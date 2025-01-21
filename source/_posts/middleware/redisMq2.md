---
title: Redis 作为消息队列及幂等性处理
comment: true   # true/false对应开启/关闭本文章评论
date: 2025-01-02
tags:
  - Redis
  - 中间件
  - 消息队列
categories:
  - [中间件,消息队列]
---

### 1. **Redis Stream 消费者初始化**

```java
@Component
@RequiredArgsConstructor
public class ShortLinkStatsStreamInitializeTask implements InitializingBean {
    private final StringRedisTemplate stringRedisTemplate;

    @Override
    public void afterPropertiesSet() throws Exception {
        Boolean hasKey = stringRedisTemplate.hasKey(SHORT_LINK_STATS_STREAM_TOPIC_KEY);
        if (hasKey == null || !hasKey) {
            stringRedisTemplate.opsForStream().createGroup(SHORT_LINK_STATS_STREAM_TOPIC_KEY, SHORT_LINK_STATS_STREAM_GROUP_KEY);
        }
    }
}
```

#### 详细解释：
- **Stream Group:** Redis Stream 中的消费者组（Consumer Group）允许多个消费者共享工作负载。每个消息被分配给消费者组中的一个消费者进行处理，避免了单个消费者的负担过重。

- **创建消费者组:** 这段代码检查是否已经存在一个消费者组，如果不存在，则通过 `createGroup` 方法创建一个新的消费者组。创建消费者组时，我们指定了流的名字（`SHORT_LINK_STATS_STREAM_TOPIC_KEY`）和消费者组的名字（`SHORT_LINK_STATS_STREAM_GROUP_KEY`）。如果该消费者组已经存在，则不会重复创建。

- **应用场景:** 这种机制适用于需要多个消费者平衡处理任务的场景，例如在短链接统计或日志处理中，不同的服务或应用可以并行地消费消息进行处理。

---

### 2. **Redis Stream 消费者配置**

```java
@Configuration
@RequiredArgsConstructor
public class RedisStreamConfiguration {
    private final RedisConnectionFactory redisConnectionFactory;
    private final ShortLinkStatsSaveConsumer shortLinkStatsSaveConsumer;

    @Bean
    public ExecutorService asyncStreamConsumer() {
        AtomicInteger index = new AtomicInteger();
        return new ThreadPoolExecutor(1,
                1,
                60,
                TimeUnit.SECONDS,
                new SynchronousQueue<>(),
                runnable -> {
                    Thread thread = new Thread(runnable);
                    thread.setName("stream_consumer_short-link_stats_" + index.incrementAndGet());
                    thread.setDaemon(true);
                    return thread;
                },
                new ThreadPoolExecutor.DiscardOldestPolicy()
        );
    }

    @Bean
    public Subscription shortLinkStatsSaveConsumerSubscription(ExecutorService asyncStreamConsumer) {
        StreamMessageListenerContainer.StreamMessageListenerContainerOptions<String, MapRecord<String, String, String>> options =
                StreamMessageListenerContainer.StreamMessageListenerContainerOptions
                        .builder()
                        .batchSize(10)
                        .executor(asyncStreamConsumer)
                        .pollTimeout(Duration.ofSeconds(3))
                        .build();

        StreamMessageListenerContainer<String, MapRecord<String, String, String>> listenerContainer = 
            StreamMessageListenerContainer.create(redisConnectionFactory, options);
        
        StreamMessageListenerContainer.StreamReadRequest<String> streamReadRequest =
                StreamMessageListenerContainer.StreamReadRequest.builder(StreamOffset.create(SHORT_LINK_STATS_STREAM_TOPIC_KEY, ReadOffset.lastConsumed()))
                        .consumer(Consumer.from(SHORT_LINK_STATS_STREAM_GROUP_KEY, "stats-consumer"))
                        .autoAcknowledge(true)
                        .build();

        Subscription subscription = listenerContainer.register(streamReadRequest, shortLinkStatsSaveConsumer);
        listenerContainer.start();
        return subscription;
    }
}
```

#### 详细解释：
- **线程池与消费者：** 通过 `ExecutorService` 使用线程池处理消费任务。消费者通过独立线程异步拉取消息并处理，确保系统的高并发处理能力，避免阻塞其他任务。

- **批量消息处理：** 配置了 `batchSize(10)`，意味着消费者每次从 Redis Stream 拉取最多 10 条消息，适应高吞吐量的需求。

- **轮询超时设置：** 设置了 `pollTimeout` 为 3 秒，表示消费者如果在 3 秒内没有拉取到消息，将会继续等待新消息到来。这样设计确保了消费者在没有消息时不浪费计算资源，只有当有新消息时才开始处理。

- **消费者组与偏移量：** 使用 `StreamOffset.lastConsumed()` 确保消费者从上次消费的偏移量开始消费，避免重复消费消息。

- **自动确认：** 配置了 `autoAcknowledge(true)`，意味着一旦消息被成功消费，Redis 将自动确认该消息，确保消息不被重复消费。

---

### 3. **消息幂等性处理**

```java
@Component
@RequiredArgsConstructor
public class MessageQueueIdempotentHandler {
    private final StringRedisTemplate stringRedisTemplate;

    private static final String IDEMPOTENT_KEY_PREFIX = "short-link:idempotent:";

    public boolean isMessageBeingConsumed(String messageId) {
        String key = IDEMPOTENT_KEY_PREFIX + messageId;
        return Boolean.FALSE.equals(stringRedisTemplate.opsForValue().setIfAbsent(key, "0", 2, TimeUnit.MINUTES));
    }

    public boolean isAccomplish(String messageId) {
        String key = IDEMPOTENT_KEY_PREFIX + messageId;
        return Objects.equals(stringRedisTemplate.opsForValue().get(key), "1");
    }

    public void setAccomplish(String messageId) {
        String key = IDEMPOTENT_KEY_PREFIX + messageId;
        stringRedisTemplate.opsForValue().set(key, "1", 2, TimeUnit.MINUTES);
    }

    public void delMessageProcessed(String messageId) {
        String key = IDEMPOTENT_KEY_PREFIX + messageId;
        stringRedisTemplate.delete(key);
    }
}
```

#### 详细解释：
- **幂等性处理：** 这段代码通过 Redis 实现消息的幂等性，确保每条消息只会被处理一次，即使在消息消费失败后重试，也不会出现重复处理的情况。

- **`setIfAbsent`：** 使用 Redis 的 `setIfAbsent` 命令，如果指定的键不存在，才会设置值为 "0"（表示消息正在处理中）。如果键已经存在，表示该消息正在被处理或已处理过，因此直接返回 `false`，防止重复处理。

- **`setAccomplish`：** 当消息处理完成后，通过 `setAccomplish` 方法将消息状态标记为完成（`1`），并设置过期时间为 2 分钟。

- **`delMessageProcessed`：** 如果在处理过程中出现异常，幂等性标志将被删除，允许系统重试该消息。

---

### 4. **生产者与消费者**

#### 消费者：
```java
@Slf4j
@Component
@RequiredArgsConstructor
public class ShortLinkStatsSaveConsumer implements StreamListener<String, MapRecord<String, String, String>> {
    private final MessageQueueIdempotentHandler messageQueueIdempotentHandler;

    @Override
    public void onMessage(MapRecord<String, String, String> message) {
        String messageId = message.getId().toString();
        if (messageQueueIdempotentHandler.isMessageBeingConsumed(messageId)) {
            if (messageQueueIdempotentHandler.isAccomplish(messageId)) {
                return; // 消息已经处理过，跳过
            }
            throw new ServiceException("消息需要重试");
        }
        // 处理消息
        messageQueueIdempotentHandler.setAccomplish(messageId); // 标记消息为已处理
    }
}
```

**解释：**
- 消费者会从 Redis 流中获取消息，然后通过 `isMessageBeingConsumed` 方法检查该消息是否正在处理中。如果是，则跳过；如果该消息已经处理完，则直接返回；如果是第一次处理该消息，则开始消费并标记消息为已处理。

---

### **工作流程总结：**

1. **生产者：** 生产者通过 `stringRedisTemplate.opsForStream().add()` 将消息推送到 Redis Stream。
2. **消费者：** 消费者从 Redis Stream 中拉取消息，使用幂等性处理确保每条消息只会被处理一次。消费者处理完成后，标记消息已处理。
3. 如果在处理过程中发生异常，幂等性标记会被清除，允许系统重试该消息。

通过这种模式，消息的处理确保了至少一次处理，并且在出现异常时，能够重试消息而不会重复处理。

---

### 5. **总结**

通过 Redis Stream 和幂等性处理，可以实现高效、可靠的消息队列系统，适合需要高吞吐量、低延迟且能处理失败重试的场景。Redis 提供的 Stream 特性和幂等性机制确保了系统的健壮性和稳定性。
