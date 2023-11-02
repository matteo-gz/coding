---
title: count
---

## count(1) and count(*)
count(*)包括了所有的列，相当于行数，在统计结果的时候，不会忽略NULL
## count(1) and count(列名)

- count(1) 会统计表中的所有的记录数，不会忽略NULL，包含字段为null 的记录
- count(列名) 会统计该列字段在表中出现的次数，会忽略字段为null 的情况，即不统计字段为null 的记录

## 执行效率
- 若列名为主键，count(列名)会比count(1)快
- 若列名不为主键，count(1)会比count(列名)快  
- 若表多个列并且没有主键，则 count（1） 的执行效率优于 count（*）
- 若表有主键，则 select count（主键）的执行效率是最优的  
- 若表只有一个字段，则 select count（*）最优。

优先使用count(*)或count(主键id);若主键不能用,则用count(1);若都不行才使用count(字段)
## 如何优化 count(*)？
- 近似值,explain 命令来表进行估算
- 额外表保存计数值 