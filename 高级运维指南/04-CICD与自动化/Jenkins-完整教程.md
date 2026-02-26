# Jenkins 完整教程

> 企业级持续集成/持续交付平台实战指南
>
> @author erik.zhou

## 📚 目录

- [1. Jenkins 简介](#1-jenkins-简介)
- [2. 安装与配置](#2-安装与配置)
- [3. 核心概念](#3-核心概念)
- [4. Pipeline 流水线](#4-pipeline-流水线)
- [5. 插件系统](#5-插件系统)
- [6. 分布式构建](#6-分布式构建)
- [7. 安全管理](#7-安全管理)
- [8. 实战案例](#8-实战案例)
- [9. 性能优化](#9-性能优化)
- [10. 故障排查](#10-故障排查)
- [11. 最佳实践](#11-最佳实践)
- [12. 学习检查清单](#12-学习检查清单)

## 🎯 学习目标

- 掌握 Jenkins 的安装、配置和管理
- 理解 Jenkins Pipeline 的核心概念和语法
- 能够构建企业级 CI/CD 流水线
- 掌握 Jenkins 插件开发和定制
- 了解分布式构建和性能优化
- 掌握安全配置和权限管理
- 能够排查常见问题和故障

## 1. Jenkins 简介

### 1.1 什么是 Jenkins

Jenkins 是一个开源的持续集成/持续交付平台，用于自动化软件开发过程中的构建、测试和部署。

**核心特性**：
- 易于安装和配置
- 丰富的插件生态（1800+ 插件）
- 支持分布式构建
- Pipeline as Code
- 强大的可扩展性

### 1.2 发展历史

```
2004年 - Hudson 项目启动（Sun Microsystems）
2011年 - 更名为 Jenkins（Oracle 收购 Sun 后分叉）
2016年 - Jenkins 2.0 发布（Pipeline as Code）
2018年 - Jenkins X 发布（云原生 CI/CD）
2020年 - Jenkins 配置即代码（JCasC）成熟
```

### 1.3 应用场景


1. **持续集成**：自动化代码构建和测试
2. **持续交付**：自动化部署到测试/生产环境
3. **定时任务**：定期执行脚本和维护任务
4. **代码质量检查**：集成 SonarQube 等工具
5. **自动化测试**：单元测试、集成测试、E2E 测试

## 2. 安装与配置

### 2.1 系统要求

```bash
# 最低配置
CPU: 2 核
内存: 4GB
磁盘: 50GB
Java: JDK 11 或 17

# 推荐配置（生产环境）
CPU: 4 核+
内存: 8GB+
磁盘: 200GB+ SSD
Java: JDK 17
```

### 2.2 Docker 安装（推荐）

```bash
# 拉取官方镜像
docker pull jenkins/jenkins:lts

# 创建数据目录
mkdir -p /data/jenkins_home
chown -R 1000:1000 /data/jenkins_home

# 启动 Jenkins
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v /data/jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --restart=always \
  jenkins/jenkins:lts

# 查看初始密码
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### 2.3 Linux 安装

```bash
# CentOS/RHEL
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install jenkins java-17-openjdk

# Ubuntu/Debian
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

# 启动服务
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

### 2.4 初始化配置

```bash
# 1. 访问 http://your-server:8080
# 2. 输入初始密码（从日志或文件获取）
cat /var/jenkins_home/secrets/initialAdminPassword

# 3. 安装推荐插件或自定义插件
# 4. 创建管理员用户
# 5. 配置 Jenkins URL
```

### 2.5 配置文件

```bash
# Jenkins 主配置文件
/var/jenkins_home/config.xml

# 系统配置
/etc/sysconfig/jenkins  # CentOS
/etc/default/jenkins    # Ubuntu

# 修改 JVM 参数
JENKINS_JAVA_OPTIONS="-Xmx4g -Xms2g -XX:MaxPermSize=512m"
```

## 3. 核心概念

### 3.1 Job/Project

Jenkins 中的基本工作单元，用于定义构建任务。

**Job 类型**：
- **Freestyle Project**：传统的自由风格项目
- **Pipeline**：流水线项目（推荐）
- **Multi-configuration Project**：矩阵项目
- **Folder**：组织项目的文件夹
- **Multibranch Pipeline**：多分支流水线

### 3.2 Build

一次 Job 的执行过程。

```groovy
// Build 信息
currentBuild.number          // 构建号
currentBuild.result          // 构建结果：SUCCESS/FAILURE/UNSTABLE
currentBuild.duration        // 构建时长
currentBuild.displayName     // 显示名称
```

### 3.3 Workspace

Job 执行时的工作目录。

```bash
# Workspace 路径
/var/jenkins_home/workspace/<job-name>

# 清理 Workspace
cleanWs()  // Pipeline 中清理
```

### 3.4 Node/Agent

执行构建任务的机器。

- **Master**：Jenkins 主节点，负责调度
- **Agent**：执行构建的从节点

### 3.5 Executor

Node 上的执行槽位，决定并发构建数量。

```groovy
// 配置 Executor 数量
// Manage Jenkins -> Configure System -> # of executors
```

## 4. Pipeline 流水线

### 4.1 Pipeline 基础

Pipeline 是 Jenkins 2.0 引入的核心特性，支持将 CI/CD 流程定义为代码。

**两种语法**：
- **Declarative Pipeline**：声明式（推荐）
- **Scripted Pipeline**：脚本式（更灵活）

### 4.2 Declarative Pipeline

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        // 环境变量
        DOCKER_REGISTRY = 'registry.example.com'
        APP_NAME = 'myapp'
        VERSION = "${env.BUILD_NUMBER}"
    }
    
    options {
        // 构建选项
        timestamps()                          // 显示时间戳
        timeout(time: 1, unit: 'HOURS')      // 超时设置
        buildDiscarder(logRotator(           // 保留策略
            numToKeepStr: '10',
            artifactNumToKeepStr: '5'
        ))
        disableConcurrentBuilds()            // 禁止并发构建
    }
    
    parameters {
        // 构建参数
        string(name: 'BRANCH', defaultValue: 'main', description: '分支名称')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'test', 'prod'], description: '部署环境')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: '跳过测试')
    }
    
    triggers {
        // 触发器
        cron('H 2 * * *')                    // 定时触发
        pollSCM('H/5 * * * *')               // 轮询 SCM
    }
    
    stages {
        stage('Checkout') {
            steps {
                // 检出代码
                git branch: "${params.BRANCH}",
                    url: 'https://github.com/example/repo.git',
                    credentialsId: 'github-credentials'
            }
        }
        
        stage('Build') {
            steps {
                script {
                    // 构建应用
                    sh '''
                        echo "Building ${APP_NAME} version ${VERSION}"
                        mvn clean package -DskipTests=${SKIP_TESTS}
                    '''
                }
            }
        }
        
        stage('Test') {
            when {
                expression { params.SKIP_TESTS == false }
            }
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'mvn verify'
                    }
                }
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Code Quality') {
            steps {
                script {
                    // SonarQube 分析
                    withSonarQubeEnv('SonarQube') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${APP_NAME}:${VERSION}")
                }
            }
        }
        
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${APP_NAME}:${VERSION}").push()
                        docker.image("${DOCKER_REGISTRY}/${APP_NAME}:${VERSION}").push('latest')
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                expression { params.ENVIRONMENT != 'prod' || env.BRANCH_NAME == 'main' }
            }
            steps {
                script {
                    // 部署到 Kubernetes
                    sh """
                        kubectl set image deployment/${APP_NAME} \
                            ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${VERSION} \
                            -n ${params.ENVIRONMENT}
                        kubectl rollout status deployment/${APP_NAME} -n ${params.ENVIRONMENT}
                    """
                }
            }
        }
    }
    
    post {
        always {
            // 清理工作空间
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
            // 发送成功通知
            emailext(
                subject: "✅ Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build succeeded. Check console output at ${env.BUILD_URL}",
                to: 'team@example.com'
            )
        }
        failure {
            echo 'Pipeline failed!'
            // 发送失败通知
            emailext(
                subject: "❌ Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build failed. Check console output at ${env.BUILD_URL}",
                to: 'team@example.com'
            )
        }
    }
}
```

### 4.3 Scripted Pipeline

```groovy
// 更灵活的脚本式 Pipeline
node {
    def app
    
    try {
        stage('Checkout') {
            checkout scm
        }
        
        stage('Build') {
            app = docker.build("myapp:${env.BUILD_NUMBER}")
        }
        
        stage('Test') {
            app.inside {
                sh 'make test'
            }
        }
        
        stage('Deploy') {
            if (env.BRANCH_NAME == 'main') {
                app.push('latest')
                app.push("${env.BUILD_NUMBER}")
            }
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        // 清理工作
        cleanWs()
    }
}
```

### 4.4 共享库（Shared Libraries）

```groovy
// vars/buildApp.groovy - 共享库
def call(Map config) {
    pipeline {
        agent any
        stages {
            stage('Build') {
                steps {
                    script {
                        sh "mvn clean package -P${config.profile}"
                    }
                }
            }
            stage('Deploy') {
                steps {
                    script {
                        deployToK8s(
                            namespace: config.namespace,
                            image: config.image
                        )
                    }
                }
            }
        }
    }
}

// Jenkinsfile - 使用共享库
@Library('my-shared-library') _

buildApp(
    profile: 'production',
    namespace: 'prod',
    image: 'myapp:latest'
)
```



## 5. 插件系统

### 5.1 核心插件

```bash
# 必装插件
Git Plugin                    # Git 集成
Pipeline                      # Pipeline 支持
Docker Pipeline              # Docker 集成
Kubernetes Plugin            # K8s 集成
Blue Ocean                   # 现代化 UI
Credentials Binding          # 凭证管理
Email Extension              # 邮件通知
SonarQube Scanner           # 代码质量
JUnit Plugin                # 测试报告
```

### 5.2 插件管理

```bash
# 通过 UI 安装
Manage Jenkins -> Manage Plugins -> Available

# 通过 CLI 安装
java -jar jenkins-cli.jar -s http://localhost:8080/ \
    install-plugin git docker-workflow

# 批量安装插件
cat plugins.txt | while read plugin; do
    java -jar jenkins-cli.jar -s http://localhost:8080/ \
        install-plugin $plugin
done

# 插件配置文件
/var/jenkins_home/plugins/
```

### 5.3 常用插件配置

```groovy
// Git 插件配置
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    userRemoteConfigs: [[
        url: 'https://github.com/example/repo.git',
        credentialsId: 'github-token'
    ]],
    extensions: [
        [$class: 'CloneOption', depth: 1, shallow: true],
        [$class: 'CheckoutOption', timeout: 10]
    ]
])

// Docker 插件配置
docker.image('maven:3.8-jdk-11').inside {
    sh 'mvn clean package'
}

// Kubernetes 插件配置
podTemplate(
    label: 'maven-pod',
    containers: [
        containerTemplate(
            name: 'maven',
            image: 'maven:3.8-jdk-11',
            command: 'cat',
            ttyEnabled: true
        )
    ]
) {
    node('maven-pod') {
        container('maven') {
            sh 'mvn clean package'
        }
    }
}
```

## 6. 分布式构建

### 6.1 Master-Agent 架构

```
┌─────────────┐
│   Master    │  - 调度任务
│  (Jenkins)  │  - 管理配置
└──────┬──────┘  - 监控状态
       │
       ├──────────┬──────────┬──────────┐
       │          │          │          │
   ┌───▼───┐  ┌──▼───┐  ┌──▼───┐  ┌──▼───┐
   │Agent 1│  │Agent2│  │Agent3│  │Agent4│
   │Linux  │  │Windows│ │macOS │  │Docker│
   └───────┘  └──────┘  └──────┘  └──────┘
```

### 6.2 添加 Agent 节点

```bash
# 方式1：通过 SSH 连接
# Manage Jenkins -> Manage Nodes -> New Node

# 方式2：通过 JNLP 连接
# 在 Agent 机器上运行
java -jar agent.jar \
    -jnlpUrl http://jenkins-master:8080/computer/agent1/slave-agent.jnlp \
    -secret <secret-key> \
    -workDir /var/jenkins

# 方式3：Docker Agent
docker run -d \
    --name jenkins-agent \
    -e JENKINS_URL=http://jenkins-master:8080 \
    -e JENKINS_AGENT_NAME=docker-agent \
    -e JENKINS_SECRET=<secret> \
    jenkins/inbound-agent
```

### 6.3 Agent 配置

```groovy
// 指定 Agent 标签
pipeline {
    agent {
        label 'linux && docker'
    }
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t myapp .'
            }
        }
    }
}

// 动态 Agent
pipeline {
    agent {
        docker {
            image 'maven:3.8-jdk-11'
            args '-v /root/.m2:/root/.m2'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}

// Kubernetes Agent
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.8-jdk-11
    command: ['cat']
    ttyEnabled: true
  - name: docker
    image: docker:latest
    command: ['cat']
    ttyEnabled: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
'''
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
        stage('Docker Build') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp .'
                }
            }
        }
    }
}
```

## 7. 安全管理

### 7.1 认证配置

```groovy
// 启用安全
// Manage Jenkins -> Configure Global Security

// 1. Jenkins 自带用户数据库
// Security Realm: Jenkins' own user database

// 2. LDAP 认证
// Security Realm: LDAP
Server: ldap://ldap.example.com
Root DN: dc=example,dc=com
User search base: ou=users
User search filter: uid={0}

// 3. OAuth 认证（GitHub）
// 安装 GitHub Authentication Plugin
// Security Realm: GitHub Authentication Plugin
GitHub Web URI: https://github.com
GitHub API URI: https://api.github.com
Client ID: <your-client-id>
Client Secret: <your-client-secret>
```

### 7.2 授权策略

```groovy
// 1. 基于矩阵的授权
// Authorization: Matrix-based security
// 配置用户/组权限

// 2. 基于项目的授权
// Authorization: Project-based Matrix Authorization Strategy
// 每个项目单独配置权限

// 3. Role-Based 授权（推荐）
// 安装 Role-based Authorization Strategy Plugin
// Manage Jenkins -> Manage and Assign Roles

// 定义角色
Global roles:
  - admin: 所有权限
  - developer: Job 相关权限
  - viewer: 只读权限

Project roles:
  - project-admin: 项目管理员
  - project-developer: 项目开发者
```

### 7.3 凭证管理

```groovy
// 添加凭证
// Manage Jenkins -> Manage Credentials

// 1. Username with password
withCredentials([usernamePassword(
    credentialsId: 'github-credentials',
    usernameVariable: 'USERNAME',
    passwordVariable: 'PASSWORD'
)]) {
    sh 'git clone https://${USERNAME}:${PASSWORD}@github.com/example/repo.git'
}

// 2. SSH Key
withCredentials([sshUserPrivateKey(
    credentialsId: 'ssh-key',
    keyFileVariable: 'SSH_KEY'
)]) {
    sh 'ssh -i ${SSH_KEY} user@server "deploy.sh"'
}

// 3. Secret text
withCredentials([string(
    credentialsId: 'api-token',
    variable: 'API_TOKEN'
)]) {
    sh 'curl -H "Authorization: Bearer ${API_TOKEN}" https://api.example.com'
}

// 4. Secret file
withCredentials([file(
    credentialsId: 'kubeconfig',
    variable: 'KUBECONFIG'
)]) {
    sh 'kubectl --kubeconfig=${KUBECONFIG} get pods'
}
```

### 7.4 安全加固

```bash
# 1. 启用 CSRF 保护
# Manage Jenkins -> Configure Global Security -> CSRF Protection

# 2. 配置代理
# Manage Jenkins -> Configure Global Security -> Agent
# TCP port for inbound agents: Fixed (50000)
# Agent protocols: JNLP4-connect

# 3. 限制脚本执行
# Manage Jenkins -> Configure Global Security -> Script Security
# 启用 Script Security Plugin

# 4. 配置内容安全策略
# Manage Jenkins -> Script Console
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", 
    "default-src 'self'; script-src 'self' 'unsafe-inline';")

# 5. 定期更新
# 保持 Jenkins 和插件最新版本
# Manage Jenkins -> Manage Plugins -> Updates
```

## 8. 实战案例

### 8.1 Java 微服务 CI/CD

```groovy
// Jenkinsfile for Spring Boot microservice
@Library('shared-library') _

pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.8-jdk-11
    command: ['cat']
    ttyEnabled: true
  - name: docker
    image: docker:latest
    command: ['cat']
    ttyEnabled: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
'''
        }
    }
    
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        APP_NAME = 'user-service'
        NAMESPACE = 'microservices'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build & Test') {
            steps {
                container('maven') {
                    sh '''
                        mvn clean package
                        mvn test
                        mvn verify
                    '''
                }
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                    jacoco(
                        execPattern: '**/target/jacoco.exec',
                        classPattern: '**/target/classes',
                        sourcePattern: '**/src/main/java'
                    )
                }
            }
        }
        
        stage('Code Quality') {
            steps {
                container('maven') {
                    withSonarQubeEnv('SonarQube') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Build Image') {
            steps {
                container('docker') {
                    script {
                        def version = sh(
                            script: 'git describe --tags --always',
                            returnStdout: true
                        ).trim()
                        
                        sh """
                            docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${version} .
                            docker tag ${DOCKER_REGISTRY}/${APP_NAME}:${version} \
                                ${DOCKER_REGISTRY}/${APP_NAME}:latest
                        """
                    }
                }
            }
        }
        
        stage('Push Image') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-registry',
                        usernameVariable: 'USERNAME',
                        passwordVariable: 'PASSWORD'
                    )]) {
                        sh """
                            echo \${PASSWORD} | docker login ${DOCKER_REGISTRY} -u \${USERNAME} --password-stdin
                            docker push ${DOCKER_REGISTRY}/${APP_NAME}:${version}
                            docker push ${DOCKER_REGISTRY}/${APP_NAME}:latest
                        """
                    }
                }
            }
        }
        
        stage('Deploy to Dev') {
            steps {
                script {
                    deployToK8s(
                        namespace: 'dev',
                        deployment: APP_NAME,
                        image: "${DOCKER_REGISTRY}/${APP_NAME}:${version}"
                    )
                }
            }
        }
        
        stage('Integration Test') {
            steps {
                script {
                    sh '''
                        # 等待服务就绪
                        kubectl wait --for=condition=ready pod \
                            -l app=${APP_NAME} -n dev --timeout=300s
                        
                        # 运行集成测试
                        mvn failsafe:integration-test -Denv=dev
                    '''
                }
            }
        }
        
        stage('Deploy to Prod') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                script {
                    deployToK8s(
                        namespace: 'prod',
                        deployment: APP_NAME,
                        image: "${DOCKER_REGISTRY}/${APP_NAME}:${version}"
                    )
                }
            }
        }
    }
    
    post {
        success {
            dingtalk(
                robot: 'jenkins-bot',
                type: 'MARKDOWN',
                title: '✅ 构建成功',
                text: [
                    "### ${env.JOB_NAME}",
                    "- 构建号: ${env.BUILD_NUMBER}",
                    "- 分支: ${env.BRANCH_NAME}",
                    "- 状态: 成功",
                    "- [查看详情](${env.BUILD_URL})"
                ]
            )
        }
        failure {
            dingtalk(
                robot: 'jenkins-bot',
                type: 'MARKDOWN',
                title: '❌ 构建失败',
                text: [
                    "### ${env.JOB_NAME}",
                    "- 构建号: ${env.BUILD_NUMBER}",
                    "- 分支: ${env.BRANCH_NAME}",
                    "- 状态: 失败",
                    "- [查看详情](${env.BUILD_URL})"
                ],
                at: ['18888888888']
            )
        }
    }
}
```

### 8.2 前端项目 CI/CD

```groovy
// Jenkinsfile for React/Vue project
pipeline {
    agent {
        docker {
            image 'node:16-alpine'
        }
    }
    
    environment {
        NPM_REGISTRY = 'https://registry.npmmirror.com'
        OSS_BUCKET = 'my-static-site'
        CDN_DOMAIN = 'cdn.example.com'
    }
    
    stages {
        stage('Install Dependencies') {
            steps {
                sh '''
                    npm config set registry ${NPM_REGISTRY}
                    npm ci
                '''
            }
        }
        
        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }
        
        stage('Unit Test') {
            steps {
                sh 'npm run test:unit'
            }
            post {
                always {
                    publishHTML([
                        reportDir: 'coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }
        
        stage('Build') {
            steps {
                sh '''
                    npm run build
                    ls -lh dist/
                '''
            }
        }
        
        stage('E2E Test') {
            steps {
                sh '''
                    npm run serve &
                    sleep 10
                    npm run test:e2e
                '''
            }
        }
        
        stage('Deploy to OSS') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aliyun-oss',
                    usernameVariable: 'ACCESS_KEY',
                    passwordVariable: 'SECRET_KEY'
                )]) {
                    sh '''
                        # 安装 ossutil
                        wget http://gosspublic.alicdn.com/ossutil/1.7.15/ossutil64
                        chmod +x ossutil64
                        
                        # 配置
                        ./ossutil64 config -e oss-cn-hangzhou.aliyuncs.com \
                            -i ${ACCESS_KEY} -k ${SECRET_KEY}
                        
                        # 上传文件
                        ./ossutil64 cp -r dist/ oss://${OSS_BUCKET}/ --update
                        
                        # 刷新 CDN
                        aliyun cdn RefreshObjectCaches \
                            --ObjectPath https://${CDN_DOMAIN}/
                    '''
                }
            }
        }
    }
}
```



### 8.3 多环境部署流水线

```groovy
// 多环境部署策略
pipeline {
    agent any
    
    parameters {
        choice(
            name: 'DEPLOY_ENV',
            choices: ['dev', 'test', 'staging', 'prod'],
            description: '部署环境'
        )
        string(
            name: 'VERSION',
            defaultValue: 'latest',
            description: '部署版本'
        )
    }
    
    stages {
        stage('Validate') {
            steps {
                script {
                    // 生产环境需要特殊权限
                    if (params.DEPLOY_ENV == 'prod') {
                        def userInput = input(
                            message: '确认部署到生产环境？',
                            parameters: [
                                string(name: 'APPROVER', description: '审批人'),
                                password(name: 'APPROVAL_CODE', description: '审批码')
                            ]
                        )
                        
                        // 验证审批码
                        if (userInput.APPROVAL_CODE != env.PROD_APPROVAL_CODE) {
                            error('审批码错误')
                        }
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    def envConfig = [
                        'dev': [
                            namespace: 'dev',
                            replicas: 1,
                            resources: [cpu: '500m', memory: '512Mi']
                        ],
                        'test': [
                            namespace: 'test',
                            replicas: 2,
                            resources: [cpu: '1', memory: '1Gi']
                        ],
                        'staging': [
                            namespace: 'staging',
                            replicas: 3,
                            resources: [cpu: '2', memory: '2Gi']
                        ],
                        'prod': [
                            namespace: 'prod',
                            replicas: 5,
                            resources: [cpu: '4', memory: '4Gi']
                        ]
                    ]
                    
                    def config = envConfig[params.DEPLOY_ENV]
                    
                    sh """
                        kubectl set image deployment/myapp \
                            myapp=registry.example.com/myapp:${params.VERSION} \
                            -n ${config.namespace}
                        
                        kubectl scale deployment/myapp \
                            --replicas=${config.replicas} \
                            -n ${config.namespace}
                        
                        kubectl rollout status deployment/myapp \
                            -n ${config.namespace} \
                            --timeout=5m
                    """
                }
            }
        }
        
        stage('Smoke Test') {
            steps {
                script {
                    def serviceUrl = sh(
                        script: "kubectl get svc myapp -n ${params.DEPLOY_ENV} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'",
                        returnStdout: true
                    ).trim()
                    
                    sh """
                        # 健康检查
                        curl -f http://${serviceUrl}/health || exit 1
                        
                        # 基本功能测试
                        curl -f http://${serviceUrl}/api/version || exit 1
                    """
                }
            }
        }
        
        stage('Rollback on Failure') {
            when {
                expression { currentBuild.result == 'FAILURE' }
            }
            steps {
                script {
                    sh """
                        kubectl rollout undo deployment/myapp \
                            -n ${params.DEPLOY_ENV}
                    """
                }
            }
        }
    }
}
```

## 9. 性能优化

### 9.1 Master 优化

```bash
# 1. JVM 参数优化
# /etc/sysconfig/jenkins
JENKINS_JAVA_OPTIONS="-Xmx4g -Xms2g \
    -XX:+UseG1GC \
    -XX:MaxGCPauseMillis=200 \
    -XX:+HeapDumpOnOutOfMemoryError \
    -XX:HeapDumpPath=/var/log/jenkins/heap-dump.hprof \
    -Djava.awt.headless=true \
    -Dhudson.model.DirectoryBrowserSupport.CSP=\"\""

# 2. 减少 Master 负载
# - 将构建任务分配到 Agent
# - 设置 Master 的 Executor 数量为 0
# Manage Jenkins -> Configure System -> # of executors: 0

# 3. 清理旧构建
# 配置构建保留策略
options {
    buildDiscarder(logRotator(
        numToKeepStr: '30',
        daysToKeepStr: '30',
        artifactNumToKeepStr: '10'
    ))
}

# 4. 定期清理工作空间
# 添加定时任务
@daily
node {
    cleanWs()
}
```

### 9.2 Pipeline 优化

```groovy
// 1. 并行执行
stage('Parallel Tests') {
    parallel {
        stage('Unit Tests') {
            steps { sh 'npm run test:unit' }
        }
        stage('Integration Tests') {
            steps { sh 'npm run test:integration' }
        }
        stage('E2E Tests') {
            steps { sh 'npm run test:e2e' }
        }
    }
}

// 2. 缓存依赖
pipeline {
    agent {
        docker {
            image 'maven:3.8-jdk-11'
            args '-v /root/.m2:/root/.m2'  // 缓存 Maven 依赖
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}

// 3. 增量构建
stage('Build') {
    when {
        changeset "src/**"  // 只在源码变更时构建
    }
    steps {
        sh 'mvn clean package'
    }
}

// 4. 跳过不必要的步骤
stage('Deploy') {
    when {
        anyOf {
            branch 'main'
            branch 'release/*'
        }
    }
    steps {
        sh 'deploy.sh'
    }
}

// 5. 使用轻量级 Checkout
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    extensions: [
        [$class: 'CloneOption', depth: 1, shallow: true],  // 浅克隆
        [$class: 'SparseCheckoutPaths', sparseCheckoutPaths: [
            [$class: 'SparseCheckoutPath', path: 'src/']
        ]]
    ],
    userRemoteConfigs: [[url: 'https://github.com/example/repo.git']]
])
```

### 9.3 Agent 优化

```bash
# 1. 使用 Docker Agent（推荐）
# - 环境隔离
# - 快速启动
# - 易于扩展

# 2. 配置 Agent 缓存
# 在 Agent 上配置本地缓存目录
-v /cache/maven:/root/.m2
-v /cache/npm:/root/.npm
-v /cache/docker:/var/lib/docker

# 3. 使用 Kubernetes 动态 Agent
# - 按需创建
# - 自动销毁
# - 资源隔离

# 4. Agent 连接优化
# 使用 WebSocket 连接（更稳定）
# Manage Jenkins -> Configure Global Security -> Agent
# WebSocket: Enable
```

### 9.4 数据库优化

```sql
-- 定期清理旧数据
-- 清理旧构建记录
DELETE FROM builds WHERE timestamp < DATE_SUB(NOW(), INTERVAL 90 DAY);

-- 清理旧日志
DELETE FROM build_logs WHERE build_id NOT IN (
    SELECT id FROM builds WHERE timestamp > DATE_SUB(NOW(), INTERVAL 30 DAY)
);

-- 优化表
OPTIMIZE TABLE builds;
OPTIMIZE TABLE build_logs;

-- 添加索引
CREATE INDEX idx_timestamp ON builds(timestamp);
CREATE INDEX idx_job_name ON builds(job_name);
```

## 10. 故障排查

### 10.1 常见问题

```bash
# 1. Jenkins 启动失败
# 查看日志
tail -f /var/log/jenkins/jenkins.log
journalctl -u jenkins -f

# 检查端口占用
netstat -tlnp | grep 8080

# 检查 Java 版本
java -version

# 2. 构建卡住
# 查看线程堆栈
jstack <jenkins-pid> > thread-dump.txt

# 查看内存使用
jmap -heap <jenkins-pid>

# 3. 插件冲突
# 禁用插件
mv /var/jenkins_home/plugins/problematic-plugin.jpi \
   /var/jenkins_home/plugins/problematic-plugin.jpi.disabled
# 重启 Jenkins

# 4. 磁盘空间不足
# 清理工作空间
find /var/jenkins_home/workspace -type d -mtime +30 -exec rm -rf {} \;

# 清理旧构建
find /var/jenkins_home/jobs/*/builds -type d -mtime +90 -exec rm -rf {} \;

# 5. Agent 连接失败
# 检查网络连接
telnet jenkins-master 50000

# 检查防火墙
firewall-cmd --list-ports

# 查看 Agent 日志
tail -f /var/log/jenkins/agent.log
```

### 10.2 日志分析

```bash
# 1. 启用详细日志
# Manage Jenkins -> System Log -> Add new log recorder
Logger: hudson.model.Run
Level: FINE

# 2. 分析构建日志
# 查找错误
grep -i error /var/jenkins_home/jobs/*/builds/*/log

# 查找超时
grep -i timeout /var/jenkins_home/jobs/*/builds/*/log

# 3. 性能分析
# 启用性能监控
# 安装 Monitoring Plugin
# Manage Jenkins -> Monitoring

# 4. 审计日志
# 安装 Audit Trail Plugin
# Manage Jenkins -> Configure System -> Audit Trail
```

### 10.3 备份与恢复

```bash
# 1. 备份 Jenkins Home
#!/bin/bash
BACKUP_DIR="/backup/jenkins"
JENKINS_HOME="/var/jenkins_home"
DATE=$(date +%Y%m%d_%H%M%S)

# 停止 Jenkins
systemctl stop jenkins

# 备份
tar -czf ${BACKUP_DIR}/jenkins_${DATE}.tar.gz \
    --exclude='workspace' \
    --exclude='caches' \
    ${JENKINS_HOME}

# 启动 Jenkins
systemctl start jenkins

# 保留最近 7 天的备份
find ${BACKUP_DIR} -name "jenkins_*.tar.gz" -mtime +7 -delete

# 2. 恢复 Jenkins
# 停止 Jenkins
systemctl stop jenkins

# 解压备份
tar -xzf jenkins_backup.tar.gz -C /

# 启动 Jenkins
systemctl start jenkins

# 3. 配置即代码备份（推荐）
# 使用 Configuration as Code Plugin
# 导出配置
curl -X POST http://localhost:8080/configuration-as-code/export \
    -o jenkins.yaml

# 导入配置
curl -X POST http://localhost:8080/configuration-as-code/reload \
    -F "jenkins.yaml=@jenkins.yaml"
```

## 11. 最佳实践

### 11.1 Pipeline 最佳实践

```groovy
// 1. 使用 Declarative Pipeline（推荐）
// 2. 将 Jenkinsfile 放在代码仓库
// 3. 使用共享库复用代码
// 4. 合理使用并行执行
// 5. 配置超时和重试
// 6. 使用参数化构建
// 7. 添加构建通知
// 8. 实现自动回滚

pipeline {
    agent any
    
    options {
        timeout(time: 1, unit: 'HOURS')
        retry(2)
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    stages {
        stage('Build') {
            steps {
                retry(3) {
                    sh 'mvn clean package'
                }
            }
        }
    }
}
```

### 11.2 安全最佳实践

```bash
# 1. 启用 HTTPS
# 配置 Nginx 反向代理
server {
    listen 443 ssl;
    server_name jenkins.example.com;
    
    ssl_certificate /etc/nginx/ssl/jenkins.crt;
    ssl_certificate_key /etc/nginx/ssl/jenkins.key;
    
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# 2. 使用凭证管理
# 不要在代码中硬编码密码
# 使用 Credentials Plugin

# 3. 限制脚本权限
# 启用 Script Security Plugin
# 审查和批准脚本

# 4. 定期更新
# 保持 Jenkins 和插件最新

# 5. 备份配置
# 定期备份 Jenkins Home
# 使用版本控制管理配置
```

### 11.3 性能最佳实践

```bash
# 1. 合理分配资源
# Master: 4C8G
# Agent: 根据构建需求配置

# 2. 使用分布式构建
# 将构建任务分配到 Agent

# 3. 优化构建缓存
# 缓存依赖、Docker 镜像等

# 4. 清理旧数据
# 定期清理旧构建和工作空间

# 5. 监控性能
# 使用 Monitoring Plugin
# 监控 CPU、内存、磁盘使用
```

### 11.4 团队协作最佳实践

```bash
# 1. 统一代码风格
# 使用 Checkstyle、ESLint 等工具

# 2. 自动化测试
# 单元测试、集成测试、E2E 测试

# 3. 代码审查
# 集成 Gerrit、GitLab MR 等

# 4. 构建通知
# 邮件、钉钉、企业微信等

# 5. 文档化
# 维护 Pipeline 文档
# 记录常见问题和解决方案
```

## 12. 学习检查清单

### 基础知识
- [ ] 理解 Jenkins 的核心概念（Job、Build、Agent 等）
- [ ] 掌握 Jenkins 的安装和配置
- [ ] 了解插件系统和常用插件
- [ ] 掌握基本的 Job 配置

### Pipeline 开发
- [ ] 掌握 Declarative Pipeline 语法
- [ ] 了解 Scripted Pipeline 语法
- [ ] 能够编写完整的 CI/CD Pipeline
- [ ] 掌握共享库的使用

### 高级特性
- [ ] 掌握分布式构建配置
- [ ] 了解 Kubernetes 集成
- [ ] 掌握安全配置和权限管理
- [ ] 能够进行性能优化

### 运维管理
- [ ] 掌握备份和恢复
- [ ] 能够排查常见问题
- [ ] 了解监控和告警
- [ ] 掌握升级和迁移

### 实战能力
- [ ] 能够搭建企业级 CI/CD 平台
- [ ] 能够设计多环境部署流程
- [ ] 能够集成各种工具（Git、Docker、K8s 等）
- [ ] 能够优化构建性能

## 📚 参考资源

### 官方文档
- [Jenkins 官方文档](https://www.jenkins.io/doc/)
- [Pipeline 语法参考](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [插件索引](https://plugins.jenkins.io/)

### 学习资源
- [Jenkins 用户手册](https://www.jenkins.io/doc/book/)
- [Pipeline 最佳实践](https://www.jenkins.io/doc/book/pipeline/pipeline-best-practices/)
- [Jenkins 中文社区](https://jenkins-zh.cn/)

### 工具和插件
- [Blue Ocean](https://www.jenkins.io/projects/blueocean/) - 现代化 UI
- [Configuration as Code](https://github.com/jenkinsci/configuration-as-code-plugin) - 配置即代码
- [Job DSL](https://github.com/jenkinsci/job-dsl-plugin) - Job 定义 DSL

### 相关技术
- Docker
- Kubernetes
- Git
- Maven/Gradle
- SonarQube

---

> 💡 **提示**：Jenkins 是 CI/CD 的核心工具，掌握 Pipeline 开发和分布式构建是关键。建议从简单的 Freestyle Job 开始，逐步过渡到 Pipeline，最后实现完整的企业级 CI/CD 平台。
>
> @author erik.zhou

