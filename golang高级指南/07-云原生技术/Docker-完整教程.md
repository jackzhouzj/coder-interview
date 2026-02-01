# Docker - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **Docker版本**：24.x+
- **Docker Compose**：2.x+
- **推荐版本**：最新稳定版

### 学习难度
- **难度等级**：⭐⭐⭐ (中等)
- **预计学习时间**：15-20小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Linux基础
- Go语言基础
- 容器化概念

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解Docker的工作原理
- [ ] 掌握Docker镜像和容器操作
- [ ] 能够编写Dockerfile
- [ ] 掌握Docker Compose的使用
- [ ] 理解Docker网络和存储
- [ ] 能够部署Go应用到Docker
- [ ] 掌握Docker的最佳实践

## 📖 目录

1. [Docker简介](#1-docker简介)
2. [Docker安装](#2-docker安装)
3. [镜像操作](#3-镜像操作)
4. [容器操作](#4-容器操作)
5. [Dockerfile](#5-dockerfile)
6. [Docker Compose](#6-docker-compose)
7. [网络和存储](#7-网络和存储)
8. [最佳实践](#8-最佳实践)

---

## 1. Docker简介

### 1.1 什么是Docker

Docker是一个开源的容器化平台，用于开发、交付和运行应用程序。

**核心特点**：
- 🔥 **轻量级**：比虚拟机更轻量
- 🔥 **可移植**：一次构建，到处运行
- 🔥 **隔离性**：应用之间相互隔离
- 🔥 **快速启动**：秒级启动
- 🔥 **版本控制**：镜像支持版本管理

### 1.2 Docker架构

```
Docker架构：
┌─────────────────────────────────────┐
│         Docker Client               │
│    (docker命令行工具)                │
└──────────────┬──────────────────────┘
               │
               ↓
┌─────────────────────────────────────┐
│         Docker Daemon               │
│    (dockerd后台进程)                 │
│  ┌──────────┬──────────┬──────────┐ │
│  │  Images  │Containers│ Networks │ │
│  └──────────┴──────────┴──────────┘ │
└─────────────────────────────────────┘
               │
               ↓
┌─────────────────────────────────────┐
│         Docker Registry             │
│    (Docker Hub / 私有仓库)           │
└─────────────────────────────────────┘
```

---

## 2. Docker安装

### 2.1 Linux安装

```bash
# 🔥 Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 🔥 启动Docker
sudo systemctl start docker
sudo systemctl enable docker

# 🔥 添加当前用户到docker组
sudo usermod -aG docker $USER

# 🔥 验证安装
docker --version
docker run hello-world
```

### 2.2 配置镜像加速

```bash
# 🔥 配置国内镜像源
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com"
  ]
}
EOF

# 🔥 重启Docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

---

## 3. 镜像操作

### 3.1 镜像基本命令

```bash
# 🔥 搜索镜像
docker search golang

# 🔥 拉取镜像
docker pull golang:1.21

# 🔥 查看本地镜像
docker images

# 🔥 删除镜像
docker rmi golang:1.21

# 🔥 查看镜像详情
docker inspect golang:1.21

# 🔥 查看镜像历史
docker history golang:1.21

# 🔥 导出镜像
docker save -o golang.tar golang:1.21

# 🔥 导入镜像
docker load -i golang.tar

# 🔥 给镜像打标签
docker tag golang:1.21 myregistry.com/golang:1.21

# 🔥 推送镜像
docker push myregistry.com/golang:1.21
```

---

## 4. 容器操作

### 4.1 容器基本命令

```bash
# 🔥 运行容器
docker run -d --name myapp -p 8080:8080 golang:1.21

# 🔥 查看运行中的容器
docker ps

# 🔥 查看所有容器
docker ps -a

# 🔥 停止容器
docker stop myapp

# 🔥 启动容器
docker start myapp

# 🔥 重启容器
docker restart myapp

# 🔥 删除容器
docker rm myapp

# 🔥 强制删除运行中的容器
docker rm -f myapp

# 🔥 查看容器日志
docker logs myapp
docker logs -f myapp  # 实时查看

# 🔥 进入容器
docker exec -it myapp /bin/bash

# 🔥 查看容器详情
docker inspect myapp

# 🔥 查看容器资源使用
docker stats myapp
```

### 4.2 容器运行选项

```bash
# 🔥 后台运行
docker run -d nginx

# 🔥 端口映射
docker run -d -p 8080:80 nginx

# 🔥 挂载卷
docker run -d -v /host/path:/container/path nginx

# 🔥 环境变量
docker run -d -e KEY=value nginx

# 🔥 容器名称
docker run -d --name mynginx nginx

# 🔥 自动重启
docker run -d --restart=always nginx

# 🔥 限制资源
docker run -d --memory=512m --cpus=1 nginx

# 🔥 网络模式
docker run -d --network=host nginx
```

---

## 5. Dockerfile

### 5.1 基础Dockerfile

```dockerfile
# 🔥 基础镜像
FROM golang:1.21-alpine

# 🔥 设置工作目录
WORKDIR /app

# 🔥 复制文件
COPY . .

# 🔥 下载依赖
RUN go mod download

# 🔥 构建应用
RUN go build -o main .

# 🔥 暴露端口
EXPOSE 8080

# 🔥 运行命令
CMD ["./main"]
```

### 5.2 多阶段构建

```dockerfile
# 🔥 第一阶段：构建
FROM golang:1.21-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# 🔥 第二阶段：运行
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

# 🔥 从构建阶段复制二进制文件
COPY --from=builder /app/main .

EXPOSE 8080

CMD ["./main"]
```

### 5.3 优化Dockerfile

```dockerfile
# 🔥 使用特定版本
FROM golang:1.21.0-alpine3.18

# 🔥 设置环境变量
ENV GO111MODULE=on \
    GOPROXY=https://goproxy.cn,direct

WORKDIR /app

# 🔥 先复制依赖文件（利用缓存）
COPY go.mod go.sum ./
RUN go mod download

# 🔥 再复制源代码
COPY . .

# 🔥 构建优化
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -ldflags="-w -s" -o main .

# 🔥 使用scratch最小镜像
FROM scratch

COPY --from=0 /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=0 /app/main /main

EXPOSE 8080

ENTRYPOINT ["/main"]
```

### 5.4 构建镜像

```bash
# 🔥 构建镜像
docker build -t myapp:v1.0 .

# 🔥 指定Dockerfile
docker build -f Dockerfile.prod -t myapp:v1.0 .

# 🔥 使用构建参数
docker build --build-arg VERSION=1.0 -t myapp:v1.0 .

# 🔥 不使用缓存
docker build --no-cache -t myapp:v1.0 .
```

---

## 6. Docker Compose

### 6.1 基础配置

```yaml
# docker-compose.yml
version: '3.8'

services:
  # 🔥 Go应用
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=db
      - DB_PORT=5432
    depends_on:
      - db
      - redis
    networks:
      - mynetwork

  # 🔥 PostgreSQL数据库
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - mynetwork

  # 🔥 Redis缓存
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - mynetwork

volumes:
  db_data:

networks:
  mynetwork:
    driver: bridge
```

### 6.2 Docker Compose命令

```bash
# 🔥 启动服务
docker-compose up

# 🔥 后台启动
docker-compose up -d

# 🔥 停止服务
docker-compose down

# 🔥 查看服务状态
docker-compose ps

# 🔥 查看日志
docker-compose logs
docker-compose logs -f app

# 🔥 重启服务
docker-compose restart

# 🔥 构建镜像
docker-compose build

# 🔥 执行命令
docker-compose exec app sh
```

### 6.3 完整示例

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: myapp
    restart: always
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_USER=user
      - DB_PASSWORD=password
      - DB_NAME=mydb
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - backend
    volumes:
      - ./logs:/app/logs

  postgres:
    image: postgres:15-alpine
    container_name: postgres
    restart: always
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: redis
    restart: always
    ports:
      - "6379:6379"
    networks:
      - backend
    volumes:
      - redis_data:/data

  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - app
    networks:
      - backend

volumes:
  postgres_data:
  redis_data:

networks:
  backend:
    driver: bridge
```

---

## 7. 网络和存储

### 7.1 Docker网络

```bash
# 🔥 查看网络
docker network ls

# 🔥 创建网络
docker network create mynetwork

# 🔥 查看网络详情
docker network inspect mynetwork

# 🔥 连接容器到网络
docker network connect mynetwork myapp

# 🔥 断开网络连接
docker network disconnect mynetwork myapp

# 🔥 删除网络
docker network rm mynetwork
```

### 7.2 Docker卷

```bash
# 🔥 查看卷
docker volume ls

# 🔥 创建卷
docker volume create myvolume

# 🔥 查看卷详情
docker volume inspect myvolume

# 🔥 删除卷
docker volume rm myvolume

# 🔥 清理未使用的卷
docker volume prune
```

---

## 8. 最佳实践

### 8.1 Dockerfile最佳实践

```dockerfile
# ✅ 使用官方基础镜像
FROM golang:1.21-alpine

# ✅ 使用多阶段构建减小镜像大小
FROM golang:1.21-alpine AS builder
# ... 构建
FROM alpine:latest
# ... 运行

# ✅ 合并RUN命令减少层数
RUN apk add --no-cache git && \
    go mod download && \
    go build -o main .

# ✅ 利用构建缓存
COPY go.mod go.sum ./
RUN go mod download
COPY . .

# ✅ 使用.dockerignore
# .dockerignore内容：
# .git
# .gitignore
# README.md
# *.md

# ✅ 不要在镜像中存储敏感信息
# 使用环境变量或secrets

# ✅ 使用非root用户运行
RUN adduser -D -u 1000 appuser
USER appuser
```

### 8.2 安全最佳实践

```dockerfile
# ✅ 使用特定版本标签
FROM golang:1.21.0-alpine3.18

# ✅ 扫描镜像漏洞
# docker scan myapp:v1.0

# ✅ 使用最小基础镜像
FROM scratch

# ✅ 不要以root用户运行
USER 1000:1000

# ✅ 只暴露必要的端口
EXPOSE 8080

# ✅ 使用只读文件系统
docker run --read-only myapp
```

### 8.3 性能优化

```bash
# ✅ 限制资源使用
docker run -d \
  --memory=512m \
  --cpus=1 \
  --name myapp \
  myapp:v1.0

# ✅ 使用健康检查
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8080/health || exit 1

# ✅ 使用日志驱动
docker run -d \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myapp:v1.0
```

---

## 📝 学习检查清单

- [ ] 理解Docker的工作原理
- [ ] 掌握Docker镜像和容器操作
- [ ] 能够编写Dockerfile
- [ ] 掌握多阶段构建
- [ ] 掌握Docker Compose的使用
- [ ] 理解Docker网络和存储
- [ ] 掌握Docker的最佳实践
- [ ] 能够部署Go应用到Docker

---

## 🔗 相关资源

- [Docker官方文档](https://docs.docker.com/)
- [Dockerfile最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Compose文档](https://docs.docker.com/compose/)
- [Docker Hub](https://hub.docker.com/)

---

@author erik.zhou
