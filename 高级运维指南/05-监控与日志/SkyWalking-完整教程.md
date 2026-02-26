# SkyWalking 完整教程

> 分布式系统应用性能监控（APM）平台实战指南
>
> @author erik.zhou

## 📚 目录

- [1. SkyWalking 简介](#1-skywalking-简介)
- [2. 架构与原理](#2-架构与原理)
- [3. 安装部署](#3-安装部署)
- [4. Agent 配置](#4-agent-配置)
- [5. 服务监控](#5-服务监控)
- [6. 链路追踪](#6-链路追踪)
- [7. 告警配置](#7-告警配置)
- [8. 日志集成](#8-日志集成)
- [9. 自定义监控](#9-自定义监控)
- [10. 性能优化](#10-性能优化)
- [11. 实战案例](#11-实战案例)
- [12. 故障排查](#12-故障排查)
- [13. 学习检查清单](#13-学习检查清单)

## 🎯 学习目标

- 理解 SkyWalking 的架构和工作原理
- 掌握 SkyWalking OAP 和 UI 的部署
- 能够配置和使用 Java Agent
- 掌握服务拓扑和链路追踪
- 能够配置告警规则
- 了解日志和指标的集成
- 掌握自定义监控的实现
- 能够进行性能调优和故障排查

## 1. SkyWalking 简介

### 1.1 什么是 SkyWalking

Apache SkyWalking 是一个开源的应用性能监控（APM）系统，专为微服务、云原生和容器化架构设计。

**核心特性**：
- 分布式追踪
- 服务拓扑分析
- 性能指标监控
- 告警通知
- 日志分析
- 多语言支持（Java、.NET、Node.js、Python、Go 等）
- 无侵入式监控
- 可扩展的插件系统

### 1.2 核心概念

```
服务（Service）：表示一组提供相同行为的工作负载
实例（Instance）：服务的一个运行实例
端点（Endpoint）：服务中的一个路径或 URI
追踪（Trace）：一次完整的请求调用链
跨度（Span）：追踪中的一个操作单元
```

### 1.3 架构组件

```
┌─────────────────────────────────────────┐
│         SkyWalking UI                   │
│      (Web Dashboard)                    │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│      SkyWalking OAP Server              │
│  ┌──────────────────────────────────┐  │
│  │  Receiver  │  Aggregator         │  │
│  │  Analyzer  │  Query Engine       │  │
│  └──────────────────────────────────┘  │
└──────────────┬──────────────────────────┘
               │
    ┌──────────┼──────────┐
    │          │          │
┌───▼───┐  ┌──▼───┐  ┌──▼────┐
│Agent  │  │Agent │  │Agent  │
│Java   │  │.NET  │  │Node.js│
└───────┘  └──────┘  └───────┘
```

**组件说明**：
- **SkyWalking Agent**：探针，负责收集应用数据
- **SkyWalking OAP**：观测分析平台，处理和存储数据
- **SkyWalking UI**：Web 界面，展示监控数据
- **Storage**：存储后端（ElasticSearch、MySQL、H2 等）

### 1.4 数据流

```
Application → Agent → OAP Server → Storage → UI
              ↓
         (gRPC/HTTP)
```

## 2. 架构与原理

### 2.1 探针架构

```java
/**
 * SkyWalking Agent 工作原理
 */
// 1. 字节码增强
// 使用 Java Agent 技术在类加载时修改字节码
// -javaagent:/path/to/skywalking-agent.jar

// 2. 插件机制
// 通过插件拦截目标方法
public class HttpClientInstrumentation extends ClassInstanceMethodsEnhancePluginDefine {
    @Override
    public ConstructorInterceptPoint[] getConstructorsInterceptPoints() {
        // 构造器拦截点
    }
    
    @Override
    public InstanceMethodsInterceptPoint[] getInstanceMethodsInterceptPoints() {
        // 实例方法拦截点
    }
}

// 3. 上下文传播
// 通过 ContextCarrier 在服务间传递追踪信息
ContextCarrier carrier = new ContextCarrier();
ContextManager.inject(carrier);
// 将 carrier 放入 HTTP Header
```

### 2.2 数据模型

```protobuf
// Trace 数据结构
message TraceSegment {
    string traceId = 1;
    string segmentId = 2;
    repeated SpanObject spans = 3;
    string service = 4;
    string serviceInstance = 5;
}

message SpanObject {
    int32 spanId = 1;
    int32 parentSpanId = 2;
    int64 startTime = 3;
    int64 endTime = 4;
    string operationName = 5;
    string peer = 6;
    SpanType spanType = 7;
    SpanLayer spanLayer = 8;
    repeated KeyValue tags = 9;
    repeated Log logs = 10;
}
```

### 2.3 采样策略

```yaml
# agent.config
# 采样率配置
agent.sample_n_per_3_secs=9  # 每3秒采样9次
# 或
agent.sample_rate=1000       # 采样率 1/1000

# 忽略特定端点
agent.ignore_suffix=.jpg,.jpeg,.js,.css,.png,.ico
```

## 3. 安装部署

### 3.1 使用 Docker Compose 部署

```yaml
# docker-compose.yml
version: '3.8'

services:
  elasticsearch:
    image: elasticsearch:7.17.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - skywalking

  oap:
    image: apache/skywalking-oap-server:9.5.0
    container_name: skywalking-oap
    depends_on:
      - elasticsearch
    environment:
      SW_STORAGE: elasticsearch
      SW_STORAGE_ES_CLUSTER_NODES: elasticsearch:9200
      SW_HEALTH_CHECKER: default
      SW_TELEMETRY: prometheus
      JAVA_OPTS: "-Xms2g -Xmx2g"
    ports:
      - "11800:11800"  # gRPC
      - "12800:12800"  # HTTP
    networks:
      - skywalking
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:12800/internal/l7check || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3

  ui:
    image: apache/skywalking-ui:9.5.0
    container_name: skywalking-ui
    depends_on:
      - oap
    environment:
      SW_OAP_ADDRESS: http://oap:12800
      SW_ZIPKIN_ADDRESS: http://oap:9412
    ports:
      - "8080:8080"
    networks:
      - skywalking

volumes:
  es-data:

networks:
  skywalking:
    driver: bridge
```

```bash
# 启动服务
docker-compose up -d

# 查看日志
docker-compose logs -f oap

# 访问 UI
# http://localhost:8080
```

### 3.2 Kubernetes 部署

```yaml
# skywalking-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: skywalking

---
# skywalking-oap.yaml
apiVersion: v1
kind: Service
metadata:
  name: skywalking-oap
  namespace: skywalking
spec:
  selector:
    app: skywalking-oap
  ports:
    - name: grpc
      port: 11800
      targetPort: 11800
    - name: http
      port: 12800
      targetPort: 12800
  type: ClusterIP

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: skywalking-oap
  namespace: skywalking
spec:
  replicas: 2
  selector:
    matchLabels:
      app: skywalking-oap
  template:
    metadata:
      labels:
        app: skywalking-oap
    spec:
      containers:
      - name: oap
        image: apache/skywalking-oap-server:9.5.0
        ports:
        - containerPort: 11800
          name: grpc
        - containerPort: 12800
          name: http
        env:
        - name: SW_STORAGE
          value: elasticsearch
        - name: SW_STORAGE_ES_CLUSTER_NODES
          value: elasticsearch:9200
        - name: JAVA_OPTS
          value: "-Xms2g -Xmx2g"
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /internal/l7check
            port: 12800
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /internal/l7check
            port: 12800
          initialDelaySeconds: 10
          periodSeconds: 5

---
# skywalking-ui.yaml
apiVersion: v1
kind: Service
metadata:
  name: skywalking-ui
  namespace: skywalking
spec:
  selector:
    app: skywalking-ui
  ports:
    - port: 8080
      targetPort: 8080
  type: LoadBalancer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: skywalking-ui
  namespace: skywalking
spec:
  replicas: 1
  selector:
    matchLabels:
      app: skywalking-ui
  template:
    metadata:
      labels:
        app: skywalking-ui
    spec:
      containers:
      - name: ui
        image: apache/skywalking-ui:9.5.0
        ports:
        - containerPort: 8080
        env:
        - name: SW_OAP_ADDRESS
          value: http://skywalking-oap:12800
```

```bash
# 部署
kubectl apply -f skywalking-namespace.yaml
kubectl apply -f skywalking-oap.yaml
kubectl apply -f skywalking-ui.yaml

# 查看状态
kubectl get pods -n skywalking
kubectl get svc -n skywalking
```

### 3.3 二进制部署

```bash
# 1. 下载 SkyWalking
wget https://archive.apache.org/dist/skywalking/9.5.0/apache-skywalking-apm-9.5.0.tar.gz
tar -xzf apache-skywalking-apm-9.5.0.tar.gz
cd apache-skywalking-apm-bin

# 2. 配置存储（ElasticSearch）
vi config/application.yml

storage:
  selector: ${SW_STORAGE:elasticsearch}
  elasticsearch:
    namespace: ${SW_NAMESPACE:"skywalking"}
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
    connectTimeout: ${SW_STORAGE_ES_CONNECT_TIMEOUT:3000}
    socketTimeout: ${SW_STORAGE_ES_SOCKET_TIMEOUT:30000}
    responseTimeout: ${SW_STORAGE_ES_RESPONSE_TIMEOUT:15000}
    numHttpClientThread: ${SW_STORAGE_ES_NUM_HTTP_CLIENT_THREAD:0}
    user: ${SW_ES_USER:""}
    password: ${SW_ES_PASSWORD:""}

# 3. 启动 OAP Server
bin/oapService.sh

# 4. 启动 UI
bin/webappService.sh

# 5. 查看日志
tail -f logs/skywalking-oap-server.log
```

## 4. Agent 配置

### 4.1 Java Agent 配置

```bash
# 1. 下载 Agent
# Agent 包含在 SkyWalking 发行版中
# apache-skywalking-apm-bin/agent/

# 2. 配置 agent.config
vi agent/config/agent.config

# 服务名称
agent.service_name=${SW_AGENT_NAME:your-service-name}

# OAP Server 地址
collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:127.0.0.1:11800}

# 采样率
agent.sample_n_per_3_secs=${SW_AGENT_SAMPLE:9}

# 日志级别
logging.level=${SW_LOGGING_LEVEL:INFO}

# 日志输出
logging.file_name=${SW_LOGGING_FILE_NAME:skywalking-api.log}
logging.dir=${SW_LOGGING_DIR:""}
logging.max_file_size=${SW_LOGGING_MAX_FILE_SIZE:300 * 1024 * 1024}
logging.max_history_files=${SW_LOGGING_MAX_HISTORY_FILES:5}

# 忽略后缀
agent.ignore_suffix=${SW_AGENT_IGNORE_SUFFIX:.jpg,.jpeg,.js,.css,.png,.bmp,.gif,.ico,.mp3,.mp4,.html,.svg}

# 插件配置
plugin.mount=${SW_MOUNT_FOLDERS:plugins,activations}
```

### 4.2 启动应用

```bash
# 方式1：命令行参数
java -javaagent:/path/to/skywalking-agent/skywalking-agent.jar \
     -Dskywalking.agent.service_name=my-service \
     -Dskywalking.collector.backend_service=127.0.0.1:11800 \
     -jar your-application.jar

# 方式2：环境变量
export SW_AGENT_NAME=my-service
export SW_AGENT_COLLECTOR_BACKEND_SERVICES=127.0.0.1:11800
java -javaagent:/path/to/skywalking-agent/skywalking-agent.jar \
     -jar your-application.jar

# 方式3：Docker
docker run -d \
  -e JAVA_OPTS="-javaagent:/skywalking/agent/skywalking-agent.jar" \
  -e SW_AGENT_NAME=my-service \
  -e SW_AGENT_COLLECTOR_BACKEND_SERVICES=oap:11800 \
  -v /path/to/skywalking-agent:/skywalking/agent \
  your-image:tag
```

### 4.3 Spring Boot 集成

```dockerfile
# Dockerfile
FROM openjdk:11-jre-slim

# 复制 SkyWalking Agent
COPY skywalking-agent /skywalking/agent

# 复制应用
COPY target/app.jar /app.jar

# 设置环境变量
ENV JAVA_OPTS="-javaagent:/skywalking/agent/skywalking-agent.jar"
ENV SW_AGENT_NAME=spring-boot-app
ENV SW_AGENT_COLLECTOR_BACKEND_SERVICES=oap:11800

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app.jar"]
```

```yaml
# application.yml
spring:
  application:
    name: ${SW_AGENT_NAME:spring-boot-app}

# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    environment:
      - SW_AGENT_NAME=spring-boot-app
      - SW_AGENT_COLLECTOR_BACKEND_SERVICES=oap:11800
      - SW_AGENT_SAMPLE=9
    ports:
      - "8080:8080"
```

### 4.4 可选插件配置

```bash
# agent/optional-plugins/ 目录包含可选插件
# 将需要的插件复制到 agent/plugins/ 目录

# 常用可选插件
cp optional-plugins/apm-trace-ignore-plugin-*.jar plugins/
cp optional-plugins/apm-spring-annotation-plugin-*.jar plugins/
cp optional-plugins/apm-customize-enhance-plugin-*.jar plugins/

# 配置插件
vi agent/config/agent.config

# Trace 忽略插件
trace.ignore_path=${SW_AGENT_TRACE_IGNORE_PATH:/health,/metrics}

# Spring 注解插件
plugin.spring.annotation.cache_or_enhance_class=${SW_SPRING_ANNOTATION_CACHE_CLASS:true}
```


## 5. 服务监控

### 5.1 服务拓扑

```
SkyWalking UI 提供服务拓扑图：
- 服务依赖关系
- 调用流量
- 响应时间
- 错误率
- 服务健康状态

访问路径：Dashboard → Topology
```

### 5.2 服务指标

```
核心指标：
- CPM (Calls Per Minute)：每分钟调用次数
- SLA (Service Level Agreement)：服务可用性
- Resp Time：响应时间（平均、P50、P75、P90、P95、P99）
- Apdex：应用性能指数

查看路径：Dashboard → Service
```

### 5.3 实例监控

```
实例级别指标：
- JVM 内存使用
- GC 统计
- 线程数
- CPU 使用率
- 类加载数

查看路径：Dashboard → Service → Instance
```

### 5.4 端点监控

```
端点（Endpoint）指标：
- 请求量
- 响应时间
- 错误率
- 慢端点分析

查看路径：Dashboard → Service → Endpoint
```

### 5.5 自定义仪表板

```yaml
# 创建自定义仪表板
# UI → Dashboard → New Dashboard

# 添加组件：
- Service Apdex
- Service Response Time
- Service Throughput
- Service Instance Load
- JVM Memory
- JVM GC
- Database Access
```

## 6. 链路追踪

### 6.1 追踪查询

```
查询条件：
- 服务名称
- 实例名称
- 端点名称
- Trace ID
- 时间范围
- 状态（成功/失败）
- 响应时间范围

查询路径：Trace → Trace Query
```

### 6.2 追踪详情

```
Trace 详情包含：
- Trace ID
- 总耗时
- Span 数量
- 服务调用链
- 每个 Span 的详细信息：
  - 服务名称
  - 端点名称
  - 开始时间
  - 耗时
  - 标签（Tags）
  - 日志（Logs）
```

### 6.3 Span 分析

```java
/**
 * Span 类型
 */
// Entry Span：服务入口
// Local Span：本地方法调用
// Exit Span：远程调用出口

/**
 * Span 层级
 */
// Database：数据库操作
// RPC Framework：RPC 调用
// HTTP：HTTP 调用
// MQ：消息队列
// Cache：缓存操作
```

### 6.4 慢追踪分析

```
慢追踪识别：
1. 设置响应时间阈值
2. 查询超过阈值的 Trace
3. 分析 Span 耗时分布
4. 定位性能瓶颈

优化建议：
- 数据库查询优化
- 缓存使用
- 异步处理
- 批量操作
```

### 6.5 手动埋点

```java
import org.apache.skywalking.apm.toolkit.trace.ActiveSpan;
import org.apache.skywalking.apm.toolkit.trace.Trace;
import org.apache.skywalking.apm.toolkit.trace.TraceContext;

public class BusinessService {
    
    /**
     * 方法级追踪
     */
    @Trace
    public void businessMethod() {
        // 业务逻辑
        processData();
    }
    
    /**
     * 添加自定义标签
     */
    public void processData() {
        ActiveSpan.tag("user_id", "12345");
        ActiveSpan.tag("order_id", "ORD-001");
        
        // 添加日志
        ActiveSpan.info("Processing order data");
        
        // 业务逻辑
    }
    
    /**
     * 获取 Trace ID
     */
    public void logTraceId() {
        String traceId = TraceContext.traceId();
        System.out.println("Current Trace ID: " + traceId);
    }
    
    /**
     * 异步追踪
     */
    @Trace
    public void asyncMethod() {
        CompletableFuture.runAsync(() -> {
            // 异步操作会自动关联到父 Span
            doAsyncWork();
        });
    }
}
```

## 7. 告警配置

### 7.1 告警规则

```yaml
# config/alarm-settings.yml
rules:
  # 服务响应时间告警
  service_resp_time_rule:
    metrics-name: service_resp_time
    op: ">"
    threshold: 1000
    period: 10
    count: 3
    silence-period: 5
    message: "服务 {name} 响应时间超过 1s"
    
  # 服务成功率告警
  service_sla_rule:
    metrics-name: service_sla
    op: "<"
    threshold: 95
    period: 10
    count: 2
    silence-period: 3
    message: "服务 {name} 成功率低于 95%"
    
  # 服务 CPM 告警
  service_cpm_rule:
    metrics-name: service_cpm
    op: ">"
    threshold: 10000
    period: 10
    count: 3
    silence-period: 5
    message: "服务 {name} 每分钟调用量超过 10000"
    
  # 端点响应时间告警
  endpoint_resp_time_rule:
    metrics-name: endpoint_resp_time
    op: ">"
    threshold: 2000
    period: 10
    count: 3
    silence-period: 5
    message: "端点 {name} 响应时间超过 2s"
    
  # 实例 JVM 内存告警
  instance_jvm_memory_heap_rule:
    metrics-name: instance_jvm_memory_heap
    op: ">"
    threshold: 80
    period: 10
    count: 3
    silence-period: 5
    message: "实例 {name} JVM 堆内存使用率超过 80%"
    
  # 数据库访问告警
  database_access_resp_time_rule:
    metrics-name: database_access_resp_time
    op: ">"
    threshold: 500
    period: 10
    count: 3
    silence-period: 5
    message: "数据库 {name} 访问响应时间超过 500ms"

# 复合规则
composite-rules:
  # 服务异常告警
  service_error_rule:
    expression: service_resp_time > 1000 && service_sla < 95
    period: 10
    count: 2
    silence-period: 5
    message: "服务 {name} 出现异常：响应时间过长且成功率下降"
```

### 7.2 Webhook 告警

```yaml
# config/alarm-settings.yml
webhooks:
  - url: http://alerting-service:8080/webhook
    headers:
      Authorization: "Bearer token123"
      Content-Type: "application/json"
```

```java
/**
 * Webhook 接收端示例
 */
@RestController
@RequestMapping("/webhook")
public class AlarmWebhookController {
    
    @PostMapping
    public ResponseEntity<String> receiveAlarm(@RequestBody AlarmMessage alarm) {
        // 处理告警
        log.warn("收到告警: {}", alarm.getMessage());
        
        // 发送到钉钉/企业微信/邮件等
        sendToDingTalk(alarm);
        
        return ResponseEntity.ok("OK");
    }
    
    private void sendToDingTalk(AlarmMessage alarm) {
        // 钉钉机器人实现
        String webhook = "https://oapi.dingtalk.com/robot/send?access_token=xxx";
        
        Map<String, Object> message = new HashMap<>();
        message.put("msgtype", "text");
        
        Map<String, String> text = new HashMap<>();
        text.put("content", String.format(
            "SkyWalking 告警\n服务: %s\n消息: %s\n时间: %s",
            alarm.getScope(),
            alarm.getMessage(),
            alarm.getStartTime()
        ));
        message.put("text", text);
        
        // 发送 HTTP 请求
        restTemplate.postForObject(webhook, message, String.class);
    }
}
```

### 7.3 钉钉告警

```yaml
# config/alarm-settings.yml
dingtalkHooks:
  - url: https://oapi.dingtalk.com/robot/send?access_token=xxx
    secret: SEC_xxx  # 可选，加签密钥
```

### 7.4 企业微信告警

```yaml
# config/alarm-settings.yml
wechatHooks:
  - corpId: ww123456
    agentId: 1000001
    secret: secret_xxx
    webhookUrl: https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxx
```

### 7.5 邮件告警

```yaml
# config/alarm-settings.yml
emailHooks:
  - host: smtp.gmail.com
    port: 587
    username: your-email@gmail.com
    password: your-password
    from: skywalking@example.com
    to:
      - admin@example.com
      - ops@example.com
    subject: "SkyWalking Alarm"
```

## 8. 日志集成

### 8.1 日志关联

```java
import org.apache.skywalking.apm.toolkit.log4j2.v1.x.TraceIdConverter;

/**
 * Log4j2 配置
 */
// log4j2.xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - [%X{tid}] - %msg%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>

/**
 * 在日志中输出 Trace ID
 */
import org.apache.skywalking.apm.toolkit.trace.TraceContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;

public class LogService {
    private static final Logger logger = LoggerFactory.getLogger(LogService.class);
    
    public void businessMethod() {
        // 方式1：使用 TraceContext
        String traceId = TraceContext.traceId();
        logger.info("Processing request, traceId: {}", traceId);
        
        // 方式2：使用 MDC
        MDC.put("tid", traceId);
        logger.info("Business logic executed");
        MDC.remove("tid");
    }
}
```

### 8.2 Logback 配置

```xml
<!-- logback-spring.xml -->
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
                <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - [%tid] - %msg%n</pattern>
            </layout>
        </encoder>
    </appender>
    
    <!-- gRPC 日志上报 -->
    <appender name="grpc-log" class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.log.GRPCLogClientAppender">
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
                <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - [%tid] - %msg%n</pattern>
            </layout>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="grpc-log"/>
    </root>
</configuration>
```

### 8.3 日志查询

```
UI 日志查询功能：
1. Log → Log Query
2. 输入查询条件：
   - 服务名称
   - 实例名称
   - Trace ID
   - 时间范围
   - 日志级别
   - 关键字
3. 查看日志详情
4. 关联 Trace 查看
```

## 9. 自定义监控

### 9.1 自定义指标

```java
import org.apache.skywalking.apm.toolkit.meter.Counter;
import org.apache.skywalking.apm.toolkit.meter.Gauge;
import org.apache.skywalking.apm.toolkit.meter.Histogram;
import org.apache.skywalking.apm.toolkit.meter.MeterFactory;

public class CustomMetrics {
    
    /**
     * 计数器
     */
    private static final Counter orderCounter = MeterFactory.counter("order_count")
        .tag("type", "online")
        .build();
    
    public void createOrder() {
        // 业务逻辑
        orderCounter.increment(1);
    }
    
    /**
     * 仪表盘
     */
    private static final Gauge queueSize = MeterFactory.gauge("queue_size", () -> {
        return getQueueSize();
    }).build();
    
    private static double getQueueSize() {
        // 返回队列大小
        return 100.0;
    }
    
    /**
     * 直方图
     */
    private static final Histogram responseTime = MeterFactory.histogram("response_time")
        .tag("api", "/api/users")
        .build();
    
    public void recordResponseTime(long time) {
        responseTime.addValue(time);
    }
}
```

### 9.2 自定义插件

```java
/**
 * 自定义插件示例
 */
// 1. 定义插件
public class CustomInstrumentation extends ClassInstanceMethodsEnhancePluginDefine {
    
    @Override
    protected ClassMatch enhanceClass() {
        return NameMatch.byName("com.example.CustomClass");
    }
    
    @Override
    public ConstructorInterceptPoint[] getConstructorsInterceptPoints() {
        return new ConstructorInterceptPoint[0];
    }
    
    @Override
    public InstanceMethodsInterceptPoint[] getInstanceMethodsInterceptPoints() {
        return new InstanceMethodsInterceptPoint[] {
            new InstanceMethodsInterceptPoint() {
                @Override
                public ElementMatcher<MethodDescription> getMethodsMatcher() {
                    return named("customMethod");
                }
                
                @Override
                public String getMethodsInterceptor() {
                    return "com.example.CustomInterceptor";
                }
                
                @Override
                public boolean isOverrideArgs() {
                    return false;
                }
            }
        };
    }
}

// 2. 定义拦截器
public class CustomInterceptor implements InstanceMethodsAroundInterceptor {
    
    @Override
    public void beforeMethod(EnhancedInstance objInst, Method method, 
                            Object[] allArguments, Class<?>[] argumentsTypes,
                            MethodInterceptResult result) {
        // 方法执行前
        AbstractSpan span = ContextManager.createLocalSpan("CustomMethod");
        span.tag("param", String.valueOf(allArguments[0]));
    }
    
    @Override
    public Object afterMethod(EnhancedInstance objInst, Method method,
                             Object[] allArguments, Class<?>[] argumentsTypes,
                             Object ret) {
        // 方法执行后
        AbstractSpan span = ContextManager.activeSpan();
        span.tag("result", String.valueOf(ret));
        ContextManager.stopSpan();
        return ret;
    }
    
    @Override
    public void handleMethodException(EnhancedInstance objInst, Method method,
                                     Object[] allArguments, Class<?>[] argumentsTypes,
                                     Throwable t) {
        // 异常处理
        AbstractSpan span = ContextManager.activeSpan();
        span.log(t);
        span.errorOccurred();
    }
}

// 3. 配置插件
// resources/skywalking-plugin.def
custom-plugin=com.example.CustomInstrumentation
```

### 9.3 自定义标签

```java
import org.apache.skywalking.apm.toolkit.trace.ActiveSpan;
import org.apache.skywalking.apm.toolkit.trace.Tag;
import org.apache.skywalking.apm.toolkit.trace.Tags;

public class TagService {
    
    /**
     * 添加标签
     */
    public void processOrder(String orderId, String userId) {
        // 添加业务标签
        ActiveSpan.tag("order_id", orderId);
        ActiveSpan.tag("user_id", userId);
        ActiveSpan.tag("payment_method", "alipay");
        
        // 业务逻辑
    }
    
    /**
     * 使用注解添加标签
     */
    @Tags({
        @Tag(key = "operation", value = "arg[0]"),
        @Tag(key = "user", value = "arg[1]")
    })
    public void annotatedMethod(String operation, String user) {
        // 业务逻辑
    }
}
```


## 10. 性能优化

### 10.1 采样优化

```yaml
# agent/config/agent.config

# 1. 固定采样率
agent.sample_n_per_3_secs=9  # 每3秒采样9次

# 2. 百分比采样
agent.sample_rate=1000       # 1/1000 的采样率

# 3. 忽略特定端点
trace.ignore_path=/health,/metrics,/actuator/*

# 4. 慢追踪采样
# 只采样响应时间超过阈值的请求
agent.slow_trace_segment_threshold=1000  # 1秒
```

### 10.2 存储优化

```yaml
# application.yml

storage:
  elasticsearch:
    # 索引分片
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:1}
    
    # 批量操作
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:5000}
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:15}
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2}
    
    # 数据保留
    recordDataTTL: ${SW_STORAGE_ES_RECORD_DATA_TTL:3}      # 3天
    metricsDataTTL: ${SW_STORAGE_ES_METRICS_DATA_TTL:7}    # 7天
    
    # 索引模式
    dayStep: ${SW_STORAGE_DAY_STEP:1}  # 按天分索引
```

### 10.3 OAP 优化

```yaml
# application.yml

core:
  default:
    # 工作线程
    gRPCThreadPoolSize: ${SW_CORE_GRPC_THREAD_POOL_SIZE:4}
    gRPCThreadPoolQueueSize: ${SW_CORE_GRPC_THREAD_POOL_QUEUE_SIZE:10000}
    
    # 缓存
    l1CacheSize: ${SW_CORE_L1_CACHE_SIZE:10000}
    l2CacheSize: ${SW_CORE_L2_CACHE_SIZE:50000}
    
    # 聚合
    downsampling:
      - Hour
      - Day
    
    # 指标计算
    metricsDataTTL: ${SW_CORE_METRICS_DATA_TTL:7}
    recordDataTTL: ${SW_CORE_RECORD_DATA_TTL:3}

receiver-sharing-server:
  default:
    # 接收器配置
    maxConcurrentCallsPerConnection: ${SW_RECEIVER_SHARING_MAX_CONCURRENT_CALL:4}
    maxMessageSize: ${SW_RECEIVER_SHARING_MAX_MESSAGE_SIZE:104857600}
    
receiver-trace:
  default:
    # Trace 接收器
    bufferPath: ${SW_RECEIVER_BUFFER_PATH:../trace-buffer/}
    bufferOffsetMaxFileSize: ${SW_RECEIVER_BUFFER_OFFSET_MAX_FILE_SIZE:100}
    bufferDataMaxFileSize: ${SW_RECEIVER_BUFFER_DATA_MAX_FILE_SIZE:500}
    bufferFileCleanWhenRestart: ${SW_RECEIVER_BUFFER_FILE_CLEAN_WHEN_RESTART:false}
    sampleRate: ${SW_TRACE_SAMPLE_RATE:10000}
```

### 10.4 Agent 优化

```properties
# agent/config/agent.config

# 1. 插件优化
# 禁用不需要的插件
plugin.exclude_plugins=tomcat-7.x-8.x-plugin,spring-plugins

# 2. 缓存优化
plugin.peer_max_length=200
plugin.cache.read_size_limit=10000

# 3. 队列优化
buffer.channel_size=5
buffer.buffer_size=300

# 4. 日志优化
logging.level=WARN
logging.max_file_size=10485760  # 10MB
logging.max_history_files=3

# 5. 性能优化
agent.is_cache_enhanced_class=true
agent.class_cache_mode=MEMORY
```

### 10.5 网络优化

```yaml
# 1. gRPC 压缩
# agent/config/agent.config
collector.grpc_upstream_timeout=30
collector.get_profile_task_interval=20
collector.get_agent_dynamic_config_interval=20

# 2. 批量发送
agent.span_limit_per_segment=300
agent.keep_tracing=false

# 3. 连接池
collector.backend_service=oap1:11800,oap2:11800,oap3:11800
collector.grpc_channel_check_interval=30
```

## 11. 实战案例

### 11.1 Spring Cloud 微服务监控

```java
/**
 * Spring Cloud 服务配置
 */
// 1. 添加依赖
// pom.xml
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-trace</artifactId>
    <version>8.16.0</version>
</dependency>

// 2. 配置服务
// application.yml
spring:
  application:
    name: order-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848

// 3. 启动服务
// java -javaagent:skywalking-agent.jar \
//      -Dskywalking.agent.service_name=order-service \
//      -jar order-service.jar

/**
 * Feign 调用监控
 */
@FeignClient(name = "user-service")
public interface UserServiceClient {
    @GetMapping("/users/{id}")
    User getUser(@PathVariable("id") Long id);
}

@Service
public class OrderService {
    @Autowired
    private UserServiceClient userServiceClient;
    
    @Trace
    public Order createOrder(OrderRequest request) {
        // Feign 调用会自动被追踪
        User user = userServiceClient.getUser(request.getUserId());
        
        // 业务逻辑
        Order order = new Order();
        order.setUserId(user.getId());
        
        return orderRepository.save(order);
    }
}
```

### 11.2 数据库监控

```java
/**
 * JDBC 监控（自动）
 */
// SkyWalking 自动监控 JDBC 操作
@Repository
public class UserRepository {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public User findById(Long id) {
        // SQL 执行会自动被追踪
        return jdbcTemplate.queryForObject(
            "SELECT * FROM users WHERE id = ?",
            new Object[]{id},
            new BeanPropertyRowMapper<>(User.class)
        );
    }
}

/**
 * MyBatis 监控（自动）
 */
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM users WHERE id = #{id}")
    User selectById(@Param("id") Long id);
}

/**
 * 慢 SQL 分析
 */
// 在 UI 中查看：
// Database → Slow SQL
// 可以看到：
// - SQL 语句
// - 执行时间
// - 调用次数
// - 关联的 Trace
```

### 11.3 Redis 监控

```java
/**
 * Redis 监控（自动）
 */
@Service
public class CacheService {
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    public User getUserFromCache(Long id) {
        // Redis 操作会自动被追踪
        String key = "user:" + id;
        return (User) redisTemplate.opsForValue().get(key);
    }
    
    public void cacheUser(User user) {
        String key = "user:" + user.getId();
        redisTemplate.opsForValue().set(key, user, 1, TimeUnit.HOURS);
    }
}

/**
 * 查看 Redis 监控
 */
// UI → Database → Redis
// 可以看到：
// - 命令类型
// - 执行时间
// - 调用次数
```

### 11.4 消息队列监控

```java
/**
 * RabbitMQ 监控
 */
@Component
public class MessageProducer {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @Trace
    public void sendMessage(String message) {
        // 发送消息会自动被追踪
        rabbitTemplate.convertAndSend("exchange", "routing.key", message);
    }
}

@Component
public class MessageConsumer {
    @RabbitListener(queues = "queue.name")
    @Trace
    public void receiveMessage(String message) {
        // 消费消息会自动被追踪
        processMessage(message);
    }
    
    private void processMessage(String message) {
        // 业务逻辑
    }
}

/**
 * Kafka 监控
 */
@Service
public class KafkaProducerService {
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;
    
    @Trace
    public void sendMessage(String topic, String message) {
        kafkaTemplate.send(topic, message);
    }
}

@Service
public class KafkaConsumerService {
    @KafkaListener(topics = "topic.name")
    @Trace
    public void consume(String message) {
        processMessage(message);
    }
}
```

### 11.5 异步任务监控

```java
/**
 * 异步方法监控
 */
@Service
public class AsyncService {
    
    @Async
    @Trace
    public CompletableFuture<String> asyncMethod() {
        // 异步操作会自动关联到父 Span
        return CompletableFuture.supplyAsync(() -> {
            // 业务逻辑
            return "result";
        });
    }
    
    /**
     * 线程池任务监控
     */
    @Autowired
    private ThreadPoolTaskExecutor executor;
    
    @Trace
    public void submitTask() {
        executor.submit(() -> {
            // 任务会自动关联到父 Span
            doWork();
        });
    }
}

/**
 * 定时任务监控
 */
@Component
public class ScheduledTasks {
    
    @Scheduled(fixedRate = 60000)
    @Trace
    public void scheduledTask() {
        // 定时任务会被追踪
        performTask();
    }
}
```

### 11.6 HTTP 客户端监控

```java
/**
 * RestTemplate 监控（自动）
 */
@Service
public class HttpService {
    @Autowired
    private RestTemplate restTemplate;
    
    public String callExternalApi() {
        // HTTP 调用会自动被追踪
        return restTemplate.getForObject(
            "http://external-service/api/data",
            String.class
        );
    }
}

/**
 * OkHttp 监控（自动）
 */
@Service
public class OkHttpService {
    private OkHttpClient client = new OkHttpClient();
    
    public String callApi() throws IOException {
        Request request = new Request.Builder()
            .url("http://external-service/api/data")
            .build();
        
        // OkHttp 调用会自动被追踪
        try (Response response = client.newCall(request).execute()) {
            return response.body().string();
        }
    }
}
```

## 12. 故障排查

### 12.1 Agent 问题

```bash
# 1. Agent 未启动
# 检查 JVM 参数
jps -v | grep skywalking

# 2. Agent 无法连接 OAP
# 查看 Agent 日志
tail -f logs/skywalking-api.log

# 常见错误：
# "Failed to connect to backend"
# 解决：检查 OAP 地址和网络连接

# 3. 插件冲突
# 禁用冲突插件
vi agent/config/agent.config
plugin.exclude_plugins=conflicting-plugin

# 4. 性能影响
# 降低采样率
agent.sample_n_per_3_secs=3

# 5. 内存占用
# 限制 Agent 内存
export JAVA_AGENT_OPTS="-Xmx512m"
```

### 12.2 OAP 问题

```bash
# 1. OAP 启动失败
# 查看日志
docker logs skywalking-oap
# 或
tail -f logs/skywalking-oap-server.log

# 2. 存储连接失败
# 检查 ElasticSearch 连接
curl http://elasticsearch:9200/_cluster/health

# 3. 内存不足
# 增加 JVM 内存
export SW_JAVA_OPTS="-Xms4g -Xmx4g"

# 4. 数据延迟
# 检查队列积压
# UI → General Service → OAP Server
# 查看 Receiver 队列大小

# 5. 索引问题
# 重建索引
curl -X DELETE http://elasticsearch:9200/sw_*
# 重启 OAP
```

### 12.3 数据问题

```bash
# 1. 数据丢失
# 检查采样率
vi agent/config/agent.config
agent.sample_n_per_3_secs=9

# 检查数据保留期
vi config/application.yml
recordDataTTL: 3
metricsDataTTL: 7

# 2. Trace 不完整
# 检查跨服务传播
# 确保所有服务都安装了 Agent

# 3. 指标不准确
# 检查时钟同步
ntpdate -u ntp.aliyun.com

# 4. 查询慢
# 优化 ES 查询
# 增加分片数
# 使用索引模板
```

### 12.4 告警问题

```bash
# 1. 告警未触发
# 检查告警规则
vi config/alarm-settings.yml

# 测试告警
# 手动触发条件，观察是否收到告警

# 2. 告警风暴
# 增加静默期
silence-period: 10

# 3. Webhook 失败
# 检查 Webhook 日志
tail -f logs/alarm.log

# 测试 Webhook
curl -X POST http://webhook-url \
  -H "Content-Type: application/json" \
  -d '{"test": "message"}'
```

### 12.5 性能问题

```bash
# 1. 高 CPU 使用
# 检查 OAP 进程
top -p $(pgrep -f skywalking-oap)

# 优化聚合配置
vi config/application.yml
downsampling: [Hour]  # 减少聚合级别

# 2. 高内存使用
# 检查内存
jmap -heap $(pgrep -f skywalking-oap)

# 调整缓存大小
l1CacheSize: 5000
l2CacheSize: 25000

# 3. 存储压力
# 检查 ES 性能
curl http://elasticsearch:9200/_nodes/stats

# 优化索引
# 减少副本数
# 增加刷新间隔
```

## 13. 学习检查清单

### 13.1 基础知识
- [ ] 理解 APM 的概念和作用
- [ ] 掌握 SkyWalking 的架构
- [ ] 了解 Agent、OAP、UI 的职责
- [ ] 理解 Trace、Span 的概念
- [ ] 掌握服务拓扑的含义

### 13.2 部署配置
- [ ] 能够使用 Docker 部署 SkyWalking
- [ ] 能够在 Kubernetes 中部署
- [ ] 掌握 Agent 的配置方法
- [ ] 了解存储后端的选择
- [ ] 能够配置高可用部署

### 13.3 监控使用
- [ ] 能够查看服务拓扑
- [ ] 掌握服务指标的含义
- [ ] 能够进行链路追踪查询
- [ ] 了解慢追踪分析方法
- [ ] 能够使用仪表板

### 13.4 告警配置
- [ ] 能够配置告警规则
- [ ] 掌握 Webhook 告警
- [ ] 能够集成钉钉/企业微信
- [ ] 了解复合告警规则
- [ ] 能够处理告警风暴

### 13.5 高级功能
- [ ] 掌握日志关联方法
- [ ] 能够实现手动埋点
- [ ] 了解自定义指标
- [ ] 能够开发自定义插件
- [ ] 掌握性能优化技巧

### 13.6 实战应用
- [ ] 能够监控 Spring Cloud 应用
- [ ] 掌握数据库监控
- [ ] 能够监控消息队列
- [ ] 了解异步任务监控
- [ ] 能够排查常见问题

## 📚 参考资源

### 官方资源
- [SkyWalking 官网](https://skywalking.apache.org/)
- [官方文档](https://skywalking.apache.org/docs/)
- [GitHub 仓库](https://github.com/apache/skywalking)
- [插件列表](https://skywalking.apache.org/docs/skywalking-java/latest/en/setup/service-agent/java-agent/supported-list/)

### 学习资源
- [SkyWalking 极简入门](https://skywalking.apache.org/zh/2020-04-19-skywalking-quick-start/)
- [SkyWalking 博客](https://skywalking.apache.org/blog/)
- [视频教程](https://www.youtube.com/c/ApacheSkyWalking)

### 社区资源
- [邮件列表](https://skywalking.apache.org/community/)
- [Slack 频道](https://join.slack.com/t/the-asf/shared_invite/zt-vlfbf7ch-HkbNHiU_uDlcH_RvaHv9gQ)
- [中文社区](https://github.com/SkyAPM/document-cn-translation-of-skywalking)

### 工具集成
- [Grafana 插件](https://github.com/SkyAPM/grafana-skywalking-datasource)
- [Kubernetes Operator](https://github.com/apache/skywalking-swck)
- [Helm Charts](https://github.com/apache/skywalking-kubernetes)

---

> 💡 **学习建议**
> 1. 先在测试环境部署，熟悉基本功能
> 2. 从简单的 Spring Boot 应用开始监控
> 3. 逐步添加数据库、缓存、消息队列监控
> 4. 学习链路追踪和性能分析
> 5. 配置告警规则，建立监控体系
> 6. 关注性能优化和故障排查
> 7. 参与社区交流，学习最佳实践

> ⚠️ **注意事项**
> 1. Agent 会对应用性能有轻微影响
> 2. 合理设置采样率，避免数据过载
> 3. 定期清理历史数据，控制存储成本
> 4. 做好 OAP 的高可用部署
> 5. 配置告警时避免告警风暴
> 6. 保护敏感信息，不要在 Trace 中记录密码等
> 7. 定期备份配置和重要数据

---

**@author erik.zhou**

**最后更新时间：2024-02**
