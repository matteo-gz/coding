---
title: 03. 资源与负载
---
> [极客时间.分布式技术原理与算法解析](https://time.geekbang.org/column/intro/100036401) 笔记
## 分布式体系结构
### 集中式结构
- Google Borg
- Kubernetes
- Mesos
### 非集中式结构
- Akka 集群
- Redis 集群
- Cassandra 集群
## 调度
而为用户任务寻找合适的服务器这个过程，在分布式领域中叫作**调度**.

调度是以任务为单位的，而不是以作业为单位.
### 单体调度
> 是由一个中央调度器去管理整个集群的资源信息和任务调度，也就是说所有任务只能通过中央调度器进行调度

Borg 调度算法
- 可行性检查，找到一组可以运行任务的机器（Borglet）
- 评分，从可行的机器中选择一个合适的机器（Borglet）
  - 最差匹配
  - 最佳匹配
### 两层调度
> 是将资源管理和任务调度分为两层来调度。
- 第一层调度器负责集群资源管理，并将可用资源发送给第二层调度
- 第二层调度接收到第一层调度发送的资源，进行任务调度
### 共享状态调度
> 多个调度器，每个调度器都可以看到集群的全局资源信息，并根据这些信息进行任务调度

## 分布式计算

- MapReduce // 分而治之
- Stream // 实时
- Actor 
  - Erlang/OTP
  - Akka
  - Quasar (Java)
- Pipeline
## 分布式通信
- RPC
- Pub/Sub
- 消息队列

## 分布式存储
### CAP


|| CA  | CP    | AP                    |
|-----|-------|-----------------------|---|
| 场景  | 单机    | 强一致性,金融银行             |  及时响应,容忍一致性,商品查询 |
| 应用  | Mysql | Redis,Hbase,ZooKeeper | CoachDB,Cassandra,DynamoDB  |


### 数据分片
- 哈希映射
- 一致性哈希环

hash算法的缺陷 [好刚: 7分钟视频详解一致性hash 算法](https://www.bilibili.com/video/BV1Hs411j73w/)

> 以图片服务器举例,我们希望图片均匀落在不同的图片服务器上.
>
> 以图片名为key,hash后再根据机器数取模,在此规则下.
> 
> 假设图片hash=6,机器数为3, %3=0,则图片放置在0号服务器.
> 
> 此时,我们增加1台机器,机器数为4,图片hash依旧为6,%4=2,则映射到了2号服务器,
> 
> 此时原本在0号服务器的图片,去了2号服务器查找不存在,又要通过后端服务查找一遍,可能导致缓存雪崩.

一致性哈希环 则可以减少数据失效程度.
![一致性哈希环](../p1.png)
hash偏斜与虚拟节点
![hash偏斜](../p2.png)

### 数据结构分类
分布式数据库
- MySQL Sharding
- Microsoft SQL Azure
- Google Spanner
- Alibaba OceanBase

KV数据库
- Redis
- Memcache

分布式存储系统
- Ceph
- GFS
- HDFS
- Swift