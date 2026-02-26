# Helm完整教程

> Helm是Kubernetes的包管理器，简化应用部署和管理
>
> @author erik.zhou

## 📋 目录

- [Helm简介](#helm简介)
- [安装配置](#安装配置)
- [Chart开发](#chart开发)
- [应用部署](#应用部署)
- [高级特性](#高级特性)
- [最佳实践](#最佳实践)

## 🎯 学习目标

- [ ] 理解Helm架构和概念
- [ ] 掌握Helm基本命令
- [ ] 能够开发自定义Chart
- [ ] 使用Helm部署应用
- [ ] 管理Chart仓库
- [ ] 掌握Helm最佳实践

## Helm简介

### 什么是Helm

Helm是Kubernetes的包管理器，类似于Linux的yum/apt，用于简化Kubernetes应用的部署和管理。

**核心概念**
- **Chart**: Helm包，包含运行应用所需的所有资源定义
- **Release**: Chart的运行实例
- **Repository**: Chart仓库，用于存储和分享Chart

**Helm的优势**
- 简化部署流程
- 版本管理和回滚
- 配置管理
- 依赖管理
- 模板化配置

### Helm架构

```
Helm v3架构（移除了Tiller）

┌─────────────┐
│  Helm CLI   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Kubernetes  │
│  API Server │
└─────────────┘
```

## 安装配置

### 安装Helm

**方法1：使用脚本安装**
```bash
# 下载安装脚本
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# 验证安装
helm version
```

**方法2：二进制安装**
```bash
# 下载Helm
wget https://get.helm.sh/helm-v3.13.0-linux-amd64.tar.gz

# 解压
tar -zxvf helm-v3.13.0-linux-amd64.tar.gz

# 移动到PATH
sudo mv linux-amd64/helm /usr/local/bin/helm

# 验证
helm version
```

**方法3：包管理器安装**
```bash
# Ubuntu/Debian
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

# CentOS/RHEL
sudo yum install helm
```

### 配置Helm

```bash
# 添加官方Chart仓库
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami

# 更新仓库
helm repo update

# 查看仓库列表
helm repo list

# 搜索Chart
helm search repo nginx

# 查看Chart信息
helm show chart bitnami/nginx
helm show values bitnami/nginx
```

## Chart开发

### Chart结构

```
mychart/
├── Chart.yaml          # Chart元数据
├── values.yaml         # 默认配置值
├── charts/             # 依赖的Chart
├── templates/          # 模板文件
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl    # 模板辅助函数
│   └── NOTES.txt       # 安装后的提示信息
└── .helmignore         # 忽略文件
```

### 创建Chart

```bash
# 创建新Chart
helm create mychart

# 查看Chart结构
tree mychart/

# 验证Chart
helm lint mychart/

# 渲染模板（不安装）
helm template mychart/

# 打包Chart
helm package mychart/
```

### Chart.yaml

```yaml
# Chart.yaml
apiVersion: v2
name: mychart
description: A Helm chart for my application
type: application
version: 0.1.0
appVersion: "1.0"
keywords:
  - web
  - nginx
home: https://example.com
sources:
  - https://github.com/example/mychart
maintainers:
  - name: Erik Zhou
    email: erik.zhou@example.com
dependencies:
  - name: mysql
    version: 9.3.4
    repository: https://charts.bitnami.com/bitnami
    condition: mysql.enabled
```

### values.yaml

```yaml
# values.yaml - 默认配置值

# 副本数
replicaCount: 3

# 镜像配置
image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.21"

# 镜像拉取密钥
imagePullSecrets: []

# 服务配置
service:
  type: ClusterIP
  port: 80
  targetPort: 8080

# Ingress配置
ingress:
  enabled: false
  className: "nginx"
  annotations: {}
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: Prefix
  tls: []

# 资源限制
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi

# 自动扩缩容
autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80

# 节点选择器
nodeSelector: {}

# 容忍度
tolerations: []

# 亲和性
affinity: {}

# 环境变量
env:
  - name: APP_ENV
    value: "production"

# ConfigMap
configMap:
  data:
    app.conf: |
      server {
        listen 80;
        server_name localhost;
      }
```

### 模板文件

**deployment.yaml**
```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.targetPort }}
          protocol: TCP
        {{- if .Values.env }}
        env:
          {{- toYaml .Values.env | nindent 10 }}
        {{- end }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
```

**service.yaml**
```yaml
# templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "mychart.selectorLabels" . | nindent 4 }}
```

**_helpers.tpl**
```yaml
# templates/_helpers.tpl
{{/*
生成Chart全名
*/}}
{{- define "mychart.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
生成标签
*/}}
{{- define "mychart.labels" -}}
helm.sh/chart: {{ include "mychart.chart" . }}
{{ include "mychart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
选择器标签
*/}}
{{- define "mychart.selectorLabels" -}}
app.kubernetes.io/name: {{ include "mychart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

### 模板函数

```yaml
# 常用模板函数

# 1. 字符串函数
{{ .Values.name | upper }}           # 转大写
{{ .Values.name | lower }}           # 转小写
{{ .Values.name | quote }}           # 添加引号
{{ .Values.name | trim }}            # 去除空格
{{ .Values.name | trunc 10 }}        # 截断字符串

# 2. 默认值
{{ .Values.name | default "default-name" }}

# 3. 条件判断
{{- if .Values.enabled }}
enabled: true
{{- else }}
enabled: false
{{- end }}

# 4. 循环
{{- range .Values.items }}
- {{ . }}
{{- end }}

# 5. 包含其他模板
{{ include "mychart.labels" . }}

# 6. 缩进
{{ .Values.data | nindent 4 }}

# 7. toYaml
{{- toYaml .Values.resources | nindent 10 }}

# 8. 字典操作
{{ .Values.config.key }}
{{ index .Values.config "key" }}

# 9. 列表操作
{{ index .Values.list 0 }}
```

## 应用部署

### 基本操作

```bash
# 安装Chart
helm install myapp mychart/

# 指定命名空间
helm install myapp mychart/ -n production

# 使用自定义values
helm install myapp mychart/ -f custom-values.yaml

# 设置单个值
helm install myapp mychart/ --set replicaCount=5

# 设置多个值
helm install myapp mychart/ \
  --set replicaCount=5 \
  --set image.tag=1.22

# 生成名称
helm install mychart/ --generate-name

# 查看安装状态
helm status myapp

# 查看Release列表
helm list
helm list -n production
helm list --all-namespaces
```

### 升级和回滚

```bash
# 升级Release
helm upgrade myapp mychart/

# 升级并修改配置
helm upgrade myapp mychart/ -f new-values.yaml

# 升级或安装（不存在则安装）
helm upgrade --install myapp mychart/

# 查看历史版本
helm history myapp

# 回滚到上一版本
helm rollback myapp

# 回滚到指定版本
helm rollback myapp 2

# 查看升级差异
helm diff upgrade myapp mychart/ -f new-values.yaml
```

### 卸载

```bash
# 卸载Release
helm uninstall myapp

# 卸载并保留历史
helm uninstall myapp --keep-history

# 查看已删除的Release
helm list --uninstalled
```

### 调试

```bash
# 渲染模板（不安装）
helm template myapp mychart/

# 调试模式安装
helm install myapp mychart/ --debug --dry-run

# 查看生成的manifest
helm get manifest myapp

# 查看values
helm get values myapp

# 查看所有信息
helm get all myapp
```

## 高级特性

### 依赖管理

**Chart.yaml中定义依赖**
```yaml
dependencies:
  - name: mysql
    version: 9.3.4
    repository: https://charts.bitnami.com/bitnami
    condition: mysql.enabled
  - name: redis
    version: 17.3.7
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

**管理依赖**
```bash
# 下载依赖
helm dependency update mychart/

# 查看依赖
helm dependency list mychart/

# 构建依赖
helm dependency build mychart/
```

**values.yaml中配置依赖**
```yaml
# 启用MySQL
mysql:
  enabled: true
  auth:
    rootPassword: "root123"
    database: "myapp"
  primary:
    persistence:
      enabled: true
      size: 10Gi

# 启用Redis
redis:
  enabled: true
  auth:
    password: "redis123"
  master:
    persistence:
      enabled: true
      size: 8Gi
```

### Hooks

Helm Hooks允许在Release生命周期的特定点执行操作。

**Hook类型**
- pre-install: 安装前
- post-install: 安装后
- pre-delete: 删除前
- post-delete: 删除后
- pre-upgrade: 升级前
- post-upgrade: 升级后
- pre-rollback: 回滚前
- post-rollback: 回滚后
- test: 测试

**示例：数据库迁移Hook**
```yaml
# templates/db-migration-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "mychart.fullname" . }}-db-migration
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "0"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: db-migration
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        command: ["./migrate.sh"]
        env:
        - name: DB_HOST
          value: {{ .Values.mysql.host }}
```

### 测试

```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "mychart.fullname" . }}-test-connection"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: wget
    image: busybox
    command: ['wget']
    args: ['{{ include "mychart.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

```bash
# 运行测试
helm test myapp

# 查看测试日志
kubectl logs -f myapp-test-connection
```

### Chart仓库

**创建Chart仓库**
```bash
# 打包Chart
helm package mychart/

# 生成索引文件
helm repo index . --url https://charts.example.com

# 上传到HTTP服务器
# 将Chart包和index.yaml上传到Web服务器
```

**使用私有仓库**
```bash
# 添加私有仓库
helm repo add myrepo https://charts.example.com

# 带认证的仓库
helm repo add myrepo https://charts.example.com \
  --username admin \
  --password password

# 更新仓库
helm repo update

# 搜索Chart
helm search repo myrepo/
```

**使用Harbor作为Chart仓库**
```bash
# 添加Harbor仓库
helm repo add harbor https://harbor.example.com/chartrepo/library \
  --username admin \
  --password Harbor12345

# 推送Chart到Harbor
helm push mychart-0.1.0.tgz harbor
```

## 最佳实践

### 1. Chart开发规范

```yaml
# 使用语义化版本
version: 1.2.3

# 提供详细的README.md
# 包含：
# - Chart描述
# - 安装说明
# - 配置参数说明
# - 示例

# 使用.helmignore排除不必要的文件
.git/
.gitignore
*.md
.DS_Store
```

### 2. Values设计

```yaml
# 使用清晰的层级结构
image:
  repository: nginx
  tag: 1.21
  pullPolicy: IfNotPresent

# 提供合理的默认值
replicaCount: 1

# 使用enabled开关
ingress:
  enabled: false

# 提供注释说明
# -- 副本数量
replicaCount: 1
```

### 3. 模板最佳实践

```yaml
# 使用辅助模板
{{- define "mychart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

# 使用条件判断
{{- if .Values.ingress.enabled }}
# ingress配置
{{- end }}

# 使用循环
{{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value }}
{{- end }}

# 使用缩进控制
{{- toYaml .Values.resources | nindent 10 }}
```

### 4. 安全实践

```yaml
# 不在values.yaml中存储敏感信息
# 使用Kubernetes Secret

# 使用imagePullSecrets
imagePullSecrets:
  - name: regcred

# 设置安全上下文
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000

# 设置资源限制
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

### 5. 生产环境部署

```bash
# 使用命名空间隔离
helm install myapp mychart/ -n production

# 使用多个values文件
helm install myapp mychart/ \
  -f values.yaml \
  -f values-production.yaml

# 设置超时时间
helm install myapp mychart/ --timeout 10m

# 等待所有资源就绪
helm install myapp mychart/ --wait

# 原子操作（失败自动回滚）
helm install myapp mychart/ --atomic
```

### 6. CI/CD集成

```yaml
# .gitlab-ci.yml
stages:
  - lint
  - package
  - deploy

lint:
  stage: lint
  script:
    - helm lint mychart/

package:
  stage: package
  script:
    - helm package mychart/
    - helm repo index .
  artifacts:
    paths:
      - "*.tgz"
      - index.yaml

deploy:
  stage: deploy
  script:
    - helm upgrade --install myapp mychart/ \
        -f values-production.yaml \
        --namespace production \
        --wait \
        --timeout 10m
  only:
    - main
```

## 实战案例

### 部署WordPress

```bash
# 添加bitnami仓库
helm repo add bitnami https://charts.bitnami.com/bitnami

# 创建自定义values
cat > wordpress-values.yaml <<EOF
wordpressUsername: admin
wordpressPassword: admin123
wordpressEmail: admin@example.com
wordpressBlogName: My Blog

service:
  type: LoadBalancer

persistence:
  enabled: true
  size: 10Gi

mariadb:
  auth:
    rootPassword: root123
    database: wordpress
  primary:
    persistence:
      enabled: true
      size: 8Gi
EOF

# 安装WordPress
helm install wordpress bitnami/wordpress \
  -f wordpress-values.yaml \
  --namespace wordpress \
  --create-namespace

# 查看状态
helm status wordpress -n wordpress

# 获取访问信息
kubectl get svc -n wordpress
```

### 部署监控栈

```bash
# 添加prometheus社区仓库
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# 安装kube-prometheus-stack
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set prometheus.prometheusSpec.retention=30d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=50Gi \
  --set grafana.adminPassword=admin123

# 访问Grafana
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

## 📚 学习检查清单

- [ ] 理解Helm核心概念
- [ ] 掌握Helm基本命令
- [ ] 能够开发自定义Chart
- [ ] 使用Helm部署应用
- [ ] 管理Chart依赖
- [ ] 掌握Helm最佳实践

## 🔗 参考资源

- [Helm官方文档](https://helm.sh/docs/)
- [Artifact Hub](https://artifacthub.io/) - Chart仓库
- [Helm最佳实践](https://helm.sh/docs/chart_best_practices/)

---

@author erik.zhou
