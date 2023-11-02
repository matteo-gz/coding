---
title: select
---
## select作用
- case后面必须是io操作
- 监听case，没满足阻塞
- 有满足，任选1个执行
- default 处理case都不满足情况
- select不产生忙轮询
- select 自身不带有循环机制 需要借助for
- break跳出一个选项