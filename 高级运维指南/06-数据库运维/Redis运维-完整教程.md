# Redis运维完整教程

> 企业级Redis集群运维与性能优化实战指南
>
> @author erik.zhou

## 📚 目录

- [1. Redis基础架构](#1-redis基础架构)
- [2. Redis安装部署](#2-redis安装部署)
- [3. Redis配置优化](#3-redis配置优化)
- [4. Redis持久化](#4-redis持久化)
- [5. Redis主从复制](#5-redis主从复制)
- [6. Redis哨兵模式](#6-redis哨兵模式)
- [7. Redis集群模式](#7-redis集群模式)
- [8. Redis性能优化](#8-redis性能优化)
- [9. Redis监控告警](#9-redis监控告警)
- [10. Redis备份恢复](#10-redis备份恢复)
- [11. Redis故障排查](#11-redis故障排查)
- [12. Redis安全加固](#12-redis安全加固)
- [13. 实战案例](#13-实战案例)
- [14. 学习检查清单](#14-学习检查清单)

## 🎯 学习目标

- 掌握Redis的架构和部署方式
- 能够配置和优化Redis性能
- 掌握Redis持久化机制
- 能够搭建Redis主从、哨兵、集群
- 掌握Redis监控和故障排查
- 了解Redis安全加固方法
- 能够处理常见的Redis运维问题

## 1. Redis基础架构

### 1.1 Redis架构模式

```yaml
# Redis部署架构对比

单机模式:
  优点:
    - 部署简单
    - 维护成本低
    - 性能最优
  缺点:
    - 无高可用
    - 容量受限
    - 单点故障
  适用场景:
    - 开发测试环境
    - 缓存场景
    - 数据可丢失

主从模式:
  优点:
    - 读写分离
    - 数据备份
    - 故障恢复
  缺点:
    - 手动故障转移
    - 主节点单点
  适用场景:
    - 读多写少
    - 需要数据备份
    - 可接受短暂中断

哨兵模式:
  优点:
    - 自动故障转移
    - 监控告警
    - 配置中心
  缺点:
    - 容量受限
    - 写能力受限
  适用场景:
    - 需要高可用
    - 中小规模数据
    - 读写分离

集群模式:
  优点:
    - 水平扩展
    - 高可用
    - 自动分片
  缺点:
    - 运维复杂
    - 某些命令受限
  适用场景:
    - 大规模数据
    - 高并发
    - 需要扩展性
```

### 1.2 Redis数据结构

```bash
# Redis核心数据结构

# 1. String（字符串）
SET key value
GET key
INCR counter
SETEX session:123 3600 "user_data"

# 2. Hash（哈希）
HSET user:1 name "Alice"
HGET user:1 name
HMSET user:1 name "Alice" age 30 email "alice@example.com"
HGETALL user:1

# 3. List（列表）
LPUSH queue:tasks "task1"
RPUSH queue:tasks "task2"
LPOP queue:tasks
LRANGE queue:tasks 0 -1

# 4. Set（集合）
SADD tags:article:1 "redis" "database" "nosql"
SMEMBERS tags:article:1
SINTER tags:article:1 tags:article:2

# 5. Sorted Set（有序集合）
ZADD leaderboard 100 "player1"
ZADD leaderboard 200 "player2"
ZRANGE leaderboard 0 -1 WITHSCORES
ZREVRANK leaderboard "player1"
```


### 1.3 Redis内存模型

```
┌─────────────────────────────────────────┐
│         Redis内存分布                    │
├─────────────────────────────────────────┤
│  数据内存（70-80%）                      │
│  - 键值对数据                            │
│  - 过期键字典                            │
│  - 数据结构开销                          │
├─────────────────────────────────────────┤
│  进程内存（10-15%）                      │
│  - 客户端缓冲区                          │
│  - 复制缓冲区                            │
│  - AOF缓冲区                             │
├─────────────────────────────────────────┤
│  碎片内存（5-10%）                       │
│  - 内存分配器碎片                        │
│  - 数据删除产生碎片                      │
└─────────────────────────────────────────┘
```

## 2. Redis安装部署

### 2.1 编译安装

```bash
# 下载Redis
cd /opt
wget https://download.redis.io/releases/redis-7.2.4.tar.gz
tar xzf redis-7.2.4.tar.gz
cd redis-7.2.4

# 编译安装
make
make test
make install PREFIX=/usr/local/redis

# 创建目录
mkdir -p /usr/local/redis/{etc,data,logs}
mkdir -p /var/run/redis

# 复制配置文件
cp redis.conf /usr/local/redis/etc/

# 创建用户
useradd -r -s /sbin/nologin redis
chown -R redis:redis /usr/local/redis
chown -R redis:redis /var/run/redis
```

### 2.2 配置文件

```bash
# /usr/local/redis/etc/redis.conf

# 基础配置
bind 0.0.0.0
port 6379
daemonize yes
pidfile /var/run/redis/redis.pid
logfile /usr/local/redis/logs/redis.log
dir /usr/local/redis/data

# 内存配置
maxmemory 4gb
maxmemory-policy allkeys-lru

# 持久化配置
save 900 1
save 300 10
save 60 10000
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec

# 复制配置
replica-read-only yes
repl-diskless-sync no
repl-backlog-size 256mb

# 安全配置
requirepass your_strong_password
rename-command FLUSHALL ""
rename-command FLUSHDB ""
rename-command CONFIG "CONFIG_ADMIN"

# 性能配置
tcp-backlog 511
timeout 300
tcp-keepalive 300
databases 16
```

### 2.3 Systemd服务

```bash
# /etc/systemd/system/redis.service
[Unit]
Description=Redis In-Memory Data Store
After=network.target

[Service]
Type=forking
User=redis
Group=redis
PIDFile=/var/run/redis/redis.pid
ExecStart=/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
ExecStop=/usr/local/redis/bin/redis-cli -h 127.0.0.1 -p 6379 -a your_password shutdown
Restart=always
RestartSec=5s
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

```bash
# 启动服务
systemctl daemon-reload
systemctl enable redis
systemctl start redis
systemctl status redis

# 验证
redis-cli -a your_password ping
# 输出: PONG
```

### 2.4 Docker部署

```yaml
# docker-compose.yml
version: '3.8'

services:
  redis:
    image: redis:7.2-alpine
    container_name: redis
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf
      - redis-data:/data
      - redis-logs:/var/log/redis
    command: redis-server /usr/local/etc/redis/redis.conf
    environment:
      - TZ=Asia/Shanghai
    networks:
      - redis-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

volumes:
  redis-data:
  redis-logs:

networks:
  redis-network:
    driver: bridge
```

```bash
# 启动
docker-compose up -d

# 查看日志
docker-compose logs -f redis

# 进入容器
docker-compose exec redis redis-cli
```

## 3. Redis配置优化

### 3.1 内存优化

```bash
# redis.conf

# 1. 内存限制
maxmemory 4gb

# 2. 淘汰策略
# volatile-lru: 对设置了过期时间的key使用LRU算法
# allkeys-lru: 对所有key使用LRU算法（推荐）
# volatile-lfu: 对设置了过期时间的key使用LFU算法
# allkeys-lfu: 对所有key使用LFU算法
# volatile-random: 随机删除设置了过期时间的key
# allkeys-random: 随机删除所有key
# volatile-ttl: 删除最近要过期的key
# noeviction: 不删除，写入时返回错误
maxmemory-policy allkeys-lru

# 3. LRU样本数
maxmemory-samples 5

# 4. 惰性删除
lazyfree-lazy-eviction yes
lazyfree-lazy-expire yes
lazyfree-lazy-server-del yes
replica-lazy-flush yes

# 5. 内存碎片整理
activedefrag yes
active-defrag-ignore-bytes 100mb
active-defrag-threshold-lower 10
active-defrag-threshold-upper 100
active-defrag-cycle-min 5
active-defrag-cycle-max 75
```

### 3.2 网络优化

```bash
# redis.conf

# 1. TCP配置
tcp-backlog 511
tcp-keepalive 300
timeout 300

# 2. 客户端连接
maxclients 10000

# 3. 输出缓冲区
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

# 4. 慢查询日志
slowlog-log-slower-than 10000
slowlog-max-len 128

# 5. 命令重命名（安全）
rename-command FLUSHALL ""
rename-command FLUSHDB ""
rename-command KEYS ""
rename-command CONFIG "CONFIG_ADMIN_ONLY"
```

### 3.3 持久化优化

```bash
# redis.conf

# RDB配置
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb

# AOF配置
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
```

### 3.4 系统参数优化

```bash
# /etc/sysctl.conf

# 内存过量使用
vm.overcommit_memory = 1

# 透明大页
echo never > /sys/kernel/mm/transparent_hugepage/enabled

# TCP配置
net.core.somaxconn = 511
net.ipv4.tcp_max_syn_backlog = 511

# 文件描述符
fs.file-max = 65535
```

```bash
# /etc/security/limits.conf
redis soft nofile 65535
redis hard nofile 65535
redis soft nproc 65535
redis hard nproc 65535
```

```bash
# 应用配置
sysctl -p
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

## 4. Redis持久化

### 4.1 RDB持久化

```bash
# RDB配置
save 900 1      # 900秒内至少1个key变化
save 300 10     # 300秒内至少10个key变化
save 60 10000   # 60秒内至少10000个key变化

# 手动触发
redis-cli -a password SAVE      # 阻塞式保存
redis-cli -a password BGSAVE    # 后台保存

# 查看最后保存时间
redis-cli -a password LASTSAVE
```

```python
"""
RDB备份脚本
"""
#!/usr/bin/env python3
import subprocess
import datetime
import os
import shutil

class RedisRDBBackup:
    def __init__(self, redis_host='127.0.0.1', redis_port=6379, 
                 redis_password='', backup_dir='/backup/redis'):
        self.redis_host = redis_host
        self.redis_port = redis_port
        self.redis_password = redis_password
        self.backup_dir = backup_dir
        
        # 创建备份目录
        os.makedirs(backup_dir, exist_ok=True)
    
    def trigger_bgsave(self):
        """触发BGSAVE"""
        cmd = [
            'redis-cli',
            '-h', self.redis_host,
            '-p', str(self.redis_port),
            '-a', self.redis_password,
            'BGSAVE'
        ]
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result.returncode == 0
    
    def wait_for_bgsave(self):
        """等待BGSAVE完成"""
        import time
        
        while True:
            cmd = [
                'redis-cli',
                '-h', self.redis_host,
                '-p', str(self.redis_port),
                '-a', self.redis_password,
                'INFO', 'persistence'
            ]
            
            result = subprocess.run(cmd, capture_output=True, text=True)
            if 'rdb_bgsave_in_progress:0' in result.stdout:
                break
            
            time.sleep(1)
    
    def backup(self):
        """执行备份"""
        # 触发BGSAVE
        if not self.trigger_bgsave():
            print("触发BGSAVE失败")
            return False
        
        # 等待完成
        self.wait_for_bgsave()
        
        # 复制RDB文件
        timestamp = datetime.datetime.now().strftime('%Y%m%d_%H%M%S')
        source = '/usr/local/redis/data/dump.rdb'
        dest = f'{self.backup_dir}/dump_{timestamp}.rdb'
        
        shutil.copy2(source, dest)
        print(f"备份完成: {dest}")
        
        # 清理旧备份（保留7天）
        self.cleanup_old_backups(days=7)
        
        return True
    
    def cleanup_old_backups(self, days=7):
        """清理旧备份"""
        import time
        
        now = time.time()
        cutoff = now - (days * 86400)
        
        for filename in os.listdir(self.backup_dir):
            if filename.startswith('dump_') and filename.endswith('.rdb'):
                filepath = os.path.join(self.backup_dir, filename)
                if os.path.getmtime(filepath) < cutoff:
                    os.remove(filepath)
                    print(f"删除旧备份: {filename}")

# 使用示例
if __name__ == '__main__':
    backup = RedisRDBBackup(
        redis_password='your_password',
        backup_dir='/backup/redis'
    )
    backup.backup()
```

### 4.2 AOF持久化

```bash
# AOF配置
appendonly yes
appendfilename "appendonly.aof"

# 同步策略
# always: 每次写入都同步（最安全，性能最差）
# everysec: 每秒同步一次（推荐）
# no: 由操作系统决定（性能最好，可能丢失数据）
appendfsync everysec

# AOF重写
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 手动触发重写
redis-cli -a password BGREWRITEAOF
```

```bash
# AOF文件修复
redis-check-aof --fix appendonly.aof
```

### 4.3 混合持久化

```bash
# redis.conf
# 开启混合持久化（Redis 4.0+）
aof-use-rdb-preamble yes

# 优点：
# 1. RDB部分加载快
# 2. AOF部分保证数据完整性
# 3. 兼顾性能和安全
```

## 5. Redis主从复制

### 5.1 主从架构

```
┌─────────────┐
│   Master    │
│  (读写)     │
└──────┬──────┘
       │
       ├──────────┬──────────┐
       │          │          │
┌──────▼──────┐ ┌▼─────────┐ ┌▼─────────┐
│   Slave 1   │ │ Slave 2  │ │ Slave 3  │
│   (只读)    │ │  (只读)  │ │  (只读)  │
└─────────────┘ └──────────┘ └──────────┘
```

### 5.2 主从配置

```bash
# Master配置 (redis-master.conf)
bind 0.0.0.0
port 6379
requirepass master_password
masterauth master_password

# 复制配置
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-backlog-size 256mb
repl-backlog-ttl 3600
min-replicas-to-write 1
min-replicas-max-lag 10
```

```bash
# Slave配置 (redis-slave.conf)
bind 0.0.0.0
port 6379
requirepass slave_password

# 主从复制配置
replicaof 192.168.1.100 6379
masterauth master_password
replica-read-only yes
replica-serve-stale-data yes
replica-priority 100
```

### 5.3 主从管理

```bash
# 查看复制信息
redis-cli -a password INFO replication

# 动态配置从节点
redis-cli -a password REPLICAOF 192.168.1.100 6379

# 取消复制
redis-cli -a password REPLICAOF NO ONE

# 查看主从延迟
redis-cli -a password INFO replication | grep lag
```

```python
"""
主从监控脚本
"""
#!/usr/bin/env python3
import redis
import time

class RedisReplicationMonitor:
    def __init__(self, master_host, master_port, password):
        self.master = redis.Redis(
            host=master_host,
            port=master_port,
            password=password,
            decode_responses=True
        )
    
    def get_replication_info(self):
        """获取复制信息"""
        info = self.master.info('replication')
        return info
    
    def check_slaves(self):
        """检查从节点状态"""
        info = self.get_replication_info()
        
        if info['role'] != 'master':
            print("当前节点不是主节点")
            return
        
        connected_slaves = info['connected_slaves']
        print(f"连接的从节点数: {connected_slaves}")
        
        for i in range(connected_slaves):
            slave_key = f'slave{i}'
            slave_info = info[slave_key]
            print(f"\n从节点 {i}:")
            print(f"  {slave_info}")
            
            # 解析从节点信息
            parts = slave_info.split(',')
            for part in parts:
                print(f"    {part}")
    
    def check_replication_lag(self):
        """检查复制延迟"""
        info = self.get_replication_info()
        
        if info['role'] != 'master':
            return
        
        for i in range(info['connected_slaves']):
            slave_key = f'slave{i}'
            slave_info = info[slave_key]
            
            # 提取lag信息
            for part in slave_info.split(','):
                if 'lag=' in part:
                    lag = int(part.split('=')[1])
                    if lag > 10:
                        print(f"警告: 从节点 {i} 延迟过高: {lag}秒")

# 使用示例
if __name__ == '__main__':
    monitor = RedisReplicationMonitor(
        master_host='192.168.1.100',
        master_port=6379,
        password='your_password'
    )
    
    while True:
        monitor.check_slaves()
        monitor.check_replication_lag()
        time.sleep(60)
```


## 6. Redis哨兵模式

### 6.1 哨兵架构

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│Sentinel 1│  │Sentinel 2│  │Sentinel 3│
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │            │            │
     └────────────┼────────────┘
                  │
     ┌────────────┴────────────┐
     │                         │
┌────▼─────┐            ┌─────▼────┐
│  Master  │◄───────────┤  Slave   │
└──────────┘            └──────────┘
```

### 6.2 哨兵配置

```bash
# sentinel.conf
port 26379
daemonize yes
pidfile /var/run/redis/sentinel.pid
logfile /usr/local/redis/logs/sentinel.log
dir /usr/local/redis/data

# 监控主节点
sentinel monitor mymaster 192.168.1.100 6379 2
sentinel auth-pass mymaster your_password

# 故障判定
sentinel down-after-milliseconds mymaster 5000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000

# 通知脚本
sentinel notification-script mymaster /usr/local/redis/scripts/notify.sh
sentinel client-reconfig-script mymaster /usr/local/redis/scripts/reconfig.sh
```

```bash
# 启动哨兵
redis-sentinel /usr/local/redis/etc/sentinel.conf

# 或者
redis-server /usr/local/redis/etc/sentinel.conf --sentinel
```

### 6.3 哨兵管理

```bash
# 连接哨兵
redis-cli -p 26379

# 查看主节点信息
SENTINEL master mymaster

# 查看从节点信息
SENTINEL slaves mymaster

# 查看哨兵信息
SENTINEL sentinels mymaster

# 手动故障转移
SENTINEL failover mymaster

# 移除监控
SENTINEL remove mymaster

# 添加监控
SENTINEL monitor mymaster 192.168.1.100 6379 2
```

### 6.4 客户端连接

```python
"""
Python客户端连接哨兵
"""
from redis.sentinel import Sentinel

# 配置哨兵
sentinel = Sentinel([
    ('192.168.1.101', 26379),
    ('192.168.1.102', 26379),
    ('192.168.1.103', 26379)
], socket_timeout=0.5)

# 获取主节点
master = sentinel.master_for(
    'mymaster',
    socket_timeout=0.5,
    password='your_password',
    db=0
)

# 获取从节点
slave = sentinel.slave_for(
    'mymaster',
    socket_timeout=0.5,
    password='your_password',
    db=0
)

# 写操作
master.set('key', 'value')

# 读操作
value = slave.get('key')
```

## 7. Redis集群模式

### 7.1 集群架构

```
┌─────────────────────────────────────────┐
│           Redis Cluster                  │
├─────────────────────────────────────────┤
│  Master1  Master2  Master3              │
│  (0-5460) (5461-  (10923-               │
│           10922)  16383)                │
│     │        │        │                 │
│  Slave1  Slave2  Slave3                │
└─────────────────────────────────────────┘
```

### 7.2 集群部署

```bash
# 创建集群目录
mkdir -p /usr/local/redis/cluster/{7000,7001,7002,7003,7004,7005}

# 配置文件模板
cat > /usr/local/redis/cluster/redis-cluster.conf << 'EOF'
port ${PORT}
bind 0.0.0.0
daemonize yes
pidfile /var/run/redis/redis-${PORT}.pid
logfile /usr/local/redis/logs/redis-${PORT}.log
dir /usr/local/redis/cluster/${PORT}

# 集群配置
cluster-enabled yes
cluster-config-file nodes-${PORT}.conf
cluster-node-timeout 5000
cluster-require-full-coverage no

# 持久化
appendonly yes
appendfilename "appendonly-${PORT}.aof"

# 安全
requirepass your_password
masterauth your_password
EOF

# 生成各节点配置
for port in 7000 7001 7002 7003 7004 7005; do
    sed "s/\${PORT}/$port/g" /usr/local/redis/cluster/redis-cluster.conf \
        > /usr/local/redis/cluster/$port/redis.conf
done

# 启动所有节点
for port in 7000 7001 7002 7003 7004 7005; do
    redis-server /usr/local/redis/cluster/$port/redis.conf
done

# 创建集群
redis-cli --cluster create \
    127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 \
    127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
    --cluster-replicas 1 \
    -a your_password
```

### 7.3 集群管理

```bash
# 查看集群信息
redis-cli -c -p 7000 -a password CLUSTER INFO

# 查看集群节点
redis-cli -c -p 7000 -a password CLUSTER NODES

# 查看槽分配
redis-cli -c -p 7000 -a password CLUSTER SLOTS

# 添加节点
redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7000 \
    -a password

# 删除节点
redis-cli --cluster del-node 127.0.0.1:7000 <node-id> \
    -a password

# 重新分片
redis-cli --cluster reshard 127.0.0.1:7000 \
    --cluster-from <source-node-id> \
    --cluster-to <target-node-id> \
    --cluster-slots 1000 \
    -a password

# 集群检查
redis-cli --cluster check 127.0.0.1:7000 -a password

# 集群修复
redis-cli --cluster fix 127.0.0.1:7000 -a password
```

### 7.4 集群客户端

```python
"""
Python客户端连接集群
"""
from rediscluster import RedisCluster

# 配置集群节点
startup_nodes = [
    {"host": "192.168.1.101", "port": "7000"},
    {"host": "192.168.1.102", "port": "7000"},
    {"host": "192.168.1.103", "port": "7000"}
]

# 创建集群连接
rc = RedisCluster(
    startup_nodes=startup_nodes,
    decode_responses=True,
    password='your_password',
    skip_full_coverage_check=True
)

# 使用集群
rc.set('key', 'value')
value = rc.get('key')

# 批量操作
pipe = rc.pipeline()
pipe.set('key1', 'value1')
pipe.set('key2', 'value2')
pipe.execute()
```

## 8. Redis性能优化

### 8.1 慢查询分析

```bash
# 配置慢查询
CONFIG SET slowlog-log-slower-than 10000  # 10ms
CONFIG SET slowlog-max-len 128

# 查看慢查询
SLOWLOG GET 10

# 清空慢查询
SLOWLOG RESET
```

```python
"""
慢查询分析工具
"""
#!/usr/bin/env python3
import redis
from collections import defaultdict

class SlowLogAnalyzer:
    def __init__(self, host, port, password):
        self.redis = redis.Redis(
            host=host,
            port=port,
            password=password,
            decode_responses=True
        )
    
    def get_slow_logs(self, count=100):
        """获取慢查询日志"""
        return self.redis.slowlog_get(count)
    
    def analyze(self):
        """分析慢查询"""
        logs = self.get_slow_logs()
        
        # 按命令统计
        cmd_stats = defaultdict(lambda: {'count': 0, 'total_time': 0})
        
        for log in logs:
            cmd = log['command'].split()[0]
            duration = log['duration']
            
            cmd_stats[cmd]['count'] += 1
            cmd_stats[cmd]['total_time'] += duration
        
        # 计算平均时间
        for cmd in cmd_stats:
            count = cmd_stats[cmd]['count']
            total = cmd_stats[cmd]['total_time']
            cmd_stats[cmd]['avg_time'] = total / count
        
        # 排序
        sorted_stats = sorted(
            cmd_stats.items(),
            key=lambda x: x[1]['total_time'],
            reverse=True
        )
        
        # 输出报告
        print("慢查询分析报告")
        print("=" * 60)
        print(f"{'命令':<15} {'次数':<10} {'总时间(μs)':<15} {'平均时间(μs)':<15}")
        print("-" * 60)
        
        for cmd, stats in sorted_stats[:10]:
            print(f"{cmd:<15} {stats['count']:<10} "
                  f"{stats['total_time']:<15} {stats['avg_time']:<15.2f}")

# 使用示例
if __name__ == '__main__':
    analyzer = SlowLogAnalyzer(
        host='127.0.0.1',
        port=6379,
        password='your_password'
    )
    analyzer.analyze()
```

### 8.2 内存优化

```bash
# 查看内存使用
INFO memory

# 内存分析
MEMORY STATS
MEMORY DOCTOR

# 查看大key
redis-cli -a password --bigkeys

# 查看热key
redis-cli -a password --hotkeys
```

```python
"""
大key扫描工具
"""
#!/usr/bin/env python3
import redis

class BigKeyScanner:
    def __init__(self, host, port, password):
        self.redis = redis.Redis(
            host=host,
            port=port,
            password=password,
            decode_responses=True
        )
    
    def scan_big_keys(self, threshold=10240):
        """扫描大key（大于10KB）"""
        big_keys = []
        cursor = 0
        
        while True:
            cursor, keys = self.redis.scan(cursor, count=100)
            
            for key in keys:
                # 获取key类型
                key_type = self.redis.type(key)
                
                # 获取key大小
                size = self.get_key_size(key, key_type)
                
                if size > threshold:
                    big_keys.append({
                        'key': key,
                        'type': key_type,
                        'size': size
                    })
            
            if cursor == 0:
                break
        
        return sorted(big_keys, key=lambda x: x['size'], reverse=True)
    
    def get_key_size(self, key, key_type):
        """获取key大小"""
        if key_type == 'string':
            return len(self.redis.get(key) or '')
        elif key_type == 'list':
            return self.redis.llen(key)
        elif key_type == 'set':
            return self.redis.scard(key)
        elif key_type == 'zset':
            return self.redis.zcard(key)
        elif key_type == 'hash':
            return self.redis.hlen(key)
        return 0
    
    def report(self):
        """生成报告"""
        big_keys = self.scan_big_keys()
        
        print("大Key报告")
        print("=" * 60)
        print(f"{'Key':<30} {'类型':<10} {'大小':<10}")
        print("-" * 60)
        
        for item in big_keys[:20]:
            print(f"{item['key']:<30} {item['type']:<10} {item['size']:<10}")

# 使用示例
if __name__ == '__main__':
    scanner = BigKeyScanner(
        host='127.0.0.1',
        port=6379,
        password='your_password'
    )
    scanner.report()
```

### 8.3 连接池优化

```python
"""
连接池配置
"""
import redis
from redis import ConnectionPool

# 创建连接池
pool = ConnectionPool(
    host='127.0.0.1',
    port=6379,
    password='your_password',
    db=0,
    max_connections=50,      # 最大连接数
    socket_timeout=5,        # 超时时间
    socket_connect_timeout=5,
    socket_keepalive=True,
    socket_keepalive_options={
        1: 1,  # TCP_KEEPIDLE
        2: 1,  # TCP_KEEPINTVL
        3: 3   # TCP_KEEPCNT
    },
    retry_on_timeout=True,
    health_check_interval=30
)

# 使用连接池
redis_client = redis.Redis(connection_pool=pool)

# 使用
redis_client.set('key', 'value')
value = redis_client.get('key')
```

### 8.4 Pipeline优化

```python
"""
Pipeline批量操作
"""
import redis
import time

redis_client = redis.Redis(
    host='127.0.0.1',
    port=6379,
    password='your_password'
)

# 不使用Pipeline
start = time.time()
for i in range(1000):
    redis_client.set(f'key:{i}', f'value:{i}')
print(f"不使用Pipeline: {time.time() - start:.2f}秒")

# 使用Pipeline
start = time.time()
pipe = redis_client.pipeline()
for i in range(1000):
    pipe.set(f'key:{i}', f'value:{i}')
pipe.execute()
print(f"使用Pipeline: {time.time() - start:.2f}秒")

# 分批Pipeline（推荐）
start = time.time()
batch_size = 100
for i in range(0, 1000, batch_size):
    pipe = redis_client.pipeline()
    for j in range(i, min(i + batch_size, 1000)):
        pipe.set(f'key:{j}', f'value:{j}')
    pipe.execute()
print(f"分批Pipeline: {time.time() - start:.2f}秒")
```

## 9. Redis监控告警

### 9.1 监控指标

```bash
# 关键监控指标

# 1. 性能指标
INFO stats
- instantaneous_ops_per_sec  # QPS
- instantaneous_input_kbps   # 输入带宽
- instantaneous_output_kbps  # 输出带宽

# 2. 内存指标
INFO memory
- used_memory                # 已用内存
- used_memory_rss            # 物理内存
- mem_fragmentation_ratio    # 内存碎片率
- evicted_keys               # 淘汰key数量

# 3. 持久化指标
INFO persistence
- rdb_last_save_time         # 最后RDB时间
- rdb_changes_since_last_save # RDB变更数
- aof_current_size           # AOF文件大小
- aof_last_rewrite_time_sec  # AOF重写耗时

# 4. 复制指标
INFO replication
- connected_slaves           # 从节点数量
- master_repl_offset         # 主节点偏移量
- slave_repl_offset          # 从节点偏移量

# 5. 客户端指标
INFO clients
- connected_clients          # 连接数
- blocked_clients            # 阻塞客户端数
- client_recent_max_input_buffer  # 最大输入缓冲区
```

### 9.2 Prometheus监控

```yaml
# redis_exporter部署
version: '3.8'

services:
  redis-exporter:
    image: oliver006/redis_exporter:latest
    container_name: redis-exporter
    restart: always
    ports:
      - "9121:9121"
    environment:
      - REDIS_ADDR=redis://192.168.1.100:6379
      - REDIS_PASSWORD=your_password
    command:
      - '--redis.addr=redis://192.168.1.100:6379'
      - '--redis.password=your_password'
```

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'redis'
    static_configs:
      - targets: ['192.168.1.100:9121']
        labels:
          instance: 'redis-master'
```

```yaml
# 告警规则
groups:
  - name: redis_alerts
    rules:
      # Redis宕机
      - alert: RedisDown
        expr: redis_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Redis实例 {{ $labels.instance }} 宕机"
      
      # 内存使用率过高
      - alert: RedisMemoryHigh
        expr: redis_memory_used_bytes / redis_memory_max_bytes > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Redis内存使用率过高"
          description: "{{ $labels.instance }} 内存使用率: {{ $value | humanizePercentage }}"
      
      # 连接数过高
      - alert: RedisTooManyConnections
        expr: redis_connected_clients > 1000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Redis连接数过高"
          description: "{{ $labels.instance }} 连接数: {{ $value }}"
      
      # 主从复制延迟
      - alert: RedisReplicationLag
        expr: redis_master_repl_offset - redis_slave_repl_offset > 1000000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Redis主从复制延迟"
          description: "{{ $labels.instance }} 延迟: {{ $value }} bytes"
```

### 9.3 自定义监控脚本

```python
"""
Redis健康检查脚本
"""
#!/usr/bin/env python3
import redis
import sys

class RedisHealthCheck:
    def __init__(self, host, port, password):
        self.redis = redis.Redis(
            host=host,
            port=port,
            password=password,
            socket_timeout=5,
            decode_responses=True
        )
        self.checks = []
    
    def check_connectivity(self):
        """检查连接"""
        try:
            self.redis.ping()
            self.checks.append(('连接检查', 'OK', ''))
            return True
        except Exception as e:
            self.checks.append(('连接检查', 'FAIL', str(e)))
            return False
    
    def check_memory(self):
        """检查内存"""
        info = self.redis.info('memory')
        used = info['used_memory']
        maxmemory = info.get('maxmemory', 0)
        
        if maxmemory > 0:
            usage = (used / maxmemory) * 100
            if usage > 90:
                self.checks.append(('内存检查', 'WARN', f'使用率: {usage:.2f}%'))
            else:
                self.checks.append(('内存检查', 'OK', f'使用率: {usage:.2f}%'))
        else:
            self.checks.append(('内存检查', 'OK', f'已用: {used / 1024 / 1024:.2f}MB'))
    
    def check_persistence(self):
        """检查持久化"""
        info = self.redis.info('persistence')
        
        # 检查RDB
        rdb_last_save = info.get('rdb_last_save_time', 0)
        import time
        if time.time() - rdb_last_save > 3600:
            self.checks.append(('RDB检查', 'WARN', '超过1小时未保存'))
        else:
            self.checks.append(('RDB检查', 'OK', ''))
        
        # 检查AOF
        if info.get('aof_enabled', 0) == 1:
            aof_last_rewrite = info.get('aof_last_rewrite_time_sec', -1)
            if aof_last_rewrite > 300:
                self.checks.append(('AOF检查', 'WARN', f'重写耗时: {aof_last_rewrite}秒'))
            else:
                self.checks.append(('AOF检查', 'OK', ''))
    
    def check_replication(self):
        """检查复制"""
        info = self.redis.info('replication')
        role = info['role']
        
        if role == 'master':
            slaves = info.get('connected_slaves', 0)
            self.checks.append(('复制检查', 'OK', f'从节点数: {slaves}'))
        else:
            master_link = info.get('master_link_status', 'down')
            if master_link == 'up':
                self.checks.append(('复制检查', 'OK', '主从连接正常'))
            else:
                self.checks.append(('复制检查', 'FAIL', '主从连接断开'))
    
    def run_all_checks(self):
        """运行所有检查"""
        if not self.check_connectivity():
            return False
        
        self.check_memory()
        self.check_persistence()
        self.check_replication()
        
        return True
    
    def report(self):
        """生成报告"""
        print("Redis健康检查报告")
        print("=" * 60)
        
        for check_name, status, message in self.checks:
            status_icon = {
                'OK': '✓',
                'WARN': '⚠',
                'FAIL': '✗'
            }.get(status, '?')
            
            print(f"{status_icon} {check_name:<20} {status:<10} {message}")
        
        # 返回状态码
        has_fail = any(status == 'FAIL' for _, status, _ in self.checks)
        return 1 if has_fail else 0

# 使用示例
if __name__ == '__main__':
    checker = RedisHealthCheck(
        host='127.0.0.1',
        port=6379,
        password='your_password'
    )
    
    if checker.run_all_checks():
        exit_code = checker.report()
        sys.exit(exit_code)
    else:
        print("无法连接到Redis")
        sys.exit(1)
```

## 10. Redis备份恢复

### 10.1 备份策略

```bash
# 备份脚本
#!/bin/bash

REDIS_CLI="/usr/local/redis/bin/redis-cli"
REDIS_HOST="127.0.0.1"
REDIS_PORT="6379"
REDIS_PASSWORD="your_password"
BACKUP_DIR="/backup/redis"
DATE=$(date +%Y%m%d_%H%M%S)

# 创建备份目录
mkdir -p $BACKUP_DIR

# 触发BGSAVE
$REDIS_CLI -h $REDIS_HOST -p $REDIS_PORT -a $REDIS_PASSWORD BGSAVE

# 等待BGSAVE完成
while true; do
    BGSAVE_STATUS=$($REDIS_CLI -h $REDIS_HOST -p $REDIS_PORT -a $REDIS_PASSWORD \
        INFO persistence | grep rdb_bgsave_in_progress | cut -d: -f2 | tr -d '\r')
    
    if [ "$BGSAVE_STATUS" == "0" ]; then
        break
    fi
    sleep 1
done

# 复制RDB文件
cp /usr/local/redis/data/dump.rdb $BACKUP_DIR/dump_$DATE.rdb

# 复制AOF文件
if [ -f /usr/local/redis/data/appendonly.aof ]; then
    cp /usr/local/redis/data/appendonly.aof $BACKUP_DIR/appendonly_$DATE.aof
fi

# 压缩备份
cd $BACKUP_DIR
tar -czf redis_backup_$DATE.tar.gz dump_$DATE.rdb appendonly_$DATE.aof
rm -f dump_$DATE.rdb appendonly_$DATE.aof

# 清理旧备份（保留7天）
find $BACKUP_DIR -name "redis_backup_*.tar.gz" -mtime +7 -delete

echo "备份完成: redis_backup_$DATE.tar.gz"
```

### 10.2 数据恢复

```bash
# 恢复步骤

# 1. 停止Redis
systemctl stop redis

# 2. 备份当前数据
cp /usr/local/redis/data/dump.rdb /usr/local/redis/data/dump.rdb.bak

# 3. 解压备份文件
cd /backup/redis
tar -xzf redis_backup_20240201_120000.tar.gz

# 4. 恢复数据文件
cp dump_20240201_120000.rdb /usr/local/redis/data/dump.rdb
cp appendonly_20240201_120000.aof /usr/local/redis/data/appendonly.aof

# 5. 修改权限
chown redis:redis /usr/local/redis/data/*

# 6. 启动Redis
systemctl start redis

# 7. 验证数据
redis-cli -a password DBSIZE
```

### 10.3 在线迁移

```python
"""
Redis数据迁移工具
"""
#!/usr/bin/env python3
import redis

class RedisMigration:
    def __init__(self, source_config, target_config):
        self.source = redis.Redis(**source_config)
        self.target = redis.Redis(**target_config)
    
    def migrate_all(self, batch_size=1000):
        """迁移所有数据"""
        cursor = 0
        total = 0
        
        while True:
            cursor, keys = self.source.scan(cursor, count=batch_size)
            
            if keys:
                self.migrate_keys(keys)
                total += len(keys)
                print(f"已迁移 {total} 个key")
            
            if cursor == 0:
                break
        
        print(f"迁移完成，共 {total} 个key")
    
    def migrate_keys(self, keys):
        """迁移指定的keys"""
        pipe_source = self.source.pipeline()
        pipe_target = self.target.pipeline()
        
        # 批量获取数据
        for key in keys:
            pipe_source.dump(key)
            pipe_source.pttl(key)
        
        results = pipe_source.execute()
        
        # 批量写入
        for i, key in enumerate(keys):
            value = results[i * 2]
            ttl = results[i * 2 + 1]
            
            if value:
                if ttl > 0:
                    pipe_target.restore(key, ttl, value, replace=True)
                else:
                    pipe_target.restore(key, 0, value, replace=True)
        
        pipe_target.execute()

# 使用示例
if __name__ == '__main__':
    source_config = {
        'host': '192.168.1.100',
        'port': 6379,
        'password': 'source_password',
        'db': 0
    }
    
    target_config = {
        'host': '192.168.1.200',
        'port': 6379,
        'password': 'target_password',
        'db': 0
    }
    
    migration = RedisMigration(source_config, target_config)
    migration.migrate_all()
```

好的，我已经完成了告警系统教程的补充，并开始创建Redis运维教程。由于内容较多，我已经完成了Redis教程的前10章内容，包括：

1. Redis基础架构
2. Redis安装部署
3. Redis配置优化
4. Redis持久化
5. Redis主从复制
6. Redis哨兵模式
7. Redis集群模式
8. Redis性能优化
9. Redis监控告警
10. Redis备份恢复

接下来我需要继续补充剩余的章节（故障排查、安全加固、实战案例、学习检查清单等）。让我继续完成。


## 11. Redis故障排查

### 11.1 常见故障类型

```yaml
# Redis常见故障分类

性能问题:
  - 响应时间慢
  - QPS下降
  - 内存使用率高
  - CPU使用率高

可用性问题:
  - 服务宕机
  - 主从切换
  - 集群节点失联
  - 连接超时

数据问题:
  - 数据丢失
  - 数据不一致
  - 主从延迟
  - 内存溢出
```

### 11.2 性能问题排查

```bash
# 1. 慢查询分析
redis-cli -a password SLOWLOG GET 10

# 2. 查看当前执行的命令
redis-cli -a password --bigkeys
redis-cli -a password --hotkeys

# 3. 监控命令执行
redis-cli -a password MONITOR

# 4. 查看客户端连接
redis-cli -a password CLIENT LIST

# 5. 查看内存使用
redis-cli -a password INFO memory

# 6. 查看统计信息
redis-cli -a password INFO stats
```

```python
"""
性能问题诊断工具
"""
#!/usr/bin/env python3
import redis
import time

class RedisPerformanceDiagnostic:
    def __init__(self, host, port, password):
        self.redis = redis.Redis(
            host=host,
            port=port,
            password=password,
            decode_responses=True
        )
    
    def check_slow_queries(self):
        """检查慢查询"""
        slow_logs = self.redis.slowlog_get(10)
        
        if slow_logs:
            print("慢查询Top 10:")
            for log in slow_logs:
                print(f"  命令: {' '.join(log['command'])}")
                print(f"  耗时: {log['duration']}μs")
                print(f"  时间: {time.ctime(log['start_time'])}")
                print()
    
    def check_memory_fragmentation(self):
        """检查内存碎片"""
        info = self.redis.info('memory')
        used_memory = info['used_memory']
        used_memory_rss = info['used_memory_rss']
        fragmentation_ratio = info['mem_fragmentation_ratio']
        
        print(f"内存使用: {used_memory / 1024 / 1024:.2f}MB")
        print(f"物理内存: {used_memory_rss / 1024 / 1024:.2f}MB")
        print(f"碎片率: {fragmentation_ratio:.2f}")
        
        if fragmentation_ratio > 1.5:
            print("警告: 内存碎片率过高，建议重启Redis")
    
    def check_blocked_clients(self):
        """检查阻塞客户端"""
        info = self.redis.info('clients')
        blocked_clients = info.get('blocked_clients', 0)
        
        if blocked_clients > 0:
            print(f"警告: 有 {blocked_clients} 个阻塞客户端")
    
    def check_evicted_keys(self):
        """检查淘汰key"""
        info = self.redis.info('stats')
        evicted_keys = info.get('evicted_keys', 0)
        
        if evicted_keys > 0:
            print(f"警告: 已淘汰 {evicted_keys} 个key，内存可能不足")
    
    def diagnose(self):
        """执行诊断"""
        print("=" * 60)
        print("Redis性能诊断报告")
        print("=" * 60)
        
        self.check_slow_queries()
        self.check_memory_fragmentation()
        self.check_blocked_clients()
        self.check_evicted_keys()

# 使用示例
if __name__ == '__main__':
    diagnostic = RedisPerformanceDiagnostic(
        host='127.0.0.1',
        port=6379,
        password='your_password'
    )
    diagnostic.diagnose()
```

### 11.3 连接问题排查

```bash
# 1. 检查网络连通性
ping 192.168.1.100
telnet 192.168.1.100 6379

# 2. 检查防火墙
firewall-cmd --list-ports
iptables -L -n | grep 6379

# 3. 检查Redis进程
ps aux | grep redis
netstat -tlnp | grep 6379

# 4. 检查连接数
redis-cli -a password INFO clients

# 5. 检查配置
redis-cli -a password CONFIG GET maxclients
redis-cli -a password CONFIG GET timeout
```

### 11.4 数据一致性问题

```python
"""
主从数据一致性检查
"""
#!/usr/bin/env python3
import redis

class RedisConsistencyChecker:
    def __init__(self, master_config, slave_config):
        self.master = redis.Redis(**master_config)
        self.slave = redis.Redis(**slave_config)
    
    def check_key_consistency(self, sample_size=1000):
        """检查key一致性"""
        inconsistent_keys = []
        cursor = 0
        checked = 0
        
        while checked < sample_size:
            cursor, keys = self.master.scan(cursor, count=100)
            
            for key in keys:
                if checked >= sample_size:
                    break
                
                # 获取主节点值
                master_value = self.master.get(key)
                
                # 获取从节点值
                slave_value = self.slave.get(key)
                
                # 比较
                if master_value != slave_value:
                    inconsistent_keys.append({
                        'key': key,
                        'master_value': master_value,
                        'slave_value': slave_value
                    })
                
                checked += 1
            
            if cursor == 0:
                break
        
        return inconsistent_keys
    
    def check_replication_lag(self):
        """检查复制延迟"""
        master_info = self.master.info('replication')
        slave_info = self.slave.info('replication')
        
        master_offset = master_info.get('master_repl_offset', 0)
        slave_offset = slave_info.get('slave_repl_offset', 0)
        
        lag = master_offset - slave_offset
        
        return lag
    
    def report(self):
        """生成报告"""
        print("数据一致性检查报告")
        print("=" * 60)
        
        # 检查复制延迟
        lag = self.check_replication_lag()
        print(f"复制延迟: {lag} bytes")
        
        if lag > 1000000:
            print("警告: 复制延迟过大")
        
        # 检查key一致性
        inconsistent = self.check_key_consistency()
        
        if inconsistent:
            print(f"\n发现 {len(inconsistent)} 个不一致的key:")
            for item in inconsistent[:10]:
                print(f"  Key: {item['key']}")
                print(f"  Master: {item['master_value']}")
                print(f"  Slave: {item['slave_value']}")
        else:
            print("\n所有检查的key都一致")

# 使用示例
if __name__ == '__main__':
    master_config = {
        'host': '192.168.1.100',
        'port': 6379,
        'password': 'master_password',
        'decode_responses': True
    }
    
    slave_config = {
        'host': '192.168.1.101',
        'port': 6379,
        'password': 'slave_password',
        'decode_responses': True
    }
    
    checker = RedisConsistencyChecker(master_config, slave_config)
    checker.report()
```

## 12. Redis安全加固

### 12.1 访问控制

```bash
# redis.conf

# 1. 绑定IP
bind 127.0.0.1 192.168.1.100

# 2. 设置密码
requirepass your_strong_password_here

# 3. 禁用危险命令
rename-command FLUSHALL ""
rename-command FLUSHDB ""
rename-command KEYS ""
rename-command CONFIG "CONFIG_ADMIN_ONLY"
rename-command SHUTDOWN "SHUTDOWN_ADMIN_ONLY"

# 4. 保护模式
protected-mode yes

# 5. 限制连接数
maxclients 10000
```

### 12.2 网络安全

```bash
# 1. 防火墙配置
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="6379" accept'
firewall-cmd --reload

# 2. iptables配置
iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 6379 -j ACCEPT
iptables -A INPUT -p tcp --dport 6379 -j DROP

# 3. 使用TLS加密
# redis.conf
tls-port 6380
tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
tls-ca-cert-file /path/to/ca.crt
```

### 12.3 ACL权限控制

```bash
# Redis 6.0+ ACL功能

# 创建用户
ACL SETUSER alice on >password ~cached:* +get +set

# 查看用户
ACL LIST
ACL GETUSER alice

# 删除用户
ACL DELUSER alice

# ACL配置文件
# redis.conf
aclfile /usr/local/redis/etc/users.acl
```

```bash
# users.acl
# 只读用户
user readonly on >readonly_password ~* +@read -@write -@dangerous

# 管理员用户
user admin on >admin_password ~* +@all

# 应用用户
user app on >app_password ~app:* +@read +@write -@dangerous
```

### 12.4 审计日志

```python
"""
Redis操作审计
"""
#!/usr/bin/env python3
import redis
import logging
from datetime import datetime

class RedisAuditLogger:
    def __init__(self, host, port, password):
        self.redis = redis.Redis(
            host=host,
            port=port,
            password=password
        )
        
        # 配置日志
        logging.basicConfig(
            filename='/var/log/redis/audit.log',
            level=logging.INFO,
            format='%(asctime)s - %(message)s'
        )
        self.logger = logging
    
    def log_operation(self, user, operation, key, result):
        """记录操作"""
        log_entry = {
            'timestamp': datetime.now().isoformat(),
            'user': user,
            'operation': operation,
            'key': key,
            'result': result
        }
        
        self.logger.info(str(log_entry))
    
    def monitored_set(self, user, key, value):
        """监控SET操作"""
        try:
            result = self.redis.set(key, value)
            self.log_operation(user, 'SET', key, 'SUCCESS')
            return result
        except Exception as e:
            self.log_operation(user, 'SET', key, f'FAILED: {str(e)}')
            raise
    
    def monitored_get(self, user, key):
        """监控GET操作"""
        try:
            result = self.redis.get(key)
            self.log_operation(user, 'GET', key, 'SUCCESS')
            return result
        except Exception as e:
            self.log_operation(user, 'GET', key, f'FAILED: {str(e)}')
            raise
    
    def monitored_delete(self, user, key):
        """监控DELETE操作"""
        try:
            result = self.redis.delete(key)
            self.log_operation(user, 'DELETE', key, 'SUCCESS')
            return result
        except Exception as e:
            self.log_operation(user, 'DELETE', key, f'FAILED: {str(e)}')
            raise
```

## 13. 实战案例

### 13.1 案例一：缓存穿透

```python
"""
缓存穿透解决方案
"""
import redis
import hashlib

class BloomFilter:
    def __init__(self, redis_client, key='bloom_filter', size=10000000):
        self.redis = redis_client
        self.key = key
        self.size = size
    
    def _hash(self, value, seed):
        """计算hash值"""
        h = hashlib.md5(f"{value}{seed}".encode()).hexdigest()
        return int(h, 16) % self.size
    
    def add(self, value):
        """添加元素"""
        for seed in range(3):
            offset = self._hash(value, seed)
            self.redis.setbit(self.key, offset, 1)
    
    def exists(self, value):
        """检查元素是否存在"""
        for seed in range(3):
            offset = self._hash(value, seed)
            if not self.redis.getbit(self.key, offset):
                return False
        return True

# 使用示例
redis_client = redis.Redis(host='127.0.0.1', port=6379, password='password')
bloom = BloomFilter(redis_client)

# 添加所有有效的key
valid_keys = ['user:1', 'user:2', 'user:3']
for key in valid_keys:
    bloom.add(key)

# 查询时先检查布隆过滤器
def get_user(user_id):
    key = f'user:{user_id}'
    
    # 检查布隆过滤器
    if not bloom.exists(key):
        return None  # 一定不存在
    
    # 查询缓存
    data = redis_client.get(key)
    if data:
        return data
    
    # 查询数据库
    data = query_database(user_id)
    if data:
        redis_client.setex(key, 3600, data)
    
    return data
```

### 13.2 案例二：缓存击穿

```python
"""
缓存击穿解决方案（互斥锁）
"""
import redis
import time

class CacheWithMutex:
    def __init__(self, redis_client):
        self.redis = redis_client
    
    def get_with_mutex(self, key, fetch_func, expire=3600):
        """使用互斥锁获取数据"""
        # 尝试从缓存获取
        data = self.redis.get(key)
        if data:
            return data
        
        # 获取锁
        lock_key = f'lock:{key}'
        lock_acquired = self.redis.set(
            lock_key,
            '1',
            nx=True,
            ex=10
        )
        
        if lock_acquired:
            try:
                # 再次检查缓存
                data = self.redis.get(key)
                if data:
                    return data
                
                # 从数据库获取
                data = fetch_func()
                
                # 写入缓存
                if data:
                    self.redis.setex(key, expire, data)
                
                return data
            finally:
                # 释放锁
                self.redis.delete(lock_key)
        else:
            # 等待锁释放
            time.sleep(0.1)
            return self.get_with_mutex(key, fetch_func, expire)

# 使用示例
cache = CacheWithMutex(redis_client)

def get_hot_data(data_id):
    key = f'hot_data:{data_id}'
    
    def fetch_from_db():
        return query_database(data_id)
    
    return cache.get_with_mutex(key, fetch_from_db)
```

### 13.3 案例三：缓存雪崩

```python
"""
缓存雪崩解决方案
"""
import redis
import random

class CacheWithRandomExpire:
    def __init__(self, redis_client):
        self.redis = redis_client
    
    def set_with_random_expire(self, key, value, base_expire=3600):
        """设置随机过期时间"""
        # 添加随机偏移（±10%）
        random_offset = int(base_expire * 0.1 * random.random())
        expire = base_expire + random_offset
        
        self.redis.setex(key, expire, value)
    
    def set_never_expire(self, key, value):
        """设置永不过期，使用逻辑过期"""
        data = {
            'value': value,
            'expire_time': time.time() + 3600
        }
        self.redis.set(key, json.dumps(data))
    
    def get_with_logical_expire(self, key, fetch_func):
        """使用逻辑过期获取数据"""
        data_str = self.redis.get(key)
        
        if not data_str:
            # 缓存不存在，直接获取
            value = fetch_func()
            self.set_never_expire(key, value)
            return value
        
        data = json.loads(data_str)
        
        # 检查逻辑过期
        if time.time() > data['expire_time']:
            # 异步更新缓存
            threading.Thread(
                target=self._async_update,
                args=(key, fetch_func)
            ).start()
        
        return data['value']
    
    def _async_update(self, key, fetch_func):
        """异步更新缓存"""
        value = fetch_func()
        self.set_never_expire(key, value)
```

### 13.4 案例四：分布式锁

```python
"""
Redis分布式锁实现
"""
import redis
import uuid
import time

class RedisLock:
    def __init__(self, redis_client, lock_name, expire=10):
        self.redis = redis_client
        self.lock_name = f'lock:{lock_name}'
        self.expire = expire
        self.identifier = str(uuid.uuid4())
    
    def acquire(self, timeout=10):
        """获取锁"""
        end_time = time.time() + timeout
        
        while time.time() < end_time:
            # 尝试获取锁
            if self.redis.set(
                self.lock_name,
                self.identifier,
                nx=True,
                ex=self.expire
            ):
                return True
            
            time.sleep(0.001)
        
        return False
    
    def release(self):
        """释放锁"""
        # 使用Lua脚本保证原子性
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        
        self.redis.eval(lua_script, 1, self.lock_name, self.identifier)
    
    def __enter__(self):
        """上下文管理器"""
        if not self.acquire():
            raise Exception("无法获取锁")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """释放锁"""
        self.release()

# 使用示例
redis_client = redis.Redis(host='127.0.0.1', port=6379, password='password')

with RedisLock(redis_client, 'my_resource') as lock:
    # 执行需要加锁的操作
    print("执行业务逻辑")
    time.sleep(1)
```


## 14. 学习检查清单

### 14.1 基础知识

```yaml
□ 理解Redis的数据结构和使用场景
□ 掌握Redis的安装和配置
□ 了解Redis的内存模型
□ 理解Redis的持久化机制
□ 掌握Redis的基本命令
□ 了解Redis的事务和Pipeline
```

### 14.2 架构部署

```yaml
□ 掌握单机模式的部署
□ 能够搭建主从复制架构
□ 掌握哨兵模式的配置和使用
□ 能够部署Redis集群
□ 理解不同架构的优缺点
□ 能够选择合适的部署架构
```

### 14.3 性能优化

```yaml
□ 掌握Redis配置优化
□ 能够分析慢查询
□ 掌握内存优化技巧
□ 了解Pipeline和批量操作
□ 能够识别和优化大key
□ 掌握连接池的使用
```

### 14.4 高可用

```yaml
□ 理解主从复制原理
□ 掌握哨兵的工作机制
□ 能够处理故障转移
□ 了解集群的分片和迁移
□ 掌握数据备份和恢复
□ 能够设计高可用方案
```

### 14.5 监控运维

```yaml
□ 掌握Redis监控指标
□ 能够配置Prometheus监控
□ 掌握告警规则设置
□ 能够分析性能问题
□ 掌握故障排查方法
□ 了解常见运维工具
```

### 14.6 安全加固

```yaml
□ 掌握访问控制配置
□ 了解网络安全设置
□ 掌握ACL权限管理
□ 能够配置TLS加密
□ 了解审计日志
□ 掌握安全最佳实践
```

### 14.7 实战能力

```yaml
□ 能够解决缓存穿透问题
□ 能够解决缓存击穿问题
□ 能够解决缓存雪崩问题
□ 掌握分布式锁的实现
□ 能够进行数据迁移
□ 具备生产环境运维能力
```

## 📚 参考资源

### 官方文档

- [Redis官方文档](https://redis.io/documentation)
- [Redis命令参考](https://redis.io/commands)
- [Redis配置参考](https://redis.io/topics/config)
- [Redis持久化](https://redis.io/topics/persistence)
- [Redis复制](https://redis.io/topics/replication)
- [Redis哨兵](https://redis.io/topics/sentinel)
- [Redis集群](https://redis.io/topics/cluster-tutorial)

### 学习资源

- [Redis设计与实现](http://redisbook.com/)
- [Redis开发与运维](https://book.douban.com/subject/26971561/)
- [Redis实战](https://book.douban.com/subject/26612779/)
- [Redis深度历险](https://book.douban.com/subject/30386804/)

### 社区资源

- [Redis中文网](http://www.redis.cn/)
- [Redis中文文档](http://redis.cn/documentation.html)
- [Redis GitHub](https://github.com/redis/redis)
- [Redis Stack Overflow](https://stackoverflow.com/questions/tagged/redis)

### 工具集成

- [redis-py](https://github.com/redis/redis-py) - Python客户端
- [redis-cli](https://redis.io/topics/rediscli) - 命令行工具
- [RedisInsight](https://redis.com/redis-enterprise/redis-insight/) - 可视化工具
- [redis-exporter](https://github.com/oliver006/redis_exporter) - Prometheus监控
- [redis-shake](https://github.com/alibaba/RedisShake) - 数据同步工具

### 开源项目

- [Redisson](https://github.com/redisson/redisson) - Java客户端
- [Lettuce](https://github.com/lettuce-io/lettuce-core) - Java异步客户端
- [ioredis](https://github.com/luin/ioredis) - Node.js客户端
- [go-redis](https://github.com/go-redis/redis) - Go客户端

## 🎓 学习建议

### 初学者路径

1. 学习Redis基础概念和数据结构
2. 掌握Redis的安装和基本配置
3. 熟悉常用命令和操作
4. 了解持久化机制
5. 学习主从复制

### 进阶路径

1. 深入理解Redis内存模型
2. 掌握哨兵和集群部署
3. 学习性能优化技巧
4. 掌握监控和故障排查
5. 了解高可用方案设计

### 实战建议

1. 从单机部署开始，逐步过渡到集群
2. 重视监控和告警配置
3. 定期进行备份和演练
4. 关注性能指标和优化
5. 学习常见问题的解决方案
6. 参与社区讨论和分享

### 注意事项

1. 生产环境必须设置密码
2. 禁用或重命名危险命令
3. 合理配置内存和淘汰策略
4. 定期备份数据
5. 监控关键指标
6. 做好容量规划
7. 避免使用KEYS等危险命令
8. 注意主从延迟问题

### 性能优化建议

1. 使用Pipeline批量操作
2. 避免大key和热key
3. 合理设置过期时间
4. 使用连接池
5. 选择合适的数据结构
6. 避免慢查询
7. 定期清理无用数据
8. 监控内存碎片率

### 高可用建议

1. 部署主从或集群架构
2. 配置哨兵实现自动故障转移
3. 做好数据备份
4. 监控复制延迟
5. 定期演练故障恢复
6. 准备应急预案

---

> 💡 **提示**: Redis是高性能的内存数据库，在缓存、会话存储、消息队列等场景有广泛应用。掌握Redis的运维技能对于保障系统稳定性至关重要。

> 📖 **相关教程**: 
> - [MySQL运维-完整教程](./MySQL运维-完整教程.md)
> - [MongoDB运维-完整教程](./MongoDB运维-完整教程.md)
> - [PostgreSQL运维-完整教程](./PostgreSQL运维-完整教程.md)
> - [数据库备份恢复-完整教程](./数据库备份恢复-完整教程.md)

**@author erik.zhou**
