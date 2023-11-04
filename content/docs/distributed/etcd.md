---
title: etcd
---
> [极客时间.etcd 实战课](https://time.geekbang.org/column/intro/100069901) 笔记
## 历史
背景:
CoreOS 团队需要一个协调服务来存储服务配置信息、提供分布式锁等能力

服务所需目标:

- 高可用
- 数据一致,提供读取"最新"数据
- 低容量、仅存储关键元数据配置
- 增删改查，监听数据变化的机制
- 可维护性

名字来源: unix `/etc` + `d` of distribute 

历史版本变化

### v0.1

- Raft算法共识
- REST API
- 数据模型使用的是基于目录的层次模式// 参考ZooKeeper
- key-value 存储引擎上,简单内存树
- Go语言

### v0.2

- 支持consistent read
- CAS提供原子性 //替换掉Test And Set 机制
### v2.0
- 支持quorum read
### v3
- 引入 B-tree,boltdb 实现一个 MVCC 数据库
- 数据模型从层次型目录结构改成扁平的 key-value
- gRPC+protobuf

特性

- 提供稳定可靠的事件通知
- 实现了事务
- 支持多 key 原子更新

同时基于 boltdb 的持久化存储，显著降低了 etcd 的内存占用、避免了 etcd v2 定期生成快照时的昂贵的资源开销.

使用了 gRPC API，使用 protobuf 定义消息，消息编解码性能相比 JSON 超过 2 倍以上，并通过 HTTP/2.0 多路复用机制，减少了大量 watcher 等场景下的连接数.

其次使用 Lease 优化 TTL 机制，每个 Lease 具有一个 TTL，相同的 TTL 的 key 关联一个 Lease，Lease 过期的时候自动删除相关联的所有 key，不再需要为每个 key 单独续期.

最后是 etcd v3 支持范围、分页查询，可避免大包等 expensive request.

{{< hint info >}}
通过历史了解到了etcd的特性与功能
{{< /hint >}}
## 概念

### quorum机制
> 比如node数量为10,我们写入3个成功,那么读取则必须为8个以上,再根据节点的最新数据获取,读才能一致.
> 
> quorum的读写最小票数可以用来做为系统在读、写性能方面的一个可调节参数。写票数Vw越大，则读票数Vr越小，这时候系统读的开销就小。反之则写的开销就小。

### bolt
[bolt](https://github.com/boltdb/bolt)是一个key-value数据库.

### Lease
[Lease](https://cloud.tencent.com/developer/article/2333360)即租约意思.
> Lease 机制是一种分布式系统中常用的协作机制，用于控制对共享资源的访问。它基于一种简单的想法：将资源的控制权租借给一个实体，以允许该实体在一段时间内独占访问资源。Lease 机制通常包括以下关键元素：
>
>租约持有者（Lease Holder）:一个实体，通常是一个进程或节点，持有资源的租约。只有租约持有者才能访问资源。
> 
>租约超时时间（Lease Timeout）:租约被授予的时间期限。一旦租约超时，资源将被释放，其他实体可以获得租约。
> 
>租约续约（Lease Renewal）:租约持有者可以在租约即将到期时请求续约，以延长其对资源的访问权限。
> 
>Lease 机制的主要目标是确保资源的独占性和一致性。通过将资源租借给一个实体，系统可以避免多个实体同时访问资源而导致的竞态条件和数据不一致性问题。

## 读场景
![](../d7.png)
### 串行读



> 如下图所示，当 client 发起一个更新 hello 为 world 请求后，若 Leader 收到写请求，它会将此请求持久化到 WAL 日志，并广播给各个节点，若一半以上节点持久化成功，则该请求对应的日志条目被标识为已提交，etcdserver 模块异步从 Raft 模块获取已提交的日志条目，应用到状态机 (boltdb 等)

WAL(write-ahead log)
![](../d3.png)
> 此时若 client 发起一个读取 hello 的请求，假设此请求直接从状态机中读取， 如果连接到的是 C 节点，若 C 节点磁盘 I/O 出现波动，可能导致它应用已提交的日志条目很慢，则会出现更新 hello 为 world 的写命令，在 client 读 hello 的时候还未被提交到状态机，因此就**可能读取到旧数据**，如上图查询 hello 流程所示。

{{< hint info >}}

适合低延时、高吞吐量,对数据一致性要求不高.
{{< /hint >}}


### 线性读
ReadIndex
![](../d4.png)

当收到一个线性读请求时，它首先会从 **Leader** 获取集群最新的已提交的**日志索引** (committed index)，
如上图中的流程二所示.

Leader 收到 ReadIndex 请求时，为防止脑裂等异常场景，会向 **Follower** 节点发送**心跳**确认，
**一半以上节点**确认 Leader 身份后才能将已提交的索引 (committed index) 返回给节点 C(上图中的流程三).

C 节点则会**等待**，直到状态机已**应用**索引 (applied index) 大于等于 Leader 的已提交索引时 (committed Index)(上图中的流程四)，
然后去**通知读请求**，数据已赶上 Leader，你可以去状态机中**访问数据**了 (上图中的流程五).
{{< hint info >}}

适合对数据一致性要求高.
{{< /hint >}}
### MVCC
{{< hint info >}}
MVCC: 解决etcd v2 不支持保存 key 的历史版本、不支持多 key 事务等问题而产生的.
{{< /hint >}}

核心组成
- 内存树形索引模块 (treeIndex)
- 嵌入式的 KV 持久化存储库 boltdb

#### boltdb

boltdb 保存一个 key 的多个历史版本,方案选择:

- 一个 key 保存多个历史版本的值// []struct
- 每次修改操作，生成一个新的版本号 (revision)，以版本号为 key， value 为用户 key-value 等信息组成的结构体 // struct

后者是etcd采用方案.

#### 读事务 
treeIndex 与 boltdb 关系如下面的读事务流程图所示
![](../d5.png)

{{< hint info >}}
etcd 在执行读请求过程中涉及磁盘 IO 吗?
{{< /hint  >}}
etcd在启动的时候会通过mmap机制将etcd db文件映射到etcd进程地址空间，并设置了mmap的MAP_POPULATE flag，
它会告诉Linux内核**预读文件**，Linux内核会将文件内容拷贝到物理内存中，此时会产生磁盘I/O。节点内存足够的请求下，后续处理读请求过程中就不会产生磁盘I/IO了。

若etcd节点内存不足，可能会导致db文件对应的内存页被换出，当读请求命中的页未在内存中时，就会产生缺页异常，导致读过程中产生磁盘IO，你可以通过观察etcd进程的majflt字段来判断etcd是否产生了主缺页中断
## 写场景


![](../d6.png)
### Quota
> 配额（Quota）模块
 
etcd db 文件大小超过了配额

> "etcdserver: mvcc: database space exceeded"
> 
> 它是指当前 etcd db 文件大小超过了配额，当出现此错误后，你的整个集群将不可写入，只读，对业务的影响非常大。

Quota 工作流程

当 etcd server 收到 put/txn 等写请求的时候，会首先检查下当前 etcd db 大小加上你请求的 key-value 大小之和是否超过了配额（quota-backend-bytes）。

如果超过了配额，它会产生一个告警（Alarm）请求，告警类型是 NO SPACE，并通过 Raft 日志同步给其它节点，告知 db 无空间了，并将告警持久化存储到 db 中

最终，无论是 API 层 gRPC 模块还是负责将 Raft 侧已提交的日志条目应用到状态机的 Apply 模块，都拒绝写入，集群只读

解决

- 调大配额,etcd 社区建议不超过 8G.
- 额外发送一个取消告警（etcdctl alarm disarm）的命令，以消除所有告警,否则集群依然拒绝写入.
- 检查 etcd 的压缩（compact）配置是否开启、配置是否合理

### Preflight Check
> 为了保证**集群稳定**性，避免雪崩，任何提交到 Raft 模块的请求，都会做一些简单的限速判断

![](../d8.png)

如果 Raft 模块已提交的日志索引（committed index）比已应用到状态机的日志索引（applied index）超过了 5000，那么它就返回一个"**etcdserver: too many requests**"错误给 client。

然后它会尝试去获取请求中的鉴权信息，若使用了密码鉴权、请求中携带了 token，如果 token 无效，则返回"**auth: invalid auth token**"错误给 client。

其次它会检查你写入的包大小是否超过默认的 1.5MB， 如果超过了会返回"**etcdserver: request is too large**"错误给给 client。

### Propose
最后通过一系列检查之后，会生成一个**唯一的ID**，将此请求关联到一个对应的消息通知 channel，然后**向 Raft 模块发起（Propose）一个提案（Proposal）**，提案内容为“大家好，请使用 put 方法执行一个 key 为 hello，value 为 world 的命令”，也就是整体架构图里的流程四。

向 Raft 模块发起提案后，KVServer 模块会等待此 put 请求，**等待写入结果**通过消息通知 channel 返回或者超时。
etcd 默认超时时间是 7 秒（5 秒磁盘 IO 延时 +2*1 秒竞选超时时间），
如果一个请求超时未返回结果，则可能会出现你熟悉的 **etcdserver: request timed out** 错误。

### WAL

Raft 模块收到提案后，如果当前节点是 Follower，它会转发给 Leader，只有 **Leader** 才能处理**写请求**。Leader 收到提案后，
通过 Raft 模块输出待转发给 **Follower** 节点的消息和**待持久化的日志条目**，日志条目则封装了我们上面所说的 put hello 提案内容。

etcdserver 从 Raft 模块获取到以上消息和日志条目后，作为 Leader，它会将 put 提案消息广播给集群各个节点，
同时需要把集群 **Leader 任期号**、**投票信息**、**已提交索引**、**提案内容**持久化到一个 WAL（Write Ahead Log）日志文件中，用于保证集群的一致性、可恢复性，也就是我们图中的流程五模块。

WAL日志结构

![](../d9.png)

上图是 WAL 结构，它由多种类型的 WAL 记录顺序追加写入组成，
每个记录由**类型、数据、循环冗余校验码**组成。
同类型的记录通过 Type 字段区分，Data 为对应记录内容，CRC 为循环校验码信息。

WAL 记录类型目前支持 5 种，分别是文件元数据记录、日志条目记录、状态信息记录、CRC 记录、快照记录：

- 文件**元数据**记录包含节点 ID、集群 ID 信息，它在 WAL 文件创建的时候写入；
- **日志**条目记录包含 Raft 日志信息，如 put 提案内容；
- **状态信息**记录，包含集群的任期号、节点投票信息等，一个日志文件中会有多条，以最后的记录为准；
- **CRC**记录包含上一个 WAL 文件的最后的 CRC（循环冗余校验码）信息， 在创建、切割 WAL 文件时，作为第一条记录写入到新的 WAL 文件， 用于校验数据文件的完整性、准确性等；
- **快照**记录包含快照的任期号、日志索引信息，用于检查快照文件的准确性。

日志条目有经过数据格式封装,fsync **持久化**到磁盘

当一半以上节点持久化此日志条目后， Raft 模块就会通过 channel 告知 etcdserver 模块，
put 提案已经被集群**多数节点确认**，提案状态为已提交，你可以执行此提案内容了

于是进入流程六，etcdserver 模块从 channel 取出提案内容，添加到先进先出（FIFO）调度队列，随后通过 Apply 模块按入队顺序，异步、依次执行提案内容。

### Apply

crash应对

> etcd 重启时，会从 WAL 中解析出 Raft 日志条目内容，追加到 Raft 日志的存储中，并**重放**已提交的日志提案给 Apply 模块执行

幂等保证

>唯一的字段能标识这个提案是Raft 日志条目中的索引（index）字段.
> 
> 日志条目索引是全局单调递增的，每个日志条目索引对应一个提案， 如果一个命令执行后，
> 我们在 db 里面也记录下当前已经执行过的日志条目索引，是不是就可以解决幂等性问题呢？
> 
> 是的。但是这还不够安全，如果执行命令的请求更新成功了，更新 index 的请求却失败了，是不是一样会导致异常？
> 
>因此我们在实现上，还需要将两个操作作为原子性事务提交，才能实现幂等。
> 
>正如我们上面的讨论的这样，etcd 通过引入一个 **consistent index** 的字段，来存储系统当前已经执行过的日志条目索引，实现幂等性。


### MVCC
- 内存索引模块 treeIndex，保存 key 的历史版本号信息
-  boltdb 模块，用来持久化存储 key-value 数据

treeIndex

**版本号**（revision）在 etcd 里面发挥着重大作用，它是 etcd 的逻辑时钟。
etcd 启动的时候默认版本号是 1，随着你对 **key** 的增、删、改操作而**全局单调递增**

![](../d10.png)

boltdb

写入 boltdb 的 value
> 为了构建索引和支持 Lease 等特性
- key 名称
- key 创建时的版本号（create_revision）、最后一次修改时的版本号（mod_revision）、key 自身修改的次数（version）
- value 值
- 租约信息

事务提交的过程，包含 B+tree 的平衡、分裂，将 boltdb 的脏数据（dirty page）、元数据信息刷新到磁盘，
因此事务提交的开销是昂贵的。
如果我们每次更新都提交事务，etcd 写性能就会较差。
> 合并再合并

首先 boltdb key 是版本号，put/delete 操作时，都会基于当前版本号递增生成新的版本号，因此属于顺序写入，可以调整 boltdb 的 bucket.FillPercent 参数，使每个 page 填充更多数据，
**减少 page 的分裂次数并降低 db 空间**。

其次 etcd 通过合并多个写事务请求，通常情况下，是**异步机制定时**（默认每隔 100ms）将**批量事务一次性提交**（pending 事务过多才会触发同步提交）， 从而大大提高吞吐量，对应上面简易写事务图的流程三。

但是这优化又引发了另外的一个问题， 因为事务未提交，读请求可能无法从 boltdb 获取到最新数据。

为了解决这个问题，etcd 引入了一个 **bucket buffer 来保存暂未提交的事务数据**。在更新 boltdb 的时候，etcd 也会同步数据到 bucket buffer。因此 etcd 处理读请求的时候会优先从 bucket buffer 里面读取，其次再从 boltdb 读，通过 bucket buffer 实现读写性能提升，同时保证数据一致性。

{{< hint info >}}
expensive request是否影响写请求性能
{{< /hint  >}}
在etcd 3.0中，线性读请求需要走一遍Raft协议持久化到WAL日志中，因此读性能非常差，写请求肯定也会被影响。

在etcd 3.1中，引入了ReadIndex机制提升读性能，读请求无需再持久化到WAL中。

在etcd 3.2中, 优化思路转移到了MVCC/boltdb模块，boltdb的事务锁由粗粒度的互斥锁，优化成**读写锁**，实现“N reads or 1 write”的并行，同时引入了**buffer**来提升吞吐量。问题就出在这个buffer，读事务会加读锁，写事务结束时要升级锁更新buffer，但是expensive request导致读事务长时间持有锁，最终导致写请求超时。

在etcd 3.4中，实现了全并发读，创建读事务的时候会**全量拷贝buffer**, 读写事务不再因为buffer阻塞，大大缓解了expensive request对etcd性能的影响。尤其是Kubernetes List Pod等资源场景来说，etcd稳定性显著提升.

## etcd数据不一致的风险

最佳实践

- 开启etcd的数据毁坏检测功能；
- 应用层的数据一致性检测；
- 定时数据备份；
- 良好的运维规范（比如使用较新稳定版本、确保版本一致性、灰度变更）