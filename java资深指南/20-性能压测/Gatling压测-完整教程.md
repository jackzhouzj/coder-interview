 # Gatling 压测完整教程

> **版本**: Gatling 3.10.3 | **难度**: ⭐⭐⭐⭐ | **重要程度**: ⭐⭐⭐⭐⭐

## 📚 技术概述

### 什么是Gatling？

Gatling是一款基于Scala、Akka和Netty的高性能负载测试工具，以其优雅的DSL和精美的报告而闻名。

**核心特性**：
- 基于Scala DSL，代码即测试
- 异步非阻塞架构，性能极高
- 实时监控和精美的HTML报告
- 支持HTTP、WebSocket、SSE等协议
- 易于集成CI/CD

### 为什么选择Gatling？

**vs JMeter**:
- 性能更高（单机可支持更多并发）
- 代码化测试，易于版本控制
- 报告更精美，分析更直观
- 更适合DevOps和CI/CD

### 学习前置知识

- ✅ Scala基础（可选，会Java即可）
- ✅ HTTP协议
- ✅ Maven/Gradle基础

### 适用场景

- 高并发API压测
- 微服务性能测试
- CI/CD自动化测试
- WebSocket实时通信测试

## 🎯 学习目标

- [ ] 掌握Gatling的安装和配置
- [ ] 理解Gatling的DSL语法
- [ ] 能够编写压测脚本
- [ ] 掌握场景设计和数据注入
- [ ] 能够分析Gatling报告
- [ ] 会集成到CI/CD流程


## 📖 一、Gatling基础

### 1.1 安装配置

#### 方式1：使用Maven项目（推荐）

**pom.xml**:

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>gatling-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <gatling.version>3.10.3</gatling.version>
        <gatling-maven-plugin.version>4.7.0</gatling-maven-plugin.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>io.gatling.highcharts</groupId>
            <artifactId>gatling-charts-highcharts</artifactId>
            <version>${gatling.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>io.gatling</groupId>
                <artifactId>gatling-maven-plugin</artifactId>
                <version>${gatling-maven-plugin.version}</version>
            </plugin>
        </plugins>
    </build>
</project>
```

#### 方式2：使用Gradle项目

**build.gradle**:

```gradle
plugins {
    id 'scala'
    id 'io.gatling.gradle' version '3.10.3.1'
}

repositories {
    mavenCentral()
}

dependencies {
    gatling 'io.gatling.highcharts:gatling-charts-highcharts:3.10.3'
}
```

#### 方式3：独立安装

```bash
# 下载
wget https://repo1.maven.org/maven2/io/gatling/highcharts/gatling-charts-highcharts-bundle/3.10.3/gatling-charts-highcharts-bundle-3.10.3.zip

# 解压
unzip gatling-charts-highcharts-bundle-3.10.3.zip

# 运行
cd gatling-charts-highcharts-bundle-3.10.3
./bin/gatling.sh
```

### 1.2 项目结构

```
gatling-demo/
├── src/
│   └── test/
│       ├── scala/
│       │   └── simulations/
│       │       └── BasicSimulation.scala
│       └── resources/
│           ├── gatling.conf
│           ├── logback-test.xml
│           └── data/
│               └── users.csv
├── target/
│   └── gatling/
│       └── results/
└── pom.xml
```

### 1.3 第一个压测脚本

**BasicSimulation.scala**:

```scala
package simulations

import io.gatling.core.Predef._
import io.gatling.http.Predef._
import scala.concurrent.duration._

class BasicSimulation extends Simulation {

  // HTTP协议配置
  val httpProtocol = http
    .baseUrl("https://api.example.com")
    .acceptHeader("application/json")
    .contentTypeHeader("application/json")

  // 场景定义
  val scn = scenario("Basic Simulation")
    .exec(
      http("Get Users")
        .get("/api/users")
        .check(status.is(200))
    )

  // 负载注入
  setUp(
    scn.inject(
      atOnceUsers(10),           // 立即10个用户
      rampUsers(100).during(60)  // 60秒内逐步增加到100个用户
    )
  ).protocols(httpProtocol)
}
```

### 1.4 运行测试

```bash
# Maven
mvn gatling:test

# Gradle
gradle gatlingRun

# 独立版本
./bin/gatling.sh
```


## 💻 二、核心DSL语法

### 2.1 HTTP请求🔥

#### GET请求

```scala
exec(
  http("Get User")
    .get("/api/users/${userId}")
    .check(status.is(200))
)
```

#### POST请求

```scala
exec(
  http("Create User")
    .post("/api/users")
    .body(StringBody("""{"name":"John","email":"john@example.com"}"""))
    .asJson
    .check(status.is(201))
    .check(jsonPath("$.id").saveAs("userId"))
)
```

#### PUT请求

```scala
exec(
  http("Update User")
    .put("/api/users/${userId}")
    .body(StringBody("""{"name":"Jane"}"""))
    .asJson
    .check(status.is(200))
)
```

#### DELETE请求

```scala
exec(
  http("Delete User")
    .delete("/api/users/${userId}")
    .check(status.is(204))
)
```

### 2.2 请求配置

#### Headers

```scala
val httpProtocol = http
  .baseUrl("https://api.example.com")
  .acceptHeader("application/json")
  .contentTypeHeader("application/json")
  .userAgentHeader("Gatling/3.10")
  .header("Authorization", "Bearer ${token}")
```

#### Query Parameters

```scala
exec(
  http("Search Users")
    .get("/api/users")
    .queryParam("page", "1")
    .queryParam("size", "20")
    .queryParam("keyword", "${keyword}")
)
```

#### Form Data

```scala
exec(
  http("Login")
    .post("/api/login")
    .formParam("username", "${username}")
    .formParam("password", "${password}")
)
```

### 2.3 检查和提取🔥

#### 状态码检查

```scala
.check(status.is(200))
.check(status.in(200, 201, 204))
.check(status.not(404))
```

#### JSON提取

```scala
.check(jsonPath("$.data.token").saveAs("token"))
.check(jsonPath("$.data.userId").saveAs("userId"))
.check(jsonPath("$.data.items[*].id").findAll.saveAs("productIds"))
```

#### 正则提取

```scala
.check(regex("""userId":"(\d+)"""").saveAs("userId"))
```

#### 响应体检查

```scala
.check(bodyString.is("""{"status":"success"}"""))
.check(substring("success").exists)
```

### 2.4 会话管理

#### 保存变量

```scala
exec(session => {
  val userId = session("userId").as[String]
  println(s"User ID: $userId")
  session
})
```

#### 设置变量

```scala
exec(session => session.set("timestamp", System.currentTimeMillis()))
```

#### 使用EL表达式

```scala
exec(
  http("Get User")
    .get("/api/users/${userId}")
    .header("X-Request-ID", "${requestId}")
)
```


## 🔥 三、场景设计

### 3.1 数据注入（Feeders）🔥

#### CSV文件

**users.csv**:
```csv
username,password,email
user1,pass1,user1@example.com
user2,pass2,user2@example.com
user3,pass3,user3@example.com
```

**使用CSV**:
```scala
val feeder = csv("data/users.csv").random

val scn = scenario("Login")
  .feed(feeder)
  .exec(
    http("Login")
      .post("/api/login")
      .body(StringBody("""{"username":"${username}","password":"${password}"}"""))
      .asJson
  )
```

#### 自定义Feeder

```scala
val userFeeder = Iterator.continually(Map(
  "userId" -> Random.nextInt(10000),
  "username" -> s"user${Random.nextInt(1000)}",
  "email" -> s"user${Random.nextInt(1000)}@example.com"
))

val scn = scenario("Custom Feeder")
  .feed(userFeeder)
  .exec(...)
```

#### JSON Feeder

```scala
val jsonFeeder = jsonFile("data/products.json").random
```

### 3.2 负载模型🔥

#### 立即注入

```scala
setUp(
  scn.inject(
    atOnceUsers(100)  // 立即100个用户
  )
)
```

#### 渐进注入

```scala
setUp(
  scn.inject(
    rampUsers(1000).during(5.minutes)  // 5分钟内逐步增加到1000用户
  )
)
```

#### 恒定速率

```scala
setUp(
  scn.inject(
    constantUsersPerSec(20).during(10.minutes)  // 每秒20个用户，持续10分钟
  )
)
```

#### 阶梯式加压

```scala
setUp(
  scn.inject(
    incrementUsersPerSec(10)      // 每秒增加10个用户
      .times(5)                    // 重复5次
      .eachLevelLasting(30.seconds) // 每个阶段持续30秒
      .separatedByRampsLasting(10.seconds) // 阶段间隔10秒
      .startingFrom(10)            // 从每秒10个用户开始
  )
)
```

#### 复杂场景

```scala
setUp(
  scn.inject(
    nothingFor(4.seconds),                    // 等待4秒
    atOnceUsers(10),                          // 立即10个用户
    rampUsers(50).during(30.seconds),         // 30秒内增加到50个用户
    constantUsersPerSec(20).during(1.minute), // 每秒20个用户，持续1分钟
    rampUsersPerSec(10).to(20).during(2.minutes) // 2分钟内从每秒10个增加到20个
  )
)
```

### 3.3 场景组合

#### 多场景并发

```scala
val loginScn = scenario("Login").exec(...)
val browseScn = scenario("Browse").exec(...)
val purchaseScn = scenario("Purchase").exec(...)

setUp(
  loginScn.inject(rampUsers(100).during(1.minute)),
  browseScn.inject(rampUsers(500).during(2.minutes)),
  purchaseScn.inject(rampUsers(50).during(1.minute))
).protocols(httpProtocol)
```

#### 场景权重

```scala
val scn = scenario("Mixed")
  .randomSwitch(
    60.0 -> exec(http("Browse").get("/api/products")),
    30.0 -> exec(http("Search").get("/api/search")),
    10.0 -> exec(http("Purchase").post("/api/orders"))
  )
```

### 3.4 循环和条件

#### 循环

```scala
repeat(10) {
  exec(http("Get Product").get("/api/products/${productId}"))
}
```

#### 条件执行

```scala
doIf(session => session("status").as[String] == "success") {
  exec(http("Next Step").get("/api/next"))
}
```

#### While循环

```scala
asLongAs(session => session("hasMore").as[Boolean]) {
  exec(http("Get Page").get("/api/items?page=${page}"))
}
```


## 🚀 四、实战案例

### 4.1 电商完整流程压测

```scala
package simulations

import io.gatling.core.Predef._
import io.gatling.http.Predef._
import scala.concurrent.duration._

class EcommerceSimulation extends Simulation {

  val httpProtocol = http
    .baseUrl("https://api.ecommerce.com")
    .acceptHeader("application/json")
    .contentTypeHeader("application/json")

  // 用户数据
  val userFeeder = csv("data/users.csv").random
  val productFeeder = csv("data/products.csv").random

  // 登录场景
  val login = exec(
    http("Login")
      .post("/api/auth/login")
      .body(StringBody("""{"username":"${username}","password":"${password}"}"""))
      .asJson
      .check(status.is(200))
      .check(jsonPath("$.token").saveAs("token"))
  )

  // 浏览商品
  val browseProducts = exec(
    http("Get Products")
      .get("/api/products?page=1&size=20")
      .header("Authorization", "Bearer ${token}")
      .check(status.is(200))
  )
  .pause(2, 5) // 思考时间2-5秒

  // 查看商品详情
  val viewProduct = feed(productFeeder)
    .exec(
      http("Get Product Detail")
        .get("/api/products/${productId}")
        .header("Authorization", "Bearer ${token}")
        .check(status.is(200))
    )
    .pause(3, 8)

  // 加入购物车
  val addToCart = exec(
    http("Add to Cart")
      .post("/api/cart")
      .header("Authorization", "Bearer ${token}")
      .body(StringBody("""{"productId":"${productId}","quantity":1}"""))
      .asJson
      .check(status.is(200))
  )
  .pause(1, 3)

  // 提交订单
  val checkout = exec(
    http("Create Order")
      .post("/api/orders")
      .header("Authorization", "Bearer ${token}")
      .body(StringBody("""{"paymentMethod":"credit_card"}"""))
      .asJson
      .check(status.is(201))
      .check(jsonPath("$.orderId").saveAs("orderId"))
  )

  // 完整场景
  val scn = scenario("E-commerce Flow")
    .feed(userFeeder)
    .exec(login)
    .exec(browseProducts)
    .repeat(3) {
      exec(viewProduct)
    }
    .exec(addToCart)
    .exec(checkout)

  // 负载注入
  setUp(
    scn.inject(
      nothingFor(5.seconds),
      rampUsers(100).during(1.minute),
      constantUsersPerSec(50).during(5.minutes)
    )
  ).protocols(httpProtocol)
   .assertions(
     global.responseTime.max.lt(5000),
     global.successfulRequests.percent.gt(95)
   )
}
```

### 4.2 秒杀场景压测

```scala
class SeckillSimulation extends Simulation {

  val httpProtocol = http
    .baseUrl("https://api.seckill.com")
    .acceptHeader("application/json")

  val userFeeder = csv("data/users.csv").circular
  val seckillProductId = "12345"

  val scn = scenario("Seckill")
    .feed(userFeeder)
    // 登录
    .exec(
      http("Login")
        .post("/api/login")
        .body(StringBody("""{"username":"${username}","password":"${password}"}"""))
        .asJson
        .check(jsonPath("$.token").saveAs("token"))
    )
    // 等待秒杀开始
    .pause(1)
    // 秒杀抢购
    .exec(
      http("Seckill")
        .post(s"/api/seckill/$seckillProductId")
        .header("Authorization", "Bearer ${token}")
        .check(status.in(200, 400, 429))
    )

  setUp(
    scn.inject(
      // 模拟秒杀瞬间流量
      nothingFor(10.seconds),
      atOnceUsers(5000) // 瞬间5000个用户
    )
  ).protocols(httpProtocol)
   .maxDuration(1.minute)
}
```

### 4.3 API压测模板

```scala
class ApiLoadTest extends Simulation {

  val httpProtocol = http
    .baseUrl(System.getProperty("baseUrl", "https://api.example.com"))
    .acceptHeader("application/json")

  val users = Integer.getInteger("users", 100)
  val duration = Integer.getInteger("duration", 300)

  val scn = scenario("API Load Test")
    .exec(
      http("Health Check")
        .get("/health")
        .check(status.is(200))
    )
    .pause(1)
    .exec(
      http("API Request")
        .get("/api/endpoint")
        .check(status.is(200))
        .check(responseTimeInMillis.lt(1000))
    )

  setUp(
    scn.inject(
      rampUsers(users).during(duration.seconds)
    )
  ).protocols(httpProtocol)
   .assertions(
     global.responseTime.percentile3.lt(1000),
     global.successfulRequests.percent.gt(99)
   )
}
```

**运行命令**:
```bash
mvn gatling:test \
  -Dgatling.simulationClass=simulations.ApiLoadTest \
  -DbaseUrl=https://api.example.com \
  -Dusers=500 \
  -Dduration=600
```


## 📊 五、报告分析

### 5.1 报告结构

Gatling生成的HTML报告包含：

#### Global Information
- 总请求数
- 成功/失败请求数
- 最小/最大/平均/标准差响应时间
- 百分位数（50th, 75th, 95th, 99th）

#### Request Statistics
- 每个请求的详细统计
- 响应时间分布
- 成功率

#### Charts
- **Response Time Distribution**: 响应时间分布图
- **Response Time Percentiles**: 百分位数趋势
- **Requests per Second**: 每秒请求数
- **Responses per Second**: 每秒响应数

### 5.2 关键指标🔥

| 指标 | 说明 | 目标值 |
|------|------|--------|
| **Mean** | 平均响应时间 | <500ms |
| **Std Dev** | 标准差 | 越小越好 |
| **95th percentile** | 95%请求响应时间 | <1000ms |
| **99th percentile** | 99%请求响应时间 | <2000ms |
| **Requests/s** | 每秒请求数（TPS） | 越高越好 |
| **OK** | 成功率 | >99% |

### 5.3 断言（Assertions）

```scala
setUp(scn.inject(...))
  .protocols(httpProtocol)
  .assertions(
    // 全局断言
    global.responseTime.max.lt(5000),              // 最大响应时间<5秒
    global.responseTime.mean.lt(1000),             // 平均响应时间<1秒
    global.responseTime.percentile3.lt(2000),      // 95%响应时间<2秒
    global.successfulRequests.percent.gt(99),      // 成功率>99%
    global.requestsPerSec.gt(100),                 // TPS>100
    
    // 特定请求断言
    details("Login").responseTime.max.lt(3000),
    details("Create Order").successfulRequests.percent.is(100)
  )
```

### 5.4 实时监控

#### Graphite集成

**gatling.conf**:
```hocon
data {
  writers = [console, file, graphite]
  
  graphite {
    host = "localhost"
    port = 2003
    protocol = "tcp"
    rootPathPrefix = "gatling"
  }
}
```

#### InfluxDB集成

```scala
// 添加依赖
libraryDependencies += "io.gatling" % "gatling-influxdb" % "1.0.0"
```


## ✨ 六、最佳实践

### 6.1 代码组织

#### 提取HTTP配置

```scala
object HttpConfig {
  val baseHttpProtocol = http
    .baseUrl(System.getProperty("baseUrl", "https://api.example.com"))
    .acceptHeader("application/json")
    .contentTypeHeader("application/json")
    .userAgentHeader("Gatling/3.10")
}
```

#### 提取场景片段

```scala
object Scenarios {
  
  val login = exec(
    http("Login")
      .post("/api/login")
      .body(StringBody("""{"username":"${username}","password":"${password}"}"""))
      .asJson
      .check(jsonPath("$.token").saveAs("token"))
  )
  
  val logout = exec(
    http("Logout")
      .post("/api/logout")
      .header("Authorization", "Bearer ${token}")
  )
}

// 使用
val scn = scenario("User Flow")
  .exec(Scenarios.login)
  .exec(...)
  .exec(Scenarios.logout)
```

#### 提取Feeders

```scala
object Feeders {
  val users = csv("data/users.csv").random
  val products = csv("data/products.csv").circular
  
  val randomUser = Iterator.continually(Map(
    "userId" -> Random.nextInt(10000)
  ))
}
```

### 6.2 性能优化

#### 1. 使用连接池

```scala
val httpProtocol = http
  .shareConnections  // 共享连接
```

#### 2. 合理设置超时

```scala
val httpProtocol = http
  .requestTimeout(30.seconds)
  .readTimeout(30.seconds)
```

#### 3. 禁用不必要的检查

```scala
val httpProtocol = http
  .inferHtmlResources(BlackList(), WhiteList())  // 不加载静态资源
  .disableCaching                                 // 禁用缓存
```

#### 4. JVM参数优化

```bash
export JAVA_OPTS="-Xms2g -Xmx2g -XX:+UseG1GC"
```

### 6.3 CI/CD集成

#### Jenkins Pipeline

```groovy
pipeline {
    agent any
    
    stages {
        stage('Performance Test') {
            steps {
                sh '''
                    mvn gatling:test \
                        -Dgatling.simulationClass=simulations.LoadTest \
                        -DbaseUrl=${TEST_URL} \
                        -Dusers=500
                '''
            }
        }
        
        stage('Publish Report') {
            steps {
                gatlingArchive()
            }
        }
        
        stage('Check Assertions') {
            steps {
                script {
                    def exitCode = sh(
                        script: 'grep "There were no failed assertions" target/gatling/*/simulation.log',
                        returnStatus: true
                    )
                    if (exitCode != 0) {
                        error("Performance test failed!")
                    }
                }
            }
        }
    }
}
```

#### GitLab CI

```yaml
performance_test:
  stage: test
  image: maven:3.8-openjdk-17
  script:
    - mvn gatling:test -Dgatling.simulationClass=simulations.LoadTest
  artifacts:
    paths:
      - target/gatling/
    reports:
      performance: target/gatling/**/js/stats.json
  only:
    - master
```

#### GitHub Actions

```yaml
name: Performance Test

on:
  push:
    branches: [ main ]

jobs:
  gatling:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: Run Gatling Tests
        run: mvn gatling:test
      
      - name: Upload Report
        uses: actions/upload-artifact@v3
        with:
          name: gatling-report
          path: target/gatling/
```

### 6.4 常见陷阱

#### 陷阱1：忽略预热

```scala
// ❌ 错误：直接全量压测
setUp(scn.inject(atOnceUsers(1000)))

// ✅ 正确：逐步加压
setUp(scn.inject(
  rampUsers(100).during(30.seconds),  // 预热
  constantUsersPerSec(50).during(5.minutes)
))
```

#### 陷阱2：没有思考时间

```scala
// ❌ 错误：连续请求
exec(request1).exec(request2).exec(request3)

// ✅ 正确：添加思考时间
exec(request1)
  .pause(2, 5)
  .exec(request2)
  .pause(1, 3)
  .exec(request3)
```

#### 陷阱3：硬编码配置

```scala
// ❌ 错误：硬编码
val httpProtocol = http.baseUrl("https://api.example.com")

// ✅ 正确：参数化
val httpProtocol = http.baseUrl(System.getProperty("baseUrl"))
```


## ❓ 七、常见问题

### Q1: Gatling vs JMeter，如何选择？

**A**: 
- **选Gatling**：高并发场景、需要代码化测试、CI/CD集成
- **选JMeter**：团队不熟悉编程、需要GUI、功能测试为主

### Q2: 如何处理CSRF Token？

**A**: 
```scala
val scn = scenario("CSRF")
  .exec(
    http("Get Form")
      .get("/form")
      .check(css("input[name='_csrf']", "value").saveAs("csrfToken"))
  )
  .exec(
    http("Submit Form")
      .post("/submit")
      .formParam("_csrf", "${csrfToken}")
  )
```

### Q3: 如何模拟文件上传？

**A**:
```scala
exec(
  http("Upload File")
    .post("/api/upload")
    .bodyPart(RawFileBodyPart("file", "test.jpg")
      .contentType("image/jpeg"))
    .asMultipartForm
)
```

### Q4: 如何处理WebSocket？

**A**:
```scala
val wsProtocol = ws.baseUrl("ws://localhost:8080")

val scn = scenario("WebSocket")
  .exec(
    ws("Connect").connect("/ws")
  )
  .exec(
    ws("Send Message")
      .sendText("""{"type":"message","content":"Hello"}""")
  )
  .exec(
    ws("Wait for Response")
      .await(30.seconds)(
        ws.checkTextMessage("check")
          .check(jsonPath("$.status").is("ok"))
      )
  )
  .exec(ws("Close").close)
```

### Q5: 如何调试脚本？

**A**:
```scala
// 1. 打印会话变量
exec(session => {
  println(s"Token: ${session("token").as[String]}")
  session
})

// 2. 打印响应
.check(bodyString.saveAs("responseBody"))
.exec(session => {
  println(session("responseBody").as[String])
  session
})

// 3. 使用日志
import io.gatling.core.Predef._
exec(session => {
  logger.info(s"Current user: ${session("username").as[String]}")
  session
})
```

### Q6: 如何实现分布式压测？

**A**: Gatling Enterprise（商业版）支持分布式压测，开源版本可以：
```bash
# 在多台机器上同时运行
# 机器1
mvn gatling:test -Dgatling.simulationClass=Test -Dusers=500

# 机器2
mvn gatling:test -Dgatling.simulationClass=Test -Dusers=500

# 手动合并结果
```

### Q7: 如何设置代理？

**A**:
```scala
val httpProtocol = http
  .proxy(Proxy("proxy.example.com", 8080)
    .httpsPort(8443)
    .credentials("username", "password"))
```

### Q8: 报告中文乱码怎么办？

**A**:
```bash
# 设置JVM参数
export JAVA_OPTS="-Dfile.encoding=UTF-8"
```


## 🔗 八、相关资源

### 官方资源

- **官方网站**: https://gatling.io/
- **文档**: https://gatling.io/docs/gatling/
- **API文档**: https://gatling.io/docs/gatling/reference/current/core/
- **GitHub**: https://github.com/gatling/gatling

### 学习资源

- **官方教程**: https://gatling.io/docs/gatling/tutorials/quickstart/
- **示例项目**: https://github.com/gatling/gatling-maven-plugin-demo
- **视频教程**: https://www.youtube.com/c/GatlingTool

### 社区

- **Google Group**: https://groups.google.com/forum/#!forum/gatling
- **Gitter**: https://gitter.im/gatling/gatling
- **Stack Overflow**: 搜索 `gatling` 标签

### 插件和扩展

- **Gradle插件**: https://github.com/gatling/gatling-gradle-plugin
- **SBT插件**: https://github.com/gatling/gatling-sbt
- **Jenkins插件**: https://plugins.jenkins.io/gatling/

### 推荐阅读

- 《Gatling性能测试实战》
- 《高性能网站建设指南》
- Gatling官方博客

## 📝 九、学习检查清单

### 基础知识
- [ ] 理解Gatling的架构和优势
- [ ] 能够搭建Gatling项目
- [ ] 掌握基本的DSL语法
- [ ] 会编写简单的HTTP请求

### 进阶技能
- [ ] 掌握Feeders数据注入
- [ ] 会使用Check提取和验证
- [ ] 理解各种负载注入模型
- [ ] 能够设计复杂场景

### 高级应用
- [ ] 会分析Gatling报告
- [ ] 能够使用Assertions
- [ ] 掌握性能优化技巧
- [ ] 会集成到CI/CD

### 实战能力
- [ ] 能够设计真实业务场景
- [ ] 会处理认证和会话
- [ ] 能够定位性能瓶颈
- [ ] 掌握最佳实践

## 🎯 十、实战练习

### 练习1：基础API压测

**目标**: 压测REST API

**要求**:
- 使用Maven项目
- 100个并发用户
- 持续5分钟
- 生成HTML报告

### 练习2：登录流程压测

**目标**: 模拟用户登录

**要求**:
- 使用CSV文件参数化
- 提取token并在后续请求中使用
- 添加响应检查
- 成功率>99%

### 练习3：电商场景压测

**目标**: 完整的购物流程

**要求**:
- 登录→浏览→详情→加购→下单
- 添加思考时间
- 使用场景权重
- 分析性能瓶颈

### 练习4：CI/CD集成

**目标**: 集成到Jenkins

**要求**:
- 编写Jenkins Pipeline
- 参数化配置
- 自动发布报告
- 失败时告警

---

> @author erik.zhou
> 
> 最后更新：2025-02-04
