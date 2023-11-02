---
title: 02. 协调与同步
---
## 分布式互斥

在分布式系统里，排他性的资源访问方式，叫作**分布式互斥**（Distributed Mutual Exclusion），
而这种被互斥访问的共享资源就叫**作临界资源**（Critical Resource）

### 集中式算法
协调者参与

{{< mermaid >}}
graph TB

subgraph 分布式系统
A[程序 A] --> C[协调者]
B[程序 B] --> C[协调者]
end

subgraph 互斥算法
C[协调者] --> D[发送请求]
D --> E[检查资源状态]
E --> |资源空闲| F[授权访问]
E --> |资源占用| G[排号等待]
G --> H[接收通知]
H --> D
F --> I[访问资源]
I --> J[释放资源]
J --> |通知协调者| C
end
{{< /mermaid >}}

优点
- 开发和实现简单,不存在节点间的协调问题
- 数据一致性易于控制
- 故障修复相对简单

缺点
- 可靠性和性能依赖于中心节点
- 中心节点出故障或压力过大会导致整体服务挂掉
- 难以水平扩展以提升吞吐量

### 分布式算法
定义
> 当一个程序要访问临界资源时，先向系统中的其他程序发送一条请求消息，在接收到所有程序返回的同意消息后，才可以访问临界资源。其中，请求消息需要包含所请求的资源、请求者的 ID，以及发起请求的时间

{{< mermaid >}}
sequenceDiagram
actor 程序1
actor 程序2
actor 程序3

程序1->>程序2: 请求访问资源A
程序1->>程序3: 请求访问资源A
程序2-->>程序1: 同意
程序3-->>程序1: 同意

程序3->>程序1: 请求访问资源A
程序3->>程序2: 请求访问资源A
程序2-->>程序3: 同意
Note over 程序3: 等待程序1使用完
程序1-->>程序3: 同意

程序1->>资源A: 使用资源A
程序1-->>资源A: 释放资源A

程序3->>资源A: 使用资源A
程序3-->>资源A: 释放资源A
{{< /mermaid >}}

在大型系统中使用分布式算法，消息数量会随着需要访问临界资源的程序数量呈指数级增加，容易导致高昂的"**沟通成本**"。


分布式算法适合**节点数目少**且**变动不频繁**的系统，且由于每个程序均需通信交互，因此适合 P2P 结构的系统.


#### Hadoop

HDFS 的文件修改
{{< mermaid >}}

sequenceDiagram
participant 计算机 1
participant 计算机 2
participant 计算机 3
计算机 1 ->> 计算机 2: 发送文件修改请求
计算机 1 ->> 计算机 3: 发送文件修改请求
计算机 2 -->> 计算机 1: 同意请求
计算机 3 -->> 计算机 1: 同意请求
计算机 1 ->> D文件: 开始修改文件
D文件 -->> 计算机 1: 修改完成
计算机 1 ->> 计算机 2: 发送修改完成消息和文件数据
计算机 1 ->> 计算机 3: 发送修改完成消息和文件数据
计算机 2 ->> E文件: 接收文件数据
计算机 3 ->> F文件: 接收文件数据

{{< /mermaid >}}

分布式算法是一个“先到先得"和“投票全票通过"的公平访问机制，但通信成本较高，可用性也比集中式算法低，适用于临界资源使用频度较低，且系统规模较小的场景.

### 令牌环算法
{{< mermaid >}}
graph TD
A -- 拥有令牌时访问 --> 资源
B -- 拥有令牌时访问 --> 资源
C -- 拥有令牌时访问 --> 资源 
D -- 拥有令牌时访问 --> 资源

A -. 传递令牌 .-> B
B -. 传递令牌 .-> C
C -. 传递令牌 .-> D
D -. 传递令牌 .-> A
{{< /mermaid >}}
令牌环算法的公平性高，在改进单点故障后，稳定性也很高，适用于系统规模较小，并且系统中每个程序使用临界资源的**频率高且使用时间比较短**的场景.

### 分布式互斥作用
- 保证资源的独占性访问
- 并发控制

传统单机上的互斥方法，为什么不能用于分布式环境呢？

> 传统单机上的互斥方法,例如mutex锁,不能直接用于分布式环境主要有以下几个原因:
>
> 分布式环境下,进程/线程在不同主机上运行,无法直接访问**共享内存**区域进行锁操作。
互斥锁依赖于共享内存模型无法实现。
>
> 不同主机上的时钟不一定同步,无法判断锁的持有时间。
> 互斥锁依赖于同步**时钟**实现锁定策略。
>
> 不同主机之间的**通信开销**很高,每次获取锁或释放锁都需要网络传输,性能很差。
>
> 分布式系统里**硬件错误**或**网络故障**很常见,单一锁持有者可能crash掉从而阻塞其他进程。缺乏容错能力。
>
> 分布式系统中加入和退出节点很动态,锁机制需要能够**自动适应节点**变化。
>
> 不同数据中心的节点需要协调进行锁操作,但网络延迟非常高。
>
> 分布式环境需要使用额外的技术如共识算法、消息传递等来实现分布式锁,
可以在不同节点间进行通信和协调,获得更强的一致性和可扩展性。

## 分布式选举
选举的作用
> 选出一个主节点，由它来协调和管理其他节点，以保证集群有序运行和节点间数据的**一致性**.


选举算法

- 基于序号选举的算法
  - Bully 算法
- 多数派算法
  - Raft 算法
  - ZAB 算法
### Bully 算法

选举原则
- 在所有活着的节点中，选取 ID 最大的节点作为主节点.

前提
> 集群中每个节点均知道其他节点的 ID.

{{<mermaid>}}
graph LR
A[节点 A]
B[节点 B]
C[节点 C]
D[节点 D-故障]

B -->|1. Election| D
B -->|1. Election| C
C -->|2. Alive| B
C -->|3. Election| D
C -->|4. Victory| B
C -->|4. Victory| A

{{</mermaid>}}
消息类型

- Election 消息，用于发起选举
- Alive 消息，对 Election 消息的应答
- Victory 消息，竞选成功的主节点向其他节点发送的宣誓主权的消息

流程
1. 集群中每个节点判断自己的 ID 是否为当前活着的节点中 ID 最大的，如果是，则直接向其他节点发送 Victory 消息，宣誓自己的主权；
2. 如果自己不是当前活着的节点中 ID 最大的，则向比自己 ID 大的所有节点发送 Election 消息，并等待其他节点的回复；
3. 若在给定的时间范围内，本节点没有收到其他节点回复的 Alive 消息，则认为自己成为主节点，并向其他节点发送 Victory 消息，宣誓自己成为主节点；若接收到来自比自己 ID 大的节点的 Alive 消息，则等待其他节点发送 Victory 消息；
4. 若本节点收到比自己 ID 小的节点发送的 Election 消息，则回复一个 Alive 消息，告知其他节点，我比你大，重新选举。

优点

- 选举速度快
- 算法复杂度低
- 简单易实现

缺点
- 每个节点有全局的节点信息
- 任意一个比当前主节点 ID 大的新节点或节点故障后恢复加入集群的时候，都可能会触发重新选举，成为新的主节点，如果该节点频繁退出、加入集群，就会导致频繁切主

### Raft 算法


| 角色 |            | 说明                                          |
|---------|------------|---------------------------------------------|
| Leader | 主节点        | 同一时刻只有一个 Leader，负责协调和管理其他节点                 |
| Candidate | 候选者        | 每一个节点都可以成为 Candidate，节点在该角色下才可以被选为新的 Leader |
| Follower | Leader 的跟随者 | 不可以发起选举                                     |

{{<mermaid>}}
graph RL
F[Follower]
C[Candidate]
L[Leader]
L-->|发现更大term|F
C-->|发现有了新的任期或新主|F
F -->|长时间没有收到leader消息,开始竞选|C
C-->|收到一半以上选票|L
{{</mermaid>}}
流程
1. 初始化时，所有节点均为 Follower 状态。
2. 开始选主时，所有节点的状态由 Follower 转化为 Candidate，并向其他节点发送选举请求。
3. 其他节点根据接收到的选举请求的先后顺序，回复是否同意成为主。这里需要注意的是，在每一轮选举中，一个节点**只能投出一张票**。
4. 若发起选举请求的节点获得超过一半的投票，则成为主节点，其状态转化为 Leader，其他节点的状态则由 Candidate 降为 Follower。Leader 节点与 Follower 节点之间会定期发送心跳包，以检测主节点是否活着。
5. 当 Leader 节点的任期到了，即发现其他服务器开始下一轮选主周期时，Leader 节点的状态由 Leader 降级为 Follower，进入新一轮选主。

优点
- 选举速度快
- 算法复杂度低
- 易于实现

缺点:
它要求系统内每个节点都可以相互通信，且需要获得过半的投票数才能选主成功，因此通信量大。该算法选举稳定性比 Bully 算法好，这是因为当有新节点加入或节点故障恢复后，会触发选主，但不一定会真正切主，除非新节点或故障后恢复的节点获得投票数过半，才会导致切主
### ZAB 算法

ZAB（ZooKeeper Atomic Broadcast）选举算法
> 是为 ZooKeeper 实现分布式协调功能而设计的。相较于 Raft 算法的投票机制，ZAB 算法增加了通过节点 ID 和数据 ID 作为参考进行选主，节点 ID 和数据 ID 越大，表示数据越新，优先成为主。相比较于 Raft 算法，ZAB 算法尽可能保证数据的最新性。所以，ZAB 算法可以说是对 Raft 算法的改进

| 状态 |   | 说明                                             |
|---------|---|------------------------------------------------|
| Looking | 选举 | 当节点处于该状态时，它会认为当前集群中没有 Leader，因此自己进入选举状态        |
| Leading | 领导 | 表示已经选出主，且当前节点为 Leader                          |
| Following | 跟随 | 集群中已经选出主后，其他非主节点状态更新为 Following，表示对 Leader 的追随 |
| Observing | 观察 | 表示当前节点为 Observer，持观望态度，没有投票权和选举权               |

核心
- 少数服从多数，ID 大的节点优先成为主

ZAB 算法性能高，对系统无特殊要求，采用广播方式发送信息，若节点中有 n 个节点，每个节点同时广播，则集群中信息量为 n*(n-1) 个消息，容易出现**广播风暴**；

且除了投票，还增加了对比节点 ID 和数据 ID，这就意味着还需要知道所有节点的 ID 和数据 ID，所以**选举时间相对较长**。

但该算法选举**稳定性比较好**，当有新节点加入或节点故障恢复后，会触发选主，但不一定会真正切主，除非新节点或故障后恢复的节点数据 ID 和节点 ID 最大，且获得投票数过半，才会导致切主。

### 选举对比

|      | Bully                  | Raft                 | ZAB              |
|------|------------------------|----------------------|------------------|
| 选举原则 | 节点ID最大                 | 投票最多                 | 数据最新或ID最大        |
| 优点   | 容易理解,选举速度快,算法复杂度低,易于实现 | 选举速度快,算法复杂度低,易于实现    | 性能高              |
| 缺点   | 信心存储量大,易频繁切主,不适大规模     | 要求系统全连接,消息通信量大,不适大规模 | 选举时间长,复杂度高       |
| 应用场景 | 适合小规模,MongoDB          | 适合中小规模,K8S集群中3个节点选举  | 适合中大规模,Zookeeper |

## 分布式共识
> 分布式共识就是在多个节点均可独自操作或记录的情况下，使得所有节点针对某个状态达成一致的过程

下文以区块链技术共识机制举例.

解决方式
- PoW // Proof-of-Work，工作量证明
- PoS // Proof-of-Stake，权益证明
- DPoS // Delegated Proof of Stake，委托权益证明

分布式共识核心
- 记账权
- 所有节点或服务器达成一致

### PoW
> 是以每个节点或服务器的计算能力（即“算力”）来竞争记账权的机制，因此是一种使用工作量证明机制的共识算法

PoW 的容错机制,允许全网 50% 的节点出错.缺点是共识达成的周期长、效率低，资源消耗大.

共识的时间影响因素
- PoW 机制每次达成共识需要全网共同参与运算，增加了每个节点的计算量.
- 如果题目过难，会导致计算时间长、资源消耗多.
- 如果题目过于简单，会导致大量节点同时获得记账权，冲突多.

### PoS
> 核心原理:由系统权益代替算力来决定区块记账权，拥有的权益越大获得记账权的概率就越大

缩短了达成共识时间,但容易出现垄断现象.
### DPoS
> DPoS 算法的原理，类似股份制公司的董事会制度，普通股民虽然拥有股权，但进不了董事会，他们可以投票选举代表（受托人）代他们做决策。DPoS 是由被社区选举的可信帐户（受托人，比如得票数排行前 101 位）来拥有记账权。

缺点:
- 持币人投票积极性不高.
- 故障问题解决效率低,易出现安全隐患.

### 拜占庭将军问题
> 假设一支军队由多个将军组成，这些将军需要就进攻或撤退的决策达成共识。然而，一些将军可能是不可信的，他们可能发送虚假的消息或者完全拒绝发送消息。此外，将军之间的通信可能会受到敌人的干扰，导致消息被篡改或丢失


## 分布式事务
> 分布式事务，就是在分布式系统中运行的事务，由多个本地事务组合而成

方案
- 基于 XA 协议的二阶段提交协议方法
- 三阶段提交协议方法
- 基于消息的最终一致性方法

### XA二阶段提交
二阶段提交: Two-Phase Commit，2PC

XA(Extended Architecture)是一个分布式事务协议

涉及对象
- 事务管理器 // 事务协调者Transaction Coordinator,负责各个本地资源的提交和回滚
- 本地资源管理器 // 分布式事务的参与者Participants,执行实际的操作,如数据库或其他资源

执行过程
- 投票（voting）
- 提交（commit）

举例

**第一阶段**
{{<mermaid>}}
sequenceDiagram
participant 协调者
participant 订单系统
participant 库存系统

协调者->>订单系统: 询问订单情况
订单系统-->>协调者: 锁定用户A相关订单,增加一条购买100件T恤的订单
订单系统-->>协调者: 回复同意消息"Yes"

协调者->>库存系统: 询问出货情况
库存系统-->>协调者: 回复库存不足信息"No"

{{</mermaid>}}
**第二阶段**
{{<mermaid>}}
sequenceDiagram
participant 协调者
participant 订单系统
participant 库存系统


协调者->>订单系统: 发送"DoAbort"消息
订单系统-->>协调者: 回复"HaveCommitted"消息

协调者->>库存系统: 发送"DoAbort"消息
库存系统-->>协调者: 回复"HaveCommitted"消息

{{</mermaid>}}

缺点:
- 同步阻塞问题
- 单点故障问题
- 数据不一致问题

### 三阶段提交方法
> 三阶段提交协议（Three-phase commit protocol，3PC）,三阶段提交引入了**超时**机制和准备阶段

- CanCommit
- PreCommit // 因为超时或条件不充分进行快速失败
- DoCommit
### 消息最终一致性
> 将需要分布式处理的事务通过消息或者日志的方式异步执行，消息或日志可以存到本地文件、数据库或消息队列中，再通过业务规则进行**失败重试**


是BASE理论体现,牺牲了强一致性采取最终一致性.

## 分布式锁
> 锁是实现多线程同时访问同一共享资源，保证同一时刻只有一个线程可访问共享资源所做的一种标记

主流方法
- 关系型数据库实现分布式锁 // 适用于并发量低，对性能要求低的场景
- cache实现分布式锁 // 利用SetNX原子操作,但锁失效时间控制不稳定
- ZooKeeper实现分布式锁 // 可靠性最高