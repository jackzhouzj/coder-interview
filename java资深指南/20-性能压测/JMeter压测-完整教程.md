# Apache JMeter 压测完整教程

> **版本**: JMeter 5.6.3 | **难度**: ⭐⭐⭐ | **重要程度**: ⭐⭐⭐⭐⭐

## 📚 技术概述

### 什么是JMeter？

Apache JMeter是一款开源的性能测试工具，最初设计用于测试Web应用程序，现在已扩展到其他测试功能。

**核心特性**：
- 支持多种协议（HTTP、HTTPS、FTP、JDBC、SOAP等）
- 图形化界面，易于使用
- 支持分布式压测
- 丰富的插件生态
- 可生成HTML报告

### 学习前置知识

- ✅ HTTP协议基础
- ✅ 基本的性能测试概念
- ✅ Java环境配置（JMeter基于Java）

### 适用场景

- Web应用性能测试
- API接口压测
- 数据库性能测试
- FTP服务器测试
- 消息队列测试

## 🎯 学习目标

- [ ] 掌握JMeter的安装和配置
- [ ] 理解JMeter的核心组件
- [ ] 能够编写基本的测试计划
- [ ] 掌握参数化和断言
- [ ] 能够进行分布式压测
- [ ] 会分析压测结果和生成报告


## 📖 一、JMeter基础

### 1.1 安装配置

#### Windows安装

```bash
# 1. 下载JMeter
# 访问 https://jmeter.apache.org/download_jmeter.cgi
# 下载 apache-jmeter-5.6.3.zip

# 2. 解压到指定目录
# 例如：D:\apache-jmeter-5.6.3

# 3. 配置环境变量（可选）
JMETER_HOME=D:\apache-jmeter-5.6.3
PATH=%PATH%;%JMETER_HOME%\bin

# 4. 启动JMeter
cd D:\apache-jmeter-5.6.3\bin
jmeter.bat
```

#### Linux/Mac安装

```bash
# 1. 下载JMeter
wget https://dlcdn.apache.org//jmeter/binaries/apache-jmeter-5.6.3.tgz

# 2. 解压
tar -xzf apache-jmeter-5.6.3.tgz

# 3. 配置环境变量
export JMETER_HOME=/path/to/apache-jmeter-5.6.3
export PATH=$PATH:$JMETER_HOME/bin

# 4. 启动JMeter
jmeter
```

#### JVM参数优化

编辑 `bin/jmeter` 或 `bin/jmeter.bat`：

```bash
# 增加堆内存
HEAP="-Xms1g -Xmx1g -XX:MaxMetaspaceSize=256m"
```

### 1.2 核心组件

#### 测试计划（Test Plan）
- 所有测试元素的容器
- 可配置用户自定义变量

#### 线程组（Thread Group）🔥
- 模拟用户并发
- 配置线程数、Ramp-Up时间、循环次数

```
线程数：100（模拟100个用户）
Ramp-Up：10秒（10秒内启动完所有线程）
循环次数：10（每个线程执行10次）
```

#### 取样器（Sampler）🔥
- HTTP请求
- JDBC请求
- FTP请求
- Java请求

#### 监听器（Listener）
- 查看结果树
- 聚合报告
- 图形结果
- 后端监听器

#### 配置元件
- CSV数据文件配置
- HTTP请求默认值
- HTTP Cookie管理器
- HTTP Header管理器

#### 断言（Assertion）
- 响应断言
- JSON断言
- 持续时间断言

#### 定时器（Timer）
- 固定定时器
- 随机定时器
- 常数吞吐量定时器


## 💻 二、快速开始

### 2.1 第一个HTTP压测

#### 步骤1：创建测试计划

1. 启动JMeter
2. 右键点击"测试计划" → 添加 → Threads(Users) → 线程组

#### 步骤2：配置线程组

```
线程数：10
Ramp-Up时间：1秒
循环次数：5
```

#### 步骤3：添加HTTP请求

右键点击"线程组" → 添加 → 取样器 → HTTP请求

```
协议：https
服务器名称或IP：www.example.com
端口号：443
路径：/api/users
方法：GET
```

#### 步骤4：添加监听器

右键点击"线程组" → 添加 → 监听器 → 查看结果树
右键点击"线程组" → 添加 → 监听器 → 聚合报告

#### 步骤5：运行测试

点击工具栏的绿色"启动"按钮

### 2.2 POST请求示例

```
方法：POST
路径：/api/login
Content-Type：application/json

Body Data：
{
  "username": "testuser",
  "password": "password123"
}
```

### 2.3 添加HTTP Header

右键点击"HTTP请求" → 添加 → 配置元件 → HTTP信息头管理器

```
Content-Type: application/json
Authorization: Bearer ${token}
```


## 🔥 三、核心功能详解

### 3.1 参数化 - CSV数据文件🔥

#### 准备CSV文件（users.csv）

```csv
username,password,email
user1,pass1,user1@example.com
user2,pass2,user2@example.com
user3,pass3,user3@example.com
```

#### 配置CSV数据文件

右键点击"线程组" → 添加 → 配置元件 → CSV Data Set Config

```
文件名：users.csv
变量名称：username,password,email
忽略首行：True
遇到文件结束符再次循环：True
遇到文件结束符停止线程：False
共享模式：All threads
```

#### 使用变量

在HTTP请求中使用：`${username}`, `${password}`, `${email}`

```json
{
  "username": "${username}",
  "password": "${password}",
  "email": "${email}"
}
```

### 3.2 关联 - 提取响应数据🔥

#### JSON提取器

右键点击"HTTP请求" → 添加 → 后置处理器 → JSON Extractor

```
变量名称：token
JSON Path表达式：$.data.token
匹配数字：1
缺省值：TOKEN_NOT_FOUND
```

#### 正则表达式提取器

```
引用名称：userId
正则表达式："userId":"(\d+)"
模板：$1$
匹配数字：1
缺省值：NOT_FOUND
```

#### 使用提取的变量

在后续请求中使用：`${token}`, `${userId}`

### 3.3 断言 - 验证响应🔥

#### 响应断言

右键点击"HTTP请求" → 添加 → 断言 → 响应断言

```
Apply to：Main sample only
响应字段：响应文本
模式匹配规则：包括
测试模式：success
```

#### JSON断言

```
Assert JSON Path exists：$.data.token
Expected Value：（留空表示只验证存在）
```

#### 持续时间断言

```
持续时间（毫秒）：1000
```

### 3.4 定时器 - 控制请求间隔

#### 固定定时器

```
线程延迟（毫秒）：1000
```

#### 随机定时器

```
延迟偏移（毫秒）：500
最大随机延迟（毫秒）：1000
```

#### 常数吞吐量定时器

```
目标吞吐量（每分钟采样数）：600
计算吞吐量基于：All active threads
```


## 🚀 四、进阶应用

### 4.1 命令行模式运行⚠️

#### 为什么使用命令行？

- GUI模式消耗资源多
- 命令行模式性能更好
- 适合CI/CD集成

#### 基本命令

```bash
# 运行测试计划
jmeter -n -t test_plan.jmx -l result.jtl -e -o report

# 参数说明：
# -n: 非GUI模式
# -t: 测试计划文件
# -l: 结果文件
# -e: 生成报告
# -o: 报告输出目录
```

#### 设置系统属性

```bash
jmeter -n -t test.jmx -l result.jtl \
  -Jthreads=100 \
  -Jrampup=10 \
  -Jduration=300
```

在测试计划中使用：`${__P(threads,10)}`

### 4.2 分布式压测⚠️

#### 架构说明

```
Master（控制机）
  ├── Slave1（压力机1）
  ├── Slave2（压力机2）
  └── Slave3（压力机3）
```

#### 配置步骤

**1. Slave机器配置**

编辑 `bin/jmeter-server.bat`（Windows）或 `bin/jmeter-server`（Linux）：

```bash
# 启动Slave
jmeter-server -Djava.rmi.server.hostname=192.168.1.101
```

**2. Master机器配置**

编辑 `bin/jmeter.properties`：

```properties
# 配置Slave地址
remote_hosts=192.168.1.101:1099,192.168.1.102:1099,192.168.1.103:1099
```

**3. 运行分布式测试**

```bash
# GUI模式：运行 → 远程启动全部
# 命令行模式：
jmeter -n -t test.jmx -r -l result.jtl
```

#### 注意事项

- 所有机器JMeter版本要一致
- 测试计划和数据文件要同步
- 防火墙开放1099端口
- 结果文件会在Master机器汇总

### 4.3 插件管理器

#### 安装插件管理器

1. 下载 `jmeter-plugins-manager-x.x.jar`
2. 放到 `lib/ext` 目录
3. 重启JMeter

#### 常用插件

**PerfMon Server Agent**
- 监控服务器资源（CPU、内存、网络）

**Custom Thread Groups**
- 更灵活的线程组配置

**3 Basic Graphs**
- 更丰富的图表展示


### 4.4 实战场景：电商秒杀压测

#### 场景设计

```
1. 用户登录（10%流量）
2. 浏览商品列表（30%流量）
3. 查看商品详情（40%流量）
4. 加入购物车（15%流量）
5. 提交订单（5%流量）
```

#### 测试计划结构

```
测试计划
├── CSV Data Set Config（用户数据）
├── HTTP Cookie Manager
├── HTTP Header Manager
├── 线程组（1000用户，10秒启动）
│   ├── 吞吐量控制器（10%）- 登录
│   │   └── HTTP请求 - POST /api/login
│   ├── 吞吐量控制器（30%）- 商品列表
│   │   └── HTTP请求 - GET /api/products
│   ├── 吞吐量控制器（40%）- 商品详情
│   │   └── HTTP请求 - GET /api/products/${productId}
│   ├── 吞吐量控制器（15%）- 加购物车
│   │   └── HTTP请求 - POST /api/cart
│   └── 吞吐量控制器（5%）- 提交订单
│       └── HTTP请求 - POST /api/orders
└── 聚合报告
```

#### 吞吐量控制器配置

```
执行百分比：10
Per User：勾选
```

### 4.5 数据库压测

#### JDBC连接配置

右键点击"测试计划" → 添加 → 配置元件 → JDBC Connection Configuration

```
Variable Name：mysql_db
Database URL：jdbc:mysql://localhost:3306/testdb
JDBC Driver class：com.mysql.cj.jdbc.Driver
Username：root
Password：password
```

#### JDBC请求

```sql
-- 查询请求
SELECT * FROM users WHERE id = ${userId}

-- 插入请求
INSERT INTO orders (user_id, product_id, amount) 
VALUES (${userId}, ${productId}, ${amount})

-- 更新请求
UPDATE products SET stock = stock - 1 
WHERE id = ${productId} AND stock > 0
```

#### 添加MySQL驱动

将 `mysql-connector-java-8.0.33.jar` 放到 `lib` 目录


## 📊 五、结果分析

### 5.1 关键指标🔥

#### 聚合报告指标

| 指标 | 说明 | 目标值 |
|------|------|--------|
| **Samples** | 请求总数 | - |
| **Average** | 平均响应时间 | <500ms |
| **Median** | 中位数响应时间 | <300ms |
| **90% Line** | 90%请求响应时间 | <1000ms |
| **95% Line** | 95%请求响应时间 | <1500ms |
| **99% Line** | 99%请求响应时间 | <2000ms |
| **Min** | 最小响应时间 | - |
| **Max** | 最大响应时间 | <5000ms |
| **Error %** | 错误率 | <1% |
| **Throughput** | 吞吐量（TPS） | 越高越好 |
| **Received KB/sec** | 接收带宽 | - |
| **Sent KB/sec** | 发送带宽 | - |

### 5.2 HTML报告生成

#### 命令行生成

```bash
# 测试时生成
jmeter -n -t test.jmx -l result.jtl -e -o report

# 已有结果文件生成
jmeter -g result.jtl -o report
```

#### 报告内容

- **Dashboard**：总览仪表盘
- **Charts**：各种性能图表
- **Statistics**：统计数据
- **Errors**：错误详情

### 5.3 性能瓶颈分析

#### 响应时间过长

**可能原因**：
- 数据库查询慢
- 接口逻辑复杂
- 网络延迟高
- 服务器资源不足

**排查方法**：
1. 查看服务器监控（CPU、内存、IO）
2. 分析慢查询日志
3. 使用APM工具（SkyWalking、Pinpoint）
4. 检查网络延迟

#### 吞吐量上不去

**可能原因**：
- 线程数不够
- 服务器处理能力达到上限
- 数据库连接池满了
- 存在锁竞争

**优化方向**：
1. 增加线程数（注意资源消耗）
2. 优化代码逻辑
3. 增加缓存
4. 数据库读写分离

#### 错误率高

**可能原因**：
- 服务器过载
- 数据库连接超时
- 接口限流
- 数据准备不充分

**解决方案**：
1. 降低并发量
2. 增加超时时间
3. 检查限流配置
4. 准备更多测试数据


## ✨ 六、最佳实践

### 6.1 测试计划设计原则

#### 1. 模拟真实场景

```
❌ 错误：所有用户同时访问同一个接口
✅ 正确：按照真实业务比例分配流量
```

#### 2. 合理设置Ramp-Up

```
❌ 错误：1000个线程，Ramp-Up=1秒（瞬间压力）
✅ 正确：1000个线程，Ramp-Up=60秒（逐步加压）
```

#### 3. 使用参数化避免缓存

```
❌ 错误：所有请求使用相同参数
✅ 正确：使用CSV文件提供不同的测试数据
```

### 6.2 性能优化建议

#### JMeter自身优化

```properties
# jmeter.properties 配置优化

# 1. 禁用不必要的监听器
# GUI模式下不要添加太多监听器

# 2. 使用非GUI模式
# 生产压测必须使用命令行模式

# 3. 增加堆内存
# HEAP="-Xms2g -Xmx2g"

# 4. 禁用不必要的功能
jmeter.save.saveservice.output_format=csv
jmeter.save.saveservice.response_data=false
jmeter.save.saveservice.samplerData=false
jmeter.save.saveservice.requestHeaders=false
jmeter.save.saveservice.url=true
jmeter.save.saveservice.responseHeaders=false
```

#### 压测机器优化

```bash
# 1. 增加文件描述符限制
ulimit -n 65535

# 2. 优化TCP参数
sysctl -w net.ipv4.tcp_tw_reuse=1
sysctl -w net.ipv4.tcp_tw_recycle=1
sysctl -w net.ipv4.ip_local_port_range="1024 65535"

# 3. 关闭不必要的服务
systemctl stop firewalld
```

### 6.3 常见陷阱

#### 陷阱1：忽略Think Time

```
问题：没有设置思考时间，请求过于密集
解决：添加随机定时器模拟用户思考
```

#### 陷阱2：数据准备不足

```
问题：CSV文件数据量少，导致数据重复
解决：准备足够多的测试数据（至少是线程数的10倍）
```

#### 陷阱3：忽略Cookie和Session

```
问题：每次请求都是新会话，不符合真实场景
解决：添加HTTP Cookie Manager
```

#### 陷阱4：压测环境与生产差异大

```
问题：压测环境配置低，结果不准确
解决：尽量使用与生产相同配置的环境
```

### 6.4 CI/CD集成

#### Jenkins集成

```groovy
pipeline {
    agent any
    stages {
        stage('Performance Test') {
            steps {
                sh '''
                    jmeter -n -t test.jmx \
                           -l result.jtl \
                           -e -o report \
                           -Jthreads=100 \
                           -Jduration=300
                '''
                
                // 发布HTML报告
                publishHTML([
                    reportDir: 'report',
                    reportFiles: 'index.html',
                    reportName: 'Performance Report'
                ])
                
                // 性能趋势分析
                perfReport sourceDataFiles: 'result.jtl'
            }
        }
    }
}
```

#### GitLab CI集成

```yaml
performance_test:
  stage: test
  script:
    - jmeter -n -t test.jmx -l result.jtl -e -o report
  artifacts:
    paths:
      - report/
    expire_in: 1 week
  only:
    - master
```


## ❓ 七、常见问题

### Q1: JMeter能支持多少并发？

**A**: 取决于压测机器配置：
- 单机：1000-5000并发（8核16G）
- 分布式：可支持数万并发
- 建议：单个Slave不超过5000线程

### Q2: 如何模拟真实的网络延迟？

**A**: 使用高斯随机定时器：

```
右键点击 → 添加 → 定时器 → Gaussian Random Timer
偏移量：100ms
标准差：50ms
```

### Q3: 如何处理动态Token？

**A**: 使用JSON提取器：

```
1. 登录接口提取token
2. 后续请求使用 ${token}
3. 如果token过期，使用BeanShell重新登录
```

### Q4: 压测时服务器CPU很高，但TPS上不去？

**A**: 可能原因：
- 代码存在性能瓶颈（慢查询、死锁）
- 线程池配置不合理
- 数据库连接池不够
- 建议使用APM工具定位具体问题

### Q5: 如何压测WebSocket？

**A**: 使用WebSocket插件：

```
1. 安装 JMeter WebSocket Samplers
2. 添加 WebSocket Sampler
3. 配置连接和消息
```

### Q6: 结果文件太大怎么办？

**A**: 优化保存配置：

```properties
# 只保存必要信息
jmeter.save.saveservice.output_format=csv
jmeter.save.saveservice.response_data=false
jmeter.save.saveservice.samplerData=false
```

### Q7: 如何实现阶梯式加压？

**A**: 使用Stepping Thread Group插件：

```
第一阶段：100用户，持续5分钟
第二阶段：200用户，持续5分钟
第三阶段：500用户，持续5分钟
```

### Q8: 分布式压测结果不准确？

**A**: 检查以下问题：
- 时钟是否同步（使用NTP）
- 网络延迟是否过高
- Slave机器配置是否一致
- 防火墙是否正确配置


## 🔗 八、相关资源

### 官方资源

- **官方网站**: https://jmeter.apache.org/
- **用户手册**: https://jmeter.apache.org/usermanual/index.html
- **API文档**: https://jmeter.apache.org/api/index.html
- **插件中心**: https://jmeter-plugins.org/

### 推荐教程

- **JMeter官方教程**: https://jmeter.apache.org/usermanual/get-started.html
- **性能测试实战**: https://www.perftest.com/
- **BlazeMeter博客**: https://www.blazemeter.com/blog

### 常用插件

- **Plugins Manager**: 插件管理器
- **PerfMon**: 服务器监控
- **Custom Thread Groups**: 自定义线程组
- **Throughput Shaping Timer**: 吞吐量控制

### 社区资源

- **GitHub**: https://github.com/apache/jmeter
- **Stack Overflow**: 搜索 `jmeter` 标签
- **JMeter中文社区**: https://jmeter.net/

### 推荐书籍

- 《性能测试实战》
- 《JMeter性能测试入门与实战》
- 《Web性能权威指南》

## 📝 九、学习检查清单

### 基础知识
- [ ] 理解JMeter的核心组件
- [ ] 能够创建基本的测试计划
- [ ] 掌握HTTP请求的配置
- [ ] 会使用监听器查看结果

### 进阶技能
- [ ] 掌握参数化（CSV文件）
- [ ] 会使用提取器（JSON、正则）
- [ ] 能够添加断言验证结果
- [ ] 理解定时器的作用

### 高级应用
- [ ] 能够使用命令行模式运行
- [ ] 掌握分布式压测配置
- [ ] 会分析性能测试报告
- [ ] 能够定位性能瓶颈

### 实战能力
- [ ] 能够设计复杂的压测场景
- [ ] 会进行数据库压测
- [ ] 能够集成到CI/CD流程
- [ ] 掌握性能优化技巧

## 🎯 十、实战练习

### 练习1：基础HTTP压测

**目标**: 压测一个REST API接口

**要求**:
- 100个并发用户
- 持续5分钟
- 生成HTML报告
- 错误率<1%

### 练习2：登录场景压测

**目标**: 模拟用户登录流程

**要求**:
- 使用CSV文件参数化用户名密码
- 提取登录后的token
- 使用token访问其他接口
- 添加响应断言

### 练习3：电商下单压测

**目标**: 模拟完整的下单流程

**要求**:
- 登录 → 浏览商品 → 加购物车 → 下单
- 按照真实比例分配流量
- 添加思考时间
- 分析性能瓶颈

### 练习4：分布式压测

**目标**: 使用3台机器进行分布式压测

**要求**:
- 配置Master和Slave
- 总并发3000用户
- 监控服务器资源
- 生成压测报告

---

> @author erik.zhou
> 
> 最后更新：2025-02-04
