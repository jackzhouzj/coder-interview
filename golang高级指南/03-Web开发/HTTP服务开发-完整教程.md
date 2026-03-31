# Go HTTP 服务开发 - 完整教程

> @author erik.zhou  
> 更新时间：2025-03-06

## 目录
- [HTTP 基础](#http-基础)
- [标准库 HTTP 服务](#标准库-http-服务)
- [路由设计](#路由设计)
- [请求处理](#请求处理)
- [响应处理](#响应处理)
- [中间件](#中间件)
- [错误处理](#错误处理)
- [实战案例](#实战案例)

## HTTP 基础

### HTTP 协议概述

HTTP（HyperText Transfer Protocol）是应用层协议，基于请求-响应模型。

### 请求结构

```
GET /api/users HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer token123

{"name": "Alice"}
```

### 响应结构

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 25

{"id": 1, "name": "Alice"}
```

## 标准库 HTTP 服务

### 最简单的 HTTP 服务

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, World!")
    })
    
    http.ListenAndServe(":8080", nil)
}
```

### 自定义 Server

```go
package main

import (
    "context"
    "log"
    "net/http"
    "os"
    "os/signal"
    "time"
)

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", homeHandler)
    
    server := &http.Server{
        Addr:         ":8080",
        Handler:      mux,
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 10 * time.Second,
        IdleTimeout:  120 * time.Second,
    }
    
    // 优雅关闭
    go func() {
        if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatalf("Server error: %v", err)
        }
    }()
    
    // 等待中断信号
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, os.Interrupt)
    <-quit
    
    log.Println("Shutting down server...")
    
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    if err := server.Shutdown(ctx); err != nil {
        log.Fatalf("Server forced to shutdown: %v", err)
    }
    
    log.Println("Server exited")
}

func homeHandler(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("Welcome!"))
}
```

## 路由设计

### 基础路由

```go
package main

import (
    "encoding/json"
    "net/http"
)

type User struct {
    ID   int    `json:"id"`
    Name string `json:"name"`
}

func main() {
    mux := http.NewServeMux()
    
    // 静态路由
    mux.HandleFunc("/", homeHandler)
    mux.HandleFunc("/users", usersHandler)
    mux.HandleFunc("/users/", userHandler)  // 注意末尾的 /
    
    http.ListenAndServe(":8080", mux)
}

func homeHandler(w http.ResponseWriter, r *http.Request) {
    if r.URL.Path != "/" {
        http.NotFound(w, r)
        return
    }
    w.Write([]byte("Home"))
}

func usersHandler(w http.ResponseWriter, r *http.Request) {
    sw