---
title: 索引
---

## 指定索引

使用FORCE INDEX关键字强制使用指定的索引,强制使用指定的索引

```sql
SELECT * FROM table FORCE INDEX (index_name) 
WHERE condition;
```

使用USE INDEX关键字建议使用指定索引,优先使用指定的索引

```sql 
SELECT * FROM table USE INDEX (index_name)
WHERE condition;
```

STRAIGHT_JOIN

强制按书写顺序连接JOIN表,固定连接顺序

## 索引分类
### 按数据结构分类索引
- B+tree索引
- Hash索引
- Full-text索引
### 按物理存储分类索引
- 聚簇索引（主键索引）
- 二级索引（辅助索引）

### 按字段特性分类索引
- 主键索引
- 唯一索引
- 普通索引
- 前缀索引
### 按字段个数分类索引
- 单列索引
- 联合索引


## 优化

### 覆盖索引

这种在二级索引的 B+Tree 就能查询到结果的过程就叫作「覆盖索引」，也就是只需要查一个 B+Tree 就能找到数据
```mysql
CREATE INDEX user_name_age ON user(name, age);
SELECT name, age FROM user WHERE name = 'Tom';
```

### 索引下推

索引下推优化（index condition pushdown)， 可以在联合索引遍历过程中，对联合索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数
```mysql
# 若存在index(username,age) , 有索引下推就会减少回表,判断username的同时不急着回表再判断age
select * from user2 where username like 'j%' and age=99;
```
### 最左匹配原则
使用联合索引时，存在最左匹配原则，也就是按照最左优先的方式进行索引的匹配
### 前缀索引
是使用某个字段中字符串的前几个字符建立索引

### 索引区分度
建立联合索引时，要把区分度大的字段排在前面，这样区分度大的字段越有可能被更多的 SQL 使用到。

区分度=distinct column / count 

区分度大前面,以小驱动大

### 主键索引最好是自增的
插入一条新记录，都是追加操作，不需要重新移动数据

### 索引最好设置为 NOT NULL
行格式中至少会用 1 字节空间存储 NULL 值列表

### 防止索引失效

- 对索引隐式类型转换