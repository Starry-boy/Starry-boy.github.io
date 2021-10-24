---
title: "MongoDB 入门"
subtitle: ""
date: 2021-10-24T16:01:05+08:00
lastmod: 2021-10-24T16:01:05+08:00
draft: false
toc:
  enable: true
weight: false
categories: ["DataBase","documentation"]
tags: ["Mongo"]
---
# MongoDB 入门
## 下载 Mongo DB 镜像

安装 Mong DB 最新版

```bash
docker pull mongo:latest
```

## 启动 Mongo DB

- #### 查看镜像

  ```bash
  docker images
  ```

  !<在这里插入图片描述>(https://img-blog.csdnimg.cn/2020040815430891.png)

- #### 创建容器并启动

  ```
  docker run -it -d --name mongo -p 27017:27017 <镜像id>
  ```

  !<在这里插入图片描述>(https://img-blog.csdnimg.cn/20200408154602645.png)

## 创建用户

- 查看容器id

  ```bash
  docker ps -a
  ```

  !<在这里插入图片描述>(https://img-blog.csdnimg.cn/20200408154819480.png)

- 进入容器

  ```bash
  docker exec -it c2e776db8573 /bin/bash
  ```

  !<在这里插入图片描述>(https://img-blog.csdnimg.cn/20200408154852444.png)

- 创建用户

  ```bash
  mongo
  ```

  !<在这里插入图片描述>(https://img-blog.csdnimg.cn/20200408155027148.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

  ```bash
  db.createUser({user:'root',pwd:'0804',roles:<{role:'root',db:'admin'}>})
  ```

  !<>(https://img-blog.csdnimg.cn/20200408155556877.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)

## 默认角色

| 数据库用户角色 | read、readWrite                                              |
| -------------- | ------------------------------------------------------------ |
| 数据库管理角色 | dbAdmin、dbOwner、userAdmin                                  |
| 集群管理角色   | clusterAdmin、clusterManager、clusterMonitor、hostManager    |
| 备份恢复角色   | backup、restore                                              |
| 所有数据库角色 | readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、 dbAdminAnyDatabase |
| 超级用户角色   | root                                                         |

## 基本命令

| 描述       | 命令            |
| ---------- | --------------- |
| 查看所有库 | show dbs        |
| 显示所有表 | show tables     |
| 切换库     | use <库名>      |
| 显示当前库 | db              |
| 元数据     | <库名>.system.* |

## MongoDB - RDBMS

| MongoDB                           | RDBMS       |
| --------------------------------- | ----------- |
| 数据库                            | 数据库      |
| Collection (集合)                 | Table (表)  |
| Document (文档)                   | Column (行) |
| Field (字段)                      | Row (列)    |
| 嵌入文档                          | 表联合      |
| 主键 (MongoDB 提供了 key 为 _id ) | 主键        |

## CRUD

### 创建数据库

```bash
use <库名>
```

### 删除数据库

```bash
#显示所有库
show dbs 
#切换库
use <库名>
#删除当前库
db.dropDatabase()
```

### 创建集合

```bash
db.createCollection("test",<options>)
```

| 字段        | 类型 | 描述                                                         |
| :---------- | :--- | :----------------------------------------------------------- |
| capped      | 布尔 | （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。 **当该值为 true 时，必须指定 size 参数。** |
| autoIndexId | 布尔 | （可选）如为 true，自动在 _id 字段创建索引。默认为 false。   |
| size        | 数值 | （可选）为固定集合指定一个最大值，以千字节计（KB）。 **如果 capped 为 true，也需要指定该字段。** |
| max         | 数值 | （可选）指定固定集合中包含文档的最大数量                     |

### 删除集合

```javascript
db.<集合名>.drop()
```

### 插入文档

```javascript
db.<集合名>.insert( <json字符串> )
```

**example**

```javascript
db.user.insert({"age":18,"email":"jxc19960306@outlook.com","id":0,"password":"0804","phone":"17680648860","registerFrom":"app","sex":0,"username":"ratel"})
WriteResult({ "nInserted" : 1 })
```

### 插入变量

- 将要插入的json赋值给变量

  ```javascript
  document = ({"age":18,"email":"jxc19960306@outlook.com","id":0,"password":"0804","phone":"17680648860","registerFrom":"app","sex":0,"username":"ratel"})
  WriteResult({ "nInserted" : 1 })
  ```

- 插入变量

  ```javascript
  db.<集合名>.insert(<变量名>)
  ```

- 插入多条文档

  ```javascript
  db.<集合名>.insertMany(<json数组>)
  ```

  **example**

  ```javascript
  db.user.insertMany(<{"username":"1"},{"username":"2"},{"username":"3"}>)
  ```

### 更新文档

  ```javascript
  db.user.update({<字段>:<值>},{$set:{<字段>:<值>}},{
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   })
  ```

- **query** : update的查询条件，类似sql update查询内where后面的。
- **update** : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
- **upsert** : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
- **multi** : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- **writeConcern** :可选，抛出异常的级别。

  **example**

  > 将用户名 = ratel 的文档的 年龄 更新为 21

  ```javascript
  db.user.update({"username":"ratel"},{$set:{"age":21}})
  ```

### 删除文档

- 方案一

  ```javascript
  db.<collection>.remove(<query>,{justOne:<boolean>,writeConcern:<document>})
  ```

  **example**

  ```javascript
  db.user.remove({"username":"ratel"})
  ```

- 方案二

  ```javascript
  db.<collection>.delete(<query>,{justOne:<boolean>,writeConcern:<document>})
  ```

  **example**

  ```javascript
  db.user.delete({"username":"ratel"})
  ```

- 方案三 (删除多条)

  ```javascript
  db.<collection>.deleteMany(<query>,{justOne:<boolean>,writeConcern:<document>})
  ```

  **example**

  ```javascript
  db.user.deleteMany({"username":"ratel"})
  ```

### 查询文档

```javascript
db.<collection>.find(<query>,projection).pretty()
```

> pretty() 	将查询结果格式化

**example**

> 查询 age = 66 的文档

```javascript
db.user.find({"age":66})
```

#### and 查询

```
db.<collection>.find({k1:v1,k2:v2},projection).pretty()
```

**example**

> 查询 username = ratel and  age = 66 

```javascript
db.user.find({"username":"ratel","age":66})
```

#### OR 查询

```
db.<collection>.find({k1:v1,{$or:[{k2:v2}]}},projection).pretty()
```

**example**

> 查 username = "ratel" and (age=66 or age = 100)

```javascript
db.user.find({"username":"ratel",$or:[{"age":66},{"age":100}]})
```

> 查 age = 66 or age = 100

```javascript
db.user.find({$or:[{"age":66},{"age":100}]})
```

#### 条件操作符查询

```
db.<collection>.find({<field>:{$<condition>:value}})
```

##### MongoDB 与 RDBMS Where 语句比较

| 操作       | 格式         | 范例                                        | RDBMS中的类似语句       |
| :--------- | :----------- | :------------------------------------------ | :---------------------- |
| 等于       | `{:`}        | `db.col.find({"by":"菜鸟教程"}).pretty()`   | `where by = '菜鸟教程'` |
| 小于       | `{:{$lt:}}`  | `db.col.find({"likes":{$lt:50}}).pretty()`  | `where likes < 50`      |
| 小于或等于 | `{:{$lte:}}` | `db.col.find({"likes":{$lte:50}}).pretty()` | `where likes <= 50`     |
| 大于       | `{:{$gt:}}`  | `db.col.find({"likes":{$gt:50}}).pretty()`  | `where likes > 50`      |
| 大于或等于 | `{:{$gte:}}` | `db.col.find({"likes":{$gte:50}}).pretty()` | `where likes >= 50`     |
| 不等于     | `{:{$ne:}}`  | `db.col.find({"likes":{$ne:50}}).pretty()`  | `where likes != 50`     |

**example**

> 查询 age >= 66

```javascript
db.user.find({"age":{$gte:66}})
```

#### 按数据类型查询

```
db.<collection>.find({<field>:{$type:<type>}})
```

**example**

> 查询 age 字段是 String 类型的文档

```javascript
db.user.find({"age":{$type:"string"}})
```

#### 分页查询

```javascript
db.<collection>.find(<query>).skip(<number>).limit(<number>)
```

- skip 跳过几行
- limit 截取几行

**exampl**

> mysql limit 2,1

```javascript
db.user.find().skip(2).limit(1)
```

#### 排序

```javascript
db.<collection>.find(<query>).sort({key:<number>})
```

number  1/升序   -1/降序

**example**

> 按 age 降序

```javascript
db.user.find().sort({age:-1})
```

