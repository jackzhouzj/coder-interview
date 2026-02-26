# ArgoCD 完整教程

> GitOps 持续交付工具实战指南
>
> @author erik.zhou

## 📚 目录

- [1. ArgoCD 简介](#1-argocd-简介)
- [2. 安装与配置](#2-安装与配置)
- [3. 核心概念](#3-核心概念)
- [4. 应用管理](#4-应用管理)
- [5. GitOps 工作流](#5-gitops-工作流)
- [6. 多集群管理](#6-多集群管理)
- [7. 同步策略](#7-同步策略)
- [8. RBAC 权限管理](#8-rbac-权限管理)
- [9. 实战案例](#9-实战案例)
- [10. 高级特性](#10-高级特性)
- [11. 最佳实践](#11-最佳实践)
- [12. 故障排查](#12-故障排查)
- [13. 学习检查清单](#13-学习检查清单)

## 🎯 学习目标

- 理解 GitOps 的核心理念和优势
- 掌握 ArgoCD 的安装和配置
- 能够创建和管理 ArgoCD 应用
- 掌握多集群和多环境管理
- 了解同步策略和健康检查
- 能够实现完整的 GitOps 工作流
- 掌握 ArgoCD 的最佳实践

## 1. ArgoCD 简介

### 1.1 什么是 ArgoCD

ArgoCD 是一个声明式的 GitOps 持续交付工具，用于 Kubernetes 应用的自动化部署和管理。

**核心特性**：
- 声明式 GitOps CD
- 自动化部署和同步
- 多集群管理
- SSO 集成
- RBAC 权限控制
- 健康状态检查
- 回滚和历史记录
- Web UI 和 CLI

### 1.2 GitOps 工作流

```
┌─────────────┐
│  Git Repo   │  ← 开发者提交配置
│  (Source)   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   ArgoCD    │  ← 监控 Git 变更
│  (Control)  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Kubernetes  │  ← 自动同步应用
│  (Target)   │
└─────────────┘
```

### 1.3 架构组件

```
┌──────────────────────────────────────┐
│           ArgoCD Server              │
│  ┌────────────┐  ┌────────────────┐ │
│  │  API Server│  │  Web UI        │ │
│  └────────────┘  └────────────────┘ │
└──────────────┬───────────────────────┘
               │
    ┌──────────┼──────────┐
    │          │          │
┌───▼───┐  ┌──▼───┐  ┌──▼────┐
│Repo   │  │App   │  │Dex    │
│Server │  │Ctrl  │  │(SSO)  │
└───────┘  └──────┘  └───────┘
```

## 2. 安装与配置

### 2.1 安装 ArgoCD

```bash
# 1. 创建命名空间
kubectl create namespace argocd

# 2. 安装 ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 3. 等待 Pod 就绪
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=argocd-server -n argocd --timeout=300s

# 4. 查看安装状态
kubectl get pods -n argocd
kubectl get svc -n argocd
```

### 2.2 访问 ArgoCD UI

```bash
# 方式1：端口转发
kubectl port-forward svc/argocd-server -n argocd 8080:443

# 方式2：LoadBalancer（生产环境）
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# 方式3：Ingress
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  rules:
  - host: argocd.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 443
  tls:
  - hosts:
    - argocd.example.com
    secretName: argocd-server-tls
EOF
```

### 2.3 获取初始密码

```bash
# 获取初始 admin 密码
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# 登录（用户名：admin）
# 访问 https://localhost:8080
```

### 2.4 安装 ArgoCD CLI

```bash
# macOS
brew install argocd

# Linux
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x /usr/local/bin/argocd

# Windows
choco install argocd-cli

# 登录
argocd login localhost:8080 --username admin --password <password> --insecure

# 修改密码
argocd account update-password
```

### 2.5 配置 ArgoCD

```yaml
# argocd-cm.yaml - ConfigMap 配置
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  # 仓库凭证
  repositories: |
    - url: https://github.com/example/repo.git
      passwordSecret:
        name: github-secret
        key: password
      usernameSecret:
        name: github-secret
        key: username
    - url: https://gitlab.com/example/repo.git
      sshPrivateKeySecret:
        name: gitlab-secret
        key: sshPrivateKey
  
  # 资源自定义
  resource.customizations: |
    argoproj.io/Application:
      health.lua: |
        hs = {}
        hs.status = "Progressing"
        hs.message = ""
        if obj.status ~= nil then
          if obj.status.health ~= nil then
            hs.status = obj.status.health.status
            if obj.status.health.message ~= nil then
              hs.message = obj.status.health.message
            end
          end
        end
        return hs
  
  # 超时设置
  timeout.reconciliation: 180s
  timeout.hard.reconciliation: 0s
  
  # UI 配置
  ui.cssurl: "https://example.com/custom.css"
  
  # 启用匿名访问（仅用于演示）
  # users.anonymous.enabled: "true"
```

## 3. 核心概念

### 3.1 Application

Application 是 ArgoCD 的核心资源，定义了应用的源和目标。

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  # 项目
  project: default
  
  # 源（Git 仓库）
  source:
    repoURL: https://github.com/example/repo.git
    targetRevision: HEAD
    path: k8s/overlays/production
    
    # Helm 配置
    helm:
      valueFiles:
        - values.yaml
      parameters:
        - name: image.tag
          value: v1.0.0
    
    # Kustomize 配置
    kustomize:
      namePrefix: prod-
      commonLabels:
        environment: production
  
  # 目标（Kubernetes 集群）
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  
  # 同步策略
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

### 3.2 Project

Project 用于逻辑分组和权限控制。

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: myproject
  namespace: argocd
spec:
  description: My Project
  
  # 允许的源仓库
  sourceRepos:
    - 'https://github.com/example/*'
    - 'https://gitlab.com/example/*'
  
  # 允许的目标集群和命名空间
  destinations:
    - namespace: 'production'
      server: https://kubernetes.default.svc
    - namespace: 'staging'
      server: https://kubernetes.default.svc
  
  # 允许的资源类型
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
    - group: 'rbac.authorization.k8s.io'
      kind: ClusterRole
  
  # 命名空间资源白名单
  namespaceResourceWhitelist:
    - group: 'apps'
      kind: Deployment
    - group: ''
      kind: Service
    - group: ''
      kind: ConfigMap
  
  # 孤立资源警告
  orphanedResources:
    warn: true
```

### 3.3 Repository

Repository 定义 Git 仓库的访问凭证。

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: github-repo
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  type: git
  url: https://github.com/example/repo.git
  username: myuser
  password: mytoken
```

### 3.4 Cluster

Cluster 定义外部 Kubernetes 集群。

```bash
# 添加集群
argocd cluster add <context-name>

# 列出集群
argocd cluster list

# 查看集群信息
argocd cluster get <cluster-name>
```

## 4. 应用管理

### 4.1 创建应用（CLI）

```bash
# 基本创建
argocd app create myapp \
  --repo https://github.com/example/repo.git \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

# Helm 应用
argocd app create myapp \
  --repo https://github.com/example/charts.git \
  --path myapp \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --helm-set image.tag=v1.0.0 \
  --values values-prod.yaml

# Kustomize 应用
argocd app create myapp \
  --repo https://github.com/example/repo.git \
  --path k8s/overlays/production \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production \
  --kustomize-image myapp=myregistry/myapp:v1.0.0
```

### 4.2 同步应用

```bash
# 手动同步
argocd app sync myapp

# 同步并等待完成
argocd app sync myapp --wait

# 同步特定资源
argocd app sync myapp --resource apps:Deployment:myapp

# 强制同步
argocd app sync myapp --force

# 预览同步（Dry Run）
argocd app sync myapp --dry-run
```

### 4.3 查看应用状态

```bash
# 查看应用列表
argocd app list

# 查看应用详情
argocd app get myapp

# 查看应用历史
argocd app history myapp

# 查看应用差异
argocd app diff myapp

# 查看应用日志
argocd app logs myapp

# 查看应用资源树
argocd app resources myapp
```

### 4.4 回滚应用

```bash
# 查看历史版本
argocd app history myapp

# 回滚到指定版本
argocd app rollback myapp <revision-id>

# 回滚到上一个版本
argocd app rollback myapp
```

### 4.5 删除应用

```bash
# 删除应用（保留资源）
argocd app delete myapp

# 删除应用和资源
argocd app delete myapp --cascade

# 批量删除
argocd app delete -l app=myapp
```

## 5. GitOps 工作流

### 5.1 标准 GitOps 流程

```yaml
# 1. 定义应用配置（Git 仓库）
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
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
        image: myregistry/myapp:v1.0.0
        ports:
        - containerPort: 8080

# 2. 创建 ArgoCD 应用
# argocd/application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo.git
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

# 3. 应用配置
kubectl apply -f argocd/application.yaml

# 4. ArgoCD 自动同步
# Git 变更 → ArgoCD 检测 → 自动部署到 K8s
```

### 5.2 多环境管理

```
repo/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── patch.yaml
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   └── patch.yaml
│   └── production/
│       ├── kustomization.yaml
│       └── patch.yaml
└── argocd/
    ├── app-dev.yaml
    ├── app-staging.yaml
    └── app-production.yaml
```

```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml

# overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
  - ../../base
namePrefix: prod-
commonLabels:
  environment: production
replicas:
  - name: myapp
    count: 5
images:
  - name: myapp
    newTag: v1.0.0

# argocd/app-production.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-production
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo.git
    targetRevision: main
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### 5.3 App of Apps 模式

```yaml
# argocd/root-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo.git
    targetRevision: main
    path: argocd/apps
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

# argocd/apps/app1.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app1
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo.git
    targetRevision: main
    path: apps/app1
  destination:
    server: https://kubernetes.default.svc
    namespace: app1
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

# argocd/apps/app2.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app2
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo.git
    targetRevision: main
    path: apps/app2
  destination:
    server: https://kubernetes.default.svc
    namespace: app2
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```



## 6. 多集群管理

### 6.1 添加外部集群

```bash
# 查看当前 kubeconfig 上下文
kubectl config get-contexts

# 添加集群到 ArgoCD
argocd cluster add <context-name>

# 使用特定命名空间
argocd cluster add <context-name> --namespace argocd

# 使用服务账号
argocd cluster add <context-name> --service-account argocd-manager

# 列出所有集群
argocd cluster list

# 查看集群详情
argocd cluster get <cluster-url>
```

### 6.2 集群凭证管理

```yaml
# 创建服务账号
apiVersion: v1
kind: ServiceAccount
metadata:
  name: argocd-manager
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: argocd-manager-role
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: argocd-manager-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argocd-manager-role
subjects:
- kind: ServiceAccount
  name: argocd-manager
  namespace: kube-system
```

### 6.3 多集群应用部署

```yaml
# 部署到多个集群
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapp-multi-cluster
  namespace: argocd
spec:
  generators:
  - list:
      elements:
      - cluster: prod-us-east
        url: https://prod-us-east.k8s.example.com
      - cluster: prod-eu-west
        url: https://prod-eu-west.k8s.example.com
      - cluster: prod-ap-south
        url: https://prod-ap-south.k8s.example.com
  template:
    metadata:
      name: 'myapp-{{cluster}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/example/repo.git
        targetRevision: main
        path: k8s
      destination:
        server: '{{url}}'
        namespace: production
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

## 7. 同步策略

### 7.1 自动同步

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  syncPolicy:
    automated:
      # 自动删除不在 Git 中的资源
      prune: true
      
      # 自动修复偏离的资源
      selfHeal: true
      
      # 允许空应用
      allowEmpty: false
    
    # 同步选项
    syncOptions:
      # 创建命名空间
      - CreateNamespace=true
      
      # 验证资源
      - Validate=true
      
      # 使用服务器端应用
      - ServerSideApply=true
      
      # 跳过 Dry Run
      - SkipDryRunOnMissingResource=true
    
    # 重试策略
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

### 7.2 同步窗口

```yaml
# argocd-cm ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  # 定义同步窗口
  sync.windows: |
    - kind: allow
      schedule: '0 9 * * 1-5'  # 工作日 9:00
      duration: 8h
      applications:
        - '*-production'
      manualSync: true
    - kind: deny
      schedule: '0 0 * * 0,6'  # 周末
      duration: 24h
      applications:
        - '*-production'
```

### 7.3 同步波次（Sync Waves）

```yaml
# 控制资源同步顺序
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
  annotations:
    argocd.argoproj.io/sync-wave: "0"  # 第一波
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  namespace: myapp
  annotations:
    argocd.argoproj.io/sync-wave: "1"  # 第二波
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp
  annotations:
    argocd.argoproj.io/sync-wave: "2"  # 第三波
```

### 7.4 资源钩子（Hooks）

```yaml
# PreSync Hook - 同步前执行
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
      - name: migration
        image: myapp:latest
        command: ["./migrate.sh"]
      restartPolicy: Never

# Sync Hook - 同步时执行
apiVersion: batch/v1
kind: Job
metadata:
  name: data-seed
  annotations:
    argocd.argoproj.io/hook: Sync
    argocd.argoproj.io/hook-delete-policy: BeforeHookCreation
spec:
  template:
    spec:
      containers:
      - name: seed
        image: myapp:latest
        command: ["./seed.sh"]
      restartPolicy: Never

# PostSync Hook - 同步后执行
apiVersion: batch/v1
kind: Job
metadata:
  name: smoke-test
  annotations:
    argocd.argoproj.io/hook: PostSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
      - name: test
        image: curlimages/curl
        command: ["curl", "-f", "http://myapp/health"]
      restartPolicy: Never
```

## 8. RBAC 权限管理

### 8.1 配置 RBAC

```yaml
# argocd-rbac-cm ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  # 策略规则
  policy.csv: |
    # 格式：p, subject, resource, action, object, effect
    
    # 管理员角色
    p, role:admin, applications, *, */*, allow
    p, role:admin, clusters, *, *, allow
    p, role:admin, repositories, *, *, allow
    p, role:admin, projects, *, *, allow
    
    # 开发者角色
    p, role:developer, applications, get, */*, allow
    p, role:developer, applications, sync, */*, allow
    p, role:developer, applications, create, */*, allow
    p, role:developer, applications, update, */*, allow
    p, role:developer, repositories, get, *, allow
    
    # 只读角色
    p, role:readonly, applications, get, */*, allow
    p, role:readonly, repositories, get, *, allow
    p, role:readonly, clusters, get, *, allow
    
    # 项目特定权限
    p, role:project-admin, applications, *, myproject/*, allow
    p, role:project-developer, applications, get, myproject/*, allow
    p, role:project-developer, applications, sync, myproject/*, allow
    
    # 用户角色绑定
    g, admin@example.com, role:admin
    g, developer@example.com, role:developer
    g, viewer@example.com, role:readonly
  
  # 默认策略
  policy.default: role:readonly
  
  # 作用域
  scopes: '[groups, email]'
```

### 8.2 SSO 集成（Dex）

```yaml
# argocd-cm ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  # GitHub OAuth
  dex.config: |
    connectors:
    - type: github
      id: github
      name: GitHub
      config:
        clientID: $GITHUB_CLIENT_ID
        clientSecret: $GITHUB_CLIENT_SECRET
        orgs:
        - name: my-org
          teams:
          - dev-team
          - ops-team
  
  # GitLab OAuth
  dex.config: |
    connectors:
    - type: gitlab
      id: gitlab
      name: GitLab
      config:
        baseURL: https://gitlab.com
        clientID: $GITLAB_CLIENT_ID
        clientSecret: $GITLAB_CLIENT_SECRET
        groups:
        - my-group
  
  # LDAP
  dex.config: |
    connectors:
    - type: ldap
      id: ldap
      name: LDAP
      config:
        host: ldap.example.com:636
        insecureNoSSL: false
        bindDN: cn=admin,dc=example,dc=com
        bindPW: $LDAP_PASSWORD
        userSearch:
          baseDN: ou=users,dc=example,dc=com
          filter: "(objectClass=person)"
          username: uid
          idAttr: uid
          emailAttr: mail
          nameAttr: cn
        groupSearch:
          baseDN: ou=groups,dc=example,dc=com
          filter: "(objectClass=groupOfNames)"
          userAttr: DN
          groupAttr: member
          nameAttr: cn
```

## 9. 实战案例

### 9.1 微服务应用部署

```yaml
# ApplicationSet - 自动发现和部署微服务
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: microservices
  namespace: argocd
spec:
  generators:
  # Git 目录生成器
  - git:
      repoURL: https://github.com/example/microservices.git
      revision: main
      directories:
      - path: services/*
  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/example/microservices.git
        targetRevision: main
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.basename}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
```

### 9.2 Helm Chart 部署

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-ingress
  namespace: argocd
spec:
  project: default
  source:
    # Helm 仓库
    repoURL: https://kubernetes.github.io/ingress-nginx
    chart: ingress-nginx
    targetRevision: 4.7.1
    helm:
      # 参数覆盖
      parameters:
        - name: controller.replicaCount
          value: "3"
        - name: controller.service.type
          value: LoadBalancer
      
      # Values 文件
      valueFiles:
        - values-production.yaml
      
      # 内联 Values
      values: |
        controller:
          metrics:
            enabled: true
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
  
  destination:
    server: https://kubernetes.default.svc
    namespace: ingress-nginx
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### 9.3 蓝绿部署

```yaml
# 蓝色版本
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-blue
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo.git
    targetRevision: v1.0.0
    path: k8s
    kustomize:
      namePrefix: blue-
      commonLabels:
        version: blue
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

# 绿色版本
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-green
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo.git
    targetRevision: v2.0.0
    path: k8s
    kustomize:
      namePrefix: green-
      commonLabels:
        version: green
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

# Service 切换
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: production
spec:
  selector:
    version: blue  # 切换到 green 实现蓝绿部署
  ports:
  - port: 80
    targetPort: 8080
```

### 9.4 金丝雀发布（Argo Rollouts）

```yaml
# 安装 Argo Rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# Rollout 资源
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 5
  strategy:
    canary:
      steps:
      - setWeight: 20
      - pause: {duration: 5m}
      - setWeight: 40
      - pause: {duration: 5m}
      - setWeight: 60
      - pause: {duration: 5m}
      - setWeight: 80
      - pause: {duration: 5m}
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
        image: myregistry/myapp:v2.0.0
        ports:
        - containerPort: 8080

# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-rollout
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo.git
    targetRevision: main
    path: k8s/rollouts
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## 10. 高级特性

### 10.1 ApplicationSet

```yaml
# 列表生成器
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: cluster-apps
  namespace: argocd
spec:
  generators:
  - list:
      elements:
      - cluster: dev
        url: https://dev.k8s.example.com
        replicas: "1"
      - cluster: staging
        url: https://staging.k8s.example.com
        replicas: "2"
      - cluster: prod
        url: https://prod.k8s.example.com
        replicas: "5"
  template:
    metadata:
      name: 'myapp-{{cluster}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/example/repo.git
        targetRevision: main
        path: k8s
        kustomize:
          commonLabels:
            environment: '{{cluster}}'
          replicas:
            - name: myapp
              count: '{{replicas}}'
      destination:
        server: '{{url}}'
        namespace: myapp
      syncPolicy:
        automated:
          prune: true
          selfHeal: true

# 集群生成器
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: all-clusters
  namespace: argocd
spec:
  generators:
  - clusters:
      selector:
        matchLabels:
          environment: production
  template:
    metadata:
      name: 'myapp-{{name}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/example/repo.git
        targetRevision: main
        path: k8s
      destination:
        server: '{{server}}'
        namespace: myapp
      syncPolicy:
        automated:
          prune: true
          selfHeal: true

# Git 文件生成器
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: config-apps
  namespace: argocd
spec:
  generators:
  - git:
      repoURL: https://github.com/example/config.git
      revision: main
      files:
      - path: "apps/*/config.json"
  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/example/repo.git
        targetRevision: '{{branch}}'
        path: '{{path}}'
      destination:
        server: '{{cluster}}'
        namespace: '{{namespace}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### 10.2 通知和告警

```yaml
# 安装 Notifications
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-notifications/stable/manifests/install.yaml

# 配置通知
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  # Slack 配置
  service.slack: |
    token: $slack-token
  
  # 钉钉配置
  service.webhook.dingtalk: |
    url: https://oapi.dingtalk.com/robot/send?access_token=$dingtalk-token
    headers:
    - name: Content-Type
      value: application/json
  
  # 模板
  template.app-deployed: |
    message: |
      Application {{.app.metadata.name}} is now running new version.
    slack:
      attachments: |
        [{
          "title": "{{ .app.metadata.name}}",
          "title_link":"{{.context.argocdUrl}}/applications/{{.app.metadata.name}}",
          "color": "#18be52",
          "fields": [
          {
            "title": "Sync Status",
            "value": "{{.app.status.sync.status}}",
            "short": true
          },
          {
            "title": "Repository",
            "value": "{{.app.spec.source.repoURL}}",
            "short": true
          }
          ]
        }]
  
  # 触发器
  trigger.on-deployed: |
    - description: Application is synced and healthy
      send:
      - app-deployed
      when: app.status.operationState.phase in ['Succeeded'] and app.status.health.status == 'Healthy'

# 订阅通知
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
  annotations:
    notifications.argoproj.io/subscribe.on-deployed.slack: my-channel
spec:
  # ...
```

### 10.3 Image Updater

```bash
# 安装 Image Updater
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml

# 配置自动更新镜像
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
  annotations:
    # 启用镜像更新
    argocd-image-updater.argoproj.io/image-list: myapp=myregistry/myapp
    
    # 更新策略
    argocd-image-updater.argoproj.io/myapp.update-strategy: latest
    
    # 允许的标签
    argocd-image-updater.argoproj.io/myapp.allow-tags: regexp:^v[0-9]+\.[0-9]+\.[0-9]+$
    
    # 写回方式
    argocd-image-updater.argoproj.io/write-back-method: git
    argocd-image-updater.argoproj.io/git-branch: main
spec:
  # ...
```

## 11. 最佳实践

### 11.1 仓库结构

```
gitops-repo/
├── apps/
│   ├── app1/
│   │   ├── base/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── kustomization.yaml
│   │   └── overlays/
│   │       ├── dev/
│   │       ├── staging/
│   │       └── production/
│   └── app2/
│       └── ...
├── argocd/
│   ├── projects/
│   │   ├── project1.yaml
│   │   └── project2.yaml
│   ├── applications/
│   │   ├── app1-dev.yaml
│   │   ├── app1-staging.yaml
│   │   ├── app1-production.yaml
│   │   └── ...
│   └── applicationsets/
│       └── microservices.yaml
└── infrastructure/
    ├── ingress-nginx/
    ├── cert-manager/
    └── monitoring/
```

### 11.2 安全配置

```yaml
# 1. 使用 Project 隔离
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  sourceRepos:
    - 'https://github.com/example/prod-*'
  destinations:
    - namespace: 'prod-*'
      server: https://kubernetes.default.svc
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace

# 2. 启用资源配额
apiVersion: v1
kind: ResourceQuota
metadata:
  name: argocd-quota
  namespace: argocd
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi

# 3. 网络策略
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: argocd-server
  namespace: argocd
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: argocd-server
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8080
```

### 11.3 性能优化

```yaml
# argocd-cm ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  # 增加并发数
  application.resourceTrackingMethod: annotation
  
  # 禁用不需要的功能
  resource.exclusions: |
    - apiGroups:
      - "*"
      kinds:
      - Event
      - Pod
  
  # 调整超时
  timeout.reconciliation: 180s
  
  # 启用缓存
  resource.compareoptions: |
    ignoreAggregatedRoles: true

# 增加资源限制
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argocd-server
  namespace: argocd
spec:
  template:
    spec:
      containers:
      - name: argocd-server
        resources:
          requests:
            cpu: 500m
            memory: 512Mi
          limits:
            cpu: 2000m
            memory: 2Gi
```

## 12. 故障排查

### 12.1 常见问题

```bash
# 1. 应用无法同步
# 查看应用状态
argocd app get myapp

# 查看同步日志
argocd app logs myapp

# 查看差异
argocd app diff myapp

# 2. 健康检查失败
# 查看资源状态
kubectl get all -n <namespace>

# 查看 Pod 日志
kubectl logs -n <namespace> <pod-name>

# 3. 权限问题
# 检查 RBAC 配置
kubectl get rolebinding,clusterrolebinding -n argocd

# 查看服务账号
kubectl get sa -n argocd

# 4. 性能问题
# 查看资源使用
kubectl top pods -n argocd

# 查看事件
kubectl get events -n argocd --sort-by='.lastTimestamp'
```

### 12.2 调试技巧

```bash
# 启用调试日志
kubectl edit cm argocd-cm -n argocd
# 添加：
# data:
#   server.log.level: debug

# 查看 ArgoCD 日志
kubectl logs -n argocd deployment/argocd-server -f
kubectl logs -n argocd deployment/argocd-repo-server -f
kubectl logs -n argocd deployment/argocd-application-controller -f

# 强制刷新应用
argocd app get myapp --refresh --hard-refresh

# 清理缓存
kubectl delete pod -n argocd -l app.kubernetes.io/name=argocd-repo-server
```

## 13. 学习检查清单

### 基础知识
- [ ] 理解 GitOps 的核心理念
- [ ] 掌握 ArgoCD 的核心概念
- [ ] 能够安装和配置 ArgoCD
- [ ] 了解 Application 和 Project 的作用

### 应用管理
- [ ] 能够创建和管理应用
- [ ] 掌握同步策略配置
- [ ] 了解健康检查机制
- [ ] 能够进行应用回滚

### 高级特性
- [ ] 掌握多集群管理
- [ ] 能够使用 ApplicationSet
- [ ] 了解同步波次和钩子
- [ ] 掌握 RBAC 权限配置

### 实战能力
- [ ] 能够实现完整的 GitOps 工作流
- [ ] 掌握多环境管理
- [ ] 能够实现蓝绿/金丝雀部署
- [ ] 能够排查常见问题

## 📚 参考资源

### 官方文档
- [ArgoCD 官方文档](https://argo-cd.readthedocs.io/)
- [ArgoCD GitHub](https://github.com/argoproj/argo-cd)
- [Argo Rollouts](https://argoproj.github.io/argo-rollouts/)

### 学习资源
- [GitOps 最佳实践](https://www.gitops.tech/)
- [ArgoCD 教程](https://codefresh.io/learn/argo-cd/)
- [Argo 项目](https://argoproj.github.io/)

### 工具和插件
- [ArgoCD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)
- [ArgoCD Notifications](https://argocd-notifications.readthedocs.io/)
- [ArgoCD Image Updater](https://argocd-image-updater.readthedocs.io/)
- [ApplicationSet](https://argocd-applicationset.readthedocs.io/)

### 相关技术
- Kubernetes
- Helm
- Kustomize
- Git

---

> 💡 **提示**：ArgoCD 是 GitOps 的最佳实践工具，将基础设施和应用配置存储在 Git 中，实现声明式管理。建议从简单的应用开始，逐步掌握多集群、ApplicationSet 等高级特性。合理使用同步策略和 RBAC 是生产环境的关键。
>
> @author erik.zhou

