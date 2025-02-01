---
title: Lua脚本在Redis中的应用
comment: true   # true/false对应开启/关闭本文章评论
date: 2025-01-16
tags:
  - Redis
  - Lua
categories:
  - [Redis]
---
## **Lua 语言详解**
Lua 是一种轻量级的脚本语言，常用于嵌入式开发，如游戏开发、数据库（如 Redis）、Nginx 以及嵌入式系统。它具有简洁的语法、快速执行速度和较低的资源消耗。

### **1. Lua 语言基础**
#### **（1）变量**
Lua 是动态类型语言，变量的类型在运行时确定。

```lua
-- 变量赋值
local x = 10
local y = "Hello, Lua!"
local z = true
```

#### **（2）数据类型**
Lua 主要有以下几种数据类型：
- `nil`：空值
- `boolean`：布尔类型（`true`/`false`）
- `number`：数值类型（整数和浮点数）
- `string`：字符串
- `table`：表（类似于 Python 的字典）
- `function`：函数

```lua
local a = nil
local b = true
local c = 3.14
local d = "Lua"
local e = { key1 = "value1", key2 = "value2" }
```

#### **（3）运算符**
```lua
-- 算术运算
local sum = 5 + 3    -- 加法
local sub = 10 - 4   -- 减法
local mul = 2 * 3    -- 乘法
local div = 10 / 2   -- 除法
local mod = 10 % 3   -- 取模
local exp = 2 ^ 3    -- 幂运算

-- 关系运算
local isEqual = (5 == 5)      -- true
local isNotEqual = (5 ~= 3)   -- true
local isGreater = (10 > 5)    -- true
```

#### **（4）控制结构**
##### **条件判断**
```lua
local age = 18
if age >= 18 then
    print("Adult")
elseif age >= 13 then
    print("Teenager")
else
    print("Child")
end
```

##### **循环**
```lua
-- for 循环
for i = 1, 5 do
    print(i)
end

-- while 循环
local i = 1
while i <= 5 do
    print(i)
    i = i + 1
end

-- repeat until 循环
local j = 1
repeat
    print(j)
    j = j + 1
until j > 5
```

#### **（5）函数**
```lua
function add(a, b)
    return a + b
end
print(add(5, 3))
```

#### **（6）Table（类似字典/数组）**
```lua
local t = { name = "Alice", age = 25 }
print(t.name)   -- 访问表中的值

t["gender"] = "female" -- 添加新键值对
print(t.gender)
```

---

## **Lua 在 Redis 中的应用**
Redis 允许用户在服务器端执行 Lua 脚本，这样可以实现高效的事务和批处理操作。Redis 使用 `EVAL` 命令执行 Lua 脚本，使得多个命令在一个事务中完成，避免网络往返和数据竞争。

### **1. 在 Redis 中使用 Lua**
**（1）基本执行**
```lua
EVAL "return 'Hello, Redis!'" 0
```
- `EVAL` 运行 Lua 脚本
- `"return 'Hello, Redis!'"` 代表返回一个字符串
- `0` 表示不需要使用 Redis 键

**（2）读取 Redis 键**
```lua
EVAL "return redis.call('GET', KEYS[1])" 1 mykey
```
- `redis.call('GET', KEYS[1])` 相当于 `GET mykey`
- `1 mykey` 代表传入一个键 `mykey`

**（3）写入 Redis**
```lua
EVAL "redis.call('SET', KEYS[1], ARGV[1])" 1 mykey "hello"
```
- `redis.call('SET', KEYS[1], ARGV[1])` 相当于 `SET mykey hello`
- `ARGV[1]` 代表传入的参数 `"hello"`

---

### **2. Redis Lua 事务**
Lua 在 Redis 中 **是原子执行的**，这意味着整个脚本会在执行时被锁定，直到完成执行后才会释放。

#### **（1）Redis 计数器**
```lua
EVAL "return redis.call('INCR', KEYS[1])" 1 counter
```
- `INCR` 命令对 `counter` 键进行自增。

#### **（2）Redis 分布式锁**
**获取锁**
```lua
EVAL "return redis.call('SET', KEYS[1], ARGV[1], 'NX', 'EX', ARGV[2])" 1 mylock "token123" 30
```
- `SET mylock "token123" NX EX 30`，表示：
   - `NX`：如果 `mylock` 不存在才设置
   - `EX`：设置过期时间 30 秒

**释放锁**
```lua
EVAL "
if redis.call('GET', KEYS[1]) == ARGV[1] then
    return redis.call('DEL', KEYS[1])
else
    return 0
end
" 1 mylock "token123"
```
- 先检查 `mylock` 是否是 `"token123"`，如果是则删除锁，否则返回 `0`。

---

### **3. 限流（Rate Limiting）**
**实现 API 访问限流，每个用户每分钟最多请求 10 次**
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
- `INCR` 递增请求次数
- `EXPIRE` 设置 60 秒过期
- `ARGV[2] = 10` 表示每分钟最多允许 10 次请求
- 如果超过 10 次，返回 `0`，否则返回 `1`

---

### **4. Redis Lua 实现转账**
**用户 A 给用户 B 转账**
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
- 检查 `userA` 余额是否足够
- 如果足够，扣除 `100`，然后给 `userB` 增加 `100`

---

## **总结**
- **Lua 语法简单**，包括变量、控制结构、函数、表等基本功能。
- **Redis Lua 脚本** 通过 `EVAL` 执行，并支持 `redis.call()` 来调用 Redis 命令。
- **Redis Lua 原子性** 确保整个脚本执行过程中数据不会被并发修改。
- **Redis Lua 应用场景**：
   1. **计数器**（如用户访问计数）
   2. **限流**（限制 API 访问频率）
   3. **分布式锁**（保证数据一致性）
   4. **转账系统**（事务性操作）