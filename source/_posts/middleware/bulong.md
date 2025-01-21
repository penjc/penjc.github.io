---
title: 步隆过滤器
comment: true   # true/false对应开启/关闭本文章评论
date: 2024-05-28
tags:
  - 步隆过滤器
  - 中间件
categories:
  - [中间件, 布隆过滤器]
#cover: cover.jpg # 文章顶部和文章介绍图（将覆盖文章主页轮播图）
---


# 检查用户名是否存在

## 直接查询数据库请求用户名是否存在。


## 存在什么问题？

- 海量用户如果查询的用户名存在或不存在，全部请求数据库，会将数据库直接打满。

### 检查用户名是否存在引起的问题

## 用户名加载缓存

### 第一版解决方案，将数据库已有的用户名全部放到缓存里。


### 该方案问题：

- 是否要设置数据的有效期？只能设置为无效期，也就是永久数据。
- 如果是永久不过期数据，占用 Redis 内存太高。

## 布隆过滤器

### 第二版解决方案，使用布隆过滤器。

### 什么是布隆过滤器

布隆过滤器是一种数据结构，用于快速判断一个元素是否存在于一个集合中。具体来说，布隆过滤器包含一个位数组和一组哈希函数。位数组的初始值全部置为 0。在插入一个元素时，将该元素经过多个哈希函数映射到位数组上的多个位置，并将这些位置的值置为 1。

- 1字节（Byte）=8位（Bit）

在查询一个元素是否存在时，会将该元素经过多个哈希函数映射到位数组上的多个位置，如果所有位置的值都为 1，则认为元素存在；如果存在任一位置的值为 0，则认为元素不存在。

### 优缺点

#### 优点：

- 高效地判断一个元素是否属于一个大规模集合。
- 节省内存。

#### 缺点：

- 可能存在一定的误判。

### 布隆过滤器误判理解

- 布隆过滤器要设置初始容量。容量设置越大，冲突几率越低。
- 布隆过滤器会设置预期的误判值。

### 误判能否接受

布隆过滤器的误判是否能够接受？

答：可以容忍。为什么？因为用户名不是特别重要的数据，如果说我设置用户名为 aaa，系统返回我不可用，那我大可以在 aaa 的基础上再加一个a，也就是 aaaa。


### 代码中使用布隆过滤器

#### 引入 Redisson 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
</dependency>
```

#### 配置 Redis 参数

```yaml
spring:
  data:
    redis:
      host: 127.0.0.1
      port: 6379
      password: 123456
```

#### 创建布隆过滤器实例

```java
import org.redisson.api.RBloomFilter;
import org.redisson.api.RedissonClient;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 布隆过滤器配置
 */
@Configuration
public class RBloomFilterConfiguration {

    /**
     * 防止用户注册查询数据库的布隆过滤器
     */
    @Bean
    public RBloomFilter<String> userRegisterCachePenetrationBloomFilter(RedissonClient redissonClient) {
        RBloomFilter<String> cachePenetrationBloomFilter = redissonClient.getBloomFilter("xxx");
        cachePenetrationBloomFilter.tryInit(0, 0);
        return cachePenetrationBloomFilter;
    }
}
```

`tryInit` 有两个核心参数：

- `expectedInsertions`：预估布隆过滤器存储的元素长度。
- `falseProbability`：运行的误判率。

错误率越低，位数组越长，布隆过滤器的内存占用越大。

错误率越低，散列 Hash 函数越多，计算耗时较长。

一个布隆过滤器占用大小的在线网站：[https://krisives.github.io/bloom-calculator/](https://krisives.github.io/bloom-calculator/)

#### 使用布隆过滤器的两种场景：

- 初始使用：注册用户时就向容器中新增数据，就不需要任务向容器存储数据了。
- 使用过程中引入：读取数据源将目标数据刷到布隆过滤器。

#### 代码中使用

```java
private final RBloomFilter<String> userRegisterCachePenetrationBloomFilter;
```

## 用户注册功能


### 如何防止用户名重复？

通过布隆过滤器把所有用户名进行加载。这样该功能就能完全隔离数据库。

### 数据库层面添加唯一索引。

### 如何防止恶意请求毫秒级触发大量请求去一个未注册的用户名？

因为用户名没注册，所以布隆过滤器不存在，代表着可以触发注册流程插入数据库。但是如果恶意请求短时间海量请求，这些请求都会落到数据库，造成数据库访问压力。这里通过分布式锁，锁定用户名进行串行执行，防止恶意请求利用未注册用户名将请求打到数据库

。


### 如果恶意请求全部使用未注册用户名发起注册

**结论：系统无法进行完全风控，只有通过类似于限流的功能进行保障系统安全。**