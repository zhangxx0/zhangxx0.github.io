---
layout:     post
title:      "亿级数据分库分表练习"
subtitle:   ""
date:       2022-01-12 00:00:00
author:     "Zhangxx"
header-img: "img/post-bg-wotplus-nine.png"
catalog: true
tags:
    - Java进阶
---

> 这才是人生难预料，不想团圆在今朝。回首繁华如梦渺，残生一线付惊涛。 柳暗花明休啼笑，善果心花可自豪。种福得福如此报，愧我当初赠木桃。

## 前言
---

一次基于MySQL的亿级数据分库分表练习；  

练习环境：  
个人笔记本xps13，i7-8550U|16g|512g  
建库前，将MySQL的数据存储位置转移到一个空闲较大的（约110g）的磁盘分区内，防止出现硬盘被写爆的情况；  




## 数据插入

![数据插入时资源消耗情况](http://zhangxx0.gitee.io/blog_image/dockerbigdata/bigdata-mysql-writein.png)  

```sql
# 查看mysql库各表容量大小
select 
table_schema as '数据库',
table_name as '表名',
table_rows as '记录数',
truncate(data_length/1024/1024, 2) as '数据容量(MB)',
truncate(index_length/1024/1024, 2) as '索引容量(MB)'
from information_schema.tables
where table_schema='m2insurance-dev'
order by data_length desc, index_length desc;
```

使用上面的SQL查看数据的插入情况时，发现其统计的数据不准确；  
通过修改`information_schema_stats_expiry`参数为`0`后，统计偏差变小，但仍不是准确数据  

![统计偏差](http://zhangxx0.gitee.io/blog_image/dockerbigdata/bigdata-mysql-100w.png)  

[MySQL8.0的information_schema.tables信息不准确](https://www.cxyzjd.com/article/londa/90480266)  

```sql
# (1)写入测试，post写入100行数据
select count(*) from user; ## 10w 2.6s
select count(*) from post; ## 100w 30s

select * from post limit 100000,2;

select * from post where user_id = 61723; # 4s

CREATE index `idx_userid` on post(`user_id`);

# user_id 加索引后 0.098s

# (2)亿级数据写入
select count(*) from user; ## 10000000 134s
select count(*) from post; ## 100000000 2999s

select * from post limit 10000000,2; # 35s

select * from post where user_id = 1097270; # 348s 10条记录

CREATE index `idx_userid` on post(`user_id`); # 562s

select * from post where user_id = 1097270; # 0.043s 10条记录

```

## 垂直拆分

将post表中的大字段拆分到post_info表中：  

```sql
create TABLE post_info(
    `post_id` bigint not null,
    `info` text,
    PRIMARY KEY(`post_id`)
);

insert into post_info select DISTINCT id as post_id,info from post;

#1206 - The total number of locks exceeds the lock table size
show variables like "%_buffer%";

SET GLOBAL innodb_buffer_pool_size=1073741824;

# Affected rows: 100000000  1475s

select count(*) from post_info; # 100000000 199s

# 删除post表的大字段
ALTER TABLE post drop COLUMN info; # 836s
```

## 水平拆分

这里使用`shardingsphere-jdbc`进行分库分表，配置文件如下：  

分为4个库s0-s4，每个库中post分为4个表t_post0-t_post4，共16张表；  
具体的分配算法见`shardingAlgorithms`配置；  

```shell
dataSources:
  # Configure the first data source
  old: !!com.zaxxer.hikari.HikariDataSource
    driverClassName: com.mysql.jdbc.Driver
    jdbcUrl: jdbc:mysql://localhost:3306/huge
    username: root
    password: **
    maxPoolSize: 30
  s0: !!com.zaxxer.hikari.HikariDataSource
    driverClassName: com.mysql.jdbc.Driver
    jdbcUrl: jdbc:mysql://localhost:3306/s0
    username: root
    password: **
    maxPoolSize: 20
  # 省略1、2
  ---
  s3: !!com.zaxxer.hikari.HikariDataSource
    driverClassName: com.mysql.jdbc.Driver
    jdbcUrl: jdbc:mysql://localhost:3306/s3
    username: root
    password: **
    maxPoolSize: 20
rules:
  # Configure sharding rule
  - !SHARDING
    tables:
      # Configure t_order table rule
      t_post:
        actualDataNodes: s${0..3}.t_post${0..3}
        # Configure database sharding strategy
        databaseStrategy:
          standard:
            shardingColumn: user_id
            shardingAlgorithmName: postdb_inline
        # Configure table sharding strategy
        tableStrategy:
          standard:
            shardingColumn: user_id
            shardingAlgorithmName: post_inline
      t_user:
        actualDataNodes: s${0..3}.t_user
        # Configure database sharding strategy
        databaseStrategy:
          standard:
            shardingColumn: id
            shardingAlgorithmName: userdb_inline
      # Omit t_order_item table rule configuration ...
      # ...

    # Configure sharding algorithms
    shardingAlgorithms:
      userdb_inline:
        type: INLINE
        props:
          algorithm-expression: s${id % 4}
      postdb_inline:
        type: INLINE
        props:
          algorithm-expression: s${user_id % 4}
      post_inline:
        type: INLINE
        props:
          algorithm-expression: t_post${user_id.intdiv(4) % 4}
```

执行速度我的机器只能跑到6w左右，不知道为什么这么慢，有时间还要再研究下写入速度这块，看别人的练习能跑到几十万甚至百万以上，从一开始的写入一亿行记录的速度来看，这个6w的速度也是太慢了，远没达到要求；  

`43 finished. 44/5000, speed=6.44W/min
44 finished. 45/5000, speed=6.44W/min`


**注意：** idea的application VM options设置为-Xmx4g -Xms4g；  




