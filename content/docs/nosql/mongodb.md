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

curd
- db.$collection.insert() # 集合变量的插入
- db.$collection.insertMany() # 集合变量的多个插入
- db.$collection.find() # 集合查找
- db.$collection.find({}) # 集合带条件查找
- db.$collection.findOne() # 集合查找1条
-  db.$collection.find({},{user:1,name:0}) # 让user字段显示,让name字段不显示出来
-  try{}catch(e){ print(e) } # 捕获错误
-  db.collection.update() # 覆盖更新
-  db.collection.update({},{$set:{}}) #局部修改
-  db.collection.update({},{},{multi:true}) # 更新多条
-  db.collection.update({},{$inc:{}}) # 自增操作

  
