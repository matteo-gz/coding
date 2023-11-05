---
title: 02.1 分布式事务
---


## 简介
> 分布式事务，就是在分布式系统中运行的事务，由多个本地事务组合而成

方案
- 基于 XA 协议的二阶段提交协议方法
- 三阶段提交协议方法
- 基于消息的最终一致性方法

## XA二阶段提交
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

### 缺点
- 同步**阻塞**问题
- 单点故障问题
- **数据不一致**问题

## 三阶段提交方法
> 三阶段提交协议（Three-phase commit protocol，3PC）,三阶段提交引入了**超时**机制和准备阶段

- CanCommit
- PreCommit // 因为超时或条件不充分进行快速失败
- DoCommit
## 消息最终一致性
> 将需要分布式处理的事务通过消息或者日志的方式异步执行，消息或日志可以存到本地文件、数据库或消息队列中，再通过业务规则进行**失败重试**


是BASE理论体现,牺牲了强一致性采取最终一致性.

### 基于 MQ 的消息投递
实现生产端和消费端的**双向确认**

- MQ 自动应答机制导致的消息丢失
  - 采取编程的方式手动发送应答
- 高并发场景下的消息积压导致消息丢失
  - 未完成的消息重新投递来进行消息补偿

