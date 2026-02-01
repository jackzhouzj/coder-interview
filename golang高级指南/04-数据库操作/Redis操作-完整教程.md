# Redis操作 - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **Redis版本**：7.0+
- **Go Redis客户端**：go-redis/redis v9
- **推荐版本**：最新稳定版

### 学习难度
- **难度等级**：⭐⭐⭐ (中等)
- **预计学习时间**：15-20小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Go语言基础
- Redis基础概念
- 数据库基础

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 掌握go-redis客户端的使用
- [ ] 理解Redis的五种基本数据类型
- [ ] 掌握Redis的高级特性
- [ ] 能够实现分布式锁
- [ ] 掌握Redis的性能优化
- [ ] 理解Redis的最佳实践

## 📖 目录

1. [Redis简介](#1-redis简介)
2. [连接Redis](#2-连接redis)
3. [基本数据类型](#3-基本数据类型)
4. [高级特性](#4-高级特性)
5. [分布式锁](#5-分布式锁)
6. [Pipeline和事务](#6-pipeline和事务)
7. [发布订阅](#7-发布订阅)
8. [最佳实践](#8-最佳实践)

---

## 1. Redis简介

### 1.1 什么是Redis

Redis是一个开源的内存数据结构存储系统，可以用作数据库、缓存和消息代理。

**核心特点**：
- 🔥 **高性能**：基于内存，读写速度极快
- 🔥 **丰富的数据类型**：String、Hash、List、Set、ZSet等
- 🔥 **持久化**：支持RDB和AOF持久化
- 🔥 **高可用**：支持主从复制、哨兵、集群
- 🔥 **原子操作**：所有操作都是原子性的

### 1.2 安装go-redis

```bash
# 🔥 安装go-redis v9
go get github.com/redis/go-redis/v9
```

---

## 2. 连接Redis

### 2.1 单机连接

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
    // 🔥 创建Redis客户端
    rdb := redis.NewClient(&redis.Options{
        Addr:     "localhost:6379",
        Password: "",  // 密码
        DB:       0,   // 数据库
        PoolSize: 10,  // 连接池大小
    })
    
    // 🔥 测试连接
    pong, err := rdb.Ping(ctx).Result()
    if err != nil {
        panic(err)
    }
    fmt.Println("连接成功:", pong)
    
    defer rdb.Close()
}
```

### 2.2 集群连接

```go
package main

import (
    "context"
    "github.com/redis/go-redis/v9"
)

func main() {
    // 🔥 连接Redis集群
    rdb := redis.NewClusterClient(&redis.ClusterOptions{
        Addrs: []string{
            "localhost:7000",
            "localhost:7001",
            "localhost:7002",
        },
        Password: "",
    })
    
    defer rdb.Close()
}
```

### 2.3 哨兵模式

```go
package main

import (
    "context"
    "github.com/redis/go-redis/v9"
)

func main() {
    // 🔥 连接Redis哨兵
    rdb := redis.NewFailoverClient(&redis.FailoverOptions{
        MasterName:    "mymaster",
        SentinelAddrs: []string{"localhost:26379"},
        Password:      "",
    })
    
    defer rdb.Close()
}
```

---

## 3. 基本数据类型

### 3.1 String类型

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
    "time"
)

var ctx = context.Background()

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    defer rdb.Close()
    
    // 🔥 SET：设置值
    err := rdb.Set(ctx, "key", "value", 0).Err()
    if err != nil {
        panic(err)
    }
    
    // 🔥 GET：获取值
    val, err := rdb.Get(ctx, "key").Result()
    if err != nil {
        panic(err)
    }
    fmt.Println("key:", val)
    
    // 🔥 SETEX：设置值并指定过期时间
    err = rdb.Set(ctx, "key2", "value2", 10*time.Second).Err()
    
    // 🔥 SETNX：仅当key不存在时设置
    ok, err := rdb.SetNX(ctx, "key3", "value3", 0).Result()
    fmt.Println("SETNX:", ok)
    
    // 🔥 INCR：自增
    val2, err := rdb.Incr(ctx, "counter").Result()
    fmt.Println("counter:", val2)
    
    // 🔥 INCRBY：增加指定值
    val3, err := rdb.IncrBy(ctx, "counter", 10).Result()
    fmt.Println("counter:", val3)
    
    // 🔥 DECR：自减
    val4, err := rdb.Decr(ctx, "counter").Result()
    fmt.Println("counter:", val4)
    
    // 🔥 MSET：批量设置
    err = rdb.MSet(ctx, "key1", "value1", "key2", "value2").Err()
    
    // 🔥 MGET：批量获取
    vals, err := rdb.MGet(ctx, "key1", "key2").Result()
    fmt.Println("MGET:", vals)
}
```

### 3.2 Hash类型

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    defer rdb.Close()
    
    // 🔥 HSET：设置字段
    err := rdb.HSet(ctx, "user:1", "name", "张三").Err()
    err = rdb.HSet(ctx, "user:1", "age", 25).Err()
    
    // 🔥 HGET：获取字段
    name, err := rdb.HGet(ctx, "user:1", "name").Result()
    fmt.Println("name:", name)
    
    // 🔥 HMSET：批量设置
    err = rdb.HMSet(ctx, "user:2", map[string]interface{}{
        "name": "李四",
        "age":  30,
        "city": "北京",
    }).Err()
    
    // 🔥 HMGET：批量获取
    vals, err := rdb.HMGet(ctx, "user:2", "name", "age", "city").Result()
    fmt.Println("HMGET:", vals)
    
    // 🔥 HGETALL：获取所有字段
    user, err := rdb.HGetAll(ctx, "user:2").Result()
    fmt.Println("HGETALL:", user)
    
    // 🔥 HDEL：删除字段
    err = rdb.HDel(ctx, "user:2", "city").Err()
    
    // 🔥 HEXISTS：检查字段是否存在
    exists, err := rdb.HExists(ctx, "user:2", "city").Result()
    fmt.Println("HEXISTS:", exists)
    
    // 🔥 HINCRBY：字段值增加
    newAge, err := rdb.HIncrBy(ctx, "user:2", "age", 1).Result()
    fmt.Println("new age:", newAge)
}
```

### 3.3 List类型

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    defer rdb.Close()
    
    // 🔥 LPUSH：从左侧插入
    err := rdb.LPush(ctx, "list", "a", "b", "c").Err()
    
    // 🔥 RPUSH：从右侧插入
    err = rdb.RPush(ctx, "list", "d", "e").Err()
    
    // 🔥 LRANGE：获取范围内的元素
    vals, err := rdb.LRange(ctx, "list", 0, -1).Result()
    fmt.Println("LRANGE:", vals)
    
    // 🔥 LPOP：从左侧弹出
    val, err := rdb.LPop(ctx, "list").Result()
    fmt.Println("LPOP:", val)
    
    // 🔥 RPOP：从右侧弹出
    val, err = rdb.RPop(ctx, "list").Result()
    fmt.Println("RPOP:", val)
    
    // 🔥 LLEN：获取列表长度
    length, err := rdb.LLen(ctx, "list").Result()
    fmt.Println("LLEN:", length)
    
    // 🔥 LINDEX：获取指定索引的元素
    val, err = rdb.LIndex(ctx, "list", 0).Result()
    fmt.Println("LINDEX:", val)
    
    // 🔥 LSET：设置指定索引的元素
    err = rdb.LSet(ctx, "list", 0, "new_value").Err()
}
```

### 3.4 Set类型

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    defer rdb.Close()
    
    // 🔥 SADD：添加成员
    err := rdb.SAdd(ctx, "set", "a", "b", "c").Err()
    
    // 🔥 SMEMBERS：获取所有成员
    members, err := rdb.SMembers(ctx, "set").Result()
    fmt.Println("SMEMBERS:", members)
    
    // 🔥 SISMEMBER：检查成员是否存在
    exists, err := rdb.SIsMember(ctx, "set", "a").Result()
    fmt.Println("SISMEMBER:", exists)
    
    // 🔥 SCARD：获取集合大小
    size, err := rdb.SCard(ctx, "set").Result()
    fmt.Println("SCARD:", size)
    
    // 🔥 SREM：删除成员
    err = rdb.SRem(ctx, "set", "a").Err()
    
    // 🔥 SPOP：随机弹出成员
    val, err := rdb.SPop(ctx, "set").Result()
    fmt.Println("SPOP:", val)
    
    // 🔥 集合运算
    rdb.SAdd(ctx, "set1", "a", "b", "c")
    rdb.SAdd(ctx, "set2", "b", "c", "d")
    
    // 交集
    inter, err := rdb.SInter(ctx, "set1", "set2").Result()
    fmt.Println("SINTER:", inter)
    
    // 并集
    union, err := rdb.SUnion(ctx, "set1", "set2").Result()
    fmt.Println("SUNION:", union)
    
    // 差集
    diff, err := rdb.SDiff(ctx, "set1", "set2").Result()
    fmt.Println("SDIFF:", diff)
}
```

### 3.5 Sorted Set类型

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    defer rdb.Close()
    
    // 🔥 ZADD：添加成员
    err := rdb.ZAdd(ctx, "rank", redis.Z{Score: 100, Member: "user1"}).Err()
    err = rdb.ZAdd(ctx, "rank", redis.Z{Score: 90, Member: "user2"}).Err()
    err = rdb.ZAdd(ctx, "rank", redis.Z{Score: 95, Member: "user3"}).Err()
    
    // 🔥 ZRANGE：按分数升序获取
    vals, err := rdb.ZRange(ctx, "rank", 0, -1).Result()
    fmt.Println("ZRANGE:", vals)
    
    // 🔥 ZREVRANGE：按分数降序获取
    vals, err = rdb.ZRevRange(ctx, "rank", 0, -1).Result()
    fmt.Println("ZREVRANGE:", vals)
    
    // 🔥 ZRANGEWITHSCORES：获取成员和分数
    valsWithScores, err := rdb.ZRangeWithScores(ctx, "rank", 0, -1).Result()
    for _, v := range valsWithScores {
        fmt.Printf("%s: %.0f\n", v.Member, v.Score)
    }
    
    // 🔥 ZSCORE：获取成员分数
    score, err := rdb.ZScore(ctx, "rank", "user1").Result()
    fmt.Println("ZSCORE:", score)
    
    // 🔥 ZRANK：获取成员排名（升序）
    rank, err := rdb.ZRank(ctx, "rank", "user1").Result()
    fmt.Println("ZRANK:", rank)
    
    // 🔥 ZREVRANK：获取成员排名（降序）
    revRank, err := rdb.ZRevRank(ctx, "rank", "user1").Result()
    fmt.Println("ZREVRANK:", revRank)
    
    // 🔥 ZINCRBY：增加成员分数
    newScore, err := rdb.ZIncrBy(ctx, "rank", 10, "user1").Result()
    fmt.Println("new score:", newScore)
    
    // 🔥 ZREM：删除成员
    err = rdb.ZRem(ctx, "rank", "user2").Err()
    
    // 🔥 ZCARD：获取集合大小
    size, err := rdb.ZCard(ctx, "rank").Result()
    fmt.Println("ZCARD:", size)
}
```

---

## 4. 高级特性

### 4.1 过期时间

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
    "time"
)

var ctx = context.Background()

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    defer rdb.Close()
    
    // 🔥 设置过期时间
    err := rdb.Set(ctx, "key", "value", 10*time.Second).Err()
    
    // 🔥 EXPIRE：设置过期时间（秒）
    err = rdb.Expire(ctx, "key", 20*time.Second).Err()
    
    // 🔥 EXPIREAT：设置过期时间戳
    err = rdb.ExpireAt(ctx, "key", time.Now().Add(30*time.Second)).Err()
    
    // 🔥 TTL：获取剩余过期时间
    ttl, err := rdb.TTL(ctx, "key").Result()
    fmt.Println("TTL:", ttl)
    
    // 🔥 PERSIST：移除过期时间
    err = rdb.Persist(ctx, "key").Err()
}
```

### 4.2 键操作

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    defer rdb.Close()
    
    // 🔥 EXISTS：检查键是否存在
    exists, err := rdb.Exists(ctx, "key").Result()
    fmt.Println("EXISTS:", exists)
    
    // 🔥 DEL：删除键
    err = rdb.Del(ctx, "key").Err()
    
    // 🔥 KEYS：查找键（生产环境慎用）
    keys, err := rdb.Keys(ctx, "user:*").Result()
    fmt.Println("KEYS:", keys)
    
    // 🔥 SCAN：迭代键（推荐）
    var cursor uint64
    for {
        var keys []string
        var err error
        keys, cursor, err = rdb.Scan(ctx, cursor, "user:*", 10).Result()
        if err != nil {
            panic(err)
        }
        fmt.Println("SCAN:", keys)
        if cursor == 0 {
            break
        }
    }
    
    // 🔥 TYPE：获取键的类型
    keyType, err := rdb.Type(ctx, "key").Result()
    fmt.Println("TYPE:", keyType)
    
    // 🔥 RENAME：重命名键
    err = rdb.Rename(ctx, "old_key", "new_key").Err()
}
```

---

## 5. 分布式锁

### 5.1 简单分布式锁

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
    "time"
)

var ctx = context.Background()

type RedisLock struct {
    client *redis.Client
    key    string
    value  string
    expire time.Duration
}

func NewRedisLock(client *redis.Client, key string, expire time.Duration) *RedisLock {
    return &RedisLock{
        client: client,
        key:    key,
        value:  fmt.Sprintf("%d", time.Now().UnixNano()),
        expire: expire,
    }
}

// 🔥 获取锁
func (l *RedisLock) Lock() bool {
    ok, err := l.client.SetNX(ctx, l.key, l.value, l.expire).Result()
    if err != nil {
        return false
    }
    return ok
}

// 🔥 释放锁
func (l *RedisLock) Unlock() bool {
    // 使用Lua脚本保证原子性
    script := `
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
    `
    result, err := l.client.Eval(ctx, script, []string{l.key}, l.value).Result()
    if err != nil {
        return false
    }
    return result.(int64) == 1
}

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    defer rdb.Close()
    
    lock := NewRedisLock(rdb, "my_lock", 10*time.Second)
    
    if lock.Lock() {
        fmt.Println("获取锁成功")
        // 执行业务逻辑
        time.Sleep(2 * time.Second)
        lock.Unlock()
        fmt.Println("释放锁成功")
    } else {
        fmt.Println("获取锁失败")
    }
}
```

---

## 6. Pipeline和事务

### 6.1 Pipeline

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    defer rdb.Close()
    
    // 🔥 使用Pipeline批量执行命令
    pipe := rdb.Pipeline()
    
    incr := pipe.Incr(ctx, "counter")
    pipe.Expire(ctx, "counter", 3600)
    
    // 执行Pipeline
    _, err := pipe.Exec(ctx)
    if err != nil {
        panic(err)
    }
    
    // 获取结果
    fmt.Println("counter:", incr.Val())
}
```

### 6.2 事务

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    defer rdb.Close()
    
    // 🔥 使用TxPipeline执行事务
    pipe := rdb.TxPipeline()
    
    pipe.Set(ctx, "key1", "value1", 0)
    pipe.Set(ctx, "key2", "value2", 0)
    pipe.Incr(ctx, "counter")
    
    // 执行事务
    _, err := pipe.Exec(ctx)
    if err != nil {
        panic(err)
    }
    
    fmt.Println("事务执行成功")
}
```

---

## 7. 发布订阅

### 7.1 发布消息

```go
package main

import (
    "context"
    "github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    defer rdb.Close()
    
    // 🔥 发布消息
    err := rdb.Publish(ctx, "channel", "hello").Err()
    if err != nil {
        panic(err)
    }
}
```

### 7.2 订阅消息

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func main() {
    rdb := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
    defer rdb.Close()
    
    // 🔥 订阅频道
    pubsub := rdb.Subscribe(ctx, "channel")
    defer pubsub.Close()
    
    // 接收消息
    ch := pubsub.Channel()
    for msg := range ch {
        fmt.Println("收到消息:", msg.Payload)
    }
}
```

---

## 8. 最佳实践

### 8.1 连接池配置

```go
rdb := redis.NewClient(&redis.Options{
    Addr:         "localhost:6379",
    PoolSize:     10,              // 连接池大小
    MinIdleConns: 5,               // 最小空闲连接数
    MaxRetries:   3,               // 最大重试次数
    DialTimeout:  5 * time.Second, // 连接超时
    ReadTimeout:  3 * time.Second, // 读超时
    WriteTimeout: 3 * time.Second, // 写超时
})
```

### 8.2 错误处理

```go
val, err := rdb.Get(ctx, "key").Result()
if err == redis.Nil {
    fmt.Println("key不存在")
} else if err != nil {
    fmt.Println("错误:", err)
} else {
    fmt.Println("值:", val)
}
```

### 8.3 使用Context

```go
// 设置超时
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

val, err := rdb.Get(ctx, "key").Result()
```

---

## 📝 学习检查清单

- [ ] 掌握go-redis客户端的使用
- [ ] 理解Redis的五种基本数据类型
- [ ] 掌握Redis的过期时间设置
- [ ] 能够实现分布式锁
- [ ] 掌握Pipeline和事务
- [ ] 理解发布订阅模式
- [ ] 掌握Redis的最佳实践

---

## 🔗 相关资源

- [go-redis文档](https://redis.uptrace.dev/)
- [Redis官方文档](https://redis.io/docs/)
- [Redis命令参考](https://redis.io/commands/)

---

@author erik.zhou
