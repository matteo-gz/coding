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