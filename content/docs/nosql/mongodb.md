---
title: mongoDB
---
https://www.mongodb.com/docs/

客户端推荐 `compass` or `navicat`

服务端由cpp编写.
- mongo 客户端,类似`redis-cli`
- mongod 服务端启动,类似`mysqld`

配置
- 日志目录
- 数据存储目录
- 守护进程方式启动
- 绑定ip启动
- 端口号 27017

命令

```shell
#显示数据库
show dbs 
#切换数据库
use <数据库> 
#打印当前数据库
db 
#删库操作
db.dropDatabase() 
# 创建集合
db.createCollection(name) 
# 显示集合
show collections 
# 删除集合
db.<集合>.drop() 
```

数据库

```shell
#后台权限库
admin 
```


## curd
```shell
# 集合变量的插入 https://www.mongodb.com/docs/manual/tutorial/insert-documents/
db.collection.insert() 
# 集合变量的多个插入
db.collection.insertMany() 
```

  
查找  
```shell
# 集合查找
db.collection.find() 
# 集合带条件查找
db.collection.find({}) 
# 集合查找1条
db.collection.findOne() 
# 让user字段显示,让name字段不显示出来
db.collection.find({},{user:1,name:0}) 
# 捕获错误
try{}catch(e){ print(e) } 
```


更新  
```shell
# 覆盖更新
db.collection.update() 
#局部修改
db.collection.update({},{$set:{}})
# 更新多条
db.collection.update({},{},{multi:true}) 
# 自增操作
db.collection.update({},{$inc:{}}) 
```


删除
```shell
# 删除
db.collection.remove() 
```
  
## 高级查询
```shell
db.collection.count() # 统计查询
db.collection.find().limit(2) # 页数
db.collection.find().limit(2).skip(4) # 页数 跳过多少条
db.collection.find().sort({key:1}) # 升序
db.collection.find().sort({key:-1}) # 降序
```


正则查询
```shell
db.collection.find(key:/expr/) # 正则匹配查询
```

比较查询
```shell
db.collection.find({key:{$gt:100 }}) # 大于操作
db.collection.find({key:{$in: ["1001","1002"] }}) # where in
```

条件查询
- $and:[{},{},{}]
- $or:[{},{},{}]
- db.collection.find({$or:[{user:"1"},{"name":"2"}]})

## 索引
- 单个索引
- 联合索引
- 地理索引
- 文本索引
- 哈希索引
- TTL索引

查看索引
- db.collection.getIndexes() # 查询所有索引

创建索引

```shell
db.collection.createIndex(keys,option) # 创建索引
db.collection.createIndex({userid:1}) # 升序索引
db.collection.createIndex({userid:1,name:-1}) # 联合索引
```

删除索引

```shell
db.collection.dropIndex(index) # 删除索引 索引名词来删
db.collection.dropIndexes() #删所有索引
```

## 性能排查
- db.collection().find().explain() # 
  
## 副本集
类似redis哨兵模式
## 分片集群

路由->配置服务->分片
## 安全认证

权限
- read
- readWrite
- root #超管
- userAdmin # 数据库创建和修改用户

副本集安全认证
- 通过key文件
