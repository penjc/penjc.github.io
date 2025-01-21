---
title: 分库分表
comment: true   # true/false对应开启/关闭本文章评论
date: 2024-08-03
tags:
  - 分库分表
  - 数据库
categories:
  - [数据库]
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
- Q：多少数据量进行分表？
- A：单表 1000w 是否要分表？回答不够标准。
假设一个表里 15 个字段，没有特别大的值（不包含 text 或其它超长度的列）数据量超过 5000 万了，依然很丝滑，因为走索引。
真正需要考虑的是：业务的增长量以及历史数量。
- Q：物理文件过大，会有什么问题？
- A：会影响公司对数据库表的一个备份。数据库表文件过大，也间接证明表数据过大，增加或删除字段导致锁表的时间过长。

## 什么情况下分库？
- 连接不够用。
  - MySQL Server 假设支持 4000 个数据库连接。一个服务连接池最大 10 个，假设有 40 个节点。已经占用了 400 个数据库连接。
当数据库的连接不够客户端使用时，可以考虑分库或读写分离。
如果说当数据库的 QPS 越来越高以及数据量越来越大的时候，就需要考虑分库分表。
- Q：为什么说连接不够用？
- A：假设 MySQL Server 能支持 4000 个数据库连接。我们有 10 个服务，40 个节点，一个节点呢数据库连接池最多 10 个。这样就把一个 MySQL Server 的连接数压榨干净了。
当 MySQL 连接不够用时，可能会报错 "Too many connections" 或者类似的错误。这是因为 MySQL 服务器同时可以处理的连接数量是有限制的，当连接数达到这个限制时，服务器就会拒绝新的连接请求，并返回这个错误消息。
## 又分库又分表？
- 高并发写入场景：当应用面临高并发的写入请求时，单一数据库可能无法满足写入压力，此时可以将数据按照一定规则拆分到多个数据库中，每个数据库处理部分数据的写入请求，从而提高写入性能。
- 数据量巨大场景：随着数据量的不断增加，单一数据库的存储和查询性能可能逐渐下降。此时，可以将数据按照一定的规则拆分到多个表中，每个表存储部分数据，从而分散数据的存储压力，提高查询性能。

## 数据库分库分表框架 ShardingSphere
- Sharding-JDBC。

## 分片键
选择分库分表中的分片键（Sharding Key）是一个关键决策，它直接影响了分库分表的性能和可扩展性。以下是一些选择分片键的关键因素：
1. 访问频率：选择分片键应考虑数据的访问频率。将经常访问的数据放在同一个分片上，可以提高查询性能和降低跨分片查询的开销。
2. 数据均匀性：分片键应该保证数据的均匀分布在各个分片上，避免出现热点数据集中在某个分片上的情况。
3. 业务关联性：分片键应该与业务关联紧密，这样可以避免跨分片查询和跨库事务的复杂性。
4. 数据不可变：一旦选择了分片键，它应该是不可变的，不能随着业务的变化而频繁修改。

基于以上考虑，我们选择使用 username 作为分片键。

## 用户名和用户ID选哪个作为分片键？
- 用户名。用户名可以登录。

## 引入 ShardingSphere-JDBC到项目
### 引入依赖
```yaml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>shardingsphere-jdbc-core</artifactId>
    <version>5.3.2</version>
</dependency>
```
### 定义分片规则
```yaml
spring:
  datasource:
  	# ShardingSphere 对 Driver 自定义，实现分库分表等隐藏逻辑
    driver-class-name: org.apache.shardingsphere.driver.ShardingSphereDriver
    # ShardingSphere 配置文件路径
    url: jdbc:shardingsphere:classpath:shardingsphere-config.yaml
```
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

## ShardingSphere-Proxy
定位为透明化的数据库代理端，通过实现数据库二进制协议，对异构语言提供支持。 目前提供 MySQL 和 PostgreSQL 协议，透明化数据库操作，对 DBA 更加友好。
- 向应用程序完全透明，可直接当做 MySQL/PostgreSQL 使用；
- 兼容 MariaDB 等基于 MySQL 协议的数据库，以及 openGauss 等基于 PostgreSQL 协议的数据库；
- 适用于任何兼容 MySQL/PostgreSQL 协议的的客户端，如：MySQL Command Client, MySQL Workbench, Navicat 等。

## JDBC 和 Proxy 优劣势
● JDBC： 
  ○ 优点：性能较高，通过 JDBC 直接向 MySQL 发起请求调用。
  ○ 优点：使用较为简单，理论上无需修改代码，仅需使用 ShardingSphere 的配置即可。
  ○ 缺点：需要修改项目配置以及引入 Jar 包。
  ○ 缺点：对应用的内存有一定影响。
● Proxy： 
  ○ 优点：无需对现有项目做任何配置或代码变更，将数据库的地址改为 Proxy 的地址即可。
  ○ 优点：Proxy 对 Java 应用内存没有任何影响。
  ○ 优点：分片后无法知道一条数据记录到底在那张表，Proxy 是屏蔽了分片逻辑，可直接操作逻辑表。
  ○ 缺点：JDBC 操作 MySQL 是点对点的，但是 Proxy 多了一层链路。

为什么选择 ShardingSphere？
● 拥有活跃的社区，能够提供及时的技术支持和更新；
● 代码质量极高，经过严格测试和验证，稳定可靠；
● 提供丰富的功能，满足多样化的业务需求。

## 用户分库分表策略

根据 2022 年的全国人口统计数据，现有 14 亿多总人口，每年新生人口约 956 万。为便于后续业务数据规模的判断，本文先基于这一人口总量和增长数据进行估算。

根据系统设计的假设，12306 的注册用户规模约为 10 亿，每年新增用户约 1000 万。
在用户数据分库或分表之前，我们需要先考虑拆分成多少个库或表才能达到最优性能。为了进行这样的决策，我们可以预估单个表的最大数据量。根据过去的经验，通常我们会选择 2000 万作为一个经验值。这个数据量既不会过小，同时又能保证增删改查等操作相对流畅。

根据当前用户表的数据量为 10 亿，并且每年新增 1000 万用户，预估未来系统的生命周期较长，数据量大概会达到 30 亿左右。基于这个数据量，我们预估单表的数据量在 2000 万左右，因此需要分大约 150 张表来容纳这些数据。
在进行分库分表容量评估时，我们通常会尽可能多地进行评估。这样做的好处是，即使每张表的数据量不多，也能及早发现拆分后是否存在数据问题，以便及时进行调整和优化。

此外，需要特别指出的是，我们对表数据量考虑的阈值相对较小，这是因为我们的系统具备良好的可扩展性，可以轻松应对大量的数据增长。因此，基于这种情况的分库分表策略，即使在几百年后，这个分库分表依然能够处理数据，并且不会出现性能问题。这为我们的系统提供了稳定可靠的性能保障。