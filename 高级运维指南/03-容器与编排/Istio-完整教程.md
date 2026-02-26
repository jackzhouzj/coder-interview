# Istio完整教程

> Istio是领先的服务网格解决方案，提供流量管理、安全、可观测性等功能
>
> @author erik.zhou

## 📋 目录

- [Istio简介](#istio简介)
- [安装部署](#安装部署)
- [流量管理](#流量管理)
- [安全管理](#安全管理)
- [可观测性](#可观测性)
- [故障排查](#故障排查)

## 🎯 学习目标

- [ ] 理解服务网格概念和架构
- [ ] 掌握Istio安装和配置
- [ ] 实现智能流量管理
- [ ] 配置服务间安全通信
- [ ] 使用Istio可观测性功能
- [ ] 排查Istio常见问题

## Istio简介

### 什么是服务网格

服务网格是处理服务间通信的基础设施层，提供：
- 流量管理
- 安全通信
- 可观测性
- 策略执行

### Istio架构

```
Istio架构（Ambient模式）

┌─────────────────────────────────────────┐
│           Control Plane                  │
│  ┌──────────┐  ┌──────────┐            │
│  │  Istiod  │  │  Ingress │            │
│  │          │  │  Gateway │            │
│  └──────────┘  └──────────┘            │
└─────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│           Data Plane                     │
│  ┌──────────┐  ┌──────────┐            │
│  │  ztunnel │  │  waypoint│            │
│  │  (L4)    │  │  (L7)    │            │
│  └──────────┘  └──────────┘            │
│                                          │
│  ┌──────────┐  ┌──────────┐            │
│  │   App1   │  │   App2   │            │
│  └──────────┘  └──────────┘            │
└─────────────────────────────────────────┘
```

**核心组件**
- **Istiod**: 控制平面，提供服务发现、配置、证书管理
- **Envoy Proxy**: 数据平面，处理服务间通信
- **Ingress Gateway**: 入口网关
- **Egress Gateway**: 出口网关

### Istio特性

**流量管理**
- 智能路由
- 负载均衡
- 故障注入
- 超时重试
- 熔断器

**安全**
- mTLS加密
- 认证授权
- 证书管理

**可观测性**
- 指标收集
- 分布式追踪
- 访问日志

## 安装部署

### 环境准备

```bash
# 确保Kubernetes集群运行正常
kubectl cluster-info

# 检查节点状态
kubectl get nodes

# 推荐配置
# - Kubernetes 1.24+
# - 至少3个节点
# - 每个节点至少2核4G
```

### 安装Istio

**方法1：使用istioctl**
```bash
# 下载Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.20.0

# 添加到PATH
export PATH=$PWD/bin:$PATH

# 安装Istio（默认配置）
istioctl install --set profile=default -y

# 安装Istio（demo配置，用于测试）
istioctl install --set profile=demo -y

# 安装Istio（生产配置）
istioctl install --set profile=production -y

# 验证安装
istioctl verify-install

# 查看Istio组件
kubectl get pods -n istio-system
```

**方法2：使用Helm**
```bash
# 添加Istio Helm仓库
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update

# 安装Istio base
helm install istio-base istio/base -n istio-system --create-namespace

# 安装Istiod
helm install istiod istio/istiod -n istio-system --wait

# 安装Ingress Gateway
helm install istio-ingress istio/gateway -n istio-system
```

### 启用Sidecar注入

**自动注入**
```bash
# 为命名空间启用自动注入
kubectl label namespace default istio-injection=enabled

# 验证标签
kubectl get namespace -L istio-injection

# 重新部署应用以注入sidecar
kubectl rollout restart deployment -n default
```

**手动注入**
```bash
# 手动注入sidecar
istioctl kube-inject -f deployment.yaml | kubectl apply -f -

# 或者
kubectl apply -f <(istioctl kube-inject -f deployment.yaml)
```

### 部署示例应用

```bash
# 部署Bookinfo示例应用
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

# 验证部署
kubectl get pods
kubectl get services

# 验证应用
kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') \
  -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
```

### 配置Ingress Gateway

```yaml
# bookinfo-gateway.yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```

```bash
# 应用配置
kubectl apply -f bookinfo-gateway.yaml

# 获取Ingress Gateway地址
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway \
  -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway \
  -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

# 访问应用
curl -s http://${GATEWAY_URL}/productpage | grep -o "<title>.*</title>"
```

## 流量管理

### VirtualService

**基本路由**
```yaml
# virtualservice-reviews.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 50
    - destination:
        host: reviews
        subset: v2
      weight: 50
```

**基于请求头路由**
```yaml
# virtualservice-reviews-header.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```

**基于URI路由**
```yaml
# virtualservice-uri.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: api
spec:
  hosts:
  - api.example.com
  http:
  - match:
    - uri:
        prefix: /v1
    route:
    - destination:
        host: api-v1
  - match:
    - uri:
        prefix: /v2
    route:
    - destination:
        host: api-v2
```

### DestinationRule

**定义服务子集**
```yaml
# destinationrule-reviews.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v3
    labels:
      version: v3
```

**连接池配置**
```yaml
# destinationrule-circuit-breaker.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 1
        http2MaxRequests: 100
        maxRequestsPerConnection: 2
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
```

### 超时和重试

```yaml
# virtualservice-timeout-retry.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
    timeout: 10s
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: 5xx,reset,connect-failure,refused-stream
```

### 故障注入

**延迟注入**
```yaml
# virtualservice-delay.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    fault:
      delay:
        percentage:
          value: 100.0
        fixedDelay: 7s
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

**中断注入**
```yaml
# virtualservice-abort.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    fault:
      abort:
        percentage:
          value: 100.0
        httpStatus: 500
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

### 流量镜像

```yaml
# virtualservice-mirror.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 100
    mirror:
      host: reviews
      subset: v2
    mirrorPercentage:
      value: 100.0
```

## 安全管理

### mTLS配置

**启用严格mTLS**
```yaml
# peer-authentication-strict.yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT
```

**命名空间级别mTLS**
```yaml
# peer-authentication-namespace.yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT
```

**服务级别mTLS**
```yaml
# peer-authentication-service.yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: reviews
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviews
  mtls:
    mode: STRICT
```

### 授权策略

**默认拒绝所有**
```yaml
# authz-deny-all.yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  {}
```

**允许特定服务访问**
```yaml
# authz-allow-productpage.yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-productpage
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviews
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/bookinfo-productpage"]
    to:
    - operation:
        methods: ["GET"]
```

**基于JWT的授权**
```yaml
# authz-jwt.yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviews
  jwtRules:
  - issuer: "testing@secure.istio.io"
    jwksUri: "https://raw.githubusercontent.com/istio/istio/release-1.20/security/tools/jwt/samples/jwks.json"
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: require-jwt
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviews
  action: ALLOW
  rules:
  - from:
    - source:
        requestPrincipals: ["testing@secure.istio.io/testing@secure.istio.io"]
```

### 证书管理

```bash
# 查看证书
istioctl proxy-config secret <pod-name> -n default

# 验证mTLS
istioctl authn tls-check <pod-name> -n default

# 查看证书详情
kubectl exec <pod-name> -c istio-proxy -n default -- \
  openssl s_client -showcerts -connect reviews:9080
```

## 可观测性

### 指标收集

**安装Prometheus**
```bash
kubectl apply -f samples/addons/prometheus.yaml

# 访问Prometheus
istioctl dashboard prometheus
```

**查询指标**
```promql
# 请求速率
rate(istio_requests_total[5m])

# 错误率
rate(istio_requests_total{response_code=~"5.*"}[5m]) / 
rate(istio_requests_total[5m])

# 请求延迟
histogram_quantile(0.99, 
  rate(istio_request_duration_milliseconds_bucket[5m]))
```

### 分布式追踪

**安装Jaeger**
```bash
kubectl apply -f samples/addons/jaeger.yaml

# 访问Jaeger UI
istioctl dashboard jaeger
```

**配置追踪采样率**
```yaml
# configmap-tracing.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: istio
  namespace: istio-system
data:
  mesh: |-
    defaultConfig:
      tracing:
        sampling: 100.0
        zipkin:
          address: jaeger-collector.istio-system:9411
```

### 可视化

**安装Kiali**
```bash
kubectl apply -f samples/addons/kiali.yaml

# 访问Kiali
istioctl dashboard kiali
```

**安装Grafana**
```bash
kubectl apply -f samples/addons/grafana.yaml

# 访问Grafana
istioctl dashboard grafana
```

### 访问日志

**启用访问日志**
```yaml
# telemetry-access-log.yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: mesh-default
  namespace: istio-system
spec:
  accessLogging:
  - providers:
    - name: envoy
```

**自定义日志格式**
```yaml
# telemetry-custom-log.yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: custom-logging
  namespace: default
spec:
  accessLogging:
  - providers:
    - name: envoy
    filter:
      expression: response.code >= 400
```

## 故障排查

### 常用诊断命令

```bash
# 查看Istio配置
istioctl analyze

# 查看代理配置
istioctl proxy-config cluster <pod-name> -n default
istioctl proxy-config listener <pod-name> -n default
istioctl proxy-config route <pod-name> -n default
istioctl proxy-config endpoint <pod-name> -n default

# 查看代理状态
istioctl proxy-status

# 查看代理日志
kubectl logs <pod-name> -c istio-proxy -n default

# 调试特定服务
istioctl experimental describe pod <pod-name> -n default
```

### 常见问题

**1. Sidecar未注入**
```bash
# 检查命名空间标签
kubectl get namespace -L istio-injection

# 检查Pod注解
kubectl get pod <pod-name> -o yaml | grep sidecar.istio.io/inject

# 手动注入
istioctl kube-inject -f deployment.yaml | kubectl apply -f -
```

**2. 服务无法访问**
```bash
# 检查VirtualService
kubectl get virtualservice -n default

# 检查DestinationRule
kubectl get destinationrule -n default

# 检查Gateway
kubectl get gateway -n default

# 验证配置
istioctl analyze -n default
```

**3. mTLS问题**
```bash
# 检查mTLS状态
istioctl authn tls-check <pod-name> -n default

# 查看PeerAuthentication
kubectl get peerauthentication -A

# 查看证书
istioctl proxy-config secret <pod-name> -n default
```

**4. 性能问题**
```bash
# 查看资源使用
kubectl top pod -n istio-system

# 查看代理统计
istioctl experimental metrics <pod-name> -n default

# 调整资源限制
kubectl set resources deployment istio-ingressgateway \
  -n istio-system \
  --limits=cpu=2000m,memory=1024Mi \
  --requests=cpu=100m,memory=128Mi
```

### 调试工具

**使用istioctl debug**
```bash
# 生成调试报告
istioctl bug-report

# 查看配置差异
istioctl experimental config diff <file1> <file2>

# 验证配置
istioctl validate -f virtualservice.yaml
```

## 生产最佳实践

### 1. 资源配置

```yaml
# istio-operator.yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  profile: production
  components:
    pilot:
      k8s:
        resources:
          requests:
            cpu: 500m
            memory: 2048Mi
          limits:
            cpu: 1000m
            memory: 4096Mi
        hpaSpec:
          minReplicas: 2
          maxReplicas: 5
    ingressGateways:
    - name: istio-ingressgateway
      enabled: true
      k8s:
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 2000m
            memory: 1024Mi
        hpaSpec:
          minReplicas: 2
          maxReplicas: 5
```

### 2. 高可用配置

```yaml
# 多副本部署
spec:
  components:
    pilot:
      k8s:
        replicaCount: 3
        affinity:
          podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  app: istiod
              topologyKey: kubernetes.io/hostname
```

### 3. 安全加固

```yaml
# 启用严格mTLS
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT

# 默认拒绝策略
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: istio-system
spec:
  {}
```

### 4. 监控告警

```yaml
# prometheus-rules.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-rules
  namespace: istio-system
data:
  alert.rules: |-
    groups:
    - name: istio
      interval: 30s
      rules:
      - alert: HighErrorRate
        expr: rate(istio_requests_total{response_code=~"5.*"}[5m]) > 0.05
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High error rate detected"
```

## 📚 学习检查清单

- [ ] 理解服务网格概念
- [ ] 掌握Istio安装配置
- [ ] 实现流量管理
- [ ] 配置安全策略
- [ ] 使用可观测性功能
- [ ] 排查常见问题

## 🔗 参考资源

- [Istio官方文档](https://istio.io/latest/docs/)
- [Istio最佳实践](https://istio.io/latest/docs/ops/best-practices/)
- [Istio性能调优](https://istio.io/latest/docs/ops/performance-and-scalability/)

---

@author erik.zhou
