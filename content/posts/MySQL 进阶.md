---
title: "MySQL 进阶"
date: 2020-08-10T10:41:54+08:00
categories: 
 - DataBase
tags: 
  - MySQL
mp3: /mp3/街道的寂寞.mp3
cover: "https://cdn.jsdelivr.net/gh/hojun2/hojun2.github.io/img/wallhaven-672007-2.jpg"
keywords:
  - DataBase
  - MySQL
description: 
---

# MySQL 进阶
查看表信息

```mysql
show table status like <table_name>;
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020050510584747.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMzYzNQ==,size_16,color_FFFFFF,t_70)



## Optimize table

> 优化表，mysql 删除数据 并不会腾出磁盘空间，需要手动释放资源，会锁表

## 存储过程

> 存储过程 == 代码里的函数

### 创建存储过程

```mysql
CREATE PROCEDURE <name>()
begin
    <code>
end;
```

### 调用存储过程

```mysql
call <name>();
```

### 删除存储过程

```mysql
drop procedure <name>;
```

### example simple

> 查询商品表 10条数据

```mysql
CREATE PROCEDURE ratelCustomizedPkgChangeStyle()
begin
    select * from scm_sys_order limit 10;
end;
```

### example simple (2)

> 查询商品表 n条数据

```mysql
CREATE PROCEDURE ratelCustomizedPkgChangeStyle(in num int)
begin
    select * from scm_sys_order limit num;
end;
```

```mysql
call ratelCustomizedPkgChangeStyle(<param>);
```

#### 参数列表

- ##### 参数类型

  - in

    传入参数

  - out

    返回参数 （调用存储过程后的返回值）

  - inout

    既可以传入参数，又能返回值 （有点类型 引用类型）

- 参数名

- 变量类型

  mysql 的数据类型 int、bigint、varchar。。。。。。

## 函数

> 和存储过程作用一致，可以作为 查询结果集 写入 SQL中

### 创建函数

```mysql
create
    function convert_sys_trade_id(sysTradeNo varchar(50)) returns bigint
begin
    declare id bigint default 0;
    set id = ifnull((select sys_trade_id from scm_sys_trade where sys_trade_no = sysTradeNo limit 1),-1);
    return id;
end;
```

- declare

  定义变量

### 调用函数

```mysql
select convert_sys_trade_id('HB123123')
```

### 删除函数

```mysql
drop function convert_sys_trade_id;
```

## 触发器

### 创建触发器

```sql
CREATE TRIGGER ratel_find_bug_trigger
    AFTER INSERT ON crm_station_interface
    FOR EACH ROW
BEGIN
    declare o2oShopId bigint default 0;
    declare o2oShopName varchar(20) default '';
    if new.remark is null and new.clue_source = '天猫电子凭证' then

    select t1.o2o_shop_id into o2oShopId from scm_trade t1 left join cloud_shop_v2 t2 on t1.o2o_shop_id = t2.guide_shop_id
    where tid = new.clue_id limit 1;
    select t2.shop_name into o2oShopName from scm_trade t1 left join cloud_shop_v2 t2 on t1.o2o_shop_id = t2.guide_shop_id
    where tid = new.clue_id limit 1;

    insert into
        ratel_find_bug(tid, create_time, o2o_shop_id, o2o_shop_id_from_code, o2o_shop_name, o2o_shop_name_from_code)
        value (new.clue_id,now(),o2oShopId,o2oShopId,o2oShopName,new.remark);
    end if;
END;
```

### 删除触发器

```sql
drop TRIGGER <触发器名字>
```

## 索引

### 普通索引

按索引字段查询时速度会很快

```mysql
alert <table> add index <index_name> (<filed_name>)
```

### 复合索引

多个字段上建立的索引,使用组合索引需要遵循最左前缀

```mysql
alert <table> add unique index <index_name> (<filed_name1>,<filed_name2>)
```

### 唯一索引

和普通一样并且建立索引字段的值必须唯一 且可以为空，如果是复合索引 则组合字段的必须唯一

```mysql
alert <table> add unique index <index_name> (<filed_name>)
```

### 主键索引

一张表只能有一个主键，必须唯一且不能为空，主键索引位于B+Tree的叶子节点

```mysql
alert <table> add <index_name> primary key (<filed_name>)
```

