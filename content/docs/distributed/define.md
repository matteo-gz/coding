---
title: 01. 分布式简介
---
> [极客时间.分布式技术原理与算法解析](https://time.geekbang.org/column/intro/100036401) 笔记
## 分布式定义

分布式其实就是将相同或相关的程序运行在多台计算机上，从而实现特定目标的一种计算方式.

分布式形态

- 数据并行
- 任务并行

分布式驱动力量

- 性能
- 可用性
- 可扩展性

## 指标

### 性能

- 吞吐量
  - QPS（Queries Per Second）
  - TPS（Transactions Per Second）
  - BPS（Bits Per Second）
- 响应时间
- 完成时间

### 资源占用
- 空载资源占用
- 满载资源占用

### 可用性

系统的可用性可以用系统停止服务的时间与总的时间之比衡量

某功能的失败次数与总的请求次数之比来衡量

### 可扩展性

当任务的需求随着具体业务不断提高时，除了升级系统的性能做**垂直** / 纵向扩展外，
另一个做法就是通过增加机器的方式去**水平** / 横向扩展系统规模。

系统可扩展性的常见指标是**加速比**（Speedup），也就是一个系统进行扩展后相对扩展前的性能提升


**不同场景下分布式系统的指标**
- 电商系统 //重吞吐量
- IoT //资源占用指标,可以资源占用KB级的
- 电信业务 // 响应时间、完成时间，以及可用性
- HPC // 任务执行时间极长,水平扩展来提高系统的加速比
- 大数据 // 扩展性
- 云计算 // 减少用户操作时间,降低系统资源开销
- 区块链 // 吞吐量和完成时间