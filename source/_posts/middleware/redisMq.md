---
title: Redis 作为消息队列
comment: true   # true/false对应开启/关闭本文章评论
date: 2025-01-02
tags:
  - Redis
  - 中间件
  - 消息队列
categories:
  - [中间件,消息队列]
---

## Redis 作为消息队列的运用和发展

Redis 是目前最受欢迎的 KV 类数据库，不仅限于作为 KV 场景使用，其消息队列功能也很重要。本文将解析 Redis 作为消息队列的发展历程，并提供具体代码示例。

---

## 一、Redis 1.0 list

Redis 在 1.0 版本就提供了 list 结构，通过下列方式，完成消息的生产与消费。

### 1. 基于 list 的消息生产与消费

- 生产消息：

```bash
lpush listA msg1
(integer) 1
```

- 消费消息：

```bash
rpop listA
"msg1"
```

- 阻塞式消费：

```bash
brpop listA 10
1) "listA"
2) "msg1"
```

### 2. 实现消息确认

通过使用 `rpoplpush` 和 `lrem` ，完成对消息的确认：

```bash
# 从 listA 中读取消息并将其写入 listB
rpoplpush listA listB
"msg1"

# 消息处理完毕，从 listB 中删除
lrem listB 1 msg1
(integer) 1
```

### 3. 优势与不足

**优势：**
- 模型简单，调用方便。
- 通过 `brpop` 实现消息处理的实时性。
- 提供 ACK 机制，避免消息丢失。

**不足：**
- 消息只能被消费一次，缺乏广播机制。

---

## 二、Redis 2.0 pubsub

Redis 2.0 提供了 pubsub 机制，解决了 list 结构缺乏广播能力的问题。

### 1. 基于 pubsub 实现消息广播

- 消费者 1 订阅 channel1：

```bash
subscribe channel1
```

- 消费者 2 订阅通配符 channel：

```bash
psubscribe channel*
```

- 生产消息：

```bash
publish channel1 msg1
publish channel2 msg2
```

### 2. 优势与不足

**优势：**
- 支持广播和通配符匹配。
- 能消息系统内部事件通知（如 keyspace notifications）。

**不足：**
- 消息无堆积能力，缺乏背压机制。
- Redis 实例或消费者崩溃时，消息将丢失。

---

## 三、Redis 5.0 stream

Redis 5.0 引入了 stream 结构，重新设计了全内存消息队列。

### 1. 基于 stream 实现消息传递

- 创建消费者组：

```bash
XGROUP CREATE mystream mygroup $ MKSTREAM
```

- 生产消息：

```bash
XADD mystream * field1 value1 field2 value2
```

- 消费消息：

```bash
XREADGROUP GROUP mygroup consumer1 COUNT 1 STREAMS mystream >
```

- ACK 确认消息：

```bash
XACK mystream mygroup <message-id>
```

- 处理存在差错的消息：

```bash
XPENDING mystream mygroup - + 10
XCLAIM mystream mygroup consumer2 0 <message-id>
```

### 2. 优势与不足

**优势：**
- 支持广播和跨消费者组使用。
- 通过 ACK 和组内同步确保消息至少被处理一次。
- 支持计量化处理，基于 id 确保消息唯一性。

**不足：**
- Redis 自身的数据持久化机制不能绝对防止数据丢失，例如带有完全处理背压的事件。

---

## 四、Redis 的总结

- **list** ：涉及基础队列模型，适合简单场景；
- **pubsub** ：做到了消息广播，但缺乏丰富的值添能力；
- **stream** ：重点解决了广播和数据第一次处理能力，适合高速解耦场景。
