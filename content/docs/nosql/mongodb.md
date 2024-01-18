# mongodb

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
- show dbs #显示数据库
- use $数据库 #切换数据库
- db #打印当前数据库
- db.dropDatabase() #删库操作
- db.createcollection(name) # 创建集合
- show collections # 显示集合
- db.$集合.drop() # 删除集合

数据库
- admin #后台权限库

## curd
- db.$collection.insert() # 集合变量的插入
- db.$collection.insertMany() # 集合变量的多个插入
  
查找  
- db.$collection.find() # 集合查找
- db.$collection.find({}) # 集合带条件查找
- db.$collection.findOne() # 集合查找1条
-  db.$collection.find({},{user:1,name:0}) # 让user字段显示,让name字段不显示出来
-  try{}catch(e){ print(e) } # 捕获错误

更新  
-  db.collection.update() # 覆盖更新
-  db.collection.update({},{$set:{}}) #局部修改
-  db.collection.update({},{},{multi:true}) # 更新多条
-  db.collection.update({},{$inc:{}}) # 自增操作

删除
- db.collection.remove() # 删除
  
## 高级查询
- db.collection.count() # 统计查询
- db.collection.find().limit(2) # 页数
- db.collection.find().limit(2).skip(4) # 页数 跳过多少条
- db.collection.find().sort({key:1}) # 升序
- db.collection.find().sort({key:-1}) # 降序

正则查询
- db.collection.find(key:/expr/) # 正则匹配查询

比较查询
- db.collection.find({key:{$gt:100 }}) # 大于操作
-  db.collection.find({key:{$in: ["1001","1002"] }}) # where in

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

查看索引
- db.collection.getIndexes() # 查询所有索引

创建索引
- db.collection.createIndex(keys,option) # 创建索引
- db.collection.createIndex({userid:1}) # 升序索引
- db.collection.createIndex({userid:1,name:-1}) # 联合索引

删除索引
- db.collection.dropIndex(index) # 删除索引 索引名词来删
- db.collection.dropIndexes() #删所有索引

## 性能排查
- db.collection().find().explain() # 
  
## 副本集
类似redis哨兵模式
## 分片集群

路由
配置服务
分片
## 安全认证
权限
- read
- readWrite
- root #超管
- userAdmin # 数据库创建和修改用户

副本集安全认证
- 通过key文件
