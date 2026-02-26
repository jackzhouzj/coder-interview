# GitLab CI/CD 完整教程

> 现代化 DevOps 平台与持续集成/交付实战指南
>
> @author erik.zhou

## 📚 目录

- [1. GitLab CI/CD 简介](#1-gitlab-cicd-简介)
- [2. 核心概念](#2-核心概念)
- [3. .gitlab-ci.yml 配置](#3-gitlab-ciyml-配置)
- [4. GitLab Runner](#4-gitlab-runner)
- [5. Pipeline 高级特性](#5-pipeline-高级特性)
- [6. 变量与密钥管理](#6-变量与密钥管理)
- [7. Docker 集成](#7-docker-集成)
- [8. Kubernetes 集成](#8-kubernetes-集成)
- [9. 实战案例](#9-实战案例)
- [10. 性能优化](#10-性能优化)
- [11. 故障排查](#11-故障排查)
- [12. 最佳实践](#12-最佳实践)
- [13. 学习检查清单](#13-学习检查清单)

## 🎯 学习目标

- 掌握 GitLab CI/CD 的核心概念和架构
- 能够编写完整的 .gitlab-ci.yml 配置文件
- 掌握 GitLab Runner 的安装和配置
- 了解 Pipeline 的高级特性和优化技巧
- 能够集成 Docker 和 Kubernetes
- 掌握多环境部署策略
- 能够排查常见问题和故障

## 1. GitLab CI/CD 简介

### 1.1 什么是 GitLab CI/CD

GitLab CI/CD 是 GitLab 内置的持续集成/持续交付工具，与代码仓库深度集成。

**核心优势**：
- 与 GitLab 无缝集成
- 配置简单（.gitlab-ci.yml）
- 强大的 Pipeline 可视化
- 支持 Docker 和 Kubernetes
- 内置容器镜像仓库
- 完整的 DevOps 生命周期管理

### 1.2 架构组件

```
┌─────────────────────────────────────────┐
│          GitLab Server                  │
│  ┌──────────────────────────────────┐  │
│  │  Git Repository + CI/CD Engine   │  │
│  └──────────────┬───────────────────┘  │
└─────────────────┼───────────────────────┘
                  │
        ┌─────────┴─────────┐
        │                   │
   ┌────▼────┐         ┌───▼────┐
   │ Runner 1│         │Runner 2│
   │ (Docker)│         │  (K8s) │
   └─────────┘         └────────┘
```

**组件说明**：
- **GitLab Server**：管理代码和 CI/CD 配置
- **GitLab Runner**：执行 CI/CD 任务的代理
- **Executor**：Runner 的执行器（Shell、Docker、Kubernetes 等）

### 1.3 工作流程

```
代码提交 → 触发 Pipeline → 分配 Runner → 执行 Job → 生成报告
```

## 2. 核心概念

### 2.1 Pipeline

Pipeline 是 CI/CD 的顶层结构，包含多个 Stage。

```yaml
# 一个 Pipeline 包含多个 Stage
stages:
  - build
  - test
  - deploy
```

### 2.2 Stage

Stage 是 Pipeline 的阶段，按顺序执行。

```yaml
stages:
  - build    # 第一阶段
  - test     # 第二阶段
  - deploy   # 第三阶段
```

### 2.3 Job

Job 是 Stage 中的具体任务，同一 Stage 的 Job 并行执行。

```yaml
build-job:
  stage: build
  script:
    - echo "Building..."
    - mvn clean package

test-job:
  stage: test
  script:
    - echo "Testing..."
    - mvn test
```

### 2.4 Runner

Runner 是执行 Job 的代理程序。

**Runner 类型**：
- **Shared Runner**：所有项目共享
- **Group Runner**：组内项目共享
- **Specific Runner**：特定项目专用

### 2.5 Executor

Executor 是 Runner 的执行环境。

**常用 Executor**：
- **Shell**：直接在主机上执行
- **Docker**：在 Docker 容器中执行（推荐）
- **Kubernetes**：在 K8s Pod 中执行
- **SSH**：通过 SSH 连接远程主机执行

## 3. .gitlab-ci.yml 配置

### 3.1 基础配置

```yaml
# .gitlab-ci.yml
# 定义 Stage
stages:
  - build
  - test
  - deploy

# 全局变量
variables:
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"
  DOCKER_DRIVER: overlay2

# 全局缓存
cache:
  paths:
    - .m2/repository
    - node_modules/

# 全局前置脚本
before_script:
  - echo "Starting job..."
  - date

# 全局后置脚本
after_script:
  - echo "Job finished"
  - date

# Build Job
build-job:
  stage: build
  image: maven:3.8-jdk-11
  script:
    - mvn clean package
  artifacts:
    paths:
      - target/*.jar
    expire_in: 1 week
  only:
    - main
    - develop

# Test Job
test-job:
  stage: test
  image: maven:3.8-jdk-11
  script:
    - mvn test
  coverage: '/Total.*?([0-9]{1,3})%/'
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml
      coverage_report:
        coverage_format: cobertura
        path: target/site/cobertura/coverage.xml

# Deploy Job
deploy-job:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache openssh-client
    - ssh user@server "deploy.sh"
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

### 3.2 Job 配置详解

```yaml
job-name:
  # 所属 Stage
  stage: build
  
  # 使用的 Docker 镜像
  image: node:16-alpine
  
  # 使用的服务（如数据库）
  services:
    - mysql:8.0
    - redis:6.2
  
  # 执行脚本
  script:
    - npm install
    - npm run build
    - npm test
  
  # 前置脚本
  before_script:
    - echo "Preparing..."
  
  # 后置脚本
  after_script:
    - echo "Cleaning up..."
  
  # 变量
  variables:
    NODE_ENV: production
  
  # 缓存
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  
  # 制品
  artifacts:
    name: "build-${CI_COMMIT_REF_NAME}"
    paths:
      - dist/
    expire_in: 30 days
  
  # 执行条件
  only:
    - main
    - /^release-.*$/
  
  except:
    - tags
  
  # 执行规则（推荐，替代 only/except）
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: always
    - if: '$CI_COMMIT_BRANCH =~ /^feature-/'
      when: manual
  
  # 依赖的 Job
  dependencies:
    - build-job
  
  # 需要的 Job
  needs:
    - job: build-job
      artifacts: true
  
  # 重试次数
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
  
  # 超时时间
  timeout: 1h
  
  # 执行时机
  when: on_success  # on_success, on_failure, always, manual, delayed
  
  # 延迟执行
  # when: delayed
  # start_in: 30 minutes
  
  # 允许失败
  allow_failure: true
  
  # 环境
  environment:
    name: staging
    url: https://staging.example.com
    on_stop: stop-staging
  
  # 资源组（防止并发）
  resource_group: production
  
  # 标签（指定 Runner）
  tags:
    - docker
    - linux
```



### 3.3 高级语法

```yaml
# 1. 锚点和引用（复用配置）
.common-config: &common
  image: alpine:latest
  before_script:
    - apk add --no-cache curl

job1:
  <<: *common
  script:
    - echo "Job 1"

job2:
  <<: *common
  script:
    - echo "Job 2"

# 2. 扩展（extends）
.base-job:
  image: node:16
  cache:
    paths:
      - node_modules/

build-job:
  extends: .base-job
  script:
    - npm run build

test-job:
  extends: .base-job
  script:
    - npm test

# 3. 包含外部文件（include）
include:
  # 本地文件
  - local: '/templates/.gitlab-ci-template.yml'
  
  # 远程文件
  - remote: 'https://example.com/ci-template.yml'
  
  # 项目文件
  - project: 'group/project'
    ref: main
    file: '/templates/.gitlab-ci-template.yml'
  
  # 模板
  - template: Security/SAST.gitlab-ci.yml

# 4. 触发器（trigger）
trigger-downstream:
  stage: deploy
  trigger:
    project: group/downstream-project
    branch: main
    strategy: depend

# 5. 父子 Pipeline
trigger-child:
  stage: build
  trigger:
    include: child-pipeline.yml
    strategy: depend

# 6. 动态子 Pipeline
generate-config:
  stage: build
  script:
    - python generate-ci-config.py > generated-config.yml
  artifacts:
    paths:
      - generated-config.yml

child-pipeline:
  stage: deploy
  trigger:
    include:
      - artifact: generated-config.yml
        job: generate-config
```

### 3.4 条件执行

```yaml
# 1. rules（推荐）
deploy-job:
  script: deploy.sh
  rules:
    # 主分支自动部署
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: always
    
    # 功能分支手动部署
    - if: '$CI_COMMIT_BRANCH =~ /^feature-/'
      when: manual
    
    # Tag 自动部署
    - if: '$CI_COMMIT_TAG'
      when: always
    
    # MR 不部署
    - if: '$CI_MERGE_REQUEST_ID'
      when: never
    
    # 文件变更触发
    - changes:
        - src/**/*
        - Dockerfile
      when: always

# 2. only/except（旧语法）
deploy-job:
  script: deploy.sh
  only:
    refs:
      - main
      - /^release-.*$/
    variables:
      - $CI_COMMIT_MESSAGE =~ /deploy/
    changes:
      - src/**/*
  except:
    refs:
      - tags
    variables:
      - $SKIP_DEPLOY

# 3. workflow（Pipeline 级别控制）
workflow:
  rules:
    # 只在 MR 或主分支运行
    - if: '$CI_MERGE_REQUEST_ID'
    - if: '$CI_COMMIT_BRANCH == "main"'
    # 其他情况不运行
    - when: never
```

## 4. GitLab Runner

### 4.1 安装 Runner

```bash
# 1. Docker 安装（推荐）
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest

# 2. Linux 安装
# Ubuntu/Debian
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
sudo apt-get install gitlab-runner

# CentOS/RHEL
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash
sudo yum install gitlab-runner

# 3. macOS 安装
brew install gitlab-runner
brew services start gitlab-runner
```

### 4.2 注册 Runner

```bash
# 交互式注册
gitlab-runner register

# 非交互式注册
gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --registration-token "PROJECT_REGISTRATION_TOKEN" \
  --executor "docker" \
  --docker-image alpine:latest \
  --description "docker-runner" \
  --tag-list "docker,linux" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"

# 注册 Kubernetes Executor
gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --registration-token "PROJECT_REGISTRATION_TOKEN" \
  --executor "kubernetes" \
  --kubernetes-host "https://k8s.example.com" \
  --kubernetes-namespace "gitlab-runner" \
  --kubernetes-privileged \
  --kubernetes-cpu-limit "2" \
  --kubernetes-memory-limit "2Gi"
```

### 4.3 Runner 配置

```toml
# /etc/gitlab-runner/config.toml
concurrent = 10  # 并发任务数
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "docker-runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "docker"
  
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
    shm_size = 0
    pull_policy = "if-not-present"
    
    # 资源限制
    cpus = "2"
    memory = "2g"
    memory_swap = "4g"
    memory_reservation = "1g"

[[runners]]
  name = "k8s-runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "kubernetes"
  
  [runners.kubernetes]
    host = "https://k8s.example.com"
    namespace = "gitlab-runner"
    privileged = true
    cpu_limit = "2"
    memory_limit = "2Gi"
    service_cpu_limit = "1"
    service_memory_limit = "1Gi"
    helper_cpu_limit = "500m"
    helper_memory_limit = "512Mi"
    
    # 节点选择器
    [runners.kubernetes.node_selector]
      "node-role.kubernetes.io/runner" = "true"
    
    # 污点容忍
    [[runners.kubernetes.node_tolerations]]
      key = "dedicated"
      operator = "Equal"
      value = "gitlab-runner"
      effect = "NoSchedule"
```

### 4.4 Runner 管理

```bash
# 查看 Runner 状态
gitlab-runner status

# 启动 Runner
gitlab-runner start

# 停止 Runner
gitlab-runner stop

# 重启 Runner
gitlab-runner restart

# 查看 Runner 列表
gitlab-runner list

# 验证配置
gitlab-runner verify

# 注销 Runner
gitlab-runner unregister --name docker-runner

# 查看日志
gitlab-runner --debug run

# 更新 Runner
# Docker
docker pull gitlab/gitlab-runner:latest
docker restart gitlab-runner

# Linux
sudo gitlab-runner stop
sudo apt-get update && sudo apt-get install gitlab-runner
sudo gitlab-runner start
```

## 5. Pipeline 高级特性

### 5.1 并行执行

```yaml
# 1. 同一 Stage 的 Job 自动并行
test:
  stage: test
  parallel: 5  # 创建 5 个并行 Job
  script:
    - npm test -- --shard=$CI_NODE_INDEX/$CI_NODE_TOTAL

# 2. 矩阵构建
test:
  stage: test
  parallel:
    matrix:
      - NODE_VERSION: ['14', '16', '18']
        OS: ['ubuntu', 'alpine']
  image: node:${NODE_VERSION}-${OS}
  script:
    - npm test

# 3. 手动并行
test-unit:
  stage: test
  script: npm run test:unit

test-integration:
  stage: test
  script: npm run test:integration

test-e2e:
  stage: test
  script: npm run test:e2e
```

### 5.2 DAG（有向无环图）

```yaml
# 使用 needs 定义依赖关系，实现更灵活的执行顺序
stages:
  - build
  - test
  - deploy

build-backend:
  stage: build
  script: mvn package

build-frontend:
  stage: build
  script: npm run build

test-backend:
  stage: test
  needs: [build-backend]  # 只依赖 build-backend
  script: mvn test

test-frontend:
  stage: test
  needs: [build-frontend]  # 只依赖 build-frontend
  script: npm test

deploy:
  stage: deploy
  needs: [test-backend, test-frontend]  # 依赖两个测试 Job
  script: deploy.sh
```

### 5.3 动态 Pipeline

```yaml
# 生成动态配置
generate-config:
  stage: .pre
  script:
    - |
      cat > generated.yml << EOF
      test-job:
        stage: test
        script:
          - echo "Generated job"
      EOF
  artifacts:
    paths:
      - generated.yml

# 使用动态配置
trigger-dynamic:
  stage: test
  trigger:
    include:
      - artifact: generated.yml
        job: generate-config
```

### 5.4 多项目 Pipeline

```yaml
# 触发下游项目
trigger-downstream:
  stage: deploy
  trigger:
    project: group/downstream-project
    branch: main
    strategy: depend  # 等待下游 Pipeline 完成
  variables:
    UPSTREAM_VERSION: $CI_COMMIT_TAG

# 接收上游变量（下游项目）
deploy:
  stage: deploy
  script:
    - echo "Deploying version $UPSTREAM_VERSION"
  only:
    - pipelines  # 只在被触发时运行
```

### 5.5 合并请求 Pipeline

```yaml
# 只在 MR 中运行
mr-check:
  script: npm run lint
  only:
    - merge_requests

# 使用 rules（推荐）
mr-check:
  script: npm run lint
  rules:
    - if: '$CI_MERGE_REQUEST_ID'

# MR 变更检测
mr-test:
  script: npm test
  rules:
    - if: '$CI_MERGE_REQUEST_ID'
      changes:
        - src/**/*
        - tests/**/*
```

## 6. 变量与密钥管理

### 6.1 预定义变量

```yaml
job:
  script:
    # CI/CD 相关
    - echo $CI_PIPELINE_ID          # Pipeline ID
    - echo $CI_PIPELINE_IID         # Pipeline 内部 ID
    - echo $CI_JOB_ID               # Job ID
    - echo $CI_JOB_NAME             # Job 名称
    - echo $CI_JOB_STAGE            # Stage 名称
    
    # Git 相关
    - echo $CI_COMMIT_SHA           # 完整 commit SHA
    - echo $CI_COMMIT_SHORT_SHA     # 短 commit SHA
    - echo $CI_COMMIT_BRANCH        # 分支名
    - echo $CI_COMMIT_TAG           # Tag 名
    - echo $CI_COMMIT_MESSAGE       # Commit 消息
    - echo $CI_COMMIT_AUTHOR        # 作者
    
    # 项目相关
    - echo $CI_PROJECT_ID           # 项目 ID
    - echo $CI_PROJECT_NAME         # 项目名
    - echo $CI_PROJECT_PATH         # 项目路径
    - echo $CI_PROJECT_URL          # 项目 URL
    - echo $CI_PROJECT_DIR          # 项目目录
    
    # Runner 相关
    - echo $CI_RUNNER_ID            # Runner ID
    - echo $CI_RUNNER_DESCRIPTION   # Runner 描述
    - echo $CI_RUNNER_TAGS          # Runner 标签
    
    # 环境相关
    - echo $CI_ENVIRONMENT_NAME     # 环境名称
    - echo $CI_ENVIRONMENT_URL      # 环境 URL
```

### 6.2 自定义变量

```yaml
# 1. 全局变量
variables:
  GLOBAL_VAR: "global value"
  DOCKER_DRIVER: overlay2

# 2. Job 变量
job:
  variables:
    JOB_VAR: "job value"
  script:
    - echo $JOB_VAR

# 3. 项目/组变量（UI 配置）
# Settings -> CI/CD -> Variables

# 4. 文件变量
variables:
  DATABASE_URL:
    value: "postgres://user:pass@host/db"
    description: "Database connection string"

# 5. 受保护变量
# 只在受保护分支/标签上可用
# Settings -> CI/CD -> Variables -> Protected

# 6. 遮蔽变量
# 在日志中自动遮蔽
# Settings -> CI/CD -> Variables -> Masked
```

### 6.3 密钥管理

```yaml
# 1. 使用项目变量存储密钥
deploy:
  script:
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - ssh user@server "deploy.sh"

# 2. 使用文件变量
deploy:
  script:
    - kubectl --kubeconfig=$KUBECONFIG apply -f deployment.yaml

# 3. 使用 Vault 集成
deploy:
  secrets:
    DATABASE_PASSWORD:
      vault: production/db/password@secret
      file: false
  script:
    - deploy.sh

# 4. 使用 AWS Secrets Manager
deploy:
  image: amazon/aws-cli
  script:
    - |
      SECRET=$(aws secretsmanager get-secret-value \
        --secret-id prod/db/password \
        --query SecretString \
        --output text)
      echo "Using secret: $SECRET"
```



## 7. Docker 集成

### 7.1 使用 Docker 镜像

```yaml
# 1. 指定镜像
job:
  image: node:16-alpine
  script:
    - npm install
    - npm test

# 2. 使用私有镜像
job:
  image: registry.example.com/myapp:latest
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - npm test

# 3. 使用服务容器
test:
  image: node:16
  services:
    - name: mysql:8.0
      alias: mysql
    - name: redis:6.2
      alias: redis
  variables:
    MYSQL_ROOT_PASSWORD: password
    MYSQL_DATABASE: testdb
  script:
    - npm test
```

### 7.2 构建 Docker 镜像

```yaml
# 1. 使用 Docker-in-Docker
build-image:
  image: docker:latest
  services:
    - docker:dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

# 2. 使用 Kaniko（推荐，无需特权模式）
build-image:
  image:
    name: gcr.io/kaniko-project/executor:debug
    entrypoint: [""]
  script:
    - mkdir -p /kaniko/.docker
    - echo "{\"auths\":{\"$CI_REGISTRY\":{\"username\":\"$CI_REGISTRY_USER\",\"password\":\"$CI_REGISTRY_PASSWORD\"}}}" > /kaniko/.docker/config.json
    - /kaniko/executor
      --context $CI_PROJECT_DIR
      --dockerfile $CI_PROJECT_DIR/Dockerfile
      --destination $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
      --destination $CI_REGISTRY_IMAGE:latest

# 3. 使用 Buildah
build-image:
  image: quay.io/buildah/stable
  script:
    - buildah bud -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - buildah login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - buildah push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

### 7.3 多阶段构建

```dockerfile
# Dockerfile
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```yaml
# .gitlab-ci.yml
build-and-push:
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

## 8. Kubernetes 集成

### 8.1 部署到 Kubernetes

```yaml
# 1. 使用 kubectl
deploy:
  image: bitnami/kubectl:latest
  stage: deploy
  script:
    - kubectl config set-cluster k8s --server="$KUBE_URL" --insecure-skip-tls-verify=true
    - kubectl config set-credentials admin --token="$KUBE_TOKEN"
    - kubectl config set-context default --cluster=k8s --user=admin
    - kubectl config use-context default
    - |
      cat <<EOF | kubectl apply -f -
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: myapp
        namespace: production
      spec:
        replicas: 3
        selector:
          matchLabels:
            app: myapp
        template:
          metadata:
            labels:
              app: myapp
          spec:
            containers:
            - name: myapp
              image: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
              ports:
              - containerPort: 8080
      EOF
    - kubectl rollout status deployment/myapp -n production

# 2. 使用 Helm
deploy:
  image: alpine/helm:latest
  stage: deploy
  script:
    - helm upgrade --install myapp ./helm-chart \
        --set image.tag=$CI_COMMIT_SHA \
        --set image.repository=$CI_REGISTRY_IMAGE \
        --namespace production \
        --create-namespace \
        --wait

# 3. 使用 Kustomize
deploy:
  image: k8s.gcr.io/kustomize/kustomize:v4.5.7
  stage: deploy
  script:
    - cd k8s/overlays/production
    - kustomize edit set image myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - kustomize build . | kubectl apply -f -
```

### 8.2 GitLab Agent for Kubernetes

```yaml
# 1. 安装 Agent
# 在 GitLab UI 中：Infrastructure -> Kubernetes clusters -> Connect a cluster

# 2. 配置 Agent
# .gitlab/agents/production/config.yaml
ci_access:
  projects:
    - id: group/project
      default_namespace: production

# 3. 使用 Agent 部署
deploy:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context group/project:production
    - kubectl apply -f k8s/
    - kubectl rollout status deployment/myapp -n production
```

### 8.3 蓝绿部署

```yaml
# 蓝绿部署策略
deploy-blue:
  stage: deploy
  script:
    - kubectl apply -f k8s/deployment-blue.yaml
    - kubectl wait --for=condition=ready pod -l version=blue -n production
  environment:
    name: production-blue
    url: https://blue.example.com

switch-to-blue:
  stage: deploy
  when: manual
  script:
    - kubectl patch service myapp -n production -p '{"spec":{"selector":{"version":"blue"}}}'
  environment:
    name: production
    url: https://example.com

deploy-green:
  stage: deploy
  script:
    - kubectl apply -f k8s/deployment-green.yaml
    - kubectl wait --for=condition=ready pod -l version=green -n production
  environment:
    name: production-green
    url: https://green.example.com

switch-to-green:
  stage: deploy
  when: manual
  script:
    - kubectl patch service myapp -n production -p '{"spec":{"selector":{"version":"green"}}}'
  environment:
    name: production
    url: https://example.com
```

## 9. 实战案例

### 9.1 完整的微服务 CI/CD

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - package
  - deploy

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"
  DOCKER_DRIVER: overlay2
  APP_NAME: user-service

# 缓存 Maven 依赖
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - .m2/repository

# 构建
build:
  stage: build
  image: maven:3.8-jdk-11
  script:
    - mvn clean compile
  artifacts:
    paths:
      - target/
    expire_in: 1 hour

# 单元测试
test-unit:
  stage: test
  image: maven:3.8-jdk-11
  script:
    - mvn test
  coverage: '/Total.*?([0-9]{1,3})%/'
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml
      coverage_report:
        coverage_format: cobertura
        path: target/site/cobertura/coverage.xml

# 集成测试
test-integration:
  stage: test
  image: maven:3.8-jdk-11
  services:
    - mysql:8.0
    - redis:6.2
  variables:
    MYSQL_ROOT_PASSWORD: password
    MYSQL_DATABASE: testdb
    SPRING_PROFILES_ACTIVE: test
  script:
    - mvn verify -DskipUnitTests
  artifacts:
    reports:
      junit: target/failsafe-reports/TEST-*.xml

# 代码质量检查
sonarqube:
  stage: test
  image: maven:3.8-jdk-11
  script:
    - mvn sonar:sonar
      -Dsonar.projectKey=$CI_PROJECT_NAME
      -Dsonar.host.url=$SONAR_HOST_URL
      -Dsonar.login=$SONAR_TOKEN
  only:
    - main
    - develop

# 打包
package:
  stage: package
  image: maven:3.8-jdk-11
  script:
    - mvn package -DskipTests
  artifacts:
    paths:
      - target/*.jar
    expire_in: 1 week

# 构建 Docker 镜像
build-image:
  stage: package
  image:
    name: gcr.io/kaniko-project/executor:debug
    entrypoint: [""]
  script:
    - mkdir -p /kaniko/.docker
    - echo "{\"auths\":{\"$CI_REGISTRY\":{\"username\":\"$CI_REGISTRY_USER\",\"password\":\"$CI_REGISTRY_PASSWORD\"}}}" > /kaniko/.docker/config.json
    - /kaniko/executor
      --context $CI_PROJECT_DIR
      --dockerfile $CI_PROJECT_DIR/Dockerfile
      --destination $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
      --destination $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
      --cache=true

# 部署到开发环境
deploy-dev:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context $KUBE_CONTEXT
    - helm upgrade --install $APP_NAME ./helm-chart
      --set image.tag=$CI_COMMIT_SHA
      --set image.repository=$CI_REGISTRY_IMAGE
      --namespace dev
      --create-namespace
      --wait
  environment:
    name: dev
    url: https://dev.example.com
    on_stop: stop-dev
  only:
    - develop

# 停止开发环境
stop-dev:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context $KUBE_CONTEXT
    - helm uninstall $APP_NAME -n dev
  environment:
    name: dev
    action: stop
  when: manual
  only:
    - develop

# 部署到测试环境
deploy-test:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context $KUBE_CONTEXT
    - helm upgrade --install $APP_NAME ./helm-chart
      --set image.tag=$CI_COMMIT_SHA
      --set image.repository=$CI_REGISTRY_IMAGE
      --set replicaCount=2
      --namespace test
      --create-namespace
      --wait
  environment:
    name: test
    url: https://test.example.com
  only:
    - main

# 部署到生产环境
deploy-prod:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context $KUBE_CONTEXT
    - helm upgrade --install $APP_NAME ./helm-chart
      --set image.tag=$CI_COMMIT_SHA
      --set image.repository=$CI_REGISTRY_IMAGE
      --set replicaCount=5
      --set resources.requests.cpu=2
      --set resources.requests.memory=4Gi
      --namespace production
      --create-namespace
      --wait
    - kubectl rollout status deployment/$APP_NAME -n production
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main

# 回滚生产环境
rollback-prod:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context $KUBE_CONTEXT
    - helm rollback $APP_NAME -n production
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

### 9.2 前端项目 CI/CD

```yaml
# .gitlab-ci.yml for React/Vue
stages:
  - install
  - lint
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "16"
  NPM_REGISTRY: "https://registry.npmmirror.com"

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .npm/

# 安装依赖
install:
  stage: install
  image: node:${NODE_VERSION}-alpine
  script:
    - npm config set registry $NPM_REGISTRY
    - npm ci --cache .npm --prefer-offline
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

# 代码检查
lint:
  stage: lint
  image: node:${NODE_VERSION}-alpine
  script:
    - npm run lint
  needs: [install]

# 单元测试
test-unit:
  stage: test
  image: node:${NODE_VERSION}-alpine
  script:
    - npm run test:unit
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
  needs: [install]

# E2E 测试
test-e2e:
  stage: test
  image: mcr.microsoft.com/playwright:v1.40.0-focal
  script:
    - npm run test:e2e
  artifacts:
    when: on_failure
    paths:
      - test-results/
    expire_in: 7 days
  needs: [install]

# 构建
build:
  stage: build
  image: node:${NODE_VERSION}-alpine
  script:
    - npm run build
    - ls -lh dist/
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
  needs: [install]

# 部署到 OSS
deploy-oss:
  stage: deploy
  image: python:3.9-alpine
  before_script:
    - pip install oss2
  script:
    - python deploy-to-oss.py
  environment:
    name: production
    url: https://cdn.example.com
  only:
    - main
  needs: [build]

# 部署到 Nginx
deploy-nginx:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh-client rsync
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
  script:
    - rsync -avz --delete dist/ user@server:/var/www/html/
    - ssh user@server "nginx -s reload"
  environment:
    name: production
    url: https://example.com
  only:
    - main
  needs: [build]
```

### 9.3 Monorepo 项目 CI/CD

```yaml
# .gitlab-ci.yml for Monorepo
stages:
  - detect
  - build
  - test
  - deploy

# 检测变更
detect-changes:
  stage: detect
  image: alpine:latest
  script:
    - apk add --no-cache git
    - |
      if git diff --name-only $CI_COMMIT_BEFORE_SHA $CI_COMMIT_SHA | grep -q "^services/user/"; then
        echo "USER_SERVICE_CHANGED=true" >> build.env
      fi
    - |
      if git diff --name-only $CI_COMMIT_BEFORE_SHA $CI_COMMIT_SHA | grep -q "^services/order/"; then
        echo "ORDER_SERVICE_CHANGED=true" >> build.env
      fi
    - |
      if git diff --name-only $CI_COMMIT_BEFORE_SHA $CI_COMMIT_SHA | grep -q "^frontend/"; then
        echo "FRONTEND_CHANGED=true" >> build.env
      fi
  artifacts:
    reports:
      dotenv: build.env

# 构建 User Service
build-user-service:
  stage: build
  image: maven:3.8-jdk-11
  script:
    - cd services/user
    - mvn clean package
  artifacts:
    paths:
      - services/user/target/*.jar
  rules:
    - if: '$USER_SERVICE_CHANGED == "true"'
  needs: [detect-changes]

# 构建 Order Service
build-order-service:
  stage: build
  image: maven:3.8-jdk-11
  script:
    - cd services/order
    - mvn clean package
  artifacts:
    paths:
      - services/order/target/*.jar
  rules:
    - if: '$ORDER_SERVICE_CHANGED == "true"'
  needs: [detect-changes]

# 构建 Frontend
build-frontend:
  stage: build
  image: node:16-alpine
  script:
    - cd frontend
    - npm ci
    - npm run build
  artifacts:
    paths:
      - frontend/dist/
  rules:
    - if: '$FRONTEND_CHANGED == "true"'
  needs: [detect-changes]

# 部署 User Service
deploy-user-service:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/user-service user-service=$CI_REGISTRY_IMAGE/user-service:$CI_COMMIT_SHA -n production
  rules:
    - if: '$USER_SERVICE_CHANGED == "true" && $CI_COMMIT_BRANCH == "main"'
  needs: [build-user-service]

# 部署 Order Service
deploy-order-service:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl set image deployment/order-service order-service=$CI_REGISTRY_IMAGE/order-service:$CI_COMMIT_SHA -n production
  rules:
    - if: '$ORDER_SERVICE_CHANGED == "true" && $CI_COMMIT_BRANCH == "main"'
  needs: [build-order-service]

# 部署 Frontend
deploy-frontend:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache openssh-client rsync
    - rsync -avz frontend/dist/ user@server:/var/www/html/
  rules:
    - if: '$FRONTEND_CHANGED == "true" && $CI_COMMIT_BRANCH == "main"'
  needs: [build-frontend]
```

## 10. 性能优化

### 10.1 缓存优化

```yaml
# 1. 全局缓存
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - .m2/repository
    - node_modules/
    - .npm/
  policy: pull-push

# 2. Job 级别缓存
job:
  cache:
    key: ${CI_COMMIT_REF_SLUG}-${CI_JOB_NAME}
    paths:
      - vendor/
    policy: pull

# 3. 分布式缓存（使用 S3）
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
  s3:
    bucket: gitlab-cache
    region: us-east-1
    access_key: $AWS_ACCESS_KEY_ID
    secret_key: $AWS_SECRET_ACCESS_KEY
```

### 10.2 并行优化

```yaml
# 1. 使用 parallel
test:
  parallel: 5
  script:
    - npm test -- --shard=$CI_NODE_INDEX/$CI_NODE_TOTAL

# 2. 使用 needs（DAG）
build:
  stage: build
  script: make build

test:
  stage: test
  needs: [build]  # 不等待其他 build stage 的 job
  script: make test

# 3. 矩阵构建
test:
  parallel:
    matrix:
      - RUBY_VERSION: ['2.7', '3.0', '3.1']
        DATABASE: ['mysql', 'postgres']
  image: ruby:${RUBY_VERSION}
  services:
    - ${DATABASE}:latest
  script:
    - bundle exec rspec
```

### 10.3 镜像优化

```yaml
# 1. 使用轻量级镜像
job:
  image: node:16-alpine  # 而不是 node:16

# 2. 使用多阶段构建
# Dockerfile
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html

# 3. 缓存 Docker 层
build:
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build --cache-from $CI_REGISTRY_IMAGE:latest -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

### 10.4 Runner 优化

```toml
# /etc/gitlab-runner/config.toml
concurrent = 10  # 增加并发数

[[runners]]
  [runners.docker]
    pull_policy = "if-not-present"  # 减少镜像拉取
    volumes = ["/cache"]  # 使用本地缓存
    
  [runners.cache]
    Type = "s3"  # 使用分布式缓存
    Shared = true
```

## 11. 故障排查

### 11.1 常见问题

```bash
# 1. Job 卡住不执行
# 检查 Runner 状态
gitlab-runner verify

# 查看 Runner 日志
gitlab-runner --debug run

# 2. Docker 镜像拉取失败
# 使用镜像加速
# .gitlab-ci.yml
variables:
  DOCKER_REGISTRY_MIRROR: https://mirror.example.com

# 3. 缓存不生效
# 检查缓存 key 是否正确
cache:
  key: ${CI_COMMIT_REF_SLUG}  # 确保 key 一致

# 4. Pipeline 超时
# 增加超时时间
job:
  timeout: 2h

# 5. 磁盘空间不足
# 清理旧的 Docker 镜像和容器
docker system prune -af

# 清理 Runner 缓存
gitlab-runner cache-archiver --delete
```

### 11.2 调试技巧

```yaml
# 1. 启用调试模式
variables:
  CI_DEBUG_TRACE: "true"

# 2. 保留失败的 artifacts
job:
  artifacts:
    when: on_failure
    paths:
      - logs/
      - test-results/

# 3. 使用 script 调试
job:
  script:
    - set -x  # 显示执行的命令
    - env | sort  # 查看环境变量
    - pwd  # 查看当前目录
    - ls -la  # 查看文件列表

# 4. 交互式调试
# 在 Job 中添加
script:
  - sleep 3600  # 保持 Job 运行
# 然后通过 kubectl exec 进入容器调试
```

## 12. 最佳实践

### 12.1 配置最佳实践

```yaml
# 1. 使用 extends 复用配置
.base-job:
  image: node:16-alpine
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  before_script:
    - npm ci

build:
  extends: .base-job
  script:
    - npm run build

test:
  extends: .base-job
  script:
    - npm test

# 2. 使用 rules 而不是 only/except
job:
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: always
    - if: '$CI_MERGE_REQUEST_ID'
      when: manual

# 3. 合理使用 needs
job:
  needs:
    - job: build
      artifacts: true

# 4. 设置合理的超时和重试
job:
  timeout: 30m
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
```

### 12.2 安全最佳实践

```yaml
# 1. 使用 CI/CD 变量存储敏感信息
# Settings -> CI/CD -> Variables

# 2. 启用变量遮蔽
# Variables -> Masked

# 3. 限制变量作用域
# Variables -> Protected (只在受保护分支可用)

# 4. 不在日志中打印敏感信息
script:
  - echo "Deploying..."  # 不要 echo $PASSWORD

# 5. 使用最小权限原则
# 为 Runner 配置最小必要权限
```

### 12.3 性能最佳实践

```yaml
# 1. 使用缓存
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/

# 2. 使用 artifacts 传递文件
build:
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

# 3. 使用并行执行
test:
  parallel: 5

# 4. 使用 needs 优化依赖
deploy:
  needs: [build, test]

# 5. 使用轻量级镜像
image: node:16-alpine
```

## 13. 学习检查清单

### 基础知识
- [ ] 理解 GitLab CI/CD 的核心概念
- [ ] 掌握 .gitlab-ci.yml 基本语法
- [ ] 了解 Pipeline、Stage、Job 的关系
- [ ] 掌握变量和缓存的使用

### Runner 管理
- [ ] 掌握 Runner 的安装和注册
- [ ] 了解不同 Executor 的特点
- [ ] 能够配置和管理 Runner
- [ ] 掌握 Runner 故障排查

### 高级特性
- [ ] 掌握 rules 条件执行
- [ ] 了解 DAG 和并行执行
- [ ] 掌握 Docker 和 Kubernetes 集成
- [ ] 能够实现多环境部署

### 实战能力
- [ ] 能够搭建完整的 CI/CD Pipeline
- [ ] 能够优化 Pipeline 性能
- [ ] 能够排查常见问题
- [ ] 掌握安全配置和最佳实践

## 📚 参考资源

### 官方文档
- [GitLab CI/CD 文档](https://docs.gitlab.com/ee/ci/)
- [.gitlab-ci.yml 参考](https://docs.gitlab.com/ee/ci/yaml/)
- [GitLab Runner 文档](https://docs.gitlab.com/runner/)

### 学习资源
- [GitLab CI/CD 示例](https://docs.gitlab.com/ee/ci/examples/)
- [GitLab CI/CD 最佳实践](https://docs.gitlab.com/ee/ci/pipelines/pipeline_efficiency.html)
- [GitLab 中文社区](https://gitlab.cn/)

### 工具和插件
- [GitLab Runner](https://gitlab.com/gitlab-org/gitlab-runner)
- [Kaniko](https://github.com/GoogleContainerTools/kaniko) - 无需特权的容器构建
- [Helm](https://helm.sh/) - Kubernetes 包管理器

### 相关技术
- Docker
- Kubernetes
- Helm
- Terraform

---

> 💡 **提示**：GitLab CI/CD 与 GitLab 深度集成，配置简单但功能强大。建议从简单的 Pipeline 开始，逐步掌握高级特性如 DAG、动态 Pipeline 等。合理使用缓存和并行执行可以显著提升 Pipeline 性能。
>
> @author erik.zhou

