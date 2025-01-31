---
title: Lua脚本在Redis中的应用
comment: true   # true/false对应开启/关闭本文章评论
date: 2025-01-30
tags:
  - Redis
  - Lua
categories:
  - [Redis]
---

## Redis Lua 脚本讲解

Redis 支持在服务器端运行 Lua 脚本，这使得可以原子地执行多个命令，避免数据竞争，提高性能。Lua 脚本在 Redis 中通常用于事务、批量操作、缓存一致性控制等场景。

---

### 1. **基本用法**
Lua 脚本可以通过 Redis 的 `EVAL` 或 `EVALSHA` 命令执行。

**语法**
```shell
EVAL script numkeys key [key ...] arg [arg ...]
```

- `script`：要执行的 Lua 脚本
- `numkeys`：脚本要操作的键的数量
- `key [key ...]`：提供给 Lua 脚本的 Redis 键
- `arg [arg ...]`：传递的额外参数

---

### 2. **示例**
**（1）简单的 Lua 脚本**
```lua
EVAL "return 'Hello, Redis!'" 0
```
- 这里 `0` 表示没有传递键，直接返回 `'Hello, Redis!'`。

**（2）操作 Redis 键**
```lua
EVAL "return redis.call('GET', KEYS[1])" 1 mykey
```
- 读取键 `mykey` 的值，相当于 `GET mykey`。

**（3）修改键值**
```lua
EVAL "redis.call('SET', KEYS[1], ARGV[1])" 1 mykey "hello"
```
- 设置 `mykey` 的值为 `"hello"`，相当于 `SET mykey hello`。

---

### 3. **Lua 脚本中的 `redis.call` 和 `redis.pcall`**
Redis 提供了两个函数来执行命令：
- `redis.call(command, arg1, arg2, ...)`：
    - **普通调用**，如果命令失败会报错并终止执行。
- `redis.pcall(command, arg1, arg2, ...)`：
    - **安全调用**，如果命令失败不会终止，而是返回错误信息。

```lua
-- redis.call 失败时会抛出异常
EVAL "return redis.call('GET', 'nonexistent_key')" 0 

-- redis.pcall 失败时不会抛出异常，而是返回错误信息
EVAL "return redis.pcall('GET', 'nonexistent_key')" 0 
```

---

### 4. **Lua 变量与 Redis 交互**
```lua
EVAL "local value = redis.call('GET', KEYS[1]); return value" 1 mykey
```
- 先从 `mykey` 读取值赋给 `value` 变量，再返回 `value`。

```lua
EVAL "redis.call('SET', KEYS[1], ARGV[1]); return 'OK'" 1 mykey "new_value"
```
- 设置 `mykey` 为 `new_value`，并返回 `OK`。

---

### 5. **Lua 脚本实现原子操作**
Redis 的 Lua 脚本是 **单线程执行的**，不会被其他命令打断。因此，它可以保证多个 Redis 操作的 **原子性**。

#### **（1）实现计数器**
```lua
EVAL "return redis.call('INCR', KEYS[1])" 1 counter
```
- 计数器 `counter` 自增，相当于 `INCR counter`。

#### **（2）模拟事务**
假设我们要实现转账逻辑：
- 从账户 A 扣 100
- 给账户 B 加 100
- 确保 A 余额足够

```lua
EVAL "
    local balance = redis.call('GET', KEYS[1])
    if tonumber(balance) < tonumber(ARGV[1]) then
        return 'Insufficient funds'
    end
    redis.call('DECRBY', KEYS[1], ARGV[1])
    redis.call('INCRBY', KEYS[2], ARGV[1])
    return 'Transfer complete'
" 2 userA userB 100
```
- 读取 `userA` 的余额，如果不足则返回 `"Insufficient funds"`。
- 如果余额足够，从 `userA` 扣款，并增加 `userB` 余额。
- 这保证了整个转账过程的原子性，不会发生中途数据不一致的问题。

---

### 6. **EVALSHA 提高效率**
每次使用 `EVAL` 直接执行脚本，Redis 都需要解析和编译 Lua 代码，这会消耗性能。

Redis 允许将脚本缓存后 **用 SHA1 哈希值执行**：
1. **缓存脚本**
   ```shell
   SCRIPT LOAD "return redis.call('GET', KEYS[1])"
   ```
   Redis 返回一个 SHA1 哈希，例如：
   ```
   "sha1_hash_value"
   ```
2. **用 SHA1 运行脚本**
   ```shell
   EVALSHA "sha1_hash_value" 1 mykey
   ```
   这样可以提高执行速度，因为 Redis 不需要每次解析 Lua 代码。

---

### 7. **Lua 脚本的限制**
1. **不能执行 `TIME`, `MONITOR`, `SUBSCRIBE` 等命令**，因为它们依赖外部状态。
2. **执行时间不可过长**，否则会阻塞 Redis，影响性能。
3. **不能直接调用 `sleep`**，Redis 不能在 Lua 脚本里阻塞。

---

### 8. **应用场景**
1. **缓存一致性控制**
    - 例如，批量删除多个键，避免数据残留：
      ```lua
      EVAL "redis.call('DEL', KEYS[1], KEYS[2])" 2 key1 key2
      ```
2. **限流（Rate Limiting）**
    - 通过 Lua 实现限流，防止接口被刷：
      ```lua
      EVAL "
          local current = redis.call('INCR', KEYS[1])
          if current == 1 then
              redis.call('EXPIRE', KEYS[1], ARGV[1])
          end
          if current > tonumber(ARGV[2]) then
              return 0
          end
          return 1
      " 1 user:1234:rate_limit 60 10
      ```
    - 这里 `user:1234:rate_limit` 记录用户请求数
    - `ARGV[1] = 60` 表示 60 秒过期
    - `ARGV[2] = 10` 表示 10 次请求限制

3. **分布式锁**
    - 获取锁：
      ```lua
      EVAL "return redis.call('SET', KEYS[1], ARGV[1], 'NX', 'EX', ARGV[2])" 1 mylock "1234" 30
      ```
    - 释放锁：
      ```lua
      EVAL "
          if redis.call('GET', KEYS[1]) == ARGV[1] then
              return redis.call('DEL', KEYS[1])
          else
              return 0
          end
      " 1 mylock "1234"
      ```

---

## 总结
- `EVAL` 运行 Lua 脚本，可以操作 Redis 键值。
- `redis.call()` 执行 Redis 命令，`redis.pcall()` 允许错误恢复。
- Lua 脚本 **原子执行**，避免竞争条件，提高性能。
- `EVALSHA` 允许用 SHA1 执行缓存脚本，提高效率。
- 适用于 **事务控制、限流、分布式锁、批量操作** 等场景。
