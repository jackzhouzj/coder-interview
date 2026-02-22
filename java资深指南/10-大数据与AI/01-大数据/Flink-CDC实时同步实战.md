# Flink CDC 实时数据同步实战教程

> @author erik.zhou  
> 难度: ⭐⭐⭐⭐  
> 技术栈: Flink 1.14.5 + MySQL CDC 2.4.1 + Elasticsearch 7 + Redis

## 📋 项目概述

本文档基于真实生产项目 **product-search-real-time-sync**,详细讲解如何使用 Flink CDC 实现 MySQL 数据实时同步到 Elasticsearch 的完整方案。

### 业务场景

电商搜索系统需要将商品数据从 MySQL 实时同步到 Elasticsearch,支持:
- 多语言站点(英语、中文、日语等 9 种语言)
- 多表关联(商品、分类、图片、标签等 20+ 张表)
- 复杂业务逻辑(库存计算、价格转换、标签处理)
- 高性能要求(秒级延迟、百万级数据)

### 技术架构

```
MySQL(主从) → Flink CDC → 数据处理 → Elasticsearch
                ↓
            侧输出流分流
                ↓
        多表 Join + 聚合
                ↓
            窗口去重
                ↓
        批量写入 ES
```

## 🎯 核心技术点

### 1. Flink CDC 数据源配置

#### 1.1 Maven 依赖

```xml
<properties>
    <flink.version>1.14.5</flink.version>
    <scala.version>2.11</scala.version>
    <flink.connector.mysql.cdc.version>2.4.1</flink.connector.mysql.cdc.version>
</properties>

<dependencies>
    <!-- Flink 核心依赖 -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-streaming-java_${scala.version}</artifactId>
        <version>${flink.version}</version>
        <scope>${build.scope}</scope>
    </dependency>
    
    <!-- MySQL CDC 连接器 -->
    <dependency>
        <groupId>com.ververica</groupId>
        <artifactId>flink-connector-mysql-cdc</artifactId>
        <version>${flink.connector.mysql.cdc.version}</version>
    </dependency>
    
    <!-- Elasticsearch 7 连接器 -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-connector-elasticsearch7_${scala.version}</artifactId>
        <version>1.14.4</version>
    </dependency>
</dependencies>
```

#### 1.2 构建 MySQL CDC Source

```java
/**
 * 构建 MySQL CDC 数据源
 * 
 * @author erik.zhou
 */
private static MySqlSource<String> buildMysqlCdcSource(
        String slaverId, 
        String host, 
        int port, 
        String username, 
        String password, 
        String[] databases, 
        String[] tables) {
    
    // 自定义转换器配置
    Map<String, Object> customConverterConfigs = new HashMap<>();
    customConverterConfigs.put(
        JsonConverterConfig.DECIMAL_FORMAT_CONFIG, 
        DecimalFormat.NUMERIC.name()
    );
    
    // Debezium 属性配置
    Properties debeziumProperties = new Properties();
    // 增量快照块大小
    debeziumProperties.setProperty("scan.incremental.snapshot.chunk.size", "8096");
    // 时间类型转换器
    debeziumProperties.setProperty("converters", "temporal");
    debeziumProperties.setProperty(
        "temporal.type", 
        "com.fs.debezium.converters.TemporalValueConverter"
    );
    debeziumProperties.setProperty(
        "temporal.selector", 
        Arrays.stream(databases)
            .map(db -> db + ".*")
            .collect(Collectors.joining(","))
    );
    
    return MySqlSource.<String>builder()
        .hostname(host)
        .port(port)
        .databaseList(databases)
        .tableList(tables)
        .username(username)
        .password(password)
        .serverTimeZone("Asia/Shanghai")
        // 启动选项: initial() 全量+增量, latest() 仅增量
        .startupOptions(StartupOptions.initial())
        // 支持动态添加新表
        .scanNewlyAddedTableEnabled(true)
        // JSON 反序列化
        .deserializer(new JsonDebeziumDeserializationSchema(false, customConverterConfigs))
        .serverId(slaverId)
        .debeziumProperties(debeziumProperties)
        .build();
}
```

### 2. 侧输出流(Side Output)实现多表分流

#### 2.1 定义输出标签

```java
/**
 * 表常量定义
 * 
 * @author erik.zhou
 */
public class TableConstant {
    
    // 需要监听的表
    public static final String[] enTables = {
        "fiberstore_spain.products",
        "fiberstore_spain.products_description",
        "fiberstore_spain.products_to_categories",
        "cms.platform_aws_image",
        // ... 更多表
    };
    
    // 侧输出标签映射
    public static final Map<String, OutputTag<Tuple4<String, String, Long, String>>> 
        tableOutputTagMap = Arrays.stream(enTables)
            .collect(Collectors.toMap(
                table -> table, 
                table -> new OutputTag<Tuple4<String, String, Long, String>>(
                    "side-output-" + table
                ) {}
            ));
}
```

#### 2.2 分流处理函数

```java
/**
 * MySQL CDC 数据分流处理
 * 
 * @author erik.zhou
 */
public class MysqlCdcDataOutputTagProcess 
    extends ProcessFunction<String, Tuple4<String, String, Long, String>> {
    
    private Map<String, OutputTag<Tuple4<String, String, Long, String>>> tableOutputTagMap;
    
    @Override
    public void processElement(
            String cdcData, 
            Context ctx, 
            Collector<Tuple4<String, String, Long, String>> out) throws Exception {
        
        JSONObject root = JSONObject.parseObject(cdcData);
        JSONObject source = root.getJSONObject("source");
        
        String db = source.getString("db");
        String table = source.getString("table");
        Long tsMs = source.getLong("ts_ms");
        String op = root.getString("op"); // r=读取, c=创建, u=更新, d=删除
        
        // 根据操作类型获取数据
        JSONObject data;
        if ("d".equals(op)) {
            data = root.getJSONObject("before");
        } else if ("r,c,u".contains(op)) {
            data = root.getJSONObject("after");
        } else {
            LOG.warn("未知操作类型: {}", cdcData);
            return;
        }
        
        String fullTable = db + "." + table;
        if (tableOutputTagMap.containsKey(fullTable)) {
            ctx.output(
                tableOutputTagMap.get(fullTable), 
                new Tuple4<>(fullTable, op, tsMs, JSONObject.toJSONString(data))
            );
        }
    }
}
```

### 3. 多流 Join 实现

#### 3.1 ConnectedStreams Join

```java
/**
 * 商品与分类关联
 * 
 * @author erik.zhou
 */
SingleOutputStreamOperator<Tuple3<Long, String, Map<String, Object>>> prodDS = 
    tableOutputTagDS.getSideOutput(fsProductsOutputTag)
        .connect(tableOutputTagDS.getSideOutput(fsProductsToCategoriesOutputTag))
        .keyBy(
            tuple4 -> JSONObject.parseObject(tuple4.f3).getLong("products_id"),
            tuple4 -> JSONObject.parseObject(tuple4.f3).getLong("products_id")
        )
        .process(new JoinProductsToCategoriesProcess(configProperties))
        .name("products join products_to_categories");
```

#### 3.2 自定义 Join 处理函数

```java
/**
 * 基础左连接处理函数
 * 
 * @author erik.zhou
 */
public abstract class BaseLeftJoinProcess<IN1, IN2, OUT> 
    extends CoProcessFunction<IN1, IN2, OUT> {
    
    // 左流状态(主表)
    protected MapState<Long, ValueState<IN1>> leftState;
    // 右流状态(关联表)
    protected MapState<Long, ValueState<IN2>> rightState;
    
    @Override
    public void open(Configuration parameters) throws Exception {
        // 初始化状态
        leftState = getRuntimeContext().getMapState(
            new MapStateDescriptor<>("left-state", Long.class, ValueState.class)
        );
        rightState = getRuntimeContext().getMapState(
            new MapStateDescriptor<>("right-state", Long.class, ValueState.class)
        );
    }
    
    @Override
    public void processElement1(IN1 left, Context ctx, Collector<OUT> out) 
            throws Exception {
        // 处理左流数据
        Long key = extractLeftKey(left);
        leftState.put(key, left);
        
        // 尝试与右流 Join
        IN2 right = rightState.get(key);
        OUT result = join(left, right);
        out.collect(result);
    }
    
    @Override
    public void processElement2(IN2 right, Context ctx, Collector<OUT> out) 
            throws Exception {
        // 处理右流数据
        Long key = extractRightKey(right);
        rightState.put(key, right);
        
        // 更新左流结果
        IN1 left = leftState.get(key);
        if (left != null) {
            OUT result = join(left, right);
            out.collect(result);
        }
    }
    
    // 抽象方法由子类实现
    protected abstract Long extractLeftKey(IN1 left);
    protected abstract Long extractRightKey(IN2 right);
    protected abstract OUT join(IN1 left, IN2 right);
}
```

### 4. 窗口聚合与去重

#### 4.1 滚动窗口去重

```java
/**
 * 1分钟滚动窗口内数据去重
 * 
 * @author erik.zhou
 */
dataStream
    .keyBy(tuple3 -> tuple3.f1) // 按产品ID分组
    .window(TumblingProcessingTimeWindows.of(Time.minutes(1)))
    .process(new ProcessWindowFunction<
        Tuple3<String, String, Map<String, Object>>, 
        Tuple3<String, String, Map<String, Object>>, 
        String, 
        TimeWindow>() {
        
        @Override
        public void process(
                String key, 
                Context context, 
                Iterable<Tuple3<String, String, Map<String, Object>>> elements, 
                Collector<Tuple3<String, String, Map<String, Object>>> out) 
                throws Exception {
            
            String esRequestType = "";
            Map<String, Object> esDoc = null;
            
            // 窗口内数据合并逻辑
            for (Tuple3<String, String, Map<String, Object>> element : elements) {
                if ("".equals(esRequestType)) {
                    esRequestType = element.f0;
                    esDoc = new HashMap<>(element.f2);
                } else if ("index".equals(esRequestType) || "update".equals(esRequestType)) {
                    if ("index".equals(element.f0)) {
                        esRequestType = element.f0;
                        esDoc = new HashMap<>(element.f2);
                    } else if ("update".equals(element.f0)) {
                        esDoc.putAll(element.f2);
                    } else if ("delete".equals(element.f0)) {
                        esRequestType = element.f0;
                        esDoc = element.f2;
                    }
                }
            }
            
            out.collect(new Tuple3<>(esRequestType, key, esDoc));
        }
    });
```

### 5. Elasticsearch Sink 配置

#### 5.1 构建 ES Sink

```java
/**
 * 构建 Elasticsearch Sink
 * 
 * @author erik.zhou
 */
public static ElasticsearchSink<Tuple3<String, String, Map<String, Object>>> 
    buildElasticsearchSink(ConfigProperties config, String indexName) {
    
    // 解析 ES 主机列表
    List<HttpHost> hosts = Arrays.stream(config.getEsHost().split(","))
        .map(host -> {
            String[] split = host.split("://|:");
            if (split.length == 2) {
                return new HttpHost(split[0], Integer.parseInt(split[1]), "http");
            } else {
                return new HttpHost(split[1], Integer.parseInt(split[2]), split[0]);
            }
        }).collect(Collectors.toList());
    
    // 构建 Sink
    ElasticsearchSink.Builder<Tuple3<String, String, Map<String, Object>>> builder = 
        new ElasticsearchSink.Builder<>(hosts, (tuple3, runtimeContext, requestIndexer) -> {
            
            if ("index".equals(tuple3.f0)) {
                // 插入或更新文档
                if (StringUtils.isNumeric(tuple3.f1) && MapUtil.isNotEmpty(tuple3.f2)) {
                    ProductInfoSearchDocEn convertJson = 
                        SinkMapToProductSearchDoc.convertJson(
                            tuple3.f2, 
                            indexName.split("_")[2]
                        );
                    requestIndexer.add(
                        new UpdateRequest(indexName, tuple3.f1)
                            .doc(JSON.toJSONString(convertJson), XContentType.JSON)
                            .docAsUpsert(true)
                    );
                }
            } else if ("update".equals(tuple3.f0)) {
                // 更新文档
                if (StringUtils.isNumeric(tuple3.f1) && MapUtil.isNotEmpty(tuple3.f2)) {
                    ProductInfoSearchDocEn convert = 
                        SinkMapToProductSearchDoc.convertJson(
                            tuple3.f2, 
                            indexName.split("_")[2]
                        );
                    requestIndexer.add(
                        new UpdateRequest(indexName, tuple3.f1)
                            .doc(JSON.toJSONString(convert), XContentType.JSON)
                    );
                }
            } else if ("delete".equals(tuple3.f0)) {
                // 删除文档
                requestIndexer.add(
                    Requests.deleteRequest(indexName).id(tuple3.f1)
                );
            }
        });
    
    // Flush 策略配置
    builder.setBulkFlushMaxActions(1000);      // 1000条刷新
    builder.setBulkFlushMaxSizeMb(3);          // 3MB刷新
    builder.setBulkFlushInterval(3000);        // 3秒刷新
    builder.setBulkFlushBackoffDelay(1000);    // 重试延迟1秒
    
    // RestClient 配置
    builder.setRestClientFactory(restClientBuilder -> {
        // 连接超时配置
        restClientBuilder.setRequestConfigCallback(requestConfigBuilder ->
            requestConfigBuilder
                .setConnectTimeout(5000)
                .setSocketTimeout(60000)
        );
        
        // HTTP 客户端配置
        restClientBuilder.setHttpClientConfigCallback(httpClientBuilder -> {
            // 认证配置
            CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
            credentialsProvider.setCredentials(
                AuthScope.ANY, 
                new UsernamePasswordCredentials(
                    config.getEsUsername(), 
                    config.getEsPassword()
                )
            );
            
            // HTTPS 证书配置
            if (config.getEsCrtPath() != null) {
                Path caCertificatePath = Paths.get(config.getEsCrtPath());
                try {
                    CertificateFactory factory = CertificateFactory.getInstance("X.509");
                    Certificate trustedCa;
                    try (InputStream is = Files.newInputStream(caCertificatePath)) {
                        trustedCa = factory.generateCertificate(is);
                    }
                    KeyStore trustStore = KeyStore.getInstance("pkcs12");
                    trustStore.load(null, null);
                    trustStore.setCertificateEntry("ca", trustedCa);
                    
                    SSLContextBuilder sslContextBuilder = 
                        SSLContexts.custom().loadTrustMaterial(trustStore, null);
                    final SSLContext sslContext = sslContextBuilder.build();
                    httpClientBuilder.setSSLContext(sslContext);
                } catch (Exception e) {
                    throw new RuntimeException("HTTPS CA 证书配置失败", e);
                }
            }
            
            httpClientBuilder.setKeepAliveStrategy((resp, context) -> 180000);
            httpClientBuilder.disableAuthCaching();
            return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
        });
    });
    
    // 失败重试策略
    builder.setFailureHandler((actionRequest, throwable, i, requestIndexer) -> {
        if (ExceptionUtils.findThrowable(throwable, SocketTimeoutException.class).isPresent()) {
            // 超时重试
            if (actionRequest instanceof IndexRequest) {
                requestIndexer.add((IndexRequest) actionRequest);
            } else if (actionRequest instanceof UpdateRequest) {
                requestIndexer.add((UpdateRequest) actionRequest);
            } else if (actionRequest instanceof DeleteRequest) {
                requestIndexer.add((DeleteRequest) actionRequest);
            }
        } else {
            LOG.error("未知错误请求: {}", actionRequest);
            throw throwable;
        }
    });
    
    return builder.build();
}
```

### 6. Watermark 与事件时间

```java
/**
 * 配置 Watermark 策略
 * 
 * @author erik.zhou
 */
DataStreamSource<String> fiberstoreSpainDS = env.fromSource(
    fiberstoreSpainMysqlSource,
    WatermarkStrategy
        .<String>forMonotonousTimestamps()
        .withTimestampAssigner((event, timestamp) -> 
            JSONObject.parseObject(event)
                .getJSONObject("source")
                .getLong("ts_ms")
        )
        .withIdleness(Duration.ofSeconds(3)), // 空闲超时3秒
    "fiberstore_spain"
);
```

## 🔧 生产环境配置

### 1. Checkpoint 配置

```java
/**
 * Checkpoint 配置
 * 
 * @author erik.zhou
 */
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

// 启用 Checkpoint,间隔30秒
env.enableCheckpointing(30000L);

// Checkpoint 超时时间30分钟
env.getCheckpointConfig().setCheckpointTimeout(1800000);

// Checkpoint 存储路径
env.getCheckpointConfig().setCheckpointStorage(
    "hdfs://namenode:9000/flink/checkpoints"
);

// 任务取消时保留 Checkpoint
env.getCheckpointConfig().setExternalizedCheckpointCleanup(
    CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION
);
```

### 2. 重启策略

```java
/**
 * 重启策略配置
 * 
 * @author erik.zhou
 */
// 固定延迟重启: 最多重启3次,每次间隔10秒
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
    3,  // 重启次数
    Time.of(10, TimeUnit.SECONDS) // 重启间隔
));

// 失败率重启: 5分钟内最多失败3次
env.setRestartStrategy(RestartStrategies.failureRateRestart(
    3,  // 最大失败次数
    Time.of(5, TimeUnit.MINUTES), // 时间窗口
    Time.of(10, TimeUnit.SECONDS) // 重启间隔
));
```

### 3. 并行度配置

```java
/**
 * 并行度配置
 * 
 * @author erik.zhou
 */
// 全局并行度
env.setParallelism(4);

// 算子级别并行度
dataStream
    .map(...)
    .setParallelism(8)  // Map 算子并行度8
    .keyBy(...)
    .window(...)
    .process(...)
    .setParallelism(4); // Window 算子并行度4
```

## 📊 性能优化

### 1. 状态后端选择

```java
/**
 * 状态后端配置
 * 
 * @author erik.zhou
 */
// RocksDB 状态后端(适合大状态)
env.setStateBackend(new EmbeddedRocksDBStateBackend());

// 增量 Checkpoint
env.getCheckpointConfig().enableUnalignedCheckpoints();
```

### 2. 批量写入优化

```java
/**
 * ES 批量写入优化
 * 
 * @author erik.zhou
 */
builder.setBulkFlushMaxActions(1000);      // 批量大小
builder.setBulkFlushMaxSizeMb(3);          // 批量大小(MB)
builder.setBulkFlushInterval(3000);        // 刷新间隔(ms)
```

### 3. 背压处理

```java
/**
 * 背压处理配置
 * 
 * @author erik.zhou
 */
Configuration configuration = new Configuration();
configuration.setString("taskmanager.memory.flink.size", "8 gb");
configuration.setString("taskmanager.memory.network.min", "256 mb");
```

## 🚨 常见问题与解决方案

### 1. CDC 全量同步慢

**问题**: 初始化全量同步耗时过长

**解决方案**:
```java
// 增大快照块大小
debeziumProperties.setProperty("scan.incremental.snapshot.chunk.size", "8096");

// 增加并行度
env.setParallelism(8);
```

### 2. Checkpoint 超时

**问题**: Checkpoint 频繁超时失败

**解决方案**:
```java
// 增大超时时间
env.getCheckpointConfig().setCheckpointTimeout(1800000); // 30分钟

// 启用非对齐 Checkpoint
env.getCheckpointConfig().enableUnalignedCheckpoints();
```

### 3. ES 写入背压

**问题**: Elasticsearch 写入速度跟不上

**解决方案**:
```java
// 增大批量大小
builder.setBulkFlushMaxActions(2000);
builder.setBulkFlushMaxSizeMb(5);

// 增加 ES 节点数量
// 优化 ES 索引配置(副本数、刷新间隔)
```

### 4. 状态过大

**问题**: 状态数据占用内存过大

**解决方案**:
```java
// 使用 RocksDB 状态后端
env.setStateBackend(new EmbeddedRocksDBStateBackend());

// 设置状态 TTL
StateTtlConfig ttlConfig = StateTtlConfig
    .newBuilder(Time.days(7))
    .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
    .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
    .build();
```

## 📈 监控指标

### 1. 关键指标

- **延迟**: Source → Sink 端到端延迟
- **吞吐量**: 每秒处理记录数
- **背压**: 算子背压情况
- **Checkpoint**: 成功率、耗时
- **状态大小**: 各算子状态占用

### 2. Flink Web UI 监控

```bash
# 访问 Flink Web UI
http://jobmanager:8081

# 查看任务详情
- Overview: 任务概览
- Running Jobs: 运行中的任务
- Completed Jobs: 已完成的任务
- Task Managers: TaskManager 状态
```

### 3. Metrics 集成

```java
/**
 * 自定义 Metrics
 * 
 * @author erik.zhou
 */
public class CustomMetricsProcess extends ProcessFunction<String, String> {
    
    private transient Counter counter;
    private transient Meter meter;
    
    @Override
    public void open(Configuration parameters) throws Exception {
        counter = getRuntimeContext()
            .getMetricGroup()
            .counter("myCounter");
        
        meter = getRuntimeContext()
            .getMetricGroup()
            .meter("myMeter", new MeterView(60));
    }
    
    @Override
    public void processElement(String value, Context ctx, Collector<String> out) {
        counter.inc();
        meter.markEvent();
        out.collect(value);
    }
}
```

## 🎓 最佳实践

### 1. 代码规范

- 遵循阿里巴巴 Java 开发规范
- 类名使用 PascalCase
- 方法名使用 camelCase
- 常量全大写+下划线
- 添加完整的 Javadoc 注释

### 2. 异常处理

```java
/**
 * 异常处理最佳实践
 * 
 * @author erik.zhou
 */
try {
    // 业务逻辑
} catch (Exception e) {
    LOG.error("处理失败: {}", data, e);
    // 发送到死信队列或告警
    return Result.error();
}
```

### 3. 资源管理

```java
/**
 * 资源管理最佳实践
 * 
 * @author erik.zhou
 */
@Override
public void open(Configuration parameters) throws Exception {
    // 初始化资源
    redisClient = createRedisClient();
}

@Override
public void close() throws Exception {
    // 释放资源
    if (redisClient != null) {
        redisClient.close();
    }
}
```

## 📚 参考资料

- [Flink 官方文档](https://flink.apache.org/)
- [Flink CDC 文档](https://ververica.github.io/flink-cdc-connectors/)
- [Elasticsearch 官方文档](https://www.elastic.co/guide/)
- [阿里巴巴 Java 开发手册](https://github.com/alibaba/p3c)

## 🔗 相关教程

- [Flink 完整教程](./Flink-完整教程.md)
- [Kafka 完整教程](../../05-消息队列/Kafka-完整教程.md)
- [Elasticsearch 完整教程](../../04-数据库/Elasticsearch-完整教程.md)

---

> 本文档基于真实生产项目编写,涵盖了 Flink CDC 实时同步的核心技术点和最佳实践。
