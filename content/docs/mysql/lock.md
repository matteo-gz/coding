---
title: 锁
---

## 死锁

> [操作系统（四）—死锁](https://www.zhihu.com/tardis/sogou/art/89878383)

### 四个必要条件

- 互斥：在一个时间只能有一个进程使用资源。
- 请求和保持（持有并等待）：进程保持至少一个资源正在等待获取其他进程持有的额外资源。
- 不可抢占：一个资源只能在进程已经完成了它的任务之后，被自愿释放。
- 循环等待：存在n个进程，进行循环等待所占资源。

### 解决

- 死锁预防
- 死锁避免
- 死锁检测
- 死锁恢复

Mysql的锁

> [MySQL 有哪些锁？](https://xiaolincoding.com/mysql/lock/mysql_lock.html)

根据锁粒度划分

## 全局锁

Flush tables with read lock (FTWRL)

全局锁主要应用于做**全库逻辑备份**，这样在备份数据库期间，不会因为数据或表结构的更新，而出现备份文件的数据与预期的不一样

## 表级锁

### 表锁

表锁的语法

``` 
lock tables … read/write
```

表锁的例子

``` mysql
#表级别的共享锁，也就是读锁；
lock tables t_student read;

#表级别的独占锁，也就是写锁；
lock tables t_stuent write;
```

主动释放
``` mysql
unlock tables
```
### 元数据锁
meta data lock,MDL

MDL 锁是系统默认会加的

- 对一张表进行 CRUD 操作时，加的是 MDL 读锁；
- 对一张表做结构变更操作的时候，加的是 MDL 写锁；


例子: 给一个小表加个字段，导致整个库挂了

- 事务不提交，就会一直占着 MDL 锁,先暂停 DDL，或者 kill 掉这个长事务
- 针对热点表,在 alter table 语句里面设定等待时间,如果在这个指定的等待时间里面能够拿到 MDL 写锁最好，拿不到也不要阻塞后面的业务语句，先放弃
``` mysql
# NOWAIT:
# 表示如果当前操作得不到表级别的元数据锁,会直接报错,不会等待锁释放。
# 通常情况下ALTER默认采用NOWAIT模式
ALTER TABLE tbl_name NOWAIT add column ...
# WAIT:
# 表示如果当前操作得不到表级别的元数据锁,会等待锁被释放,而不是直接报错。
# 可以指定等待的最大秒数,如WAIT 30。如果超过设定时间还没获得锁就报错。
ALTER TABLE tbl_name WAIT N add column ... 
```

### Intention Lock 意向锁 

锁的英文缩写

- S shared lock // 共享锁
- X exclusive lock // 排他锁
- IS intention shared lock // 意向共享锁
- IX intention exclusive lock // 意向排他锁

```mysql
# 先在表上加上意向共享锁，然后对读取的记录加共享锁
select ... lock in share mode;

# 先表上加上意向独占锁，然后对读取的记录加独占锁
select ... for update;
```

> https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-intention-locks

锁之间的兼容性

|  | X | IX | S | IS |
| --- | --- | --- | --- | --- |
| X | ❌ | ❌ | ❌ | ❌ |
| IX | ❌ | ✅ | ❌ | ✅ |
| S | ❌ | ❌ | ✅ | ✅ |
| IS | ❌ | ✅ | ✅ | ✅ |

意向锁的目的是为了**快速判断**表里是否有记录被加锁
### AUTO-INC 锁

AUTO-INC 锁是特殊的表锁机制，锁不是再一个事务提交后才释放，而是再执行完插入语句后就会立即释放。

## 行锁

### Record Locks 记录锁

```mysql
mysql > begin;
mysql > select * from t_test where id = 1 for update;
```

### Gap Locks 间隙锁

```mysql
# 区域中所有现有值 15 之间的间隙都是锁定的
 SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE;
```
间隙锁之间是兼容的，即两个事务可以同时持有包含共同间隙范围的间隙锁，并不存在互斥关系，因为间隙锁的目的是防止插入**幻影记录**而提出的。



### Next-Key Locks 临间锁
锁定一个范围，并且锁定记录本身

next-key lock 是包含间隙锁(Gap Locks)+记录锁(Record Locks)的，
如果一个事务获取了 X 型的 next-key lock，
那么另外一个事务在获取相同范围的 X 型的 next-key lock 时，是会被阻塞的
### Insert Intention Locks 插入意向锁

一个事务在插入一条记录的时候，需要判断插入位置是否已被其他事务加了间隙锁（next-key lock 也包含间隙锁）

如果有的话，插入操作就会发生**阻塞**，直到拥有间隙锁的那个事务提交为止（释放间隙锁的时刻），在此期间会生成一个**插入意向锁**，表明有事务想在某个区间插入新记录，但是现在处于等待状态

## 锁策略

### 乐观锁

- 增加版本号version字段,在更新时带上version进行判断

### 悲观锁

- SELECT FOR UPDATE

读多写少使用乐观锁,写多使用悲观锁。