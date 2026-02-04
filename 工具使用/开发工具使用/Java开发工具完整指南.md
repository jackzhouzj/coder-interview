# Java 开发工具完整指南

> **作者**: @author erik.zhou  
> **适用场景**: Java 开发全流程工具链  
> **目标**: 掌握 Java 开发常用工具的使用方法和最佳实践

---

## 📋 目录

1. [IDE 工具](#1-ide-工具)
2. [构建工具](#2-构建工具)
3. [版本控制](#3-版本控制)
4. [代码质量工具](#4-代码质量工具)
5. [调试工具](#5-调试工具)
6. [性能分析工具](#6-性能分析工具)
7. [测试工具](#7-测试工具)
8. [数据库工具](#8-数据库工具)
9. [API 测试工具](#9-api-测试工具)
10. [容器化工具](#10-容器化工具)
11. [监控工具](#11-监控工具)
12. [文档工具](#12-文档工具)

---

## 1. IDE 工具

### 1.1 IntelliJ IDEA

#### 使用指南

**安装与配置**
```bash
# macOS 使用 Homebrew 安装
brew install --cask intellij-idea

# 或下载社区版（免费）
# https://www.jetbrains.com/idea/download/
```

**核心快捷键**

| 功能 | macOS | Windows/Linux |
|------|-------|---------------|
| 智能提示 | `Ctrl + Space` | `Ctrl + Space` |
| 快速修复 | `Option + Enter` | `Alt + Enter` |
| 查找类 | `Cmd + O` | `Ctrl + N` |
| 查找文件 | `Cmd + Shift + O` | `Ctrl + Shift + N` |
| 查找符号 | `Cmd + Option + O` | `Ctrl + Alt + Shift + N` |
| 重构重命名 | `Shift + F6` | `Shift + F6` |
| 格式化代码 | `Cmd + Option + L` | `Ctrl + Alt + L` |
| 优化导入 | `Ctrl + Option + O` | `Ctrl + Alt + O` |
| 运行 | `Ctrl + R` | `Shift + F10` |
| 调试 | `Ctrl + D` | `Shift + F9` |
| 查看方法调用层次 | `Ctrl + Option + H` | `Ctrl + Alt + H` |

#### 最佳实践

**1. 代码模板（Live Templates）**

创建常用代码模板：`Settings` → `Editor` → `Live Templates`

```java
// 快速创建单例模式：输入 singleton + Tab
public static final $CLASS_NAME$ INSTANCE = new $CLASS_NAME$();
private $CLASS_NAME$() {}

// 快速创建日志：输入 logger + Tab
private static final Logger logger = LoggerFactory.getLogger($CLASS_NAME$.class);

// 快速创建测试方法：输入 test + Tab
@Test
public void test$NAME$() {
    // given
    $END$
    
    // when
    
    // then
}
```

**2. 插件推荐**

```
必装插件：
- Lombok：简化 Java 代码
- Maven Helper：Maven 依赖分析
- SonarLint：代码质量检查
- GitToolBox：Git 增强
- Rainbow Brackets：彩虹括号
- String Manipulation：字符串处理
- Translation：翻译插件
- Alibaba Java Coding Guidelines：阿里巴巴代码规范检查

可选插件：
- JRebel：热部署
- MyBatisX：MyBatis 增强
- RestfulTool：接口测试
- SequenceDiagram：生成时序图
- JPA Buddy：JPA 开发助手
```

**3. 性能优化配置**

编辑 `idea.vmoptions`：

```properties
# 增加内存
-Xms2048m
-Xmx4096m
-XX:ReservedCodeCacheSize=512m

# 优化 GC
-XX:+UseG1GC
-XX:SoftRefLRUPolicyMSPerMB=50

# 减少 CPU 占用
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
```

**4. 代码检查配置**

`Settings` → `Editor` → `Inspections`

推荐启用：
- Java → Probable bugs（可能的 Bug）
- Java → Performance issues（性能问题）
- Java → Code style issues（代码风格）
- Java → Serialization issues（序列化问题）

---

### 1.2 VSCode

详见：[Java项目VSCODE配置标准流程.md](./Java项目VSCODE配置标准流程.md)

**推荐扩展**
```
核心扩展：
- Extension Pack for Java（Java 扩展包）
- Spring Boot Extension Pack（Spring Boot 扩展包）
- Debugger for Java（Java 调试器）
- Maven for Java（Maven 支持）
- Test Runner for Java（测试运行器）

增强扩展：
- SonarLint（代码质量）
- GitLens（Git 增强）
- REST Client（API 测试）
- Database Client（数据库客户端）
- Docker（容器支持）
```

---

## 2. 构建工具

### 2.1 Maven

#### 使用指南

**安装**
```bash
# macOS
brew install maven

# 验证安装
mvn -version
```

**核心命令**
```bash
# 清理项目
mvn clean

# 编译
mvn compile

# 测试
mvn test

# 打包
mvn package

# 安装到本地仓库
mvn install

# 部署到远程仓库
mvn deploy

# 跳过测试打包
mvn package -DskipTests

# 只编译测试代码，不运行
mvn test-compile

# 查看依赖树
mvn dependency:tree

# 分析依赖
mvn dependency:analyze

# 下载源码
mvn dependency:sources

# 清理并重新构建
mvn clean install -U
```

#### 最佳实践

**1. settings.xml 配置**

位置：`~/.m2/settings.xml` 或 `${MAVEN_HOME}/conf/settings.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
          http://maven.apache.org/xsd/settings-1.0.0.xsd">
    
    <!-- 本地仓库路径 -->
    <localRepository>/path/to/maven/repository</localRepository>
    
    <!-- 镜像配置（加速下载） -->
    <mirrors>
        <mirror>
            <id>aliyun-maven</id>
            <name>阿里云公共仓库</name>
            <url>https://maven.aliyun.com/repository/public</url>
            <mirrorOf>central</mirrorOf>
        </mirror>
    </mirrors>
    
    <!-- 私服配置 -->
    <servers>
        <server>
            <id>nexus-releases</id>
            <username>admin</username>
            <password>admin123</password>
        </server>
        <server>
            <id>nexus-snapshots</id>
            <username>admin</username>
            <password>admin123</password>
        </server>
    </servers>
    
    <!-- 全局配置 -->
    <profiles>
        <profile>
            <id>jdk-17</id>
            <activation>
                <activeByDefault>true</activeByDefault>
                <jdk>17</jdk>
            </activation>
            <properties>
                <maven.compiler.source>17</maven.compiler.source>
                <maven.compiler.target>17</maven.compiler.target>
                <maven.compiler.compilerVersion>17</maven.compiler.compilerVersion>
            </properties>
        </profile>
    </profiles>
</settings>
```

**2. pom.xml 最佳实践**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>demo-project</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    
    <name>Demo Project</name>
    <description>项目描述</description>
    
    <!-- 统一管理版本号 -->
    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        
        <!-- 依赖版本 -->
        <spring-boot.version>3.2.0</spring-boot.version>
        <lombok.version>1.18.30</lombok.version>
        <mysql.version>8.0.33</mysql.version>
    </properties>
    
    <!-- 依赖管理 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
    <dependencies>
        <!-- Spring Boot Starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>
        
        <!-- 测试依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <!-- Spring Boot 打包插件 -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            
            <!-- 编译插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <encoding>${project.build.sourceEncoding}</encoding>
                </configuration>
            </plugin>
            
            <!-- 测试插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0</version>
            </plugin>
        </plugins>
    </build>
    
    <!-- 私服配置 -->
    <distributionManagement>
        <repository>
            <id>nexus-releases</id>
            <name>Nexus Release Repository</name>
            <url>http://nexus.example.com/repository/maven-releases/</url>
        </repository>
        <snapshotRepository>
            <id>nexus-snapshots</id>
            <name>Nexus Snapshot Repository</name>
            <url>http://nexus.example.com/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
</project>
```

**3. 常见问题解决**

```bash
# 依赖冲突解决
mvn dependency:tree -Dverbose -Dincludes=<groupId>:<artifactId>

# 强制更新依赖
mvn clean install -U

# 清理本地仓库损坏的依赖
find ~/.m2/repository -name "*.lastUpdated" -delete

# 查看有效 POM
mvn help:effective-pom

# 查看有效 settings
mvn help:effective-settings
```

---

### 2.2 Gradle

#### 使用指南

**安装**
```bash
# macOS
brew install gradle

# 验证安装
gradle -version
```

**核心命令**
```bash
# 清理
gradle clean

# 编译
gradle compileJava

# 测试
gradle test

# 构建
gradle build

# 跳过测试构建
gradle build -x test

# 查看依赖
gradle dependencies

# 查看任务
gradle tasks

# 运行应用
gradle bootRun
```

#### 最佳实践

**1. build.gradle 配置（Kotlin DSL）**

```kotlin
plugins {
    java
    id("org.springframework.boot") version "3.2.0"
    id("io.spring.dependency-management") version "1.1.4"
}

group = "com.example"
version = "1.0.0-SNAPSHOT"

java {
    sourceCompatibility = JavaVersion.VERSION_17
}

configurations {
    compileOnly {
        extendsFrom(configurations.annotationProcessor.get())
    }
}

repositories {
    mavenCentral()
    maven { url = uri("https://maven.aliyun.com/repository/public") }
}

dependencies {
    // Spring Boot
    implementation("org.springframework.boot:spring-boot-starter-web")
    
    // Lombok
    compileOnly("org.projectlombok:lombok")
    annotationProcessor("org.projectlombok:lombok")
    
    // 测试
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

**2. Gradle Wrapper 使用**

```bash
# 生成 Wrapper
gradle wrapper --gradle-version 8.5

# 使用 Wrapper（推荐）
./gradlew build
./gradlew test
./gradlew bootRun
```

---

## 3. 版本控制

### 3.1 Git

#### 使用指南

**常用命令**
```bash
# 初始化仓库
git init

# 克隆仓库
git clone <url>

# 查看状态
git status

# 添加文件
git add .
git add <file>

# 提交
git commit -m "提交信息"

# 推送
git push origin main

# 拉取
git pull origin main

# 创建分支
git checkout -b feature/new-feature

# 切换分支
git checkout main

# 合并分支
git merge feature/new-feature

# 查看日志
git log --oneline --graph --all

# 暂存修改
git stash
git stash pop

# 撤销修改
git checkout -- <file>

# 回退版本
git reset --hard HEAD^
git reset --soft HEAD^

# 查看差异
git diff
git diff --staged
```

#### 最佳实践

**1. .gitignore 配置**

```gitignore
# Java
*.class
*.jar
*.war
*.ear
*.log

# Maven
target/
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
pom.xml.next
release.properties

# Gradle
.gradle/
build/

# IDE
.idea/
*.iml
.vscode/
.settings/
.classpath
.project

# OS
.DS_Store
Thumbs.db

# 日志
logs/
*.log

# 临时文件
*.tmp
*.bak
*.swp
*~
```

**2. Git 提交规范**

```
格式：<type>(<scope>): <subject>

type 类型：
- feat: 新功能
- fix: 修复 Bug
- docs: 文档更新
- style: 代码格式（不影响代码运行）
- refactor: 重构
- perf: 性能优化
- test: 测试相关
- chore: 构建过程或辅助工具的变动

示例：
feat(user): 添加用户登录功能
fix(order): 修复订单计算错误
docs(readme): 更新安装文档
refactor(service): 重构用户服务层代码
```

**3. Git Flow 工作流**

```bash
# 主分支
main/master    # 生产环境
develop        # 开发环境

# 功能分支
feature/*      # 新功能开发
git checkout -b feature/user-login develop

# 发布分支
release/*      # 发布准备
git checkout -b release/1.0.0 develop

# 修复分支
hotfix/*       # 紧急修复
git checkout -b hotfix/critical-bug main

# 完成功能开发
git checkout develop
git merge --no-ff feature/user-login
git branch -d feature/user-login
git push origin develop
```

---

## 4. 代码质量工具

### 4.1 SonarQube

#### 使用指南

**Docker 快速启动**
```bash
# 启动 SonarQube
docker run -d --name sonarqube \
  -p 9000:9000 \
  -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
  sonarqube:latest

# 访问：http://localhost:9000
# 默认账号：admin/admin
```

**Maven 集成**
```xml
<plugin>
    <groupId>org.sonarsource.scanner.maven</groupId>
    <artifactId>sonar-maven-plugin</artifactId>
    <version>3.10.0.2594</version>
</plugin>
```

**执行扫描**
```bash
mvn clean verify sonar:sonar \
  -Dsonar.projectKey=my-project \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=<token>
```

#### 最佳实践

**1. 质量门禁配置**

```
推荐规则：
- 代码覆盖率 ≥ 80%
- 重复代码率 ≤ 3%
- 主要问题数 = 0
- 次要问题数 ≤ 10
- 技术债务比率 ≤ 5%
```

**2. 排除规则**

创建 `sonar-project.properties`：

```properties
sonar.projectKey=my-project
sonar.projectName=My Project
sonar.projectVersion=1.0.0

# 源码目录
sonar.sources=src/main/java
sonar.tests=src/test/java

# 排除文件
sonar.exclusions=**/generated/**,**/dto/**,**/entity/**

# Java 版本
sonar.java.source=17
sonar.java.target=17

# 编码
sonar.sourceEncoding=UTF-8
```

---

### 4.2 Checkstyle

#### 使用指南

**Maven 集成**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>3.3.1</version>
    <configuration>
        <configLocation>checkstyle.xml</configLocation>
        <encoding>UTF-8</encoding>
        <consoleOutput>true</consoleOutput>
        <failsOnError>true</failsOnError>
    </configuration>
    <executions>
        <execution>
            <id>validate</id>
            <phase>validate</phase>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

**执行检查**
```bash
mvn checkstyle:check
```

---

### 4.3 SpotBugs

#### 使用指南

**Maven 集成**
```xml
<plugin>
    <groupId>com.github.spotbugs</groupId>
    <artifactId>spotbugs-maven-plugin</artifactId>
    <version>4.8.2.0</version>
    <configuration>
        <effort>Max</effort>
        <threshold>Low</threshold>
        <xmlOutput>true</xmlOutput>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

**执行检查**
```bash
mvn spotbugs:check
```

---

## 5. 调试工具

### 5.1 IDEA 调试器

#### 使用指南

**断点类型**
```
1. 行断点：点击行号左侧
2. 方法断点：在方法签名行设置
3. 异常断点：Run → View Breakpoints → + → Java Exception Breakpoints
4. 条件断点：右键断点 → Condition
5. 字段断点：在字段声明行设置
```

**调试快捷键**

| 功能 | macOS | Windows/Linux |
|------|-------|---------------|
| 单步执行（进入方法） | `F7` | `F7` |
| 单步执行（跳过方法） | `F8` | `F8` |
| 继续执行 | `Cmd + Option + R` | `F9` |
| 执行到光标处 | `Option + F9` | `Alt + F9` |
| 计算表达式 | `Option + F8` | `Alt + F8` |
| 查看断点 | `Cmd + Shift + F8` | `Ctrl + Shift + F8` |

#### 最佳实践

**1. 条件断点**
```java
// 只在特定条件下暂停
// 右键断点 → Condition → 输入条件
userId.equals("admin")
list.size() > 100
```

**2. 日志断点**
```java
// 不暂停程序，只打印日志
// 右键断点 → 取消勾选 Suspend → 勾选 Evaluate and log
"User ID: " + userId + ", Name: " + userName
```

**3. 远程调试**
```bash
# 启动应用时添加 JVM 参数
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 -jar app.jar

# IDEA 配置：Run → Edit Configurations → + → Remote JVM Debug
# Host: localhost
# Port: 5005
```

---

### 5.2 Arthas

#### 使用指南

**安装与启动**
```bash
# 下载
curl -O https://arthas.aliyun.com/arthas-boot.jar

# 启动（选择要诊断的 Java 进程）
java -jar arthas-boot.jar
```

**常用命令**
```bash
# 查看 JVM 信息
dashboard

# 查看线程信息
thread
thread -n 3  # 查看 CPU 占用最高的 3 个线程
thread <id>  # 查看指定线程堆栈

# 反编译
jad com.example.UserService

# 查看方法调用
watch com.example.UserService getUser "{params,returnObj}" -x 2

# 追踪方法调用
trace com.example.UserService getUser

# 查看方法耗时
monitor -c 5 com.example.UserService getUser

# 查看 JVM 参数
sysprop
sysenv

# 查看类加载信息
sc -d com.example.UserService
sm com.example.UserService  # 查看方法

# 退出
quit
```

#### 最佳实践

**1. 线上问题排查**
```bash
# 1. 查看 CPU 占用高的线程
thread -n 3

# 2. 查看线程堆栈
thread <thread-id>

# 3. 查看方法调用耗时
trace com.example.OrderService createOrder

# 4. 查看方法参数和返回值
watch com.example.OrderService createOrder "{params,returnObj,throwExp}" -x 2
```

**2. 热更新代码**
```bash
# 1. 反编译查看当前代码
jad com.example.UserService

# 2. 修改代码后重新编译
javac -cp /path/to/lib/* UserService.java

# 3. 重新加载类
redefine /path/to/UserService.class
```

---

## 6. 性能分析工具

### 6.1 JProfiler

#### 使用指南

**安装**
```bash
# macOS
brew install --cask jprofiler

# 或下载：https://www.ej-technologies.com/products/jprofiler/overview.html
```

**核心功能**
```
1. CPU 分析：查看方法调用耗时
2. 内存分析：查看对象分配和内存泄漏
3. 线程分析：查看线程状态和死锁
4. 数据库分析：查看 SQL 执行情况
5. 实时监控：查看 JVM 实时状态
```

#### 最佳实践

**1. 内存泄漏排查**
```
步骤：
1. 启动应用并连接 JProfiler
2. 运行一段时间后，点击 "Record Objects"
3. 执行 GC：Memory → Run GC
4. 查看 "Biggest Objects"
5. 分析对象引用链：右键对象 → Show Selection in Heap Walker
```

**2. CPU 性能分析**
```
步骤：
1. 点击 "Start CPU Recording"
2. 执行需要分析的操作
3. 点击 "Stop CPU Recording"
4. 查看 "Hot Spots" 找出耗时最多的方法
5. 查看 "Call Tree" 分析调用链
```

---

### 6.2 VisualVM

#### 使用指南

**安装**
```bash
# macOS
brew install --cask visualvm

# 或使用 JDK 自带的（JDK 8）
jvisualvm
```

**核心功能**
```bash
# 监控：实时查看 CPU、内存、线程、类加载
# 线程：查看线程状态和死锁检测
# 采样：CPU 和内存采样分析
# 堆转储：生成和分析堆转储文件
```

#### 最佳实践

**1. 生成堆转储**
```bash
# 方式 1：VisualVM 界面操作
# 右键应用 → Heap Dump

# 方式 2：命令行
jmap -dump:format=b,file=heap.hprof <pid>

# 分析堆转储
jhat heap.hprof
# 访问：http://localhost:7000
```

**2. 线程死锁检测**
```bash
# VisualVM：Threads → Thread Dump
# 或命令行
jstack <pid> > thread.txt
```

---

### 6.3 JMeter

#### 使用指南

**安装**
```bash
# macOS
brew install jmeter

# 启动
jmeter
```

**创建性能测试**
```
1. 添加线程组：右键 Test Plan → Add → Threads → Thread Group
   - Number of Threads: 100（并发用户数）
   - Ramp-Up Period: 10（启动时间）
   - Loop Count: 10（循环次数）

2. 添加 HTTP 请求：右键 Thread Group → Add → Sampler → HTTP Request
   - Server Name: localhost
   - Port: 8080
   - Path: /api/users

3. 添加监听器：右键 Thread Group → Add → Listener
   - View Results Tree（查看结果树）
   - Summary Report（汇总报告）
   - Aggregate Report（聚合报告）

4. 运行测试：点击绿色播放按钮
```

#### 最佳实践

**1. 参数化测试**
```
1. 创建 CSV 文件：users.csv
userId,userName
1,user1
2,user2

2. 添加 CSV Data Set Config
   - Filename: users.csv
   - Variable Names: userId,userName

3. 在 HTTP 请求中使用：${userId}
```

**2. 断言验证**
```
右键 HTTP Request → Add → Assertions → Response Assertion
- Response Code: 200
- Response Message: OK
```

---

## 7. 测试工具

### 7.1 JUnit 5

#### 使用指南

**Maven 依赖**
```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.1</version>
    <scope>test</scope>
</dependency>
```

**基本测试**
```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

@DisplayName("用户服务测试")
class UserServiceTest {
    
    private UserService userService;
    
    @BeforeAll
    static void initAll() {
        // 所有测试前执行一次
    }
    
    @BeforeEach
    void init() {
        // 每个测试前执行
        userService = new UserService();
    }
    
    @Test
    @DisplayName("测试获取用户")
    void testGetUser() {
        // given
        Long userId = 1L;
        
        // when
        User user = userService.getUser(userId);
        
        // then
        assertNotNull(user);
        assertEquals("张三", user.getName());
    }
    
    @Test
    @DisplayName("测试用户不存在")
    void testUserNotFound() {
        assertThrows(UserNotFoundException.class, () -> {
            userService.getUser(999L);
        });
    }
    
    @ParameterizedTest
    @ValueSource(strings = {"admin", "user", "guest"})
    @DisplayName("参数化测试")
    void testMultipleUsers(String username) {
        assertTrue(username.length() > 0);
    }
    
    @AfterEach
    void tearDown() {
        // 每个测试后执行
    }
    
    @AfterAll
    static void tearDownAll() {
        // 所有测试后执行一次
    }
}
```

#### 最佳实践

**1. 测试命名规范**
```java
// 方法名：should_ExpectedBehavior_When_StateUnderTest
@Test
void should_ReturnUser_When_UserExists() {
    // ...
}

@Test
void should_ThrowException_When_UserNotFound() {
    // ...
}
```

**2. 使用 AssertJ**
```xml
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.24.2</version>
    <scope>test</scope>
</dependency>
```

```java
import static org.assertj.core.api.Assertions.*;

@Test
void testUser() {
    User user = userService.getUser(1L);
    
    assertThat(user)
        .isNotNull()
        .extracting("name", "age")
        .containsExactly("张三", 25);
    
    assertThat(user.getOrders())
        .hasSize(3)
        .extracting("orderNo")
        .contains("ORDER001", "ORDER002");
}
```

---

### 7.2 Mockito

#### 使用指南

**Maven 依赖**
```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.8.0</version>
    <scope>test</scope>
</dependency>
```

**基本使用**
```java
import org.mockito.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private OrderRepository orderRepository;
    
    @InjectMocks
    private OrderService orderService;
    
    @Test
    void testCreateOrder() {
        // given
        User user = new User(1L, "张三");
        when(userRepository.findById(1L)).thenReturn(Optional.of(user));
        
        Order order = new Order();
        when(orderRepository.save(any(Order.class))).thenReturn(order);
        
        // when
        Order result = orderService.createOrder(1L, "商品A");
        
        // then
        assertNotNull(result);
        verify(userRepository, times(1)).findById(1L);
        verify(orderRepository, times(1)).save(any(Order.class));
    }
    
    @Test
    void testCreateOrder_UserNotFound() {
        // given
        when(userRepository.findById(999L)).thenReturn(Optional.empty());
        
        // when & then
        assertThrows(UserNotFoundException.class, () -> {
            orderService.createOrder(999L, "商品A");
        });
        
        verify(orderRepository, never()).save(any(Order.class));
    }
}
```

#### 最佳实践

**1. 参数匹配器**
```java
// 精确匹配
when(userRepository.findById(1L)).thenReturn(user);

// 任意参数
when(userRepository.findById(anyLong())).thenReturn(user);

// 参数捕获
ArgumentCaptor<Order> captor = ArgumentCaptor.forClass(Order.class);
verify(orderRepository).save(captor.capture());
Order savedOrder = captor.getValue();
assertEquals("ORDER001", savedOrder.getOrderNo());
```

**2. 异常模拟**
```java
when(userRepository.findById(anyLong()))
    .thenThrow(new RuntimeException("数据库连接失败"));
```

---

## 8. 数据库工具

### 8.1 DBeaver

#### 使用指南

**安装**
```bash
# macOS
brew install --cask dbeaver-community

# 或下载：https://dbeaver.io/download/
```

**连接数据库**
```
1. 点击 "新建连接"
2. 选择数据库类型（MySQL、PostgreSQL、Oracle 等）
3. 填写连接信息：
   - Host: localhost
   - Port: 3306
   - Database: mydb
   - Username: root
   - Password: ******
4. 测试连接
5. 完成
```

#### 最佳实践

**1. SQL 格式化**
```sql
-- 选中 SQL → 右键 → Format → Format SQL
-- 或快捷键：Ctrl + Shift + F（Windows）/ Cmd + Shift + F（macOS）
```

**2. 数据导出**
```
1. 右键表 → Export Data
2. 选择格式：CSV、JSON、XML、SQL
3. 配置导出选项
4. 导出
```

**3. ER 图生成**
```
1. 右键数据库 → View Diagram
2. 自动生成表关系图
3. 可导出为图片
```

---

### 8.2 Flyway（数据库版本管理）

#### 使用指南

**Maven 依赖**
```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
    <version>10.4.1</version>
</dependency>
```

**配置**
```yaml
# application.yml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
```

**创建迁移脚本**
```
目录结构：
src/main/resources/db/migration/
├── V1__Create_user_table.sql
├── V2__Add_email_to_user.sql
└── V3__Create_order_table.sql

命名规则：V{version}__{description}.sql
```

**迁移脚本示例**
```sql
-- V1__Create_user_table.sql
CREATE TABLE `user` (
    `id` BIGINT NOT NULL AUTO_INCREMENT,
    `username` VARCHAR(50) NOT NULL,
    `password` VARCHAR(100) NOT NULL,
    `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    UNIQUE KEY `uk_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- V2__Add_email_to_user.sql
ALTER TABLE `user` ADD COLUMN `email` VARCHAR(100) NULL AFTER `username`;
```

#### 最佳实践

**1. 版本号规范**
```
V1.0.0__Initial_schema.sql
V1.0.1__Add_user_status.sql
V1.1.0__Create_order_module.sql
V2.0.0__Refactor_database.sql
```

**2. 回滚策略**
```sql
-- Flyway 不支持自动回滚，需要手动创建回滚脚本
-- U2__Rollback_add_email.sql
ALTER TABLE `user` DROP COLUMN `email`;
```

---

## 9. API 测试工具

### 9.1 Postman

#### 使用指南

**安装**
```bash
# macOS
brew install --cask postman

# 或下载：https://www.postman.com/downloads/
```

**基本使用**
```
1. 创建请求
   - Method: GET/POST/PUT/DELETE
   - URL: http://localhost:8080/api/users
   - Headers: Content-Type: application/json
   - Body: {"name": "张三", "age": 25}

2. 发送请求
   - 点击 Send 按钮
   - 查看响应结果

3. 保存请求
   - Save → 输入名称 → 选择集合
```

#### 最佳实践

**1. 环境变量**
```
创建环境：
- dev: http://localhost:8080
- test: http://test.example.com
- prod: http://api.example.com

使用变量：
{{baseUrl}}/api/users
```

**2. 测试脚本**
```javascript
// Tests 标签页
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has user data", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.name).to.eql("张三");
    pm.expect(jsonData.age).to.be.above(0);
});

// 保存响应数据到环境变量
var jsonData = pm.response.json();
pm.environment.set("userId", jsonData.id);
```

**3. 批量测试**
```
1. 创建 Collection
2. 添加多个请求
3. 点击 Collection → Run
4. 查看测试报告
```

---

### 9.2 Swagger/OpenAPI

#### 使用指南

**Maven 依赖（SpringDoc）**
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

**配置**
```yaml
# application.yml
springdoc:
  api-docs:
    path: /v3/api-docs
  swagger-ui:
    path: /swagger-ui.html
    enabled: true
```

**使用注解**
```java
@RestController
@RequestMapping("/api/users")
@Tag(name = "用户管理", description = "用户相关接口")
public class UserController {
    
    @Operation(summary = "获取用户", description = "根据用户ID获取用户信息")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "成功"),
        @ApiResponse(responseCode = "404", description = "用户不存在")
    })
    @GetMapping("/{id}")
    public Result<User> getUser(
        @Parameter(description = "用户ID", required = true)
        @PathVariable Long id
    ) {
        return Result.success(userService.getUser(id));
    }
    
    @Operation(summary = "创建用户")
    @PostMapping
    public Result<User> createUser(
        @io.swagger.v3.oas.annotations.parameters.RequestBody(
            description = "用户信息",
            required = true
        )
        @RequestBody @Valid UserDTO userDTO
    ) {
        return Result.success(userService.createUser(userDTO));
    }
}
```

**访问文档**
```
Swagger UI: http://localhost:8080/swagger-ui.html
OpenAPI JSON: http://localhost:8080/v3/api-docs
```

---

## 10. 容器化工具

### 10.1 Docker

#### 使用指南

**安装**
```bash
# macOS
brew install --cask docker

# 验证
docker --version
```

**常用命令**

```bash
# 镜像操作
docker images                    # 查看镜像
docker pull <image>              # 拉取镜像
docker build -t <name> .         # 构建镜像
docker rmi <image>               # 删除镜像
docker tag <image> <new-name>    # 标记镜像

# 容器操作
docker ps                        # 查看运行中的容器
docker ps -a                     # 查看所有容器
docker run <image>               # 运行容器
docker start <container>         # 启动容器
docker stop <container>          # 停止容器
docker restart <container>       # 重启容器
docker rm <container>            # 删除容器
docker logs <container>          # 查看日志
docker exec -it <container> bash # 进入容器

# 网络操作
docker network ls                # 查看网络
docker network create <name>     # 创建网络
docker network connect <network> <container>  # 连接网络

# 数据卷操作
docker volume ls                 # 查看数据卷
docker volume create <name>      # 创建数据卷
docker volume rm <name>          # 删除数据卷
```

#### 最佳实践

**1. Dockerfile 编写**

```dockerfile
# 多阶段构建
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar

# 添加非 root 用户
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# 暴露端口
EXPOSE 8080

# 启动命令
ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-Djava.security.egd=file:/dev/./urandom", \
  "-jar", "app.jar"]
```

**2. docker-compose.yml**

```yaml
version: '3.8'

services:
  # MySQL 数据库
  mysql:
    image: mysql:8.0
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: mydb
      TZ: Asia/Shanghai
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis 缓存
  redis:
    image: redis:7-alpine
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - app-network
    command: redis-server --appendonly yes

  # Spring Boot 应用
  app:
    build: .
    container_name: spring-app
    environment:
      SPRING_PROFILES_ACTIVE: prod
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/mydb
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: root123
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PORT: 6379
    ports:
      - "8080:8080"
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - app-network
    restart: unless-stopped

volumes:
  mysql-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

**3. 常用命令组合**

```bash
# 构建并启动
docker-compose up -d --build

# 查看日志
docker-compose logs -f app

# 停止并删除
docker-compose down

# 停止并删除（包括数据卷）
docker-compose down -v

# 重启服务
docker-compose restart app

# 进入容器
docker-compose exec app bash
```

---

### 10.2 Kubernetes

#### 使用指南

**安装 kubectl**
```bash
# macOS
brew install kubectl

# 验证
kubectl version --client
```

**常用命令**
```bash
# 集群信息
kubectl cluster-info
kubectl get nodes

# Pod 操作
kubectl get pods
kubectl get pods -n <namespace>
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs -f <pod-name>
kubectl exec -it <pod-name> -- bash

# Deployment 操作
kubectl get deployments
kubectl create deployment <name> --image=<image>
kubectl scale deployment <name> --replicas=3
kubectl rollout status deployment/<name>
kubectl rollout undo deployment/<name>

# Service 操作
kubectl get services
kubectl expose deployment <name> --port=8080 --type=LoadBalancer

# 配置操作
kubectl apply -f <file.yaml>
kubectl delete -f <file.yaml>
kubectl get all
```

#### 最佳实践

**1. Deployment 配置**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
  labels:
    app: spring-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: spring-app
        image: myregistry/spring-app:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: SPRING_DATASOURCE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database.url
        - name: SPRING_DATASOURCE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: database.password
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 20
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: spring-app-service
spec:
  selector:
    app: spring-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

**2. ConfigMap 和 Secret**

```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.url: "jdbc:mysql://mysql:3306/mydb"
  redis.host: "redis"
  redis.port: "6379"

---
# Secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  database.password: cm9vdDEyMw==  # base64 编码
  redis.password: cmVkaXMxMjM=
```

---

## 11. 监控工具

### 11.1 Prometheus + Grafana

#### 使用指南

**Docker Compose 部署**

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
    networks:
      - monitoring
    depends_on:
      - prometheus

volumes:
  prometheus-data:
  grafana-data:

networks:
  monitoring:
    driver: bridge
```

**prometheus.yml 配置**

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']
```

**Spring Boot 集成**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

#### 最佳实践

**1. Grafana 仪表板配置**

```
1. 访问 Grafana：http://localhost:3000
2. 登录：admin/admin
3. 添加数据源：Configuration → Data Sources → Add Prometheus
   - URL: http://prometheus:9090
4. 导入仪表板：
   - JVM (Micrometer): 4701
   - Spring Boot: 12900
```

**2. 自定义指标**

```java
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Service;

@Service
public class OrderService {
    
    private final Counter orderCounter;
    
    public OrderService(MeterRegistry registry) {
        this.orderCounter = Counter.builder("orders.created")
            .description("订单创建数量")
            .tag("type", "online")
            .register(registry);
    }
    
    public void createOrder(Order order) {
        // 业务逻辑
        orderCounter.increment();
    }
}
```

---

### 11.2 ELK Stack（日志管理）

#### 使用指南

**Docker Compose 部署**

```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - es-data:/usr/share/elasticsearch/data
    networks:
      - elk

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    container_name: logstash
    ports:
      - "5000:5000"
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    networks:
      - elk
    depends_on:
      - elasticsearch

volumes:
  es-data:

networks:
  elk:
    driver: bridge
```

**Logback 配置**

```xml
<!-- logback-spring.xml -->
<configuration>
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>localhost:5000</destination>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <customFields>{"app":"spring-app","env":"prod"}</customFields>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="LOGSTASH" />
    </root>
</configuration>
```

---

## 12. 文档工具

### 12.1 Maven Site

#### 使用指南

**pom.xml 配置**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-site-plugin</artifactId>
            <version>4.0.0-M11</version>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-project-info-reports-plugin</artifactId>
            <version>3.5.0</version>
        </plugin>
    </plugins>
</build>

<reporting>
    <plugins>
        <!-- Javadoc -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <version>3.6.3</version>
        </plugin>
        
        <!-- 测试报告 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-report-plugin</artifactId>
            <version>3.2.3</version>
        </plugin>
        
        <!-- 代码覆盖率 -->
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.11</version>
        </plugin>
    </plugins>
</reporting>
```

**生成文档**

```bash
# 生成站点
mvn site

# 查看文档
open target/site/index.html
```

---

### 12.2 Javadoc

#### 使用指南

**生成 Javadoc**

```bash
# Maven
mvn javadoc:javadoc

# Gradle
gradle javadoc

# 查看
open target/site/apidocs/index.html
```

#### 最佳实践

**1. 注释规范**

```java
/**
 * 用户服务类
 * <p>
 * 提供用户相关的业务操作，包括用户查询、创建、更新和删除
 * </p>
 *
 * @author erik.zhou
 * @version 1.0.0
 * @since 2024-01-01
 */
@Service
public class UserService {
    
    /**
     * 根据用户ID获取用户信息
     *
     * @param userId 用户ID，不能为空
     * @return 用户信息
     * @throws UserNotFoundException 当用户不存在时抛出
     * @throws IllegalArgumentException 当用户ID为空时抛出
     */
    public User getUser(@NonNull Long userId) {
        if (userId == null) {
            throw new IllegalArgumentException("用户ID不能为空");
        }
        
        return userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException("用户不存在: " + userId));
    }
    
    /**
     * 创建新用户
     *
     * @param userDTO 用户信息DTO
     * @return 创建成功的用户
     * @see UserDTO
     * @deprecated 使用 {@link #createUserV2(UserDTO)} 替代
     */
    @Deprecated
    public User createUser(UserDTO userDTO) {
        // ...
    }
}
```

---

## 13. 工具链整合最佳实践

### 13.1 开发环境配置

**推荐工具组合**

```
IDE: IntelliJ IDEA Ultimate / VSCode
构建: Maven 3.9+ / Gradle 8+
版本控制: Git 2.40+
数据库: DBeaver / DataGrip
API 测试: Postman / Swagger UI
容器: Docker Desktop
监控: Prometheus + Grafana
日志: ELK Stack
```

### 13.2 CI/CD 流程

**GitLab CI 示例**

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - quality
  - deploy

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"

cache:
  paths:
    - .m2/repository

build:
  stage: build
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn clean compile
  artifacts:
    paths:
      - target/

test:
  stage: test
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn test
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml

quality:
  stage: quality
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn sonar:sonar -Dsonar.host.url=$SONAR_URL -Dsonar.login=$SONAR_TOKEN
  only:
    - main

deploy:
  stage: deploy
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
  only:
    - tags
```

---

## 14. 总结

### 14.1 工具选择建议

| 场景 | 推荐工具 | 备选方案 |
|------|----------|----------|
| IDE | IntelliJ IDEA | VSCode + 扩展 |
| 构建 | Maven | Gradle |
| 版本控制 | Git | - |
| 代码质量 | SonarQube | Checkstyle + SpotBugs |
| 调试 | IDEA Debugger | Arthas |
| 性能分析 | JProfiler | VisualVM |
| 测试 | JUnit 5 + Mockito | TestNG |
| 数据库 | DBeaver | DataGrip |
| API 测试 | Postman | Swagger UI |
| 容器 | Docker | Podman |
| 监控 | Prometheus + Grafana | SkyWalking |
| 日志 | ELK Stack | Loki |

### 14.2 学习路径

```
1. 基础工具（1-2周）
   - IDE 配置和快捷键
   - Maven/Gradle 基本使用
   - Git 常用命令

2. 开发工具（2-3周）
   - 调试技巧
   - 单元测试
   - API 测试

3. 进阶工具（3-4周）
   - 性能分析
   - 代码质量检查
   - 容器化部署

4. 运维工具（4-6周）
   - 监控告警
   - 日志分析
   - CI/CD 流程
```

---

**文档版本**: v1.0  
**最后更新**: 2026-02-04  
**作者**: @author erik.zhou

