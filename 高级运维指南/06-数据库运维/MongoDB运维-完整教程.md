# MongoDB运维完整教程

> 企业级MongoDB集群运维与性能优化实战指南
>
> @author erik.zhou

## 📚 目录

- [1. MongoDB基础架构](#1-mongodb基础架构)
- [2. MongoDB安装部署](#2-mongodb安装部署)
- [3. MongoDB配置优化](#3-mongodb配置优化)
- [4. 副本集架构](#4-副本集架构)
- [5. 分片集群](#5-分片集群)
- [6. 索引优化](#6-索引优化)
- [7. 查询优化](#7-查询优化)
- [8. 性能监控](#8-性能监控)
- [9. 备份恢复](#9-备份恢复)
- [10. 安全加固](#10-安全加固)
- [11. 故障排查](#11-故障排查)
- [12. 实战案例](#12-实战案例)
- [13. 学习检查清单](#13-学习检查清单)

## 🎯 学习目标

- 掌握MongoDB的架构和部署方式
- 能够配置和优化MongoDB性能
- 掌握副本集和分片集群的搭建
- 能够进行索引和查询优化
- 掌握MongoDB监控和故障排查
- 了解MongoDB安全加固方法
- 能够处理常见的MongoDB运维问题

## 1. MongoDB基础架构

### 1.1 MongoDB架构模式

```yaml
# MongoDB部署架构对比

单机模式:
  优点:
    - 部署简单
    - 维护成本低
    - 适合开发测试
  缺点:
    - 无高可用
    - 单点故障
    - 性能受限
  适用场景:
    - 开发测试环境
    - 小规模应用
    - 数据可丢失

副本集模式:
  优点:
    - 高可用
    - 自动故障转移
    - 读写分离
    - 数据冗余
  缺点:
    - 存储容量受限
    - 写性能受限
  适用场景:
    - 生产环境
    - 需要高可用
    - 中小规模数据

分片集群:
  优点:
    - 水平扩展
    - 海量数据存储
    - 高并发支持
  缺点:
    - 架构复杂
    - 运维成本高
    - 某些操作受限
  适用场景:
    - 大规模数据
    - 高并发场景
    - 需要扩展性
```

### 1.2 MongoDB核心概念

```javascript
// MongoDB核心概念对比

// 1. 数据库（Database）
// 类似关系型数据库的Database
use mydb

// 2. 集合（Collection）
// 类似关系型数据库的Table
db.createCollection("users")

// 3. 文档（Document）
// 类似关系型数据库的Row，使用BSON格式
{
    "_id": ObjectId("507f1f77bcf86cd799439011"),
    "name": "Alice",
    "age": 30,
    "email": "alice@example.com",
    "tags": ["developer", "mongodb"],
    "address": {
        "city": "Beijing",
        "country": "China"
    }
}

// 4. 字段（Field）
// 类似关系型数据库的Column

// 5. 索引（Index）
// 提高查询性能
db.users.createIndex({ email: 1 })

// 6. 聚合（Aggregation）
// 数据处理和分析
db.users.aggregate([
    { $match: { age: { $gte: 18 } } },
    { $group: { _id: "$city", count: { $sum: 1 } } }
])
```

### 1.3 存储引擎

```yaml
# WiredTiger存储引擎（默认）

特点:
  - 文档级并发控制
  - 压缩支持
  - 检查点机制
  - 日志预写

配置:
  storage:
    engine: wiredTiger
    wiredTiger:
      engineConfig:
        cacheSizeGB: 4
        journalCompressor: snappy
      collectionConfig:
        blockCompressor: snappy
      indexConfig:
        prefixCompression: true
```

## 2. MongoDB安装部署

### 2.1 YUM安装

```bash
# 配置MongoDB仓库
cat > /etc/yum.repos.d/mongodb-org-7.0.repo << 'EOF'
[mongodb-org-7.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/7.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-7.0.asc
EOF

# 安装MongoDB
yum install -y mongodb-org

# 创建数据和日志目录
mkdir -p /data/mongodb/{data,logs}
chown -R mongod:mongod /data/mongodb

# 启动服务
systemctl enable mongod
systemctl start mongod
systemctl status mongod

# 验证安装
mongosh --eval "db.version()"
```

### 2.2 配置文件

```yaml
# /etc/mongod.conf

# 系统日志
systemLog:
  destination: file
  path: /data/mongodb/logs/mongod.log
  logAppend: true
  logRotate: reopen

# 数据存储
storage:
  dbPath: /data/mongodb/data
  journal:
    enabled: true
  engine: wiredTiger
  wiredTiger:
    engineConfig:
      cacheSizeGB: 4
      journalCompressor: snappy
    collectionConfig:
      blockCompressor: snappy

# 网络配置
net:
  port: 27017
  bindIp: 0.0.0.0
  maxIncomingConnections: 10000

# 安全配置
security:
  authorization: enabled
  keyFile: /etc/mongodb/keyfile

# 操作分析
operationProfiling:
  mode: slowOp
  slowOpThresholdMs: 100

# 副本集配置
replication:
  replSetName: rs0

# 分片配置
sharding:
  clusterRole: shardsvr
```

### 2.3 Docker部署

```yaml
# docker-compose.yml
version: '3.8'

services:
  mongodb:
    image: mongo:7.0
    container_name: mongodb
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: your_password
      MONGO_INITDB_DATABASE: mydb
    volumes:
      - mongodb-data:/data/db
      - mongodb-config:/data/configdb
      - ./mongod.conf:/etc/mongod.conf
      - ./init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js
    command: mongod --config /etc/mongod.conf
    networks:
      - mongodb-network
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 3

volumes:
  mongodb-data:
  mongodb-config:

networks:
  mongodb-network:
    driver: bridge
```

```javascript
// init-mongo.js
db = db.getSiblingDB('mydb');

// 创建应用用户
db.createUser({
    user: 'appuser',
    pwd: 'app_password',
    roles: [
        {
            role: 'readWrite',
            db: 'mydb'
        }
    ]
});

// 创建集合
db.createCollection('users');
db.createCollection('orders');

// 创建索引
db.users.createIndex({ email: 1 }, { unique: true });
db.orders.createIndex({ user_id: 1, created_at: -1 });
```

## 3. MongoDB配置优化

### 3.1 内存配置

```yaml
# mongod.conf

storage:
  wiredTiger:
    engineConfig:
      # WiredTiger缓存大小（默认为物理内存的50%）
      cacheSizeGB: 4
      
      # 日志压缩
      journalCompressor: snappy
      
      # 目录配置
      directoryForIndexes: true
    
    collectionConfig:
      # 集合数据压缩
      blockCompressor: snappy
    
    indexConfig:
      # 索引前缀压缩
      prefixCompression: true
```

### 3.2 连接池配置

```yaml
# mongod.conf

net:
  # 最大连接数
  maxIncomingConnections: 10000
  
  # 连接超时
  connectionOptions:
    maxPoolSize: 100
    minPoolSize: 10
    maxIdleTimeMS: 300000
    waitQueueTimeoutMS: 10000
```

### 3.3 系统参数优化

```bash
# /etc/sysctl.conf

# 虚拟内存
vm.swappiness = 1
vm.dirty_ratio = 15
vm.dirty_background_ratio = 5

# 文件系统
fs.file-max = 65535

# 网络
net.core.somaxconn = 4096
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 300

# 应用配置
sysctl -p
```

```bash
# /etc/security/limits.conf
mongod soft nofile 64000
mongod hard nofile 64000
mongod soft nproc 64000
mongod hard nproc 64000
```

```bash
# 禁用透明大页
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag

# 永久生效
cat > /etc/systemd/system/disable-thp.service << 'EOF'
[Unit]
Description=Disable Transparent Huge Pages

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo never > /sys/kernel/mm/transparent_hugepage/enabled'
ExecStart=/bin/sh -c 'echo never > /sys/kernel/mm/transparent_hugepage/defrag'

[Install]
WantedBy=multi-user.target
EOF

systemctl enable disable-thp
systemctl start disable-thp
```

## 4. 副本集架构

### 4.1 副本集部署

```bash
# 1. 生成keyfile（用于副本集认证）
openssl rand -base64 756 > /etc/mongodb/keyfile
chmod 400 /etc/mongodb/keyfile
chown mongod:mongod /etc/mongodb/keyfile

# 2. 配置三个节点
# 节点1: mongodb-1 (Primary)
# 节点2: mongodb-2 (Secondary)
# 节点3: mongodb-3 (Secondary)

# 3. 每个节点的配置文件
cat > /etc/mongod.conf << 'EOF'
systemLog:
  destination: file
  path: /data/mongodb/logs/mongod.log
  logAppend: true

storage:
  dbPath: /data/mongodb/data
  journal:
    enabled: true

net:
  port: 27017
  bindIp: 0.0.0.0

security:
  authorization: enabled
  keyFile: /etc/mongodb/keyfile

replication:
  replSetName: rs0
EOF

# 4. 启动所有节点
systemctl start mongod

# 5. 初始化副本集（在任意节点执行）
mongosh --host mongodb-1:27017

rs.initiate({
    _id: "rs0",
    members: [
        { _id: 0, host: "mongodb-1:27017", priority: 2 },
        { _id: 1, host: "mongodb-2:27017", priority: 1 },
        { _id: 2, host: "mongodb-3:27017", priority: 1 }
    ]
})

# 6. 查看副本集状态
rs.status()
rs.conf()
```

### 4.2 副本集管理

```javascript
// 连接到副本集
mongosh "mongodb://mongodb-1:27017,mongodb-2:27017,mongodb-3:27017/?replicaSet=rs0"

// 查看副本集状态
rs.status()

// 查看副本集配置
rs.conf()

// 添加节点
rs.add("mongodb-4:27017")

// 添加仲裁节点
rs.addArb("mongodb-5:27017")

// 删除节点
rs.remove("mongodb-4:27017")

// 修改节点优先级
cfg = rs.conf()
cfg.members[1].priority = 2
rs.reconfig(cfg)

// 手动触发选举
rs.stepDown()

// 查看复制延迟
rs.printReplicationInfo()
rs.printSecondaryReplicationInfo()
```

### 4.3 读写分离

```javascript
// 设置读偏好

// 1. primary（默认）- 只从主节点读
db.users.find().readPref("primary")

// 2. primaryPreferred - 优先主节点，主节点不可用时从从节点读
db.users.find().readPref("primaryPreferred")

// 3. secondary - 只从从节点读
db.users.find().readPref("secondary")

// 4. secondaryPreferred - 优先从节点，从节点不可用时从主节点读
db.users.find().readPref("secondaryPreferred")

// 5. nearest - 从网络延迟最小的节点读
db.users.find().readPref("nearest")
```

```python
"""
Python客户端读写分离
"""
from pymongo import MongoClient, ReadPreference

# 连接副本集
client = MongoClient(
    'mongodb://mongodb-1:27017,mongodb-2:27017,mongodb-3:27017/',
    replicaSet='rs0',
    username='admin',
    password='password'
)

# 写操作（主节点）
db = client.mydb
db.users.insert_one({'name': 'Alice', 'age': 30})

# 读操作（从节点）
db_read = client.get_database(
    'mydb',
    read_preference=ReadPreference.SECONDARY_PREFERRED
)
users = db_read.users.find({'age': {'$gte': 18}})
```


## 5. 分片集群

### 5.1 分片集群架构

```
┌─────────────────────────────────────────┐
│         MongoDB Sharded Cluster          │
├─────────────────────────────────────────┤
│  mongos  mongos  mongos                 │
│  (路由)  (路由)  (路由)                 │
├─────────────────────────────────────────┤
│         Config Servers (副本集)          │
│  config-1  config-2  config-3           │
├─────────────────────────────────────────┤
│  Shard 1      Shard 2      Shard 3      │
│  (副本集)     (副本集)     (副本集)     │
└─────────────────────────────────────────┘
```

### 5.2 分片集群部署

```bash
# 1. 部署Config Server副本集
# config-1, config-2, config-3

# config server配置
cat > /etc/mongod-config.conf << 'EOF'
systemLog:
  destination: file
  path: /data/mongodb/logs/mongod-config.log
  logAppend: true

storage:
  dbPath: /data/mongodb/config
  journal:
    enabled: true

net:
  port: 27019
  bindIp: 0.0.0.0

security:
  keyFile: /etc/mongodb/keyfile

replication:
  replSetName: configReplSet

sharding:
  clusterRole: configsvr
EOF

# 启动config servers
systemctl start mongod-config

# 初始化config server副本集
mongosh --host config-1:27019
rs.initiate({
    _id: "configReplSet",
    configsvr: true,
    members: [
        { _id: 0, host: "config-1:27019" },
        { _id: 1, host: "config-2:27019" },
        { _id: 2, host: "config-3:27019" }
    ]
})

# 2. 部署Shard副本集
# shard1: shard1-1, shard1-2, shard1-3
# shard2: shard2-1, shard2-2, shard2-3

# shard配置
cat > /etc/mongod-shard.conf << 'EOF'
systemLog:
  destination: file
  path: /data/mongodb/logs/mongod-shard.log
  logAppend: true

storage:
  dbPath: /data/mongodb/shard
  journal:
    enabled: true

net:
  port: 27018
  bindIp: 0.0.0.0

security:
  keyFile: /etc/mongodb/keyfile

replication:
  replSetName: shard1

sharding:
  clusterRole: shardsvr
EOF

# 启动shard servers
systemctl start mongod-shard

# 初始化shard副本集
mongosh --host shard1-1:27018
rs.initiate({
    _id: "shard1",
    members: [
        { _id: 0, host: "shard1-1:27018" },
        { _id: 1, host: "shard1-2:27018" },
        { _id: 2, host: "shard1-3:27018" }
    ]
})

# 3. 部署mongos路由
cat > /etc/mongos.conf << 'EOF'
systemLog:
  destination: file
  path: /data/mongodb/logs/mongos.log
  logAppend: true

net:
  port: 27017
  bindIp: 0.0.0.0

security:
  keyFile: /etc/mongodb/keyfile

sharding:
  configDB: configReplSet/config-1:27019,config-2:27019,config-3:27019
EOF

# 启动mongos
mongos --config /etc/mongos.conf

# 4. 添加分片
mongosh --host mongos:27017
sh.addShard("shard1/shard1-1:27018,shard1-2:27018,shard1-3:27018")
sh.addShard("shard2/shard2-1:27018,shard2-2:27018,shard2-3:27018")

# 5. 启用数据库分片
sh.enableSharding("mydb")

# 6. 对集合进行分片
sh.shardCollection("mydb.users", { user_id: "hashed" })
```

### 5.3 分片策略

```javascript
// 1. 哈希分片（Hash Sharding）
// 优点：数据分布均匀
// 缺点：范围查询性能差
sh.shardCollection("mydb.users", { user_id: "hashed" })

// 2. 范围分片（Range Sharding）
// 优点：范围查询性能好
// 缺点：可能数据分布不均
sh.shardCollection("mydb.orders", { order_date: 1 })

// 3. 复合分片键
// 结合多个字段
sh.shardCollection("mydb.logs", { user_id: 1, timestamp: 1 })

// 4. 区域分片（Zone Sharding）
// 将数据分配到特定分片
sh.addShardTag("shard1", "US")
sh.addShardTag("shard2", "EU")
sh.addTagRange(
    "mydb.users",
    { country: "US", user_id: MinKey },
    { country: "US", user_id: MaxKey },
    "US"
)
```

### 5.4 分片管理

```javascript
// 查看分片状态
sh.status()

// 查看集合分片信息
db.users.getShardDistribution()

// 查看chunk分布
db.chunks.find({ ns: "mydb.users" }).count()

// 平衡器管理
sh.getBalancerState()
sh.startBalancer()
sh.stopBalancer()
sh.setBalancerState(true)

// 手动移动chunk
sh.moveChunk("mydb.users", { user_id: 12345 }, "shard2")

// 分割chunk
sh.splitAt("mydb.users", { user_id: 50000 })
```

## 6. 索引优化

### 6.1 索引类型

```javascript
// 1. 单字段索引
db.users.createIndex({ email: 1 })

// 2. 复合索引
db.users.createIndex({ age: 1, city: 1 })

// 3. 多键索引（数组字段）
db.users.createIndex({ tags: 1 })

// 4. 文本索引
db.articles.createIndex({ content: "text" })

// 5. 地理空间索引
db.places.createIndex({ location: "2dsphere" })

// 6. 哈希索引
db.users.createIndex({ user_id: "hashed" })

// 7. 唯一索引
db.users.createIndex({ email: 1 }, { unique: true })

// 8. 部分索引
db.users.createIndex(
    { email: 1 },
    { partialFilterExpression: { age: { $gte: 18 } } }
)

// 9. TTL索引（自动过期）
db.sessions.createIndex(
    { created_at: 1 },
    { expireAfterSeconds: 3600 }
)

// 10. 稀疏索引
db.users.createIndex(
    { phone: 1 },
    { sparse: true }
)
```

### 6.2 索引管理

```javascript
// 查看索引
db.users.getIndexes()

// 查看索引大小
db.users.stats().indexSizes

// 删除索引
db.users.dropIndex("email_1")
db.users.dropIndexes()  // 删除所有索引（除了_id）

// 重建索引
db.users.reIndex()

// 后台创建索引
db.users.createIndex({ age: 1 }, { background: true })

// 查看索引使用情况
db.users.aggregate([
    { $indexStats: {} }
])
```

### 6.3 索引优化策略

```javascript
// 1. ESR规则（Equality, Sort, Range）
// 等值查询 -> 排序 -> 范围查询
db.users.createIndex({ status: 1, age: 1, created_at: 1 })

// 查询示例
db.users.find({
    status: "active",      // Equality
    age: { $gte: 18 }      // Range
}).sort({ created_at: -1 }) // Sort

// 2. 覆盖索引
// 查询字段都在索引中，无需访问文档
db.users.createIndex({ email: 1, name: 1, age: 1 })
db.users.find(
    { email: "alice@example.com" },
    { _id: 0, name: 1, age: 1 }
)

// 3. 索引交集
// MongoDB可以使用多个索引
db.users.createIndex({ age: 1 })
db.users.createIndex({ city: 1 })
db.users.find({ age: 30, city: "Beijing" })
```

```python
"""
索引分析工具
"""
#!/usr/bin/env python3
from pymongo import MongoClient

class IndexAnalyzer:
    def __init__(self, uri, database):
        self.client = MongoClient(uri)
        self.db = self.client[database]
    
    def analyze_collection(self, collection_name):
        """分析集合索引"""
        collection = self.db[collection_name]
        
        # 获取索引信息
        indexes = collection.index_information()
        
        print(f"集合 {collection_name} 的索引分析:")
        print("=" * 60)
        
        for index_name, index_info in indexes.items():
            print(f"\n索引名称: {index_name}")
            print(f"  字段: {index_info['key']}")
            
            # 获取索引大小
            stats = collection.aggregate([
                { "$collStats": { "storageStats": {} } }
            ]).next()
            
            if 'indexSizes' in stats['storageStats']:
                size = stats['storageStats']['indexSizes'].get(index_name, 0)
                print(f"  大小: {size / 1024 / 1024:.2f} MB")
        
        # 获取索引使用统计
        index_stats = list(collection.aggregate([
            { "$indexStats": {} }
        ]))
        
        print("\n索引使用统计:")
        for stat in index_stats:
            print(f"  {stat['name']}: {stat['accesses']['ops']} 次访问")
    
    def find_unused_indexes(self, collection_name):
        """查找未使用的索引"""
        collection = self.db[collection_name]
        
        index_stats = list(collection.aggregate([
            { "$indexStats": {} }
        ]))
        
        unused = [
            stat['name'] for stat in index_stats
            if stat['accesses']['ops'] == 0 and stat['name'] != '_id_'
        ]
        
        if unused:
            print(f"\n未使用的索引:")
            for index_name in unused:
                print(f"  - {index_name}")
        else:
            print("\n所有索引都在使用中")

# 使用示例
if __name__ == '__main__':
    analyzer = IndexAnalyzer(
        uri='mongodb://admin:password@localhost:27017/',
        database='mydb'
    )
    analyzer.analyze_collection('users')
    analyzer.find_unused_indexes('users')
```

## 7. 查询优化

### 7.1 查询分析

```javascript
// 1. explain()分析查询
db.users.find({ age: { $gte: 18 } }).explain("executionStats")

// 2. 查看查询计划
db.users.find({ email: "alice@example.com" }).explain("queryPlanner")

// 3. 查看所有可能的查询计划
db.users.find({ age: 30, city: "Beijing" }).explain("allPlansExecution")

// 4. 分析聚合管道
db.orders.aggregate([
    { $match: { status: "completed" } },
    { $group: { _id: "$user_id", total: { $sum: "$amount" } } }
], { explain: true })
```

### 7.2 查询优化技巧

```javascript
// 1. 使用投影减少返回字段
db.users.find(
    { age: { $gte: 18 } },
    { name: 1, email: 1, _id: 0 }
)

// 2. 使用limit限制返回数量
db.users.find({ age: { $gte: 18 } }).limit(10)

// 3. 避免使用$where
// 不推荐
db.users.find({ $where: "this.age > 18" })
// 推荐
db.users.find({ age: { $gt: 18 } })

// 4. 使用$in代替多个$or
// 不推荐
db.users.find({
    $or: [
        { status: "active" },
        { status: "pending" }
    ]
})
// 推荐
db.users.find({ status: { $in: ["active", "pending"] } })

// 5. 避免正则表达式开头使用通配符
// 不推荐（无法使用索引）
db.users.find({ email: /.*@example.com/ })
// 推荐（可以使用索引）
db.users.find({ email: /^alice@example.com/ })

// 6. 使用聚合管道优化
db.orders.aggregate([
    // 先过滤，减少数据量
    { $match: { status: "completed" } },
    // 再分组
    { $group: { _id: "$user_id", total: { $sum: "$amount" } } },
    // 最后排序
    { $sort: { total: -1 } },
    // 限制结果
    { $limit: 10 }
])
```

### 7.3 慢查询分析

```javascript
// 1. 启用profiler
db.setProfilingLevel(1, { slowms: 100 })

// 2. 查看慢查询
db.system.profile.find().sort({ ts: -1 }).limit(10)

// 3. 分析慢查询
db.system.profile.find({
    millis: { $gt: 100 }
}).sort({ millis: -1 })

// 4. 查看特定操作的慢查询
db.system.profile.find({
    ns: "mydb.users",
    op: "query"
})

// 5. 关闭profiler
db.setProfilingLevel(0)
```

```python
"""
慢查询分析工具
"""
#!/usr/bin/env python3
from pymongo import MongoClient
from collections import defaultdict

class SlowQueryAnalyzer:
    def __init__(self, uri, database):
        self.client = MongoClient(uri)
        self.db = self.client[database]
    
    def analyze_slow_queries(self, threshold_ms=100):
        """分析慢查询"""
        profile = self.db.system.profile
        
        slow_queries = profile.find({
            'millis': { '$gt': threshold_ms }
        }).sort('millis', -1).limit(20)
        
        print(f"慢查询分析（阈值: {threshold_ms}ms）")
        print("=" * 80)
        
        for query in slow_queries:
            print(f"\n命名空间: {query.get('ns')}")
            print(f"操作: {query.get('op')}")
            print(f"耗时: {query.get('millis')}ms")
            print(f"查询: {query.get('command', {}).get('filter', {})}")
            
            if 'planSummary' in query:
                print(f"执行计划: {query['planSummary']}")
    
    def get_slow_query_stats(self):
        """获取慢查询统计"""
        profile = self.db.system.profile
        
        # 按集合统计
        pipeline = [
            { '$group': {
                '_id': '$ns',
                'count': { '$sum': 1 },
                'avg_time': { '$avg': '$millis' },
                'max_time': { '$max': '$millis' }
            }},
            { '$sort': { 'count': -1 } }
        ]
        
        stats = list(profile.aggregate(pipeline))
        
        print("\n慢查询统计（按集合）:")
        print("=" * 80)
        print(f"{'集合':<30} {'次数':<10} {'平均耗时':<15} {'最大耗时':<15}")
        print("-" * 80)
        
        for stat in stats:
            print(f"{stat['_id']:<30} {stat['count']:<10} "
                  f"{stat['avg_time']:<15.2f} {stat['max_time']:<15.2f}")

# 使用示例
if __name__ == '__main__':
    analyzer = SlowQueryAnalyzer(
        uri='mongodb://admin:password@localhost:27017/',
        database='mydb'
    )
    analyzer.analyze_slow_queries(threshold_ms=100)
    analyzer.get_slow_query_stats()
```

## 8. 性能监控

### 8.1 监控指标

```javascript
// 1. 服务器状态
db.serverStatus()

// 2. 数据库统计
db.stats()

// 3. 集合统计
db.users.stats()

// 4. 当前操作
db.currentOp()

// 5. 连接信息
db.serverStatus().connections

// 6. 内存使用
db.serverStatus().mem

// 7. 网络统计
db.serverStatus().network

// 8. 操作计数
db.serverStatus().opcounters

// 9. 锁信息
db.serverStatus().locks

// 10. 复制延迟
rs.printReplicationInfo()
rs.printSecondaryReplicationInfo()
```

### 8.2 Prometheus监控

```yaml
# mongodb_exporter部署
version: '3.8'

services:
  mongodb-exporter:
    image: percona/mongodb_exporter:latest
    container_name: mongodb-exporter
    restart: always
    ports:
      - "9216:9216"
    environment:
      - MONGODB_URI=mongodb://monitor:password@mongodb:27017
    command:
      - '--mongodb.uri=mongodb://monitor:password@mongodb:27017'
      - '--collect-all'
      - '--compatible-mode'
```

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'mongodb'
    static_configs:
      - targets: ['mongodb-exporter:9216']
        labels:
          instance: 'mongodb-prod'
```

```yaml
# 告警规则
groups:
  - name: mongodb_alerts
    rules:
      # MongoDB宕机
      - alert: MongoDBDown
        expr: mongodb_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "MongoDB实例 {{ $labels.instance }} 宕机"
      
      # 副本集成员不足
      - alert: MongoDBReplicaSetMemberDown
        expr: mongodb_replset_number_of_members < 3
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "副本集成员数量不足"
      
      # 复制延迟过高
      - alert: MongoDBReplicationLag
        expr: mongodb_replset_member_replication_lag > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "复制延迟过高"
          description: "延迟: {{ $value }}秒"
      
      # 连接数过高
      - alert: MongoDBTooManyConnections
        expr: mongodb_connections{state="current"} > 1000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "连接数过高"
          description: "当前连接数: {{ $value }}"
```


### 8.3 性能监控脚本

```python
"""
MongoDB性能监控
"""
#!/usr/bin/env python3
from pymongo import MongoClient
import time

class MongoDBMonitor:
    def __init__(self, uri):
        self.client = MongoClient(uri)
        self.admin_db = self.client.admin
    
    def get_server_status(self):
        """获取服务器状态"""
        return self.admin_db.command('serverStatus')
    
    def monitor_connections(self):
        """监控连接数"""
        status = self.get_server_status()
        connections = status['connections']
        
        print("连接监控:")
        print(f"  当前连接: {connections['current']}")
        print(f"  可用连接: {connections['available']}")
        print(f"  总创建: {connections['totalCreated']}")
        
        usage = connections['current'] / (connections['current'] + connections['available'])
        if usage > 0.8:
            print(f"  警告: 连接使用率过高 ({usage:.2%})")
    
    def monitor_operations(self):
        """监控操作统计"""
        status = self.get_server_status()
        opcounters = status['opcounters']
        
        print("\n操作统计:")
        print(f"  查询: {opcounters['query']}")
        print(f"  插入: {opcounters['insert']}")
        print(f"  更新: {opcounters['update']}")
        print(f"  删除: {opcounters['delete']}")
    
    def monitor_memory(self):
        """监控内存使用"""
        status = self.get_server_status()
        mem = status['mem']
        
        print("\n内存使用:")
        print(f"  常驻内存: {mem['resident']} MB")
        print(f"  虚拟内存: {mem['virtual']} MB")
        
        if 'wiredTiger' in status:
            wt_cache = status['wiredTiger']['cache']
            cache_used = wt_cache['bytes currently in the cache'] / 1024 / 1024
            cache_max = wt_cache['maximum bytes configured'] / 1024 / 1024
            
            print(f"  WiredTiger缓存: {cache_used:.2f} MB / {cache_max:.2f} MB")
            
            usage = cache_used / cache_max
            if usage > 0.9:
                print(f"  警告: 缓存使用率过高 ({usage:.2%})")
    
    def monitor_replication(self):
        """监控复制状态"""
        try:
            status = self.admin_db.command('replSetGetStatus')
            
            print("\n复制状态:")
            print(f"  副本集: {status['set']}")
            
            for member in status['members']:
                print(f"  节点: {member['name']}")
                print(f"    状态: {member['stateStr']}")
                
                if 'optimeDate' in member:
                    print(f"    最后操作时间: {member['optimeDate']}")
                
                if 'lastHeartbeatRecv' in member:
                    print(f"    最后心跳: {member['lastHeartbeatRecv']}")
        except Exception as e:
            print(f"\n非副本集模式或无权限: {e}")
    
    def monitor_locks(self):
        """监控锁信息"""
        status = self.get_server_status()
        
        if 'locks' in status:
            print("\n锁统计:")
            for lock_type, lock_info in status['locks'].items():
                if 'acquireCount' in lock_info:
                    print(f"  {lock_type}:")
                    for mode, count in lock_info['acquireCount'].items():
                        print(f"    {mode}: {count}")
    
    def run_monitoring(self):
        """运行监控"""
        print("=" * 60)
        print("MongoDB性能监控")
        print("=" * 60)
        
        self.monitor_connections()
        self.monitor_operations()
        self.monitor_memory()
        self.monitor_replication()
        self.monitor_locks()

# 使用示例
if __name__ == '__main__':
    monitor = MongoDBMonitor('mongodb://admin:password@localhost:27017/')
    
    while True:
        monitor.run_monitoring()
        time.sleep(60)
```

## 9. 备份恢复

### 9.1 备份策略

```bash
# 1. mongodump备份
mongodump \
    --host localhost:27017 \
    --username admin \
    --password password \
    --authenticationDatabase admin \
    --out /backup/mongodb/$(date +%Y%m%d)

# 2. 备份特定数据库
mongodump \
    --host localhost:27017 \
    --username admin \
    --password password \
    --authenticationDatabase admin \
    --db mydb \
    --out /backup/mongodb/mydb_$(date +%Y%m%d)

# 3. 备份特定集合
mongodump \
    --host localhost:27017 \
    --username admin \
    --password password \
    --authenticationDatabase admin \
    --db mydb \
    --collection users \
    --out /backup/mongodb/users_$(date +%Y%m%d)

# 4. 压缩备份
mongodump \
    --host localhost:27017 \
    --username admin \
    --password password \
    --authenticationDatabase admin \
    --gzip \
    --out /backup/mongodb/$(date +%Y%m%d)
```

```bash
# 备份脚本
#!/bin/bash

MONGO_HOST="localhost:27017"
MONGO_USER="admin"
MONGO_PASS="password"
BACKUP_DIR="/backup/mongodb"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7

# 创建备份目录
mkdir -p $BACKUP_DIR

# 执行备份
mongodump \
    --host $MONGO_HOST \
    --username $MONGO_USER \
    --password $MONGO_PASS \
    --authenticationDatabase admin \
    --gzip \
    --out $BACKUP_DIR/$DATE

# 压缩备份
cd $BACKUP_DIR
tar -czf mongodb_backup_$DATE.tar.gz $DATE
rm -rf $DATE

# 清理旧备份
find $BACKUP_DIR -name "mongodb_backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "备份完成: mongodb_backup_$DATE.tar.gz"
```

### 9.2 数据恢复

```bash
# 1. 恢复所有数据库
mongorestore \
    --host localhost:27017 \
    --username admin \
    --password password \
    --authenticationDatabase admin \
    /backup/mongodb/20240201

# 2. 恢复特定数据库
mongorestore \
    --host localhost:27017 \
    --username admin \
    --password password \
    --authenticationDatabase admin \
    --db mydb \
    /backup/mongodb/20240201/mydb

# 3. 恢复特定集合
mongorestore \
    --host localhost:27017 \
    --username admin \
    --password password \
    --authenticationDatabase admin \
    --db mydb \
    --collection users \
    /backup/mongodb/20240201/mydb/users.bson

# 4. 恢复压缩备份
mongorestore \
    --host localhost:27017 \
    --username admin \
    --password password \
    --authenticationDatabase admin \
    --gzip \
    /backup/mongodb/20240201

# 5. 删除现有数据后恢复
mongorestore \
    --host localhost:27017 \
    --username admin \
    --password password \
    --authenticationDatabase admin \
    --drop \
    /backup/mongodb/20240201
```

### 9.3 增量备份

```bash
# 使用oplog进行增量备份

# 1. 备份oplog
mongodump \
    --host localhost:27017 \
    --username admin \
    --password password \
    --authenticationDatabase admin \
    --db local \
    --collection oplog.rs \
    --out /backup/mongodb/oplog_$(date +%Y%m%d_%H%M%S)

# 2. 恢复时应用oplog
mongorestore \
    --host localhost:27017 \
    --username admin \
    --password password \
    --authenticationDatabase admin \
    --oplogReplay \
    /backup/mongodb/20240201
```

### 9.4 在线备份

```python
"""
MongoDB在线备份工具
"""
#!/usr/bin/env python3
from pymongo import MongoClient
import subprocess
import datetime
import os

class MongoDBBackup:
    def __init__(self, uri, backup_dir):
        self.client = MongoClient(uri)
        self.backup_dir = backup_dir
        os.makedirs(backup_dir, exist_ok=True)
    
    def backup_database(self, database_name):
        """备份数据库"""
        timestamp = datetime.datetime.now().strftime('%Y%m%d_%H%M%S')
        backup_path = os.path.join(
            self.backup_dir,
            f'{database_name}_{timestamp}'
        )
        
        cmd = [
            'mongodump',
            '--uri', str(self.client._MongoClient__options.pool_options._credentials),
            '--db', database_name,
            '--gzip',
            '--out', backup_path
        ]
        
        try:
            subprocess.run(cmd, check=True)
            print(f"备份成功: {backup_path}")
            return backup_path
        except subprocess.CalledProcessError as e:
            print(f"备份失败: {e}")
            return None
    
    def backup_collection(self, database_name, collection_name):
        """备份集合"""
        timestamp = datetime.datetime.now().strftime('%Y%m%d_%H%M%S')
        backup_path = os.path.join(
            self.backup_dir,
            f'{database_name}_{collection_name}_{timestamp}'
        )
        
        cmd = [
            'mongodump',
            '--uri', str(self.client._MongoClient__options.pool_options._credentials),
            '--db', database_name,
            '--collection', collection_name,
            '--gzip',
            '--out', backup_path
        ]
        
        try:
            subprocess.run(cmd, check=True)
            print(f"备份成功: {backup_path}")
            return backup_path
        except subprocess.CalledProcessError as e:
            print(f"备份失败: {e}")
            return None
    
    def cleanup_old_backups(self, days=7):
        """清理旧备份"""
        import time
        
        now = time.time()
        cutoff = now - (days * 86400)
        
        for item in os.listdir(self.backup_dir):
            item_path = os.path.join(self.backup_dir, item)
            if os.path.isdir(item_path):
                if os.path.getmtime(item_path) < cutoff:
                    import shutil
                    shutil.rmtree(item_path)
                    print(f"删除旧备份: {item}")

# 使用示例
if __name__ == '__main__':
    backup = MongoDBBackup(
        uri='mongodb://admin:password@localhost:27017/',
        backup_dir='/backup/mongodb'
    )
    
    backup.backup_database('mydb')
    backup.cleanup_old_backups(days=7)
```

## 10. 安全加固

### 10.1 认证授权

```javascript
// 1. 创建管理员用户
use admin
db.createUser({
    user: "admin",
    pwd: "strong_password",
    roles: [
        { role: "userAdminAnyDatabase", db: "admin" },
        { role: "readWriteAnyDatabase", db: "admin" },
        { role: "dbAdminAnyDatabase", db: "admin" },
        { role: "clusterAdmin", db: "admin" }
    ]
})

// 2. 创建数据库用户
use mydb
db.createUser({
    user: "appuser",
    pwd: "app_password",
    roles: [
        { role: "readWrite", db: "mydb" }
    ]
})

// 3. 创建只读用户
use mydb
db.createUser({
    user: "readonly",
    pwd: "readonly_password",
    roles: [
        { role: "read", db: "mydb" }
    ]
})

// 4. 查看用户
db.getUsers()

// 5. 修改用户密码
db.changeUserPassword("appuser", "new_password")

// 6. 授予角色
db.grantRolesToUser("appuser", [
    { role: "dbAdmin", db: "mydb" }
])

// 7. 撤销角色
db.revokeRolesFromUser("appuser", [
    { role: "dbAdmin", db: "mydb" }
])

// 8. 删除用户
db.dropUser("appuser")
```

### 10.2 网络安全

```yaml
# mongod.conf

net:
  # 绑定IP
  bindIp: 127.0.0.1,192.168.1.100
  
  # 启用TLS
  tls:
    mode: requireTLS
    certificateKeyFile: /etc/mongodb/mongodb.pem
    CAFile: /etc/mongodb/ca.pem
```

```bash
# 生成TLS证书
# 1. 生成CA私钥
openssl genrsa -out ca.key 4096

# 2. 生成CA证书
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt

# 3. 生成服务器私钥
openssl genrsa -out mongodb.key 4096

# 4. 生成证书签名请求
openssl req -new -key mongodb.key -out mongodb.csr

# 5. 签名证书
openssl x509 -req -days 3650 -in mongodb.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out mongodb.crt

# 6. 合并证书和私钥
cat mongodb.key mongodb.crt > mongodb.pem
```

### 10.3 审计日志

```yaml
# mongod.conf

auditLog:
  destination: file
  format: JSON
  path: /data/mongodb/logs/audit.json
  filter: '{ atype: { $in: ["authenticate", "createUser", "dropUser", "dropDatabase", "dropCollection"] } }'
```

### 10.4 安全检查清单

```yaml
安全检查清单:
  □ 启用认证授权
  □ 使用强密码策略
  □ 限制网络访问（bindIp）
  □ 启用TLS加密
  □ 配置防火墙规则
  □ 启用审计日志
  □ 定期更新MongoDB版本
  □ 最小权限原则
  □ 禁用不必要的功能
  □ 定期备份数据
  □ 监控异常访问
  □ 定期安全审计
```

## 11. 故障排查

### 11.1 常见问题

```yaml
# MongoDB常见故障

性能问题:
  - 查询慢
  - 写入慢
  - 内存不足
  - CPU使用率高
  - 磁盘IO高

可用性问题:
  - 服务宕机
  - 副本集选举失败
  - 分片不可用
  - 连接超时

数据问题:
  - 数据丢失
  - 数据不一致
  - 复制延迟
  - 磁盘空间不足
```

### 11.2 诊断工具

```bash
# 1. 查看当前操作
mongosh --eval "db.currentOp()"

# 2. 杀死慢查询
mongosh --eval "db.killOp(opid)"

# 3. 查看连接
mongosh --eval "db.serverStatus().connections"

# 4. 查看锁信息
mongosh --eval "db.serverStatus().locks"

# 5. 查看复制状态
mongosh --eval "rs.status()"

# 6. 查看分片状态
mongosh --eval "sh.status()"

# 7. 验证集合
mongosh --eval "db.users.validate()"

# 8. 修复数据库
mongod --repair --dbpath /data/mongodb/data
```

### 11.3 故障排查脚本

```python
"""
MongoDB故障诊断工具
"""
#!/usr/bin/env python3
from pymongo import MongoClient
import sys

class MongoDBDiagnostic:
    def __init__(self, uri):
        try:
            self.client = MongoClient(uri, serverSelectionTimeoutMS=5000)
            self.admin_db = self.client.admin
        except Exception as e:
            print(f"连接失败: {e}")
            sys.exit(1)
    
    def check_connectivity(self):
        """检查连接"""
        try:
            self.admin_db.command('ping')
            print("✓ 连接正常")
            return True
        except Exception as e:
            print(f"✗ 连接失败: {e}")
            return False
    
    def check_replication(self):
        """检查复制状态"""
        try:
            status = self.admin_db.command('replSetGetStatus')
            
            print("\n副本集状态:")
            healthy_members = 0
            
            for member in status['members']:
                state = member['stateStr']
                print(f"  {member['name']}: {state}")
                
                if state in ['PRIMARY', 'SECONDARY']:
                    healthy_members += 1
            
            if healthy_members >= 2:
                print("✓ 副本集健康")
            else:
                print("✗ 副本集成员不足")
        except Exception as e:
            print(f"非副本集模式或无权限: {e}")
    
    def check_slow_queries(self):
        """检查慢查询"""
        try:
            for db_name in self.client.list_database_names():
                if db_name in ['admin', 'local', 'config']:
                    continue
                
                db = self.client[db_name]
                profile = db.system.profile
                
                slow_count = profile.count_documents({
                    'millis': { '$gt': 100 }
                })
                
                if slow_count > 0:
                    print(f"\n✗ 数据库 {db_name} 有 {slow_count} 个慢查询")
                else:
                    print(f"\n✓ 数据库 {db_name} 无慢查询")
        except Exception as e:
            print(f"检查慢查询失败: {e}")
    
    def check_disk_space(self):
        """检查磁盘空间"""
        try:
            status = self.admin_db.command('serverStatus')
            
            for db_name in self.client.list_database_names():
                db = self.client[db_name]
                stats = db.command('dbStats')
                
                data_size = stats['dataSize'] / 1024 / 1024 / 1024
                storage_size = stats['storageSize'] / 1024 / 1024 / 1024
                
                print(f"\n数据库 {db_name}:")
                print(f"  数据大小: {data_size:.2f} GB")
                print(f"  存储大小: {storage_size:.2f} GB")
        except Exception as e:
            print(f"检查磁盘空间失败: {e}")
    
    def diagnose(self):
        """执行诊断"""
        print("=" * 60)
        print("MongoDB故障诊断")
        print("=" * 60)
        
        if not self.check_connectivity():
            return
        
        self.check_replication()
        self.check_slow_queries()
        self.check_disk_space()

# 使用示例
if __name__ == '__main__':
    diagnostic = MongoDBDiagnostic(
        'mongodb://admin:password@localhost:27017/'
    )
    diagnostic.diagnose()
```

## 12. 实战案例

### 12.1 案例一：高并发写入优化

```python
"""
批量写入优化
"""
from pymongo import MongoClient, InsertOne, UpdateOne
import time

client = MongoClient('mongodb://localhost:27017/')
db = client.mydb
collection = db.users

# 方案1：单条插入（慢）
start = time.time()
for i in range(1000):
    collection.insert_one({'user_id': i, 'name': f'user{i}'})
print(f"单条插入耗时: {time.time() - start:.2f}秒")

# 方案2：批量插入（快）
start = time.time()
documents = [{'user_id': i, 'name': f'user{i}'} for i in range(1000)]
collection.insert_many(documents, ordered=False)
print(f"批量插入耗时: {time.time() - start:.2f}秒")

# 方案3：bulk_write（最快）
start = time.time()
operations = [
    InsertOne({'user_id': i, 'name': f'user{i}'})
    for i in range(1000)
]
collection.bulk_write(operations, ordered=False)
print(f"bulk_write耗时: {time.time() - start:.2f}秒")
```

### 12.2 案例二：分页查询优化

```python
"""
分页查询优化
"""
from pymongo import MongoClient

client = MongoClient('mongodb://localhost:27017/')
db = client.mydb
collection = db.orders

# 方案1：skip+limit（慢，深分页性能差）
def paginate_skip(page, page_size):
    skip = (page - 1) * page_size
    return list(collection.find().skip(skip).limit(page_size))

# 方案2：基于_id的范围查询（快）
def paginate_range(last_id, page_size):
    query = {}
    if last_id:
        query['_id'] = {'$gt': last_id}
    
    results = list(collection.find(query).limit(page_size))
    return results

# 方案3：使用索引字段分页
def paginate_indexed(last_created_at, page_size):
    query = {}
    if last_created_at:
        query['created_at'] = {'$gt': last_created_at}
    
    results = list(
        collection.find(query)
        .sort('created_at', 1)
        .limit(page_size)
    )
    return results
```

### 12.3 案例三：聚合查询优化

```javascript
// 订单统计优化

// 方案1：不优化（慢）
db.orders.aggregate([
    { $match: { status: "completed" } },
    { $lookup: {
        from: "users",
        localField: "user_id",
        foreignField: "_id",
        as: "user"
    }},
    { $unwind: "$user" },
    { $group: {
        _id: "$user.city",
        total: { $sum: "$amount" },
        count: { $sum: 1 }
    }},
    { $sort: { total: -1 } }
])

// 方案2：优化（快）
db.orders.aggregate([
    // 1. 先过滤，减少数据量
    { $match: { 
        status: "completed",
        created_at: { $gte: ISODate("2024-01-01") }
    }},
    // 2. 只投影需要的字段
    { $project: {
        user_id: 1,
        amount: 1
    }},
    // 3. 先分组再join
    { $group: {
        _id: "$user_id",
        total: { $sum: "$amount" },
        count: { $sum: 1 }
    }},
    // 4. 最后join用户信息
    { $lookup: {
        from: "users",
        localField: "_id",
        foreignField: "_id",
        as: "user"
    }},
    { $unwind: "$user" },
    // 5. 按城市分组
    { $group: {
        _id: "$user.city",
        total: { $sum: "$total" },
        count: { $sum: "$count" }
    }},
    { $sort: { total: -1 } },
    { $limit: 10 }
])
```

### 12.4 案例四：数据迁移

```python
"""
MongoDB数据迁移
"""
from pymongo import MongoClient

class MongoDBMigration:
    def __init__(self, source_uri, target_uri):
        self.source = MongoClient(source_uri)
        self.target = MongoClient(target_uri)
    
    def migrate_database(self, db_name, batch_size=1000):
        """迁移数据库"""
        source_db = self.source[db_name]
        target_db = self.target[db_name]
        
        # 获取所有集合
        collections = source_db.list_collection_names()
        
        for coll_name in collections:
            if coll_name.startswith('system.'):
                continue
            
            print(f"迁移集合: {coll_name}")
            self.migrate_collection(db_name, coll_name, batch_size)
    
    def migrate_collection(self, db_name, coll_name, batch_size):
        """迁移集合"""
        source_coll = self.source[db_name][coll_name]
        target_coll = self.target[db_name][coll_name]
        
        # 复制索引
        indexes = source_coll.list_indexes()
        for index in indexes:
            if index['name'] != '_id_':
                target_coll.create_index(
                    list(index['key'].items()),
                    **{k: v for k, v in index.items() 
                       if k not in ['key', 'v', 'ns']}
                )
        
        # 迁移数据
        total = source_coll.count_documents({})
        migrated = 0
        
        cursor = source_coll.find().batch_size(batch_size)
        
        batch = []
        for doc in cursor:
            batch.append(doc)
            
            if len(batch) >= batch_size:
                target_coll.insert_many(batch, ordered=False)
                migrated += len(batch)
                print(f"  进度: {migrated}/{total}")
                batch = []
        
        # 插入剩余数据
        if batch:
            target_coll.insert_many(batch, ordered=False)
            migrated += len(batch)
        
        print(f"  完成: {migrated}/{total}")

# 使用示例
if __name__ == '__main__':
    migration = MongoDBMigration(
        source_uri='mongodb://source:27017/',
        target_uri='mongodb://target:27017/'
    )
    migration.migrate_database('mydb')
```

好的，我已经完成了MongoDB运维教程的大部分内容。现在让我补充最后的学习检查清单和参考资源部分。


## 13. 学习检查清单

### 13.1 基础知识

```yaml
MongoDB基础:
  □ 理解MongoDB的文档模型
  □ 掌握BSON数据格式
  □ 了解MongoDB与关系型数据库的区别
  □ 熟悉MongoDB的数据类型
  □ 掌握基本的CRUD操作
  □ 理解集合和数据库的概念
  □ 了解MongoDB的应用场景

数据结构:
  □ 掌握String、Number、Boolean等基本类型
  □ 理解Document嵌套文档
  □ 掌握Array数组的使用
  □ 了解ObjectId的生成规则
  □ 理解Date日期类型
  □ 掌握Null和Undefined的区别
  □ 了解Binary二进制数据
```

### 13.2 架构部署

```yaml
单机部署:
  □ 能够安装MongoDB
  □ 掌握配置文件的编写
  □ 了解存储引擎的选择
  □ 能够启动和停止服务
  □ 掌握基本的运维命令

副本集部署:
  □ 理解副本集的架构
  □ 能够搭建副本集
  □ 掌握副本集的管理
  □ 理解选举机制
  □ 掌握读写分离配置
  □ 了解复制延迟的处理
  □ 能够进行故障转移

分片集群:
  □ 理解分片集群的架构
  □ 能够部署Config Server
  □ 能够部署Shard副本集
  □ 能够配置mongos路由
  □ 掌握分片键的选择
  □ 理解chunk的概念
  □ 能够管理分片集群
  □ 掌握数据平衡机制
```

### 13.3 性能优化

```yaml
索引优化:
  □ 理解索引的工作原理
  □ 掌握各种索引类型
  □ 能够创建合适的索引
  □ 理解ESR规则
  □ 掌握覆盖索引的使用
  □ 能够分析索引使用情况
  □ 了解索引的维护成本

查询优化:
  □ 能够使用explain分析查询
  □ 掌握查询优化技巧
  □ 理解查询计划的选择
  □ 能够优化聚合管道
  □ 掌握慢查询的分析
  □ 了解查询缓存机制

配置优化:
  □ 掌握内存配置
  □ 了解连接池配置
  □ 理解持久化配置
  □ 能够优化系统参数
  □ 掌握WiredTiger配置
  □ 了解网络参数优化
```

### 13.4 高可用

```yaml
副本集高可用:
  □ 理解副本集的高可用机制
  □ 掌握自动故障转移
  □ 能够配置优先级
  □ 了解仲裁节点的作用
  □ 掌握心跳机制
  □ 能够处理脑裂问题

分片集群高可用:
  □ 理解分片集群的高可用
  □ 掌握Config Server的高可用
  □ 了解mongos的高可用
  □ 能够处理分片故障
  □ 掌握数据迁移
  □ 了解区域分片

备份恢复:
  □ 掌握mongodump/mongorestore
  □ 能够制定备份策略
  □ 了解增量备份方法
  □ 掌握oplog的使用
  □ 能够进行数据恢复
  □ 了解在线备份方案
```

### 13.5 监控运维

```yaml
性能监控:
  □ 掌握serverStatus命令
  □ 能够监控关键指标
  □ 了解Prometheus监控
  □ 掌握mongodb_exporter
  □ 能够配置告警规则
  □ 了解日志分析

故障排查:
  □ 能够诊断性能问题
  □ 掌握常见故障的处理
  □ 了解日志分析方法
  □ 能够使用诊断工具
  □ 掌握数据修复方法
  □ 了解问题定位技巧

日常运维:
  □ 掌握用户权限管理
  □ 能够进行容量规划
  □ 了解版本升级方法
  □ 掌握数据迁移
  □ 能够进行性能调优
  □ 了解运维自动化
```

### 13.6 安全加固

```yaml
认证授权:
  □ 掌握用户管理
  □ 理解角色权限体系
  □ 能够配置认证机制
  □ 了解LDAP集成
  □ 掌握最小权限原则

网络安全:
  □ 能够配置网络访问控制
  □ 掌握TLS加密配置
  □ 了解防火墙规则
  □ 能够配置VPN访问
  □ 掌握IP白名单

数据安全:
  □ 了解数据加密方法
  □ 掌握审计日志配置
  □ 能够进行安全审计
  □ 了解合规要求
  □ 掌握数据脱敏方法
```

### 13.7 实战能力

```yaml
架构设计:
  □ 能够根据业务选择架构
  □ 掌握容量规划方法
  □ 了解性能基准测试
  □ 能够设计高可用方案
  □ 掌握扩展性设计

问题解决:
  □ 能够快速定位问题
  □ 掌握性能优化方法
  □ 了解常见陷阱
  □ 能够处理紧急故障
  □ 掌握问题预防措施

项目经验:
  □ 有实际项目经验
  □ 了解最佳实践
  □ 能够进行技术选型
  □ 掌握团队协作
  □ 了解运维流程
```

## 📚 参考资源

### 官方文档

```yaml
MongoDB官方资源:
  - MongoDB官方文档: https://docs.mongodb.com/
  - MongoDB大学: https://university.mongodb.com/
  - MongoDB博客: https://www.mongodb.com/blog
  - MongoDB GitHub: https://github.com/mongodb/mongo
  - MongoDB社区: https://www.mongodb.com/community

中文资源:
  - MongoDB中文文档: https://docs.mongoing.com/
  - MongoDB中文社区: https://mongoing.com/
  - MongoDB中文教程: https://www.runoob.com/mongodb/
```

### 学习资源

```yaml
在线教程:
  - MongoDB University（免费课程）
  - Coursera MongoDB课程
  - Udemy MongoDB课程
  - 极客时间《MongoDB核心原理与实战》
  - 慕课网MongoDB教程

书籍推荐:
  - 《MongoDB权威指南》（第3版）
  - 《MongoDB实战》（第2版）
  - 《深入学习MongoDB》
  - 《MongoDB性能调优实战》
  - 《MongoDB高可用架构设计》

视频教程:
  - B站MongoDB教程合集
  - YouTube MongoDB官方频道
  - 腾讯课堂MongoDB课程
  - 网易云课堂MongoDB教程
```

### 社区资源

```yaml
技术社区:
  - Stack Overflow MongoDB标签
  - MongoDB中文社区论坛
  - Reddit r/mongodb
  - MongoDB Slack频道
  - CSDN MongoDB专栏

技术博客:
  - MongoDB官方博客
  - Percona MongoDB博客
  - DBA Stack MongoDB文章
  - 美团技术团队MongoDB实践
  - 阿里云MongoDB最佳实践

开源项目:
  - mongo-tools: MongoDB工具集
  - mongo-connector: 数据同步工具
  - mongoengine: Python ODM
  - mongoose: Node.js ODM
  - spring-data-mongodb: Spring集成
```

### 工具集成

```yaml
管理工具:
  - MongoDB Compass: 官方GUI工具
  - Robo 3T: 免费MongoDB客户端
  - Studio 3T: 商业MongoDB IDE
  - NoSQLBooster: MongoDB客户端
  - MongoDB Ops Manager: 企业级管理平台

监控工具:
  - Prometheus + mongodb_exporter
  - Grafana MongoDB仪表板
  - MongoDB Cloud Manager
  - Datadog MongoDB监控
  - New Relic MongoDB插件

备份工具:
  - mongodump/mongorestore
  - Percona Backup for MongoDB
  - MongoDB Cloud Backup
  - Ops Manager Backup
  - 自定义备份脚本

性能分析:
  - MongoDB Profiler
  - explain()分析工具
  - mongostat实时统计
  - mongotop集合统计
  - mtools日志分析
```

### 开源项目

```yaml
推荐项目:
  - mongo-express: Web管理界面
  - mongo-connector: 数据同步
  - mongo-hadoop: Hadoop集成
  - mongo-spark: Spark集成
  - mongo-kafka: Kafka集成

Python生态:
  - pymongo: 官方Python驱动
  - motor: 异步MongoDB驱动
  - mongoengine: ODM框架
  - ming: 轻量级ODM
  - umongo: 异步ODM

Node.js生态:
  - mongodb: 官方Node.js驱动
  - mongoose: 流行的ODM
  - monk: 简化的MongoDB API
  - mongoist: Promise封装
  - mongorito: 现代ODM

Java生态:
  - mongo-java-driver: 官方Java驱动
  - spring-data-mongodb: Spring集成
  - morphia: Java ODM
  - jongo: 类似mongo shell的API
  - mongojack: Jackson集成
```

### 最佳实践

```yaml
架构设计:
  - MongoDB架构设计最佳实践
  - 分片键选择指南
  - 索引设计模式
  - 数据建模最佳实践
  - 高可用架构设计

性能优化:
  - MongoDB性能调优指南
  - 查询优化技巧
  - 索引优化策略
  - 内存配置优化
  - 网络优化建议

运维管理:
  - MongoDB运维手册
  - 备份恢复策略
  - 监控告警方案
  - 故障处理流程
  - 容量规划方法

安全加固:
  - MongoDB安全加固指南
  - 认证授权配置
  - 网络安全配置
  - 审计日志配置
  - 合规性要求
```

## 🎓 学习建议

### 学习路径

```yaml
初级阶段（1-2个月）:
  1. 学习MongoDB基础概念
  2. 掌握基本的CRUD操作
  3. 了解数据类型和文档模型
  4. 学习基本的索引使用
  5. 完成简单的项目实践

中级阶段（2-3个月）:
  1. 深入学习索引优化
  2. 掌握聚合框架
  3. 学习副本集部署
  4. 了解分片集群
  5. 掌握性能优化技巧
  6. 学习备份恢复

高级阶段（3-6个月）:
  1. 深入理解MongoDB架构
  2. 掌握分片集群部署
  3. 学习高级查询优化
  4. 掌握故障排查方法
  5. 学习安全加固
  6. 参与大型项目实践

专家阶段（持续学习）:
  1. 研究MongoDB源码
  2. 深入理解存储引擎
  3. 掌握内核优化
  4. 参与社区贡献
  5. 分享实践经验
```

### 实践建议

```yaml
动手实践:
  - 搭建本地MongoDB环境
  - 完成官方教程练习
  - 参与开源项目
  - 解决实际问题
  - 编写运维脚本

项目经验:
  - 从小项目开始
  - 逐步增加复杂度
  - 关注性能优化
  - 积累故障处理经验
  - 总结最佳实践

持续学习:
  - 关注MongoDB新版本
  - 学习新特性
  - 阅读技术博客
  - 参与技术讨论
  - 分享学习心得
```

### 注意事项

```yaml
常见陷阱:
  - 不要滥用嵌套文档
  - 避免过度索引
  - 注意分片键的选择
  - 防止慢查询
  - 关注内存使用

性能优化:
  - 合理设计数据模型
  - 创建合适的索引
  - 使用投影减少返回数据
  - 批量操作提高效率
  - 监控关键指标

安全注意:
  - 启用认证授权
  - 限制网络访问
  - 使用强密码
  - 定期备份数据
  - 及时更新版本

运维规范:
  - 制定运维流程
  - 建立监控告警
  - 定期巡检
  - 做好文档记录
  - 定期演练故障恢复
```

---

**祝你学习愉快，成为MongoDB专家！** 🚀

> 本教程持续更新中，欢迎反馈建议
>
> @author erik.zhou
