---
layout:       post
title:        "Mybatis-Plus"
# subtitle: ""
author:       "Peng"
# header-img: ""
header-style: text
catalog:      true
# lang: en
tags:
    - Java
    - Database
---

# 用户分库分表

## 为什么要分库分表？
- 数据量庞大。
- 查询性能缓慢，之前可能是 20ms，后续随着数据量的增长，查询时间呈指数增长。
- 数据库连接不够。

## 什么是分库分表？
分库和分表有两种模式，垂直和水平。
- 分库两种模式：
  - 垂直分库：电商数据库拆分为用户、订单、商品、交易等数据库。
  - 水平分库：用户数据库，拆分为多个，比如User_DB_0 - x。
- 分表两种模式：
  - 垂直分表：将数据库表按照业务维度进行拆分，将不常用的信息放到一个扩展表。
  - 水平分表：将用户表水平拆分，展现形式就是 User_Table_0 - x。

## 什么场景下分表？
- 数据量过大或者数据库表对应的磁盘文件过大。
- Q: 多少数据需要分表
- A: 字段过多，text文件占比较大这种类型的数据字段多

## 什么情况下分库？
- 连接不够用。
  - MySQL Server 假设支持 4000 个数据库连接。一个服务连接池最大 10 个，假设有 40 个节点。已经占用了 400 个数据库连接。

## 又分库又分表？
- 高并发写入或查询场景。
- 数据量巨大场景。

## 数据库分库分表框架 ShardingSphere
- Sharding-JDBC。

## 分片键
用于将数据库（表）水平拆分的数据库字段。 分库分表中的分片键（Sharding Key）是一个关键决策，它直接影响了分库分表的性能和可扩展性。以下是一些选择分片键的关键因素：
- 访问频率：选择分片键应考虑数据的访问频率。将经常访问的数据放在同一个分片上，可以提高查询性能和降低跨分片查询的开销。
- 数据均匀性：分片键应该保证数据的均匀分布在各个分片上，避免出现热点数据集中在某个分片上的情况。
- 数据不可变：一旦选择了分片键，它应该是不可变的，不能随着业务的变化而频繁修改。

## 用户名和用户ID选哪个作为分片键？
- 用户名。用户名可以登录。

## 引入 ShardingSphere-JDBC到项目
### 引入依赖
### 定义分片规则
#### shardingsphere-config.yaml
```yaml
# 数据源集合
dataSources:
  ds_0:
    dataSourceClassName: com.zaxxer.hikari.HikariDataSource
    driverClassName: com.mysql.cj.jdbc.Driver
    jdbcUrl: jdbc:mysql://127.0.0.1:3306/link?useUnicode=true&characterEncoding=UTF-8&rewriteBatchedStatements=true&allowMultiQueries=true&serverTimezone=Asia/Shanghai
    username: root
    password: root

rules:
  - !SHARDING
    tables:
      t_user:
        # 真实数据节点，比如数据库源以及数据库在数据库中真实存在的
        actualDataNodes: ds_0.t_user_${0..15}
        # 分表策略
        tableStrategy:
          # 用于单分片键的标准分片场景
          standard:
            # 分片键
            shardingColumn: username
            # 分片算法，对应 rules[0].shardingAlgorithms
            shardingAlgorithmName: user_table_hash_mod
    # 分片算法
    shardingAlgorithms:
      # 数据表分片算法
      user_table_hash_mod:
        # 根据分片键 Hash 分片
        type: HASH_MOD
        # 分片数量
        props:
          sharding-count: 16
# 展现逻辑 SQL & 真实 SQL
props:
  sql-show: true
```

## ShardingSphere 数据分片核心概念
- 逻辑表
  - 相同结构的水平拆分数据库（表）的逻辑名称，是 SQL 中表的逻辑标识。 
- 真实表
  - 在水平拆分的数据库中真实存在的物理表。