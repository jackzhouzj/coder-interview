# Zabbix 完整教程

> 企业级开源监控解决方案实战指南
>
> @author erik.zhou

## 📚 目录

- [1. Zabbix 简介](#1-zabbix-简介)
- [2. 安装与配置](#2-安装与配置)
- [3. 核心概念](#3-核心概念)
- [4. 主机监控](#4-主机监控)
- [5. 模板管理](#5-模板管理)
- [6. 触发器与告警](#6-触发器与告警)
- [7. 可视化](#7-可视化)
- [8. 自动发现](#8-自动发现)
- [9. 分布式监控](#9-分布式监控)
- [10. API 使用](#10-api-使用)
- [11. 实战案例](#11-实战案例)
- [12. 性能优化](#12-性能优化)
- [13. 故障排查](#13-故障排查)
- [14. 学习检查清单](#14-学习检查清单)

## 🎯 学习目标

- 理解 Zabbix 的架构和工作原理
- 掌握 Zabbix Server 的安装和配置
- 能够配置主机和监控项
- 掌握模板的创建和使用
- 了解触发器和告警机制
- 能够创建可视化图表和仪表板
- 掌握自动发现和分布式监控
- 能够使用 Zabbix API 进行自动化

## 1. Zabbix 简介

### 1.1 什么是 Zabbix

Zabbix 是一个企业级开源监控解决方案，用于监控网络、服务器、虚拟机、云服务等。

**核心特性**：
- 分布式监控
- 自动发现
- 实时监控
- 灵活的告警机制
- 强大的可视化
- 历史数据存储
- 丰富的 API
- 多种数据采集方式

### 1.2 架构组件

```
┌─────────────────────────────────────┐
│        Zabbix Frontend              │
│         (Web UI)                    │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│        Zabbix Server                │
│  ┌──────────┐  ┌──────────────┐   │
│  │Poller    │  │Trapper       │   │
│  │Alerter   │  │Discovery     │   │
│  └──────────┘  └──────────────┘   │
└──────────────┬──────────────────────┘
               │
    ┌──────────┼──────────┐
    │          │          │
┌───▼───┐  ┌──▼───┐  ┌──▼────┐
│Agent  │  │Proxy │  │SNMP   │
│       │  │      │  │JMX    │
└───────┘  └──────┘  └───────┘
```

**组件说明**：
- **Zabbix Server**：核心组件，负责数据收集、处理和告警
- **Zabbix Agent**：部署在被监控主机上，收集本地数据
- **Zabbix Proxy**：代理服务器，用于分布式监控
- **Zabbix Frontend**：Web 界面，用于配置和查看监控数据
- **Database**：存储配置和历史数据（MySQL/PostgreSQL）

### 1.3 数据采集方式

1. **Zabbix Agent**：主动或被动采集
2. **SNMP**：网络设备监控
3. **IPMI**：硬件监控
4. **JMX**：Java 应用监控
5. **SSH/Telnet**：远程命令执行
6. **HTTP Agent**：Web 服务监控
7. **Trapper**：主动推送数据

## 2. 安装与配置

### 2.1 系统要求

```bash
# 最低配置
CPU: 2 核
内存: 2GB
磁盘: 50GB

# 推荐配置（生产环境）
CPU: 4 核+
内存: 8GB+
磁盘: 200GB+ SSD
```

### 2.2 Docker 安装（快速部署）

```bash
# 创建网络
docker network create zabbix-net

# 启动 MySQL
docker run -d \
  --name mysql-server \
  --network zabbix-net \
  -e MYSQL_DATABASE=zabbix \
  -e MYSQL_USER=zabbix \
  -e MYSQL_PASSWORD=zabbix_pwd \
  -e MYSQL_ROOT_PASSWORD=root_pwd \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0 \
  --character-set-server=utf8mb4 \
  --collation-server=utf8mb4_bin

# 启动 Zabbix Server
docker run -d \
  --name zabbix-server \
  --network zabbix-net \
  -e DB_SERVER_HOST=mysql-server \
  -e MYSQL_DATABASE=zabbix \
  -e MYSQL_USER=zabbix \
  -e MYSQL_PASSWORD=zabbix_pwd \
  -p 10051:10051 \
  zabbix/zabbix-server-mysql:latest

# 启动 Zabbix Web
docker run -d \
  --name zabbix-web \
  --network zabbix-net \
  -e ZBX_SERVER_HOST=zabbix-server \
  -e DB_SERVER_HOST=mysql-server \
  -e MYSQL_DATABASE=zabbix \
  -e MYSQL_USER=zabbix \
  -e MYSQL_PASSWORD=zabbix_pwd \
  -e PHP_TZ=Asia/Shanghai \
  -p 80:8080 \
  zabbix/zabbix-web-nginx-mysql:latest

# 访问 Web 界面
# http://localhost
# 默认用户名：Admin
# 默认密码：zabbix
```

### 2.3 Linux 安装（Ubuntu/Debian）

```bash
# 1. 安装仓库
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update

# 2. 安装 Zabbix Server、Frontend 和 Agent
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts zabbix-agent

# 3. 安装 MySQL
sudo apt install mysql-server

# 4. 创建数据库
mysql -uroot -p
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> set global log_bin_trust_function_creators = 1;
mysql> quit;

# 5. 导入初始数据
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix

# 6. 配置数据库连接
sudo vi /etc/zabbix/zabbix_server.conf
# DBPassword=password

# 7. 配置 Nginx
sudo vi /etc/zabbix/nginx.conf
# listen 80;
# server_name example.com;

# 8. 启动服务
sudo systemctl restart zabbix-server zabbix-agent nginx php8.1-fpm
sudo systemctl enable zabbix-server zabbix-agent nginx php8.1-fpm
```

### 2.4 Zabbix Server 配置

```bash
# /etc/zabbix/zabbix_server.conf
# 监听端口
ListenPort=10051

# 数据库配置
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=password
DBPort=3306

# 性能配置
StartPollers=10
StartPollersUnreachable=5
StartTrappers=5
StartPingers=5
StartDiscoverers=5
StartHTTPPollers=5

# 缓存配置
CacheSize=128M
HistoryCacheSize=64M
HistoryIndexCacheSize=32M
TrendCacheSize=32M
ValueCacheSize=128M

# 超时配置
Timeout=30
TrapperTimeout=300

# 日志配置
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=10
DebugLevel=3

# 告警脚本目录
AlertScriptsPath=/usr/lib/zabbix/alertscripts

# 外部脚本目录
ExternalScripts=/usr/lib/zabbix/externalscripts
```

### 2.5 Zabbix Agent 安装

```bash
# 安装 Agent
sudo apt install zabbix-agent

# 配置 Agent
sudo vi /etc/zabbix/zabbix_agentd.conf

# Server 地址
Server=zabbix-server-ip
ServerActive=zabbix-server-ip:10051

# 主机名
Hostname=web-server-01

# 监听端口
ListenPort=10050

# 日志配置
LogFile=/var/log/zabbix/zabbix_agentd.log

# 启用远程命令
EnableRemoteCommands=1
UnsafeUserParameters=1

# 自定义参数
UserParameter=custom.check,/path/to/script.sh

# 启动 Agent
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent

# 验证连接
zabbix_get -s 127.0.0.1 -k agent.ping
```

## 3. 核心概念

### 3.1 主机（Host）

主机是被监控的实体，可以是物理服务器、虚拟机、网络设备等。

```
主机配置：
- 主机名称
- 可见名称
- 主机组
- 接口（Agent/SNMP/JMX/IPMI）
- 模板
- 宏变量
```

### 3.2 监控项（Item）

监控项是从主机收集的具体指标。

```
监控项类型：
- Zabbix Agent：agent.ping, system.cpu.load
- SNMP：snmp.get, snmp.walk
- Simple Check：icmpping, tcp.port
- Internal：zabbix[process,poller,avg,busy]
- Calculated：计算项
- Dependent：依赖项
- HTTP Agent：http.get, http.post
- Script：自定义脚本
```

### 3.3 触发器（Trigger）

触发器定义了问题的条件，当条件满足时产生告警。

```
触发器表达式：
- last()：最新值
- avg()：平均值
- min()：最小值
- max()：最大值
- sum()：总和
- count()：计数
- nodata()：无数据
- change()：值变化
```

### 3.4 模板（Template）

模板是监控项、触发器、图形等的集合，可以应用到多个主机。

```
模板包含：
- 监控项
- 触发器
- 图形
- 仪表板
- 自动发现规则
- Web 场景
```

### 3.5 动作（Action）

动作定义了当触发器触发时执行的操作。

```
动作类型：
- 发送消息
- 执行远程命令
- 添加/移除主机
- 启用/禁用主机
```

## 4. 主机监控

### 4.1 添加主机

```
1. 进入 Configuration -> Hosts
2. 点击 "Create host"
3. 配置主机信息：
   - Host name: web-server-01
   - Visible name: Web Server 01
   - Groups: Linux servers
   - Interfaces:
     - Type: Agent
     - IP address: 192.168.1.100
     - Port: 10050
4. 关联模板：
   - Templates: Linux by Zabbix agent
5. 点击 "Add"
```

### 4.2 创建监控项

```
1. 进入 Configuration -> Hosts -> Items
2. 点击 "Create item"
3. 配置监控项：
   - Name: CPU Load Average
   - Type: Zabbix agent
   - Key: system.cpu.load[percpu,avg1]
   - Type of information: Numeric (float)
   - Units: %
   - Update interval: 1m
   - History storage period: 7d
   - Trend storage period: 365d
4. 点击 "Add"
```

### 4.3 常用监控项

```bash
# CPU
system.cpu.load[percpu,avg1]        # CPU 负载
system.cpu.util[,idle]              # CPU 空闲率
system.cpu.util[,system]            # CPU 系统使用率

# 内存
vm.memory.size[total]               # 总内存
vm.memory.size[available]           # 可用内存
vm.memory.util[available]           # 内存使用率

# 磁盘
vfs.fs.size[/,total]                # 磁盘总大小
vfs.fs.size[/,used]                 # 磁盘已用
vfs.fs.size[/,pfree]                # 磁盘空闲百分比
vfs.fs.inode[/,pfree]               # inode 空闲百分比

# 网络
net.if.in[eth0]                     # 网络入流量
net.if.out[eth0]                    # 网络出流量
net.tcp.listen[80]                  # TCP 端口监听

# 进程
proc.num[nginx]                     # 进程数量
proc.mem[nginx]                     # 进程内存使用

# 服务
net.tcp.service[http]               # HTTP 服务检查
net.tcp.service.perf[http]          # HTTP 响应时间

# 日志
log[/var/log/messages,error]        # 日志监控
logrt["/var/log/app/*.log",error]   # 日志轮转监控
```

### 4.4 自定义监控项

```bash
# 1. 创建脚本
sudo vi /usr/lib/zabbix/externalscripts/check_mysql.sh
#!/bin/bash
mysql -u monitor -p'password' -e "show status like 'Threads_connected'" | awk 'NR==2 {print $2}'

sudo chmod +x /usr/lib/zabbix/externalscripts/check_mysql.sh

# 2. 配置 Agent
sudo vi /etc/zabbix/zabbix_agentd.conf
UserParameter=mysql.threads,/usr/lib/zabbix/externalscripts/check_mysql.sh

# 3. 重启 Agent
sudo systemctl restart zabbix-agent

# 4. 测试
zabbix_get -s 127.0.0.1 -k mysql.threads

# 5. 在 Web 界面创建监控项
# Key: mysql.threads
# Type: Zabbix agent
```

## 5. 模板管理

### 5.1 创建模板

```
1. 进入 Configuration -> Templates
2. 点击 "Create template"
3. 配置模板：
   - Template name: Template App MySQL
   - Visible name: MySQL Database
   - Groups: Templates/Databases
   - Description: MySQL monitoring template
4. 添加监控项：
   - MySQL Threads Connected
   - MySQL Questions
   - MySQL Slow Queries
   - MySQL Uptime
5. 添加触发器：
   - MySQL is down
   - Too many connections
   - High slow query rate
6. 添加图形：
   - MySQL Performance
   - MySQL Connections
7. 点击 "Add"
```

### 5.2 模板继承

```
1. 创建基础模板：Template OS Linux
2. 创建应用模板：Template App Nginx
3. 配置继承：
   - Template App Nginx
   - Linked templates: Template OS Linux
4. 应用到主机：
   - Host: web-server-01
   - Templates: Template App Nginx
   # 自动继承 Template OS Linux
```

### 5.3 模板宏

```
# 在模板中定义宏
{$MYSQL.HOST} = localhost
{$MYSQL.PORT} = 3306
{$MYSQL.USER} = monitor
{$MYSQL.PASSWORD} = password

# 在监控项中使用宏
mysql.ping[{$MYSQL.HOST},{$MYSQL.PORT}]

# 在主机级别覆盖宏
Host: db-server-01
Macros:
  {$MYSQL.HOST} = 192.168.1.200
  {$MYSQL.PORT} = 3307
```

### 5.4 导入导出模板

```bash
# 导出模板
1. Configuration -> Templates
2. 选择模板
3. 点击 "Export"
4. 选择格式：YAML/XML/JSON
5. 下载文件

# 导入模板
1. Configuration -> Templates
2. 点击 "Import"
3. 选择文件
4. 配置导入选项：
   - Create new
   - Update existing
   - Delete missing
5. 点击 "Import"
```

## 6. 触发器与告警

### 6.1 创建触发器

```
1. Configuration -> Hosts -> Triggers
2. 点击 "Create trigger"
3. 配置触发器：
   - Name: High CPU load on {HOST.NAME}
   - Severity: Warning
   - Expression:
     avg(/Linux server/system.cpu.load[percpu,avg1],5m)>5
   - OK event generation: Expression
   - OK event closes: All problems
   - Description: CPU load is too high
4. 点击 "Add"
```

### 6.2 触发器表达式

```bash
# 基本语法
function(/host/item,parameter)operator value

# 示例
# CPU 负载过高
avg(/web-server/system.cpu.load[percpu,avg1],5m)>5

# 内存使用率过高
last(/web-server/vm.memory.util[available])<20

# 磁盘空间不足
last(/web-server/vfs.fs.size[/,pfree])<10

# 服务不可用
last(/web-server/net.tcp.service[http])=0

# 无数据
nodata(/web-server/agent.ping,5m)=1

# 值变化
change(/web-server/system.hostname)<>0

# 复合条件
avg(/web-server/system.cpu.load,5m)>5 and 
last(/web-server/vm.memory.util)<20

# 时间条件
avg(/web-server/system.cpu.load,5m)>5 and 
time()>093000 and time()<180000
```

### 6.3 配置告警动作

```
1. Configuration -> Actions -> Trigger actions
2. 点击 "Create action"
3. Action 配置：
   - Name: Notify admins
   - Conditions:
     - Trigger severity >= Warning
     - Host group = Linux servers
4. Operations 配置：
   - Send message to: Admin group
   - Send only to: Email
   - Custom message:
     Subject: Problem: {TRIGGER.NAME}
     Message:
       Problem started at {EVENT.TIME} on {EVENT.DATE}
       Problem name: {TRIGGER.NAME}
       Host: {HOST.NAME}
       Severity: {TRIGGER.SEVERITY}
       Original problem ID: {EVENT.ID}
       {TRIGGER.URL}
5. Recovery operations:
   - Send message to: Admin group
   - Custom message:
     Subject: Resolved: {TRIGGER.NAME}
6. 点击 "Add"
```

### 6.4 告警媒介配置

```
# Email 配置
1. Administration -> Media types -> Email
2. 配置 SMTP：
   - SMTP server: smtp.gmail.com
   - SMTP server port: 587
   - SMTP helo: gmail.com
   - SMTP email: your-email@gmail.com
   - Connection security: STARTTLS
   - Authentication: Username and password
   - Username: your-email@gmail.com
   - Password: your-app-password
3. 测试发送
4. 配置用户媒介：
   - Administration -> Users -> Admin -> Media
   - Type: Email
   - Send to: admin@example.com
   - When active: 1-7,00:00-24:00
   - Use if severity: All
```

### 6.5 告警升级

```
1. Configuration -> Actions -> Trigger actions
2. 编辑动作
3. Operations 配置：
   - Step 1 (0-30m):
     Send to: Developer group
     Send only to: Email
   - Step 2 (30m-1h):
     Send to: Team Lead
     Send only to: Email + SMS
   - Step 3 (1h-):
     Send to: Manager
     Send only to: Email + SMS + Phone
```

## 7. 可视化

### 7.1 创建图形

```
1. Configuration -> Hosts -> Graphs
2. 点击 "Create graph"
3. 配置图形：
   - Name: CPU Usage
   - Width: 900
   - Height: 200
   - Graph type: Normal
   - Show legend: Yes
   - Items:
     - system.cpu.util[,user]
     - system.cpu.util[,system]
     - system.cpu.util[,iowait]
4. 点击 "Add"
```

### 7.2 创建仪表板

```
1. Monitoring -> Dashboards
2. 点击 "Create dashboard"
3. 添加小部件：
   - Graph: CPU Usage
   - Graph: Memory Usage
   - Graph: Network Traffic
   - Plain text: System Info
   - Problems: Current Problems
   - Map: Network Topology
4. 调整布局
5. 保存仪表板
```

### 7.3 创建地图

```
1. Monitoring -> Maps
2. 点击 "Create map"
3. 添加元素：
   - Host: web-server-01
   - Host group: Web Servers
   - Trigger: Critical triggers
   - Image: Server icon
4. 添加链接：
   - From: web-server-01
   - To: db-server-01
   - Type: Network link
5. 配置样式：
   - OK: Green
   - Problem: Red
   - Unknown: Gray
6. 保存地图
```

### 7.4 创建屏幕

```
1. Monitoring -> Screens
2. 点击 "Create screen"
3. 配置屏幕：
   - Name: Infrastructure Overview
   - Columns: 2
   - Rows: 3
4. 添加元素：
   - [0,0] Graph: CPU Load
   - [0,1] Graph: Memory Usage
   - [1,0] Graph: Disk I/O
   - [1,1] Graph: Network Traffic
   - [2,0] Problems: Current Problems
   - [2,1] Data overview: Latest data
5. 保存屏幕
```


## 8. 自动发现

### 8.1 网络发现

```
1. Configuration -> Discovery
2. 点击 "Create discovery rule"
3. 配置规则：
   - Name: Local network discovery
   - IP range: 192.168.1.1-254
   - Update interval: 1h
   - Checks:
     - ICMP ping
     - Zabbix agent "system.uname"
     - HTTP check on port 80
     - HTTPS check on port 443
4. 配置动作：
   - Configuration -> Actions -> Discovery actions
   - Create action:
     - Name: Auto-register Linux servers
     - Conditions:
       - Service type = Zabbix agent
       - Received value contains Linux
     - Operations:
       - Add host
       - Link to template: Linux by Zabbix agent
       - Add to host group: Discovered hosts
5. 点击 "Add"
```

### 8.2 主动注册

```bash
# 1. 配置 Agent 主动注册
sudo vi /etc/zabbix/zabbix_agentd.conf

# Server 地址
ServerActive=zabbix-server:10051

# 主机元数据
HostMetadata=linux mysql webserver
# 或使用脚本
HostMetadataItem=system.uname

# 2. 配置自动注册动作
# Configuration -> Actions -> Autoregistration actions
# Create action:
#   Name: Auto-register by metadata
#   Conditions:
#     - Host metadata contains mysql
#   Operations:
#     - Add host
#     - Link to template: Template App MySQL
#     - Add to host group: MySQL servers
#     - Set host inventory mode: Automatic

# 3. 重启 Agent
sudo systemctl restart zabbix-agent
```

### 8.3 低级发现（LLD）

```bash
# 1. 创建发现规则脚本
sudo vi /usr/lib/zabbix/externalscripts/discover_disks.sh
#!/bin/bash
# 发现所有磁盘分区
echo '{"data":['
first=1
for disk in $(df -h | awk 'NR>1 {print $6}'); do
    if [ $first -eq 0 ]; then
        echo ','
    fi
    echo -n "    {\"{#FSNAME}\":\"$disk\"}"
    first=0
done
echo
echo ']}'

sudo chmod +x /usr/lib/zabbix/externalscripts/discover_disks.sh

# 2. 配置 Agent
sudo vi /etc/zabbix/zabbix_agentd.conf
UserParameter=custom.disk.discovery,/usr/lib/zabbix/externalscripts/discover_disks.sh
UserParameter=custom.disk.size[*],df -h $1 | awk 'NR==2 {print $$2}'
UserParameter=custom.disk.used[*],df -h $1 | awk 'NR==2 {print $$3}'

# 3. 创建发现规则
# Configuration -> Templates -> Create template
# Template name: Template Custom Disk Discovery
# Discovery rules -> Create discovery rule:
#   Name: Disk discovery
#   Type: Zabbix agent
#   Key: custom.disk.discovery
#   Update interval: 1h
#
# Item prototypes:
#   Name: Disk {#FSNAME} total size
#   Key: custom.disk.size[{#FSNAME}]
#
#   Name: Disk {#FSNAME} used space
#   Key: custom.disk.used[{#FSNAME}]
#
# Trigger prototypes:
#   Name: Disk {#FSNAME} is full
#   Expression: last(/Template/custom.disk.used[{#FSNAME}])>90
```

### 8.4 Docker 容器发现

```bash
# 1. 创建容器发现脚本
sudo vi /usr/lib/zabbix/externalscripts/discover_containers.sh
#!/bin/bash
echo '{"data":['
first=1
for container in $(docker ps --format "{{.Names}}"); do
    if [ $first -eq 0 ]; then
        echo ','
    fi
    id=$(docker ps -qf "name=$container")
    echo -n "    {\"{#CONTAINER_NAME}\":\"$container\",\"{#CONTAINER_ID}\":\"$id\"}"
    first=0
done
echo
echo ']}'

sudo chmod +x /usr/lib/zabbix/externalscripts/discover_containers.sh

# 2. 配置监控项
UserParameter=docker.discovery,/usr/lib/zabbix/externalscripts/discover_containers.sh
UserParameter=docker.cpu[*],docker stats --no-stream --format "{{.CPUPerc}}" $1 | sed 's/%//'
UserParameter=docker.mem[*],docker stats --no-stream --format "{{.MemPerc}}" $1 | sed 's/%//'
UserParameter=docker.status[*],docker inspect -f '{{.State.Status}}' $1

# 3. 创建 LLD 规则
# Discovery rule:
#   Key: docker.discovery
# Item prototypes:
#   docker.cpu[{#CONTAINER_ID}]
#   docker.mem[{#CONTAINER_ID}]
#   docker.status[{#CONTAINER_ID}]
```

## 9. 分布式监控

### 9.1 Zabbix Proxy 架构

```
┌─────────────────────────────────────┐
│        Zabbix Server                │
│         (Central)                   │
└──────────────┬──────────────────────┘
               │
    ┌──────────┼──────────┐
    │          │          │
┌───▼───┐  ┌──▼───┐  ┌──▼────┐
│Proxy  │  │Proxy │  │Proxy  │
│ IDC-A │  │ IDC-B│  │Cloud  │
└───┬───┘  └──┬───┘  └───┬───┘
    │         │          │
  Agents    Agents     Agents
```

### 9.2 安装 Zabbix Proxy

```bash
# 1. 安装 Proxy
sudo apt install zabbix-proxy-mysql zabbix-sql-scripts

# 2. 创建数据库
mysql -uroot -p
mysql> create database zabbix_proxy character set utf8mb4 collate utf8mb4_bin;
mysql> create user 'zabbix_proxy'@'localhost' identified by 'password';
mysql> grant all privileges on zabbix_proxy.* to 'zabbix_proxy'@'localhost';
mysql> quit;

# 3. 导入数据
zcat /usr/share/zabbix-sql-scripts/mysql/proxy.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix_proxy -p zabbix_proxy

# 4. 配置 Proxy
sudo vi /etc/zabbix/zabbix_proxy.conf

# Proxy 模式
ProxyMode=0  # 0=主动模式, 1=被动模式

# Server 地址
Server=zabbix-server-ip

# Proxy 名称（必须与 Server 上配置的一致）
Hostname=Proxy-IDC-A

# 数据库配置
DBHost=localhost
DBName=zabbix_proxy
DBUser=zabbix_proxy
DBPassword=password

# 本地缓存时间
ProxyLocalBuffer=24
ProxyOfflineBuffer=720

# 数据上传间隔
ConfigFrequency=3600
DataSenderFrequency=1

# 5. 启动 Proxy
sudo systemctl restart zabbix-proxy
sudo systemctl enable zabbix-proxy
```

### 9.3 配置 Proxy

```
1. Administration -> Proxies
2. 点击 "Create proxy"
3. 配置 Proxy：
   - Proxy name: Proxy-IDC-A
   - Proxy mode: Active
   - Proxy address: (主动模式不需要)
   - Description: Proxy for IDC-A
4. 点击 "Add"

# 将主机分配给 Proxy
1. Configuration -> Hosts
2. 选择主机
3. 修改 "Monitored by proxy": Proxy-IDC-A
4. 保存
```

### 9.4 Proxy 监控

```bash
# 监控 Proxy 状态
# Configuration -> Hosts -> Zabbix server
# Items:
zabbix[proxy,Proxy-IDC-A,lastaccess]     # 最后访问时间
zabbix[proxy,Proxy-IDC-A,delay]          # 数据延迟
zabbix[proxy_history]                     # 历史数据队列

# 触发器
# Proxy is unreachable
nodata(/Zabbix server/zabbix[proxy,Proxy-IDC-A,lastaccess],10m)=1

# Proxy data delay is too high
last(/Zabbix server/zabbix[proxy,Proxy-IDC-A,delay])>300
```

## 10. API 使用

### 10.1 API 认证

```python
import requests
import json

# Zabbix API 地址
ZABBIX_URL = "http://zabbix-server/api_jsonrpc.php"
ZABBIX_USER = "Admin"
ZABBIX_PASSWORD = "zabbix"

# 获取认证 token
def get_auth_token():
    payload = {
        "jsonrpc": "2.0",
        "method": "user.login",
        "params": {
            "username": ZABBIX_USER,
            "password": ZABBIX_PASSWORD
        },
        "id": 1
    }
    
    response = requests.post(ZABBIX_URL, json=payload)
    result = response.json()
    
    if 'result' in result:
        return result['result']
    else:
        raise Exception(f"Authentication failed: {result}")

# 使用 token
auth_token = get_auth_token()
print(f"Auth token: {auth_token}")
```

### 10.2 主机管理

```python
# 获取主机列表
def get_hosts(auth_token):
    payload = {
        "jsonrpc": "2.0",
        "method": "host.get",
        "params": {
            "output": ["hostid", "host", "name", "status"],
            "selectInterfaces": ["interfaceid", "ip"],
            "selectGroups": ["groupid", "name"]
        },
        "auth": auth_token,
        "id": 2
    }
    
    response = requests.post(ZABBIX_URL, json=payload)
    return response.json()['result']

# 创建主机
def create_host(auth_token, hostname, ip, groupid, templateid):
    payload = {
        "jsonrpc": "2.0",
        "method": "host.create",
        "params": {
            "host": hostname,
            "name": hostname,
            "interfaces": [
                {
                    "type": 1,  # Agent
                    "main": 1,
                    "useip": 1,
                    "ip": ip,
                    "dns": "",
                    "port": "10050"
                }
            ],
            "groups": [{"groupid": groupid}],
            "templates": [{"templateid": templateid}]
        },
        "auth": auth_token,
        "id": 3
    }
    
    response = requests.post(ZABBIX_URL, json=payload)
    return response.json()

# 删除主机
def delete_host(auth_token, hostid):
    payload = {
        "jsonrpc": "2.0",
        "method": "host.delete",
        "params": [hostid],
        "auth": auth_token,
        "id": 4
    }
    
    response = requests.post(ZABBIX_URL, json=payload)
    return response.json()

# 更新主机
def update_host(auth_token, hostid, status):
    payload = {
        "jsonrpc": "2.0",
        "method": "host.update",
        "params": {
            "hostid": hostid,
            "status": status  # 0=启用, 1=禁用
        },
        "auth": auth_token,
        "id": 5
    }
    
    response = requests.post(ZABBIX_URL, json=payload)
    return response.json()
```

### 10.3 监控项管理

```python
# 获取监控项
def get_items(auth_token, hostid):
    payload = {
        "jsonrpc": "2.0",
        "method": "item.get",
        "params": {
            "output": ["itemid", "name", "key_", "lastvalue"],
            "hostids": hostid,
            "sortfield": "name"
        },
        "auth": auth_token,
        "id": 6
    }
    
    response = requests.post(ZABBIX_URL, json=payload)
    return response.json()['result']

# 创建监控项
def create_item(auth_token, hostid, name, key, value_type):
    payload = {
        "jsonrpc": "2.0",
        "method": "item.create",
        "params": {
            "name": name,
            "key_": key,
            "hostid": hostid,
            "type": 0,  # Zabbix agent
            "value_type": value_type,  # 0=float, 1=char, 3=numeric
            "delay": "60s",
            "history": "7d",
            "trends": "365d"
        },
        "auth": auth_token,
        "id": 7
    }
    
    response = requests.post(ZABBIX_URL, json=payload)
    return response.json()

# 获取监控项历史数据
def get_history(auth_token, itemid, time_from, time_till):
    payload = {
        "jsonrpc": "2.0",
        "method": "history.get",
        "params": {
            "output": "extend",
            "itemids": itemid,
            "time_from": time_from,
            "time_till": time_till,
            "sortfield": "clock",
            "sortorder": "DESC",
            "limit": 100
        },
        "auth": auth_token,
        "id": 8
    }
    
    response = requests.post(ZABBIX_URL, json=payload)
    return response.json()['result']
```

### 10.4 告警管理

```python
# 获取问题列表
def get_problems(auth_token):
    payload = {
        "jsonrpc": "2.0",
        "method": "problem.get",
        "params": {
            "output": "extend",
            "selectAcknowledges": "extend",
            "recent": "true",
            "sortfield": ["eventid"],
            "sortorder": "DESC"
        },
        "auth": auth_token,
        "id": 9
    }
    
    response = requests.post(ZABBIX_URL, json=payload)
    return response.json()['result']

# 确认告警
def acknowledge_event(auth_token, eventid, message):
    payload = {
        "jsonrpc": "2.0",
        "method": "event.acknowledge",
        "params": {
            "eventids": eventid,
            "action": 6,  # 1=close, 2=acknowledge, 4=message, 6=acknowledge+message
            "message": message
        },
        "auth": auth_token,
        "id": 10
    }
    
    response = requests.post(ZABBIX_URL, json=payload)
    return response.json()

# 获取触发器
def get_triggers(auth_token, hostid):
    payload = {
        "jsonrpc": "2.0",
        "method": "trigger.get",
        "params": {
            "output": "extend",
            "hostids": hostid,
            "selectFunctions": "extend",
            "selectItems": ["itemid", "key_"]
        },
        "auth": auth_token,
        "id": 11
    }
    
    response = requests.post(ZABBIX_URL, json=payload)
    return response.json()['result']
```

### 10.5 批量操作示例

```python
#!/usr/bin/env python3
"""
批量添加主机到 Zabbix
"""
import csv
import sys

def batch_add_hosts(auth_token, csv_file):
    """从 CSV 文件批量添加主机"""
    with open(csv_file, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            hostname = row['hostname']
            ip = row['ip']
            groupid = row['groupid']
            templateid = row['templateid']
            
            try:
                result = create_host(auth_token, hostname, ip, groupid, templateid)
                if 'result' in result:
                    print(f"✓ Created host: {hostname}")
                else:
                    print(f"✗ Failed to create {hostname}: {result['error']}")
            except Exception as e:
                print(f"✗ Error creating {hostname}: {e}")

# CSV 格式示例
# hostname,ip,groupid,templateid
# web-01,192.168.1.101,2,10001
# web-02,192.168.1.102,2,10001
# db-01,192.168.1.201,3,10002

if __name__ == "__main__":
    auth_token = get_auth_token()
    batch_add_hosts(auth_token, "hosts.csv")
```

## 11. 实战案例

### 11.1 监控 Nginx

```bash
# 1. 启用 Nginx status 模块
sudo vi /etc/nginx/sites-available/default

server {
    listen 80;
    
    location /nginx_status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }
}

sudo nginx -t
sudo systemctl reload nginx

# 2. 创建监控脚本
sudo vi /usr/lib/zabbix/externalscripts/nginx_status.sh
#!/bin/bash
METRIC=$1
curl -s http://127.0.0.1/nginx_status | awk -v metric="$METRIC" '
    /Active connections/ {active=$3}
    /Reading/ {reading=$2; writing=$4; waiting=$6}
    /^[[:space:]]*[0-9]+ [0-9]+ [0-9]+/ {accepts=$1; handled=$2; requests=$3}
    END {
        if (metric == "active") print active
        if (metric == "reading") print reading
        if (metric == "writing") print writing
        if (metric == "waiting") print waiting
        if (metric == "accepts") print accepts
        if (metric == "handled") print handled
        if (metric == "requests") print requests
    }
'

sudo chmod +x /usr/lib/zabbix/externalscripts/nginx_status.sh

# 3. 配置 Agent
sudo vi /etc/zabbix/zabbix_agentd.conf
UserParameter=nginx.status[*],/usr/lib/zabbix/externalscripts/nginx_status.sh $1

sudo systemctl restart zabbix-agent

# 4. 创建模板
# Template App Nginx
# Items:
#   nginx.status[active]    - Active connections
#   nginx.status[reading]   - Reading
#   nginx.status[writing]   - Writing
#   nginx.status[waiting]   - Waiting
#   nginx.status[accepts]   - Accepts
#   nginx.status[handled]   - Handled
#   nginx.status[requests]  - Requests
#
# Triggers:
#   Nginx is down
#   Too many active connections
#   High waiting connections
```

### 11.2 监控 MySQL

```bash
# 1. 创建监控用户
mysql -uroot -p
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'password';
GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;

# 2. 创建监控脚本
sudo vi /usr/lib/zabbix/externalscripts/mysql_stats.sh
#!/bin/bash
METRIC=$1
MYSQL_USER="zabbix"
MYSQL_PASS="password"

case $METRIC in
    threads_connected)
        mysql -u$MYSQL_USER -p$MYSQL_PASS -e "SHOW STATUS LIKE 'Threads_connected'" | awk 'NR==2 {print $2}'
        ;;
    threads_running)
        mysql -u$MYSQL_USER -p$MYSQL_PASS -e "SHOW STATUS LIKE 'Threads_running'" | awk 'NR==2 {print $2}'
        ;;
    questions)
        mysql -u$MYSQL_USER -p$MYSQL_PASS -e "SHOW STATUS LIKE 'Questions'" | awk 'NR==2 {print $2}'
        ;;
    slow_queries)
        mysql -u$MYSQL_USER -p$MYSQL_PASS -e "SHOW STATUS LIKE 'Slow_queries'" | awk 'NR==2 {print $2}'
        ;;
    uptime)
        mysql -u$MYSQL_USER -p$MYSQL_PASS -e "SHOW STATUS LIKE 'Uptime'" | awk 'NR==2 {print $2}'
        ;;
    ping)
        mysqladmin -u$MYSQL_USER -p$MYSQL_PASS ping | grep -c alive
        ;;
esac

sudo chmod +x /usr/lib/zabbix/externalscripts/mysql_stats.sh

# 3. 配置 Agent
UserParameter=mysql.stats[*],/usr/lib/zabbix/externalscripts/mysql_stats.sh $1

# 4. 创建模板
# Template App MySQL
# Items:
#   mysql.stats[ping]               - MySQL is alive
#   mysql.stats[threads_connected]  - Threads connected
#   mysql.stats[threads_running]    - Threads running
#   mysql.stats[questions]          - Questions
#   mysql.stats[slow_queries]       - Slow queries
#   mysql.stats[uptime]             - Uptime
#
# Triggers:
#   MySQL is down
#   Too many connections
#   High slow query rate
```

### 11.3 监控 Redis

```bash
# 1. 创建监控脚本
sudo vi /usr/lib/zabbix/externalscripts/redis_stats.sh
#!/bin/bash
METRIC=$1
REDIS_CLI="redis-cli"
REDIS_HOST="127.0.0.1"
REDIS_PORT="6379"

case $METRIC in
    ping)
        $REDIS_CLI -h $REDIS_HOST -p $REDIS_PORT ping | grep -c PONG
        ;;
    connected_clients)
        $REDIS_CLI -h $REDIS_HOST -p $REDIS_PORT info clients | grep connected_clients | cut -d: -f2 | tr -d '\r'
        ;;
    used_memory)
        $REDIS_CLI -h $REDIS_HOST -p $REDIS_PORT info memory | grep used_memory: | cut -d: -f2 | tr -d '\r'
        ;;
    used_memory_rss)
        $REDIS_CLI -h $REDIS_HOST -p $REDIS_PORT info memory | grep used_memory_rss: | cut -d: -f2 | tr -d '\r'
        ;;
    keys)
        $REDIS_CLI -h $REDIS_HOST -p $REDIS_PORT dbsize | cut -d: -f2 | tr -d '\r'
        ;;
    evicted_keys)
        $REDIS_CLI -h $REDIS_HOST -p $REDIS_PORT info stats | grep evicted_keys | cut -d: -f2 | tr -d '\r'
        ;;
    keyspace_hits)
        $REDIS_CLI -h $REDIS_HOST -p $REDIS_PORT info stats | grep keyspace_hits | cut -d: -f2 | tr -d '\r'
        ;;
    keyspace_misses)
        $REDIS_CLI -h $REDIS_HOST -p $REDIS_PORT info stats | grep keyspace_misses | cut -d: -f2 | tr -d '\r'
        ;;
esac

sudo chmod +x /usr/lib/zabbix/externalscripts/redis_stats.sh

# 2. 配置 Agent
UserParameter=redis.stats[*],/usr/lib/zabbix/externalscripts/redis_stats.sh $1

# 3. 创建模板
# Template App Redis
# Items:
#   redis.stats[ping]              - Redis is alive
#   redis.stats[connected_clients] - Connected clients
#   redis.stats[used_memory]       - Used memory
#   redis.stats[keys]              - Total keys
#   redis.stats[evicted_keys]      - Evicted keys
#   redis.stats[keyspace_hits]     - Keyspace hits
#   redis.stats[keyspace_misses]   - Keyspace misses
#
# Calculated items:
#   Hit rate = keyspace_hits / (keyspace_hits + keyspace_misses) * 100
```

### 11.4 监控 Kubernetes

```bash
# 1. 部署 Zabbix Agent DaemonSet
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: zabbix-agent
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: zabbix-agent
  template:
    metadata:
      labels:
        app: zabbix-agent
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - name: zabbix-agent
        image: zabbix/zabbix-agent2:latest
        env:
        - name: ZBX_SERVER_HOST
          value: "zabbix-server"
        - name: ZBX_HOSTNAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: ZBX_METADATA
          value: "kubernetes node"
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
        - name: docker
          mountPath: /var/run/docker.sock
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      - name: docker
        hostPath:
          path: /var/run/docker.sock
EOF

# 2. 创建 Kubernetes 监控脚本
sudo vi /usr/lib/zabbix/externalscripts/k8s_stats.sh
#!/bin/bash
METRIC=$1

case $METRIC in
    nodes_ready)
        kubectl get nodes --no-headers | grep -c Ready
        ;;
    nodes_total)
        kubectl get nodes --no-headers | wc -l
        ;;
    pods_running)
        kubectl get pods --all-namespaces --no-headers | grep -c Running
        ;;
    pods_pending)
        kubectl get pods --all-namespaces --no-headers | grep -c Pending
        ;;
    pods_failed)
        kubectl get pods --all-namespaces --no-headers | grep -c Failed
        ;;
    deployments_total)
        kubectl get deployments --all-namespaces --no-headers | wc -l
        ;;
    services_total)
        kubectl get services --all-namespaces --no-headers | wc -l
        ;;
esac

# 3. 配置监控项
UserParameter=k8s.stats[*],/usr/lib/zabbix/externalscripts/k8s_stats.sh $1
```

### 11.5 监控 Docker

```bash
# 1. 创建 Docker 监控脚本
sudo vi /usr/lib/zabbix/externalscripts/docker_stats.sh
#!/bin/bash
METRIC=$1
CONTAINER=$2

case $METRIC in
    containers_running)
        docker ps -q | wc -l
        ;;
    containers_total)
        docker ps -aq | wc -l
        ;;
    images_total)
        docker images -q | wc -l
        ;;
    cpu_usage)
        docker stats --no-stream --format "{{.CPUPerc}}" $CONTAINER | sed 's/%//'
        ;;
    mem_usage)
        docker stats --no-stream --format "{{.MemPerc}}" $CONTAINER | sed 's/%//'
        ;;
    mem_limit)
        docker stats --no-stream --format "{{.MemUsage}}" $CONTAINER | awk '{print $3}' | sed 's/[^0-9.]//g'
        ;;
    net_input)
        docker stats --no-stream --format "{{.NetIO}}" $CONTAINER | awk '{print $1}' | sed 's/[^0-9.]//g'
        ;;
    net_output)
        docker stats --no-stream --format "{{.NetIO}}" $CONTAINER | awk '{print $3}' | sed 's/[^0-9.]//g'
        ;;
    status)
        docker inspect -f '{{.State.Status}}' $CONTAINER
        ;;
esac

sudo chmod +x /usr/lib/zabbix/externalscripts/docker_stats.sh

# 2. 配置 Agent
UserParameter=docker.stats[*],/usr/lib/zabbix/externalscripts/docker_stats.sh $1 $2
UserParameter=docker.discovery,/usr/lib/zabbix/externalscripts/discover_containers.sh
```

### 11.6 日志监控

```bash
# 1. 监控错误日志
# Configuration -> Hosts -> Items
# Create item:
#   Name: Error logs in /var/log/app.log
#   Type: Zabbix agent (active)
#   Key: log[/var/log/app.log,ERROR,,,skip]
#   Type of information: Log
#   Update interval: 1m

# 2. 监控日志轮转
# Key: logrt["/var/log/app/*.log",ERROR,,,skip]

# 3. 创建触发器
# Name: Too many errors in application log
# Expression:
#   count(/Host/log[/var/log/app.log,ERROR],5m)>10

# 4. 监控特定模式
# Key: log[/var/log/nginx/access.log,"HTTP/1.1\" 5[0-9]{2}",,,skip]
# 监控 HTTP 5xx 错误

# 5. 日志分析脚本
sudo vi /usr/lib/zabbix/externalscripts/analyze_log.sh
#!/bin/bash
LOG_FILE=$1
PATTERN=$2
TIME_RANGE=$3  # 分钟

# 统计最近 N 分钟内匹配的行数
find $LOG_FILE -mmin -$TIME_RANGE -exec grep -c "$PATTERN" {} \;

# 配置
UserParameter=log.analyze[*],/usr/lib/zabbix/externalscripts/analyze_log.sh $1 $2 $3
```

## 12. 性能优化

### 12.1 数据库优化

```sql
-- 1. 清理历史数据
-- 配置 Housekeeper
-- Administration -> General -> Housekeeping
-- Enable internal housekeeping: Yes
-- Override item history period: 7d
-- Override item trend period: 365d

-- 2. 分区表（MySQL）
-- 对 history 和 trends 表进行分区
ALTER TABLE history_uint PARTITION BY RANGE (clock) (
    PARTITION p2024_01 VALUES LESS THAN (UNIX_TIMESTAMP('2024-02-01 00:00:00')),
    PARTITION p2024_02 VALUES LESS THAN (UNIX_TIMESTAMP('2024-03-01 00:00:00')),
    PARTITION p2024_03 VALUES LESS THAN (UNIX_TIMESTAMP('2024-04-01 00:00:00')),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- 3. 索引优化
CREATE INDEX idx_history_clock ON history (clock);
CREATE INDEX idx_trends_clock ON trends (clock);

-- 4. 数据库配置优化
-- /etc/mysql/my.cnf
[mysqld]
innodb_buffer_pool_size = 4G
innodb_log_file_size = 512M
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT
max_connections = 500
query_cache_size = 0
query_cache_type = 0
```

### 12.2 Zabbix Server 优化

```bash
# /etc/zabbix/zabbix_server.conf

# 1. 进程数优化
StartPollers=20              # 轮询进程（根据监控项数量调整）
StartPollersUnreachable=10   # 不可达主机轮询进程
StartTrappers=10             # Trapper 进程
StartPingers=10              # ICMP ping 进程
StartDiscoverers=5           # 自动发现进程
StartHTTPPollers=5           # HTTP 轮询进程
StartPreprocessors=5         # 预处理进程
StartTimers=2                # 定时器进程
StartEscalators=2            # 告警升级进程
StartAlerters=5              # 告警发送进程

# 2. 缓存优化
CacheSize=256M               # 配置缓存
HistoryCacheSize=128M        # 历史数据缓存
HistoryIndexCacheSize=64M    # 历史索引缓存
TrendCacheSize=64M           # 趋势数据缓存
ValueCacheSize=256M          # 值缓存

# 3. 超时优化
Timeout=30                   # 超时时间
TrapperTimeout=300           # Trapper 超时
UnreachablePeriod=120        # 不可达周期
UnavailableDelay=60          # 不可用延迟

# 4. 日志优化
LogSlowQueries=3000          # 记录慢查询（毫秒）
```

### 12.3 监控项优化

```
1. 减少不必要的监控项
   - 删除未使用的监控项
   - 禁用不重要的监控项
   - 合并相似的监控项

2. 调整采集频率
   - 静态数据：5m-1h
   - 动态数据：1m-5m
   - 关键指标：30s-1m

3. 优化历史数据保留
   - 重要数据：30d-90d
   - 一般数据：7d-30d
   - 临时数据：1d-7d

4. 使用依赖项
   - 减少重复采集
   - 通过计算项派生数据
```

### 12.4 网络优化

```bash
# 1. 启用数据压缩
# Agent 配置
sudo vi /etc/zabbix/zabbix_agentd.conf
TLSConnect=psk
TLSAccept=psk
TLSPSKIdentity=PSK001
TLSPSKFile=/etc/zabbix/zabbix_agentd.psk

# 生成 PSK
openssl rand -hex 32 > /etc/zabbix/zabbix_agentd.psk

# 2. 使用 Proxy 分流
# 远程站点使用 Proxy
# 减少 Server 负载

# 3. 批量数据发送
# Agent 配置
BufferSend=5                 # 批量发送间隔
BufferSize=100               # 缓冲区大小
```

### 12.5 性能监控

```bash
# 1. 监控 Zabbix 内部指标
# Configuration -> Hosts -> Zabbix server
# Items:
zabbix[process,poller,avg,busy]           # Poller 繁忙度
zabbix[process,trapper,avg,busy]          # Trapper 繁忙度
zabbix[wcache,values]                     # 写缓存值数量
zabbix[wcache,history,pfree]              # 历史缓存空闲百分比
zabbix[queue,10m]                         # 10分钟队列长度
zabbix[rcache,buffer,pfree]               # 配置缓存空闲百分比

# 2. 数据库性能监控
# 慢查询
SHOW FULL PROCESSLIST;
SELECT * FROM information_schema.processlist WHERE time > 5;

# 表大小
SELECT 
    table_name,
    ROUND(((data_length + index_length) / 1024 / 1024), 2) AS size_mb
FROM information_schema.tables
WHERE table_schema = 'zabbix'
ORDER BY size_mb DESC;
```

## 13. 故障排查

### 13.1 常见问题

```bash
# 1. Agent 无法连接
# 检查 Agent 状态
sudo systemctl status zabbix-agent

# 检查端口
netstat -tlnp | grep 10050

# 测试连接
zabbix_get -s 127.0.0.1 -k agent.ping

# 检查防火墙
sudo firewall-cmd --list-all
sudo firewall-cmd --add-port=10050/tcp --permanent
sudo firewall-cmd --reload

# 检查 SELinux
sudo setenforce 0
sudo vi /etc/selinux/config
SELINUX=disabled

# 2. Server 无法启动
# 查看日志
sudo tail -f /var/log/zabbix/zabbix_server.log

# 检查数据库连接
mysql -uzabbix -p -h localhost zabbix

# 检查配置文件
sudo zabbix_server -c /etc/zabbix/zabbix_server.conf -t

# 3. 数据采集延迟
# 检查队列
# Administration -> Queue
# 查看延迟的监控项

# 增加 Poller 进程
sudo vi /etc/zabbix/zabbix_server.conf
StartPollers=30

# 4. 数据库性能问题
# 检查慢查询
sudo vi /etc/mysql/my.cnf
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2

# 分析慢查询
mysqldumpslow -s t -t 10 /var/log/mysql/slow.log
```

### 13.2 日志分析

```bash
# 1. Server 日志
sudo tail -f /var/log/zabbix/zabbix_server.log

# 常见错误
# "database is down"
# 解决：检查数据库连接

# "cannot send list of active checks"
# 解决：检查 Agent 配置

# "timeout while executing operation"
# 解决：增加 Timeout 参数

# 2. Agent 日志
sudo tail -f /var/log/zabbix/zabbix_agentd.log

# 常见错误
# "cannot connect to server"
# 解决：检查 Server 地址和防火墙

# "item not supported"
# 解决：检查监控项 Key 是否正确

# 3. Web 日志
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/php-fpm/error.log
```

### 13.3 调试技巧

```bash
# 1. 启用调试模式
sudo vi /etc/zabbix/zabbix_server.conf
DebugLevel=4  # 0-5，5为最详细

sudo systemctl restart zabbix-server

# 2. 测试监控项
zabbix_get -s <host> -k <key>
zabbix_get -s 192.168.1.100 -k system.cpu.load[percpu,avg1]

# 3. 测试 Agent 配置
zabbix_agentd -t <key>
zabbix_agentd -t system.cpu.load[percpu,avg1]

# 4. 检查 Trapper 数据
zabbix_sender -z <server> -s <host> -k <key> -o <value>
zabbix_sender -z 192.168.1.1 -s web-01 -k test.key -o 100

# 5. 数据库查询
# 查看主机
SELECT hostid, host, name, status FROM hosts;

# 查看监控项
SELECT itemid, hostid, name, key_, status FROM items WHERE hostid = 10084;

# 查看最新数据
SELECT * FROM history_uint WHERE itemid = 23296 ORDER BY clock DESC LIMIT 10;

# 查看触发器
SELECT triggerid, description, expression, priority FROM triggers WHERE status = 0;
```

### 13.4 备份恢复

```bash
# 1. 数据库备份
# 完整备份
mysqldump -uzabbix -p zabbix > zabbix_backup_$(date +%Y%m%d).sql

# 仅备份配置（不含历史数据）
mysqldump -uzabbix -p zabbix \
  --ignore-table=zabbix.history \
  --ignore-table=zabbix.history_uint \
  --ignore-table=zabbix.history_str \
  --ignore-table=zabbix.history_text \
  --ignore-table=zabbix.history_log \
  --ignore-table=zabbix.trends \
  --ignore-table=zabbix.trends_uint \
  > zabbix_config_backup_$(date +%Y%m%d).sql

# 2. 配置文件备份
tar -czf zabbix_conf_backup_$(date +%Y%m%d).tar.gz \
  /etc/zabbix/ \
  /usr/lib/zabbix/alertscripts/ \
  /usr/lib/zabbix/externalscripts/

# 3. 恢复数据库
mysql -uzabbix -p zabbix < zabbix_backup_20240101.sql

# 4. 自动备份脚本
sudo vi /usr/local/bin/zabbix_backup.sh
#!/bin/bash
BACKUP_DIR="/backup/zabbix"
DATE=$(date +%Y%m%d)
RETENTION_DAYS=7

# 创建备份目录
mkdir -p $BACKUP_DIR

# 备份数据库
mysqldump -uzabbix -ppassword zabbix | gzip > $BACKUP_DIR/zabbix_db_$DATE.sql.gz

# 备份配置
tar -czf $BACKUP_DIR/zabbix_conf_$DATE.tar.gz /etc/zabbix/

# 删除旧备份
find $BACKUP_DIR -name "zabbix_*" -mtime +$RETENTION_DAYS -delete

sudo chmod +x /usr/local/bin/zabbix_backup.sh

# 添加定时任务
sudo crontab -e
0 2 * * * /usr/local/bin/zabbix_backup.sh
```

## 14. 学习检查清单

### 14.1 基础知识

- [ ] 理解 Zabbix 的架构和组件
- [ ] 掌握 Zabbix Server 的安装和配置
- [ ] 了解 Zabbix Agent 的工作原理
- [ ] 理解主机、监控项、触发器的概念
- [ ] 掌握模板的创建和使用
- [ ] 了解数据采集的多种方式

### 14.2 监控配置

- [ ] 能够添加和配置主机
- [ ] 能够创建自定义监控项
- [ ] 掌握常用监控项的配置
- [ ] 能够创建和管理模板
- [ ] 理解模板继承和宏的使用
- [ ] 能够配置触发器表达式
- [ ] 掌握告警动作的配置
- [ ] 能够配置多种告警媒介

### 14.3 可视化

- [ ] 能够创建图形展示数据
- [ ] 掌握仪表板的配置
- [ ] 能够创建网络拓扑图
- [ ] 了解屏幕的使用
- [ ] 能够自定义可视化展示

### 14.4 高级功能

- [ ] 掌握网络发现的配置
- [ ] 理解主动注册机制
- [ ] 能够创建低级发现规则
- [ ] 掌握 Proxy 的部署和配置
- [ ] 了解分布式监控架构
- [ ] 能够使用 Zabbix API
- [ ] 掌握批量操作的方法

### 14.5 实战应用

- [ ] 能够监控 Nginx/Apache
- [ ] 能够监控 MySQL/PostgreSQL
- [ ] 能够监控 Redis/MongoDB
- [ ] 能够监控 Docker 容器
- [ ] 能够监控 Kubernetes 集群
- [ ] 能够实现日志监控
- [ ] 能够创建自定义监控脚本

### 14.6 性能优化

- [ ] 了解数据库优化方法
- [ ] 掌握 Zabbix Server 参数调优
- [ ] 能够优化监控项配置
- [ ] 理解缓存机制
- [ ] 能够分析性能瓶颈
- [ ] 掌握数据清理策略

### 14.7 故障排查

- [ ] 能够分析 Zabbix 日志
- [ ] 掌握常见问题的解决方法
- [ ] 能够使用调试工具
- [ ] 理解数据库查询方法
- [ ] 能够进行备份和恢复
- [ ] 掌握故障诊断流程

### 14.8 安全加固

- [ ] 了解 Zabbix 安全最佳实践
- [ ] 能够配置 TLS/PSK 加密
- [ ] 掌握用户权限管理
- [ ] 能够配置防火墙规则
- [ ] 了解审计日志的使用
- [ ] 能够实施安全加固措施

## 📚 参考资源

### 官方文档

- [Zabbix 官方文档](https://www.zabbix.com/documentation/current/)
- [Zabbix API 文档](https://www.zabbix.com/documentation/current/manual/api)
- [Zabbix 模板库](https://www.zabbix.com/integrations)
- [Zabbix 论坛](https://www.zabbix.com/forum/)

### 学习资源

- [Zabbix 官方培训](https://www.zabbix.com/training)
- [Zabbix 认证考试](https://www.zabbix.com/certification)
- [Zabbix GitHub](https://github.com/zabbix/zabbix)
- [Zabbix Docker](https://github.com/zabbix/zabbix-docker)

### 社区资源

- [Zabbix Share](https://share.zabbix.com/) - 模板和脚本分享
- [Awesome Zabbix](https://github.com/zabbix/awesome-zabbix) - 资源列表
- [Zabbix 中文社区](https://www.zabbix.org.cn/)
- [Zabbix 博客](https://blog.zabbix.com/)

### 工具和插件

- [Grafana Zabbix Plugin](https://grafana.com/grafana/plugins/alexanderzobnin-zabbix-app/) - Grafana 集成
- [Zabbix CLI](https://github.com/usit-gd/zabbix-cli) - 命令行工具
- [Zabbix Ansible](https://github.com/ansible-collections/community.zabbix) - Ansible 集成
- [Zabbix Terraform](https://registry.terraform.io/providers/tpretz/zabbix/latest) - Terraform Provider

### 监控模板

- Linux 系统监控
- Windows 系统监控
- 网络设备监控（Cisco、Huawei）
- 数据库监控（MySQL、PostgreSQL、Oracle）
- 中间件监控（Nginx、Apache、Tomcat）
- 容器监控（Docker、Kubernetes）
- 云平台监控（AWS、Azure、阿里云）

### 最佳实践

- [Zabbix 性能调优指南](https://www.zabbix.com/documentation/current/manual/appendix/performance_tuning)
- [Zabbix 安全加固指南](https://www.zabbix.com/documentation/current/manual/installation/requirements/best_practices)
- [Zabbix 高可用部署](https://www.zabbix.com/documentation/current/manual/installation/requirements/high_availability)
- [Zabbix 大规模部署](https://www.zabbix.com/documentation/current/manual/installation/requirements/large_scale)

### 书籍推荐

- 《Zabbix 监控实战》
- 《Zabbix 企业级分布式监控系统》
- 《Learning Zabbix》
- 《Mastering Zabbix》

### 视频教程

- [Zabbix 官方 YouTube 频道](https://www.youtube.com/c/Zabbix)
- [Zabbix Summit 演讲](https://www.zabbix.com/summit)
- [Zabbix Webinar](https://www.zabbix.com/webinars)

---

> 💡 **学习建议**
>
> 1. 先在测试环境搭建 Zabbix，熟悉基本操作
> 2. 从简单的主机监控开始，逐步增加复杂度
> 3. 多实践自定义监控项和模板的创建
> 4. 学习使用 API 进行自动化管理
> 5. 关注性能优化和故障排查技巧
> 6. 参与社区交流，学习他人经验
> 7. 定期查看官方文档，了解新特性
> 8. 在生产环境部署前做好充分测试

> ⚠️ **注意事项**
>
> 1. 生产环境部署前务必做好备份
> 2. 合理规划监控项，避免过度监控
> 3. 定期清理历史数据，控制数据库大小
> 4. 做好权限管理，保护敏感信息
> 5. 配置告警时避免告警风暴
> 6. 使用 Proxy 分流，减轻 Server 压力
> 7. 定期检查系统性能，及时优化
> 8. 建立完善的监控体系和应急预案

---

**@author erik.zhou**

**最后更新时间：2024-02**
