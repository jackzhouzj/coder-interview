# ELK 完整教程

> Elasticsearch + Logstash + Kibana 日志分析平台实战指南
>
> @author erik.zhou

## 📚 目录

- [1. ELK Stack 简介](#1-elk-stack-简介)
- [2. Elasticsearch 安装与配置](#2-elasticsearch-安装与配置)
- [3. Logstash 安装与配置](#3-logstash-安装与配置)
- [4. Kibana 安装与配置](#4-kibana-安装与配置)
- [5. Filebeat 日志采集](#5-filebeat-日志采集)
- [6. 索引管理](#6-索引管理)
- [7. 查询与分析](#7-查询与分析)
- [8. 可视化与仪表板](#8-可视化与仪表板)
- [9. 告警配置](#9-告警配置)
- [10. 性能优化](#10-性能优化)
- [11. 实战案例](#11-实战案例)
- [12. 故障排查](#12-故障排查)
- [13. 学习检查清单](#13-学习检查清单)

## 🎯 学习目标

- 理解 ELK Stack 的架构和组件
- 掌握 Elasticsearch 的安装和配置
- 能够使用 Logstash 进行日志处理
- 掌握 Kibana 的可视化功能
- 了解 Filebeat 的日志采集
- 能够进行日志查询和分析
- 掌握索引管理和性能优化
- 能够搭建完整的日志分析平台

## 1. ELK Stack 简介

### 1.1 什么是 ELK

ELK 是三个开源项目的首字母缩写：Elasticsearch、Logstash 和 Kibana。

**组件说明**：
- **Elasticsearch**：分布式搜索和分析引擎
- **Logstash**：服务器端数据处理管道
- **Kibana**：数据可视化和探索工具
- **Beats**：轻量级数据采集器（Filebeat、Metricbeat 等）

### 1.2 架构图

```
┌─────────────┐
│ Application │
│   Logs      │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Filebeat   │  ← 日志采集
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Logstash   │  ← 日志处理
└──────┬──────┘
       │
       ▼
┌─────────────┐
│Elasticsearch│  ← 存储和搜索
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Kibana    │  ← 可视化
└─────────────┘
```

### 1.3 应用场景

1. **日志聚合**：集中收集和分析应用日志
2. **性能监控**：实时监控系统和应用性能
3. **安全分析**：检测和分析安全事件
4. **业务分析**：分析用户行为和业务指标
5. **故障排查**：快速定位和解决问题

## 2. Elasticsearch 安装与配置

### 2.1 系统要求

```bash
# 最低配置
CPU: 2 核
内存: 4GB
磁盘: 50GB

# 推荐配置（生产环境）
CPU: 8 核+
内存: 32GB+
磁盘: 500GB+ SSD
```

### 2.2 Docker 安装（单节点）

```bash
# 创建网络
docker network create elastic

# 启动 Elasticsearch
docker run -d \
  --name elasticsearch \
  --net elastic \
  -p 9200:9200 \
  -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e "ES_JAVA_OPTS=-Xms2g -Xmx2g" \
  -e "xpack.security.enabled=false" \
  -v es-data:/usr/share/elasticsearch/data \
  docker.elastic.co/elasticsearch/elasticsearch:8.11.0

# 验证安装
curl http://localhost:9200
```

### 2.3 Linux 安装

```bash
# 导入 GPG 密钥
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

# 添加仓库
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

# 安装
sudo apt-get update && sudo apt-get install elasticsearch

# 启动服务
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch

# 查看状态
sudo systemctl status elasticsearch
```

### 2.4 集群配置

```yaml
# /etc/elasticsearch/elasticsearch.yml
# 集群名称
cluster.name: my-cluster

# 节点名称
node.name: node-1

# 节点角色
node.roles: [ master, data, ingest ]

# 网络配置
network.host: 0.0.0.0
http.port: 9200
transport.port: 9300

# 集群发现
discovery.seed_hosts: ["node-1:9300", "node-2:9300", "node-3:9300"]
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]

# 路径配置
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch

# 内存配置
bootstrap.memory_lock: true

# 安全配置
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
```

### 2.5 JVM 配置

```bash
# /etc/elasticsearch/jvm.options
# 堆内存大小（不超过物理内存的 50%，不超过 32GB）
-Xms16g
-Xmx16g

# GC 配置
-XX:+UseG1GC
-XX:G1ReservePercent=25
-XX:InitiatingHeapOccupancyPercent=30

# GC 日志
-Xlog:gc*,gc+age=trace,safepoint:file=/var/log/elasticsearch/gc.log:utctime,pid,tags:filecount=32,filesize=64m
```

## 3. Logstash 安装与配置

### 3.1 Docker 安装

```bash
# 启动 Logstash
docker run -d \
  --name logstash \
  --net elastic \
  -p 5044:5044 \
  -p 9600:9600 \
  -v $(pwd)/logstash.conf:/usr/share/logstash/pipeline/logstash.conf \
  -v $(pwd)/logstash.yml:/usr/share/logstash/config/logstash.yml \
  docker.elastic.co/logstash/logstash:8.11.0
```

### 3.2 Linux 安装

```bash
# 安装
sudo apt-get install logstash

# 启动服务
sudo systemctl enable logstash
sudo systemctl start logstash
```

### 3.3 Pipeline 配置

```ruby
# /etc/logstash/conf.d/logstash.conf
input {
  # Beats 输入
  beats {
    port => 5044
  }
  
  # TCP 输入
  tcp {
    port => 5000
    codec => json
  }
  
  # HTTP 输入
  http {
    port => 8080
  }
  
  # Kafka 输入
  kafka {
    bootstrap_servers => "kafka:9092"
    topics => ["logs"]
    codec => json
  }
}

filter {
  # Grok 解析
  if [type] == "nginx" {
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
    
    date {
      match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
    
    geoip {
      source => "clientip"
    }
  }
  
  # JSON 解析
  if [type] == "app" {
    json {
      source => "message"
    }
  }
  
  # 字段处理
  mutate {
    # 添加字段
    add_field => { "environment" => "production" }
    
    # 删除字段
    remove_field => [ "host", "agent" ]
    
    # 重命名字段
    rename => { "old_field" => "new_field" }
    
    # 类型转换
    convert => { "response_time" => "integer" }
  }
  
  # 条件过滤
  if [level] == "ERROR" {
    mutate {
      add_tag => [ "error" ]
    }
  }
}

output {
  # Elasticsearch 输出
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
    
    # 认证
    user => "elastic"
    password => "changeme"
  }
  
  # 标准输出（调试用）
  stdout {
    codec => rubydebug
  }
  
  # 条件输出
  if [level] == "ERROR" {
    email {
      to => "admin@example.com"
      subject => "Error Alert"
      body => "Error: %{message}"
    }
  }
}
```

### 3.4 常用 Grok 模式

```ruby
# Nginx 访问日志
%{COMBINEDAPACHELOG}

# Java 异常堆栈
%{JAVASTACKTRACEPART}

# Syslog
%{SYSLOGBASE}

# 自定义模式
(?<field_name>pattern)

# 示例：解析自定义日志
%{TIMESTAMP_ISO8601:timestamp} \[%{LOGLEVEL:level}\] %{GREEDYDATA:message}
```

## 4. Kibana 安装与配置

### 4.1 Docker 安装

```bash
# 启动 Kibana
docker run -d \
  --name kibana \
  --net elastic \
  -p 5601:5601 \
  -e "ELASTICSEARCH_HOSTS=http://elasticsearch:9200" \
  docker.elastic.co/kibana/kibana:8.11.0

# 访问 Kibana
# http://localhost:5601
```

### 4.2 Linux 安装

```bash
# 安装
sudo apt-get install kibana

# 启动服务
sudo systemctl enable kibana
sudo systemctl start kibana
```

### 4.3 配置文件

```yaml
# /etc/kibana/kibana.yml
# 服务器配置
server.port: 5601
server.host: "0.0.0.0"
server.name: "kibana"

# Elasticsearch 配置
elasticsearch.hosts: ["http://elasticsearch:9200"]
elasticsearch.username: "kibana_system"
elasticsearch.password: "changeme"

# 日志配置
logging.dest: /var/log/kibana/kibana.log
logging.verbose: false

# 国际化
i18n.locale: "zh-CN"

# 安全配置
xpack.security.enabled: true
xpack.encryptedSavedObjects.encryptionKey: "something_at_least_32_characters"
```

## 5. Filebeat 日志采集

### 5.1 安装 Filebeat

```bash
# Docker
docker run -d \
  --name filebeat \
  --net elastic \
  -v $(pwd)/filebeat.yml:/usr/share/filebeat/filebeat.yml \
  -v /var/log:/var/log:ro \
  docker.elastic.co/beats/filebeat:8.11.0

# Linux
sudo apt-get install filebeat
```

### 5.2 配置文件

```yaml
# /etc/filebeat/filebeat.yml
filebeat.inputs:
# 文件输入
- type: log
  enabled: true
  paths:
    - /var/log/nginx/*.log
  fields:
    type: nginx
    environment: production
  
  # 多行日志
  multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
  multiline.negate: true
  multiline.match: after
  
  # 排除行
  exclude_lines: ['^DEBUG']
  
  # 包含行
  include_lines: ['^ERR', '^WARN']

# 容器日志
- type: container
  paths:
    - '/var/lib/docker/containers/*/*.log'
  processors:
    - add_docker_metadata:
        host: "unix:///var/run/docker.sock"

# 系统日志
- type: syslog
  protocol.tcp:
    host: "0.0.0.0:514"

# 输出到 Logstash
output.logstash:
  hosts: ["logstash:5044"]
  
  # 负载均衡
  loadbalance: true
  
  # 压缩
  compression_level: 3

# 输出到 Elasticsearch
output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  username: "elastic"
  password: "changeme"
  
  # 索引配置
  index: "filebeat-%{[agent.version]}-%{+yyyy.MM.dd}"
  
  # ILM 配置
  ilm.enabled: true
  ilm.rollover_alias: "filebeat"
  ilm.pattern: "{now/d}-000001"

# 处理器
processors:
  # 添加主机信息
  - add_host_metadata:
      when.not.contains.tags: forwarded
  
  # 添加云元数据
  - add_cloud_metadata: ~
  
  # 添加 Docker 元数据
  - add_docker_metadata: ~
  
  # 添加 Kubernetes 元数据
  - add_kubernetes_metadata:
      host: ${NODE_NAME}
      matchers:
      - logs_path:
          logs_path: "/var/log/containers/"

# 日志配置
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644
```

### 5.3 模块配置

```bash
# 启用模块
filebeat modules enable nginx
filebeat modules enable mysql
filebeat modules enable system

# 配置模块
# /etc/filebeat/modules.d/nginx.yml
- module: nginx
  access:
    enabled: true
    var.paths: ["/var/log/nginx/access.log*"]
  
  error:
    enabled: true
    var.paths: ["/var/log/nginx/error.log*"]

# 测试配置
filebeat test config
filebeat test output

# 启动 Filebeat
sudo systemctl start filebeat
```



## 6. 索引管理

### 6.1 索引模板

```json
// 创建索引模板
PUT _index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1,
      "refresh_interval": "30s",
      "index.codec": "best_compression"
    },
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "level": {
          "type": "keyword"
        },
        "message": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "host": {
          "type": "keyword"
        },
        "response_time": {
          "type": "integer"
        }
      }
    }
  },
  "priority": 100
}
```

### 6.2 索引生命周期管理（ILM）

```json
// 创建 ILM 策略
PUT _ilm/policy/logs_policy
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "1d",
            "max_docs": 100000000
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          },
          "set_priority": {
            "priority": 50
          }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "freeze": {},
          "set_priority": {
            "priority": 0
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}

// 应用 ILM 策略到索引模板
PUT _index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "index.lifecycle.name": "logs_policy",
      "index.lifecycle.rollover_alias": "logs"
    }
  }
}
```

### 6.3 索引别名

```json
// 创建别名
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "logs-2024.01.01",
        "alias": "logs-current"
      }
    }
  ]
}

// 切换别名
POST _aliases
{
  "actions": [
    {
      "remove": {
        "index": "logs-2024.01.01",
        "alias": "logs-current"
      }
    },
    {
      "add": {
        "index": "logs-2024.01.02",
        "alias": "logs-current"
      }
    }
  ]
}

// 过滤别名
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "logs-*",
        "alias": "logs-error",
        "filter": {
          "term": {
            "level": "ERROR"
          }
        }
      }
    }
  ]
}
```

### 6.4 索引维护

```bash
# 查看索引
GET _cat/indices?v

# 查看索引设置
GET logs-2024.01.01/_settings

# 更新索引设置
PUT logs-2024.01.01/_settings
{
  "index": {
    "number_of_replicas": 2,
    "refresh_interval": "60s"
  }
}

# 关闭索引
POST logs-2024.01.01/_close

# 打开索引
POST logs-2024.01.01/_open

# 删除索引
DELETE logs-2024.01.01

# 重建索引
POST _reindex
{
  "source": {
    "index": "logs-old"
  },
  "dest": {
    "index": "logs-new"
  }
}

# 强制合并
POST logs-2024.01.01/_forcemerge?max_num_segments=1
```

## 7. 查询与分析

### 7.1 基本查询

```json
// 匹配所有文档
GET logs-*/_search
{
  "query": {
    "match_all": {}
  }
}

// 全文搜索
GET logs-*/_search
{
  "query": {
    "match": {
      "message": "error database"
    }
  }
}

// 精确匹配
GET logs-*/_search
{
  "query": {
    "term": {
      "level": "ERROR"
    }
  }
}

// 多字段匹配
GET logs-*/_search
{
  "query": {
    "multi_match": {
      "query": "connection timeout",
      "fields": ["message", "error"]
    }
  }
}

// 范围查询
GET logs-*/_search
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-1h",
        "lte": "now"
      }
    }
  }
}
```

### 7.2 复合查询

```json
// Bool 查询
GET logs-*/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "message": "error" } }
      ],
      "filter": [
        { "term": { "level": "ERROR" } },
        { "range": { "@timestamp": { "gte": "now-1h" } } }
      ],
      "should": [
        { "term": { "host": "server1" } },
        { "term": { "host": "server2" } }
      ],
      "must_not": [
        { "term": { "status": "ignored" } }
      ],
      "minimum_should_match": 1
    }
  }
}
```

### 7.3 聚合分析

```json
// 统计聚合
GET logs-*/_search
{
  "size": 0,
  "aggs": {
    "error_count": {
      "value_count": {
        "field": "level"
      }
    },
    "avg_response_time": {
      "avg": {
        "field": "response_time"
      }
    },
    "max_response_time": {
      "max": {
        "field": "response_time"
      }
    }
  }
}

// 桶聚合
GET logs-*/_search
{
  "size": 0,
  "aggs": {
    "by_level": {
      "terms": {
        "field": "level",
        "size": 10
      }
    },
    "by_host": {
      "terms": {
        "field": "host",
        "size": 20
      },
      "aggs": {
        "avg_response": {
          "avg": {
            "field": "response_time"
          }
        }
      }
    }
  }
}

// 时间直方图
GET logs-*/_search
{
  "size": 0,
  "aggs": {
    "logs_over_time": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "1h"
      },
      "aggs": {
        "by_level": {
          "terms": {
            "field": "level"
          }
        }
      }
    }
  }
}

// 百分位聚合
GET logs-*/_search
{
  "size": 0,
  "aggs": {
    "response_time_percentiles": {
      "percentiles": {
        "field": "response_time",
        "percents": [50, 95, 99]
      }
    }
  }
}
```

## 8. 可视化与仪表板

### 8.1 创建索引模式

```
1. 打开 Kibana
2. 进入 Management -> Stack Management -> Index Patterns
3. 点击 "Create index pattern"
4. 输入索引模式：logs-*
5. 选择时间字段：@timestamp
6. 点击 "Create index pattern"
```

### 8.2 Discover 探索

```
1. 进入 Discover
2. 选择索引模式：logs-*
3. 设置时间范围：Last 24 hours
4. 添加过滤器：level: ERROR
5. 选择显示字段：@timestamp, level, message, host
6. 保存搜索：Error Logs
```

### 8.3 创建可视化

```
# 1. 折线图 - 日志趋势
Visualization Type: Line
Data:
  Y-axis: Count
  X-axis: Date Histogram (@timestamp, 1 hour)
  Split Series: Terms (level)

# 2. 饼图 - 日志级别分布
Visualization Type: Pie
Data:
  Slice Size: Count
  Split Slices: Terms (level, Top 5)

# 3. 数据表 - Top 错误
Visualization Type: Data Table
Data:
  Metrics: Count
  Buckets: Terms (message.keyword, Top 10)
  
# 4. 热力图 - 响应时间
Visualization Type: Heat Map
Data:
  Y-axis: Terms (host)
  X-axis: Date Histogram (@timestamp, 1 hour)
  Cell: Average (response_time)

# 5. 指标 - 总数
Visualization Type: Metric
Data:
  Metric: Count
  Filter: level: ERROR
```

### 8.4 创建仪表板

```
1. 进入 Dashboard
2. 点击 "Create dashboard"
3. 添加可视化：
   - 日志趋势（折线图）
   - 日志级别分布（饼图）
   - Top 错误（数据表）
   - 响应时间热力图
   - 错误总数（指标）
4. 调整布局和大小
5. 保存仪表板：Application Logs Dashboard
```

## 9. 告警配置

### 9.1 Watcher 告警

```json
// 创建 Watcher
PUT _watcher/watch/error_alert
{
  "trigger": {
    "schedule": {
      "interval": "5m"
    }
  },
  "input": {
    "search": {
      "request": {
        "indices": ["logs-*"],
        "body": {
          "query": {
            "bool": {
              "must": [
                {
                  "term": {
                    "level": "ERROR"
                  }
                },
                {
                  "range": {
                    "@timestamp": {
                      "gte": "now-5m"
                    }
                  }
                }
              ]
            }
          }
        }
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.hits.total": {
        "gt": 10
      }
    }
  },
  "actions": {
    "send_email": {
      "email": {
        "to": "admin@example.com",
        "subject": "Error Alert: {{ctx.payload.hits.total}} errors in last 5 minutes",
        "body": "Found {{ctx.payload.hits.total}} errors in the last 5 minutes."
      }
    },
    "webhook": {
      "webhook": {
        "scheme": "https",
        "host": "hooks.slack.com",
        "port": 443,
        "method": "post",
        "path": "/services/YOUR/WEBHOOK/URL",
        "params": {},
        "headers": {
          "Content-Type": "application/json"
        },
        "body": "{\"text\": \"Error Alert: {{ctx.payload.hits.total}} errors\"}"
      }
    }
  }
}
```

### 9.2 Kibana Alerting

```
1. 进入 Stack Management -> Alerts and Insights -> Rules
2. 点击 "Create rule"
3. 选择规则类型：Elasticsearch query
4. 配置查询：
   - Index: logs-*
   - Query: level: ERROR
   - Time field: @timestamp
   - Threshold: > 10 in 5 minutes
5. 配置动作：
   - Email
   - Slack
   - Webhook
6. 保存规则
```

## 10. 性能优化

### 10.1 Elasticsearch 优化

```yaml
# elasticsearch.yml
# 1. 内存优化
bootstrap.memory_lock: true
indices.memory.index_buffer_size: 30%

# 2. 线程池优化
thread_pool.write.queue_size: 1000
thread_pool.search.queue_size: 1000

# 3. 缓存优化
indices.queries.cache.size: 10%
indices.fielddata.cache.size: 20%

# 4. 刷新间隔
index.refresh_interval: 30s

# 5. 合并策略
index.merge.scheduler.max_thread_count: 1
```

```bash
# 6. 禁用 swap
sudo swapoff -a

# 7. 增加文件描述符
ulimit -n 65535

# 8. 增加虚拟内存
sudo sysctl -w vm.max_map_count=262144
```

### 10.2 索引优化

```json
// 1. 使用批量 API
POST _bulk
{ "index": { "_index": "logs-2024.01.01" } }
{ "@timestamp": "2024-01-01T00:00:00", "message": "log1" }
{ "index": { "_index": "logs-2024.01.01" } }
{ "@timestamp": "2024-01-01T00:00:01", "message": "log2" }

// 2. 优化映射
PUT logs-2024.01.01
{
  "mappings": {
    "properties": {
      "message": {
        "type": "text",
        "index_options": "offsets",  // 加速短语查询
        "norms": false  // 不需要评分时禁用
      },
      "level": {
        "type": "keyword"  // 使用 keyword 而不是 text
      }
    }
  }
}

// 3. 禁用不需要的功能
PUT logs-2024.01.01
{
  "mappings": {
    "_source": {
      "enabled": false  // 不需要原始文档时禁用
    },
    "properties": {
      "field": {
        "type": "text",
        "index": false  // 不需要搜索时禁用索引
      }
    }
  }
}
```

### 10.3 查询优化

```json
// 1. 使用过滤器而不是查询
GET logs-*/_search
{
  "query": {
    "bool": {
      "filter": [  // filter 会被缓存
        { "term": { "level": "ERROR" } },
        { "range": { "@timestamp": { "gte": "now-1h" } } }
      ]
    }
  }
}

// 2. 限制返回字段
GET logs-*/_search
{
  "_source": ["@timestamp", "level", "message"],
  "query": {
    "match_all": {}
  }
}

// 3. 使用 scroll API 处理大量数据
POST logs-*/_search?scroll=1m
{
  "size": 1000,
  "query": {
    "match_all": {}
  }
}

// 4. 使用聚合而不是搜索
GET logs-*/_search
{
  "size": 0,  // 不返回文档
  "aggs": {
    "error_count": {
      "filter": {
        "term": { "level": "ERROR" }
      }
    }
  }
}
```

## 11. 实战案例

### 11.1 应用日志分析

```yaml
# Filebeat 配置
filebeat.inputs:
- type: log
  paths:
    - /var/log/app/*.log
  json.keys_under_root: true
  json.add_error_key: true
  fields:
    app: myapp
    environment: production

# Logstash 配置
filter {
  if [app] == "myapp" {
    # 解析 JSON
    json {
      source => "message"
    }
    
    # 提取字段
    mutate {
      add_field => {
        "request_id" => "%{[request][id]}"
        "user_id" => "%{[user][id]}"
      }
    }
    
    # 计算响应时间
    ruby {
      code => "
        if event.get('[response][time]')
          event.set('response_time_ms', event.get('[response][time]').to_i)
        end
      "
    }
  }
}

# Kibana 查询
GET logs-*/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "app": "myapp" } },
        { "range": { "response_time_ms": { "gte": 1000 } } }
      ]
    }
  },
  "aggs": {
    "slow_endpoints": {
      "terms": {
        "field": "endpoint.keyword",
        "size": 10
      },
      "aggs": {
        "avg_response_time": {
          "avg": {
            "field": "response_time_ms"
          }
        }
      }
    }
  }
}
```

### 11.2 Nginx 日志分析

```yaml
# Filebeat 配置
filebeat.modules:
- module: nginx
  access:
    enabled: true
    var.paths: ["/var/log/nginx/access.log*"]
  error:
    enabled: true
    var.paths: ["/var/log/nginx/error.log*"]

# Kibana 可视化
# 1. 请求趋势
Visualization: Line Chart
Y-axis: Count
X-axis: Date Histogram (@timestamp, 1 hour)

# 2. 状态码分布
Visualization: Pie Chart
Slice Size: Count
Split Slices: Terms (http.response.status_code)

# 3. Top URL
Visualization: Data Table
Metrics: Count
Buckets: Terms (url.original, Top 20)

# 4. 响应时间百分位
Visualization: Line Chart
Y-axis: Percentiles (nginx.access.response_time, 50, 95, 99)
X-axis: Date Histogram (@timestamp, 1 hour)
```

### 11.3 Kubernetes 日志分析

```yaml
# Filebeat DaemonSet
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: filebeat
  template:
    metadata:
      labels:
        app: filebeat
    spec:
      serviceAccountName: filebeat
      containers:
      - name: filebeat
        image: docker.elastic.co/beats/filebeat:8.11.0
        args: [
          "-c", "/etc/filebeat.yml",
          "-e",
        ]
        env:
        - name: ELASTICSEARCH_HOST
          value: elasticsearch
        - name: ELASTICSEARCH_PORT
          value: "9200"
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        volumeMounts:
        - name: config
          mountPath: /etc/filebeat.yml
          subPath: filebeat.yml
        - name: data
          mountPath: /usr/share/filebeat/data
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: varlog
          mountPath: /var/log
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: filebeat-config
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: varlog
        hostPath:
          path: /var/log
      - name: data
        hostPath:
          path: /var/lib/filebeat-data
          type: DirectoryOrCreate

# Filebeat 配置
filebeat.inputs:
- type: container
  paths:
    - /var/log/containers/*.log
  processors:
    - add_kubernetes_metadata:
        host: ${NODE_NAME}
        matchers:
        - logs_path:
            logs_path: "/var/log/containers/"

output.elasticsearch:
  hosts: ['${ELASTICSEARCH_HOST}:${ELASTICSEARCH_PORT}']
  index: "k8s-logs-%{+yyyy.MM.dd}"
```

## 12. 故障排查

### 12.1 常见问题

```bash
# 1. Elasticsearch 无法启动
# 查看日志
tail -f /var/log/elasticsearch/my-cluster.log

# 检查配置
elasticsearch-plugin list
curl -X GET "localhost:9200/_cluster/health?pretty"

# 2. 内存不足
# 查看堆内存使用
GET _cat/nodes?v&h=name,heap.percent,heap.current,heap.max

# 清理缓存
POST _cache/clear

# 3. 磁盘空间不足
# 查看磁盘使用
GET _cat/allocation?v

# 删除旧索引
DELETE logs-2023.*

# 4. 查询慢
# 查看慢查询日志
GET _cat/thread_pool?v&h=name,active,queue,rejected

# 启用慢查询日志
PUT logs-*/_settings
{
  "index.search.slowlog.threshold.query.warn": "10s",
  "index.search.slowlog.threshold.query.info": "5s"
}
```

### 12.2 性能监控

```bash
# 集群状态
GET _cluster/health
GET _cluster/stats

# 节点状态
GET _cat/nodes?v
GET _nodes/stats

# 索引状态
GET _cat/indices?v&s=store.size:desc
GET logs-*/_stats

# 任务状态
GET _cat/tasks?v
GET _tasks?detailed=true&actions=*search*
```

## 13. 学习检查清单

### 基础知识
- [ ] 理解 ELK Stack 的架构和组件
- [ ] 掌握 Elasticsearch 的核心概念
- [ ] 了解 Logstash 的数据处理流程
- [ ] 熟悉 Kibana 的基本功能

### 安装配置
- [ ] 能够安装和配置 Elasticsearch 集群
- [ ] 掌握 Logstash Pipeline 配置
- [ ] 能够配置 Filebeat 采集日志
- [ ] 了解 Kibana 的配置选项

### 数据管理
- [ ] 掌握索引模板和映射
- [ ] 了解索引生命周期管理
- [ ] 能够进行索引维护操作
- [ ] 掌握数据备份和恢复

### 查询分析
- [ ] 掌握基本查询语法
- [ ] 能够使用聚合分析数据
- [ ] 了解查询优化技巧
- [ ] 能够创建复杂查询

### 可视化
- [ ] 能够创建各种可视化图表
- [ ] 掌握仪表板设计
- [ ] 了解告警配置
- [ ] 能够分享和导出报表

### 运维能力
- [ ] 掌握性能优化方法
- [ ] 能够监控集群状态
- [ ] 能够排查常见问题
- [ ] 了解高可用架构

## 📚 参考资源

### 官方文档
- [Elasticsearch 官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Logstash 官方文档](https://www.elastic.co/guide/en/logstash/current/index.html)
- [Kibana 官方文档](https://www.elastic.co/guide/en/kibana/current/index.html)
- [Filebeat 官方文档](https://www.elastic.co/guide/en/beats/filebeat/current/index.html)

### 学习资源
- [Elastic 官方培训](https://www.elastic.co/training/)
- [Elasticsearch 权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)
- [ELK Stack 实战](https://elk-stack.readthedocs.io/)

### 工具
- [Cerebro](https://github.com/lmenezes/cerebro) - Elasticsearch 管理工具
- [ElasticHQ](https://www.elastichq.org/) - 监控和管理工具
- [Curator](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/index.html) - 索引管理工具

### 相关技术
- Kafka
- Redis
- Prometheus
- Grafana

---

> 💡 **提示**：ELK Stack 是日志分析的标准解决方案，掌握 Elasticsearch 的查询和聚合是关键。建议从小规模开始，逐步扩展到生产环境。合理的索引设计和生命周期管理对性能至关重要。
>
> @author erik.zhou

