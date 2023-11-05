---
title: k8s网络
---
## 网络栈

- 网卡（Network Interface）
- 回环设备（Loopback Device）
- 路由表（Routing Table）
- iptables 规则

> 被隔离在它自己的 Network Namespace 当中的


在大多数情况下，我们都希望容器进程能使用自己 Network Namespace 里的网络栈，即：**拥有属于自己的 IP 地址和端口**