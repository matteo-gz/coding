---
title: delete、truncate、drop
---

速度:drop > truncate >> DELETE

delete :dml

drop,truncate:ddl

- DELETE:删除表中的某些行,但不删除表结构。删除后数据行数减少,但表还存在。

- TRUNCATE:清空表内所有数据,但不删除表结构。与DELETE不同的是,TRUNCATE没有事务日志,效率更高。

- DROP:完全删除表,表结构和表数据一起删除。删除后无法恢复。

可以这么理解，一本书，delete是把目录撕了，truncate是把书的内容撕下来烧了，drop是把书烧了