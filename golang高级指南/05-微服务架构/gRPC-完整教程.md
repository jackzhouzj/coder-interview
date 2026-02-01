# gRPC - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **gRPC版本**：1.60+
- **Protocol Buffers**：3.0+
- **Go版本**：1.21+

### 学习难度
- **难度等级**：⭐⭐⭐⭐ (较难)
- **预计学习时间**：20-25小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Go语言基础
- 网络编程基础
- Protocol Buffers基础
- 微服务概念

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解gRPC的工作原理
- [ ] 掌握Protocol Buffers的使用
- [ ] 能够开发gRPC服务
- [ ] 掌握四种通信模式
- [ ] 理解拦截器的使用
- [ ] 掌握错误处理和超时控制
- [ ] 能够实现服务注册与发现

## 📖 目录

1. [gRPC简介](#1-grpc简介)
2. [Protocol Buffers](#2-protocol-buffers)
3. [简单RPC](#3-简单rpc)
4. [流式RPC](#4-流式rpc)
5. [拦截器](#5-拦截器)
6. [错误处理](#6-错误处理)
7. [高级特性](#7-高级特性)
8. [最佳实践](#8-最佳实践)

---

## 1. gRPC简介

### 1.1 什么是gRPC

gRPC是Google开源的高性能RPC框架，基于HTTP/2协议。

**核心特点**：
- 🔥 **高性能**：基于HTTP/2，支持多路复用
- 🔥 **跨语言**：支持多种编程语言
- 🔥 **强类型**：使用Protocol Buffers定义接口
- 🔥 **流式传输**：支持客户端流、服务端流、双向流
- 🔥 **负载均衡**：内置负载均衡支持

### 1.2 安装依赖

```bash
# 🔥 安装gRPC
go get -u google.golang.org/grpc

# 🔥 安装protoc编译器
# 下载地址：https://github.com/protocolbuffers/protobuf/releases

# 🔥 安装Go插件
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

# 🔥 配置PATH
export PATH="$PATH:$(go env GOPATH)/bin"
```

---

## 2. Protocol Buffers

### 2.1 定义消息

```protobuf
// user.proto
syntax = "proto3";

package user;

option go_package = "./proto";

// 🔥 定义消息
message User {
  int64 id = 1;
  string name = 2;
  string email = 3;
  int32 age = 4;
}

message GetUserRequest {
  int64 id = 1;
}

message GetUserResponse {
  User user = 1;
}

message ListUsersRequest {
  int32 page = 1;
  int32 page_size = 2;
}

message ListUsersResponse {
  repeated User users = 1;
  int32 total = 2;
}
```

### 2.2 定义服务

```protobuf
// user.proto
syntax = "proto3";

package user;

option go_package = "./proto";

// 🔥 定义服务
service UserService {
  // 简单RPC
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  
  // 服务端流式RPC
  rpc ListUsers(ListUsersRequest) returns (stream User);
  
  // 客户端流式RPC
  rpc CreateUsers(stream User) returns (CreateUsersResponse);
  
  // 双向流式RPC
  rpc Chat(stream ChatMessage) returns (stream ChatMessage);
}

message CreateUsersResponse {
  int32 count = 1;
}

message ChatMessage {
  string user = 1;
  string message = 2;
}
```

### 2.3 生成代码

```bash
# 🔥 生成Go代码
protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    user.proto
```

---

## 3. 简单RPC

### 3.1 实现服务端

```go
// server/main.go
package main

import (
    "context"
    "fmt"
    "log"
    "net"
    
    "google.golang.org/grpc"
    pb "your-module/proto"
)

// 🔥 实现UserService接口
type userServer struct {
    pb.UnimplementedUserServiceServer
}

func (s *userServer) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.GetUserResponse, error) {
    // 模拟从数据库获取用户
    user := &pb.User{
        Id:    req.Id,
        Name:  "张三",
        Email: "zhangsan@example.com",
        Age:   25,
    }
    
    return &pb.GetUserResponse{User: user}, nil
}

func main() {
    // 🔥 创建监听器
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }
    
    // 🔥 创建gRPC服务器
    s := grpc.NewServer()
    
    // 🔥 注册服务
    pb.RegisterUserServiceServer(s, &userServer{})
    
    fmt.Println("gRPC服务器启动在 :50051")
    
    // 🔥 启动服务器
    if err := s.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
}
```

### 3.2 实现客户端

```go
// client/main.go
package main

import (
    "context"
    "fmt"
    "log"
    "time"
    
    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
    pb "your-module/proto"
)

func main() {
    // 🔥 建立连接
    conn, err := grpc.Dial("localhost:50051", 
        grpc.WithTransportCredentials(insecure.NewCredentials()))
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()
    
    // 🔥 创建客户端
    client := pb.NewUserServiceClient(conn)
    
    // 🔥 调用RPC方法
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    defer cancel()
    
    resp, err := client.GetUser(ctx, &pb.GetUserRequest{Id: 1})
    if err != nil {
        log.Fatalf("could not get user: %v", err)
    }
    
    fmt.Printf("User: %+v\n", resp.User)
}
```

---

## 4. 流式RPC

### 4.1 服务端流式RPC

```go
// 🔥 服务端实现
func (s *userServer) ListUsers(req *pb.ListUsersRequest, stream pb.UserService_ListUsersServer) error {
    // 模拟分页查询
    users := []*pb.User{
        {Id: 1, Name: "张三", Email: "zhangsan@example.com", Age: 25},
        {Id: 2, Name: "李四", Email: "lisi@example.com", Age: 30},
        {Id: 3, Name: "王五", Email: "wangwu@example.com", Age: 28},
    }
    
    // 🔥 流式发送数据
    for _, user := range users {
        if err := stream.Send(user); err != nil {
            return err
        }
        time.Sleep(time.Second)  // 模拟延迟
    }
    
    return nil
}

// 🔥 客户端调用
func listUsers(client pb.UserServiceClient) {
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    stream, err := client.ListUsers(ctx, &pb.ListUsersRequest{
        Page:     1,
        PageSize: 10,
    })
    if err != nil {
        log.Fatalf("could not list users: %v", err)
    }
    
    // 🔥 接收流式数据
    for {
        user, err := stream.Recv()
        if err == io.EOF {
            break
        }
        if err != nil {
            log.Fatalf("error receiving: %v", err)
        }
        fmt.Printf("User: %+v\n", user)
    }
}
```

### 4.2 客户端流式RPC

```go
// 🔥 服务端实现
func (s *userServer) CreateUsers(stream pb.UserService_CreateUsersServer) error {
    count := 0
    
    // 🔥 接收流式数据
    for {
        user, err := stream.Recv()
        if err == io.EOF {
            // 客户端发送完毕
            return stream.SendAndClose(&pb.CreateUsersResponse{
                Count: int32(count),
            })
        }
        if err != nil {
            return err
        }
        
        // 处理用户数据
        fmt.Printf("Received user: %+v\n", user)
        count++
    }
}

// 🔥 客户端调用
func createUsers(client pb.UserServiceClient) {
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    stream, err := client.CreateUsers(ctx)
    if err != nil {
        log.Fatalf("could not create users: %v", err)
    }
    
    users := []*pb.User{
        {Name: "用户1", Email: "user1@example.com", Age: 20},
        {Name: "用户2", Email: "user2@example.com", Age: 25},
        {Name: "用户3", Email: "user3@example.com", Age: 30},
    }
    
    // 🔥 流式发送数据
    for _, user := range users {
        if err := stream.Send(user); err != nil {
            log.Fatalf("error sending: %v", err)
        }
        time.Sleep(time.Second)
    }
    
    // 🔥 关闭流并接收响应
    resp, err := stream.CloseAndRecv()
    if err != nil {
        log.Fatalf("error closing: %v", err)
    }
    
    fmt.Printf("Created %d users\n", resp.Count)
}
```

### 4.3 双向流式RPC

```go
// 🔥 服务端实现
func (s *userServer) Chat(stream pb.UserService_ChatServer) error {
    for {
        // 🔥 接收消息
        msg, err := stream.Recv()
        if err == io.EOF {
            return nil
        }
        if err != nil {
            return err
        }
        
        fmt.Printf("Received: %s from %s\n", msg.Message, msg.User)
        
        // 🔥 发送响应
        response := &pb.ChatMessage{
            User:    "Server",
            Message: fmt.Sprintf("Echo: %s", msg.Message),
        }
        
        if err := stream.Send(response); err != nil {
            return err
        }
    }
}

// 🔥 客户端调用
func chat(client pb.UserServiceClient) {
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    stream, err := client.Chat(ctx)
    if err != nil {
        log.Fatalf("could not chat: %v", err)
    }
    
    // 🔥 启动接收Goroutine
    go func() {
        for {
            msg, err := stream.Recv()
            if err == io.EOF {
                return
            }
            if err != nil {
                log.Fatalf("error receiving: %v", err)
            }
            fmt.Printf("Received: %s from %s\n", msg.Message, msg.User)
        }
    }()
    
    // 🔥 发送消息
    messages := []string{"Hello", "How are you?", "Goodbye"}
    for _, text := range messages {
        msg := &pb.ChatMessage{
            User:    "Client",
            Message: text,
        }
        if err := stream.Send(msg); err != nil {
            log.Fatalf("error sending: %v", err)
        }
        time.Sleep(time.Second)
    }
    
    stream.CloseSend()
    time.Sleep(2 * time.Second)
}
```

---

## 5. 拦截器

### 5.1 服务端拦截器

```go
package main

import (
    "context"
    "log"
    "time"
    
    "google.golang.org/grpc"
)

// 🔥 一元拦截器
func unaryInterceptor(
    ctx context.Context,
    req interface{},
    info *grpc.UnaryServerInfo,
    handler grpc.UnaryHandler,
) (interface{}, error) {
    start := time.Now()
    
    // 前置处理
    log.Printf("Method: %s, Request: %v", info.FullMethod, req)
    
    // 调用实际的RPC方法
    resp, err := handler(ctx, req)
    
    // 后置处理
    log.Printf("Method: %s, Duration: %v", info.FullMethod, time.Since(start))
    
    return resp, err
}

// 🔥 流式拦截器
func streamInterceptor(
    srv interface{},
    ss grpc.ServerStream,
    info *grpc.StreamServerInfo,
    handler grpc.StreamHandler,
) error {
    start := time.Now()
    
    log.Printf("Stream Method: %s", info.FullMethod)
    
    err := handler(srv, ss)
    
    log.Printf("Stream Method: %s, Duration: %v", info.FullMethod, time.Since(start))
    
    return err
}

func main() {
    // 🔥 注册拦截器
    s := grpc.NewServer(
        grpc.UnaryInterceptor(unaryInterceptor),
        grpc.StreamInterceptor(streamInterceptor),
    )
    
    // ... 注册服务
}
```

### 5.2 客户端拦截器

```go
package main

import (
    "context"
    "log"
    "time"
    
    "google.golang.org/grpc"
)

// 🔥 一元拦截器
func clientUnaryInterceptor(
    ctx context.Context,
    method string,
    req, reply interface{},
    cc *grpc.ClientConn,
    invoker grpc.UnaryInvoker,
    opts ...grpc.CallOption,
) error {
    start := time.Now()
    
    log.Printf("Calling: %s", method)
    
    err := invoker(ctx, method, req, reply, cc, opts...)
    
    log.Printf("Called: %s, Duration: %v", method, time.Since(start))
    
    return err
}

// 🔥 流式拦截器
func clientStreamInterceptor(
    ctx context.Context,
    desc *grpc.StreamDesc,
    cc *grpc.ClientConn,
    method string,
    streamer grpc.Streamer,
    opts ...grpc.CallOption,
) (grpc.ClientStream, error) {
    log.Printf("Stream Calling: %s", method)
    
    return streamer(ctx, desc, cc, method, opts...)
}

func main() {
    // 🔥 注册拦截器
    conn, err := grpc.Dial("localhost:50051",
        grpc.WithTransportCredentials(insecure.NewCredentials()),
        grpc.WithUnaryInterceptor(clientUnaryInterceptor),
        grpc.WithStreamInterceptor(clientStreamInterceptor),
    )
    
    // ... 使用连接
}
```

---

## 6. 错误处理

### 6.1 返回错误

```go
package main

import (
    "context"
    
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
    pb "your-module/proto"
)

func (s *userServer) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.GetUserResponse, error) {
    if req.Id <= 0 {
        // 🔥 返回错误
        return nil, status.Errorf(codes.InvalidArgument, "invalid user id: %d", req.Id)
    }
    
    // 查询用户
    user, err := s.db.GetUser(req.Id)
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, status.Errorf(codes.NotFound, "user not found: %d", req.Id)
        }
        return nil, status.Errorf(codes.Internal, "database error: %v", err)
    }
    
    return &pb.GetUserResponse{User: user}, nil
}
```

### 6.2 处理错误

```go
package main

import (
    "context"
    "log"
    
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
    pb "your-module/proto"
)

func getUser(client pb.UserServiceClient, id int64) {
    resp, err := client.GetUser(context.Background(), &pb.GetUserRequest{Id: id})
    if err != nil {
        // 🔥 解析错误
        st, ok := status.FromError(err)
        if ok {
            switch st.Code() {
            case codes.InvalidArgument:
                log.Printf("Invalid argument: %s", st.Message())
            case codes.NotFound:
                log.Printf("User not found: %s", st.Message())
            case codes.Internal:
                log.Printf("Internal error: %s", st.Message())
            default:
                log.Printf("Unknown error: %s", st.Message())
            }
        } else {
            log.Printf("Error: %v", err)
        }
        return
    }
    
    log.Printf("User: %+v", resp.User)
}
```

---

## 7. 高级特性

### 7.1 超时控制

```go
package main

import (
    "context"
    "time"
    
    pb "your-module/proto"
)

func getUserWithTimeout(client pb.UserServiceClient, id int64) {
    // 🔥 设置超时
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    resp, err := client.GetUser(ctx, &pb.GetUserRequest{Id: id})
    if err != nil {
        log.Printf("Error: %v", err)
        return
    }
    
    log.Printf("User: %+v", resp.User)
}
```

### 7.2 元数据传递

```go
package main

import (
    "context"
    
    "google.golang.org/grpc/metadata"
    pb "your-module/proto"
)

// 🔥 客户端发送元数据
func getUserWithMetadata(client pb.UserServiceClient, id int64) {
    md := metadata.Pairs(
        "authorization", "Bearer token123",
        "request-id", "req-001",
    )
    ctx := metadata.NewOutgoingContext(context.Background(), md)
    
    resp, err := client.GetUser(ctx, &pb.GetUserRequest{Id: id})
    // ...
}

// 🔥 服务端接收元数据
func (s *userServer) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.GetUserResponse, error) {
    md, ok := metadata.FromIncomingContext(ctx)
    if ok {
        auth := md.Get("authorization")
        requestID := md.Get("request-id")
        log.Printf("Auth: %v, RequestID: %v", auth, requestID)
    }
    
    // ...
}
```

### 7.3 负载均衡

```go
package main

import (
    "google.golang.org/grpc"
    "google.golang.org/grpc/balancer/roundrobin"
    "google.golang.org/grpc/credentials/insecure"
)

func main() {
    // 🔥 使用轮询负载均衡
    conn, err := grpc.Dial(
        "dns:///localhost:50051,localhost:50052,localhost:50053",
        grpc.WithTransportCredentials(insecure.NewCredentials()),
        grpc.WithDefaultServiceConfig(`{"loadBalancingPolicy":"round_robin"}`),
    )
    
    // ... 使用连接
}
```

---

## 8. 最佳实践

### 8.1 项目结构

```
project/
├── proto/
│   ├── user.proto
│   └── user.pb.go
├── server/
│   ├── main.go
│   └── handler.go
├── client/
│   └── main.go
└── go.mod
```

### 8.2 错误处理规范

```go
// ✅ 使用标准错误码
codes.OK
codes.Canceled
codes.Unknown
codes.InvalidArgument
codes.DeadlineExceeded
codes.NotFound
codes.AlreadyExists
codes.PermissionDenied
codes.ResourceExhausted
codes.FailedPrecondition
codes.Aborted
codes.OutOfRange
codes.Unimplemented
codes.Internal
codes.Unavailable
codes.DataLoss
codes.Unauthenticated
```

### 8.3 性能优化

```go
// ✅ 使用连接池
conn, err := grpc.Dial(
    "localhost:50051",
    grpc.WithTransportCredentials(insecure.NewCredentials()),
    grpc.WithDefaultCallOptions(
        grpc.MaxCallRecvMsgSize(10*1024*1024),  // 10MB
        grpc.MaxCallSendMsgSize(10*1024*1024),  // 10MB
    ),
)

// ✅ 使用流式RPC处理大量数据
// ✅ 合理设置超时时间
// ✅ 使用拦截器记录日志和监控
```

---

## 📝 学习检查清单

- [ ] 理解gRPC的工作原理
- [ ] 掌握Protocol Buffers的使用
- [ ] 能够开发简单RPC服务
- [ ] 掌握四种流式RPC
- [ ] 理解拦截器的使用
- [ ] 掌握错误处理
- [ ] 理解超时控制和元数据
- [ ] 掌握gRPC的最佳实践

---

## 🔗 相关资源

- [gRPC官方文档](https://grpc.io/docs/)
- [Protocol Buffers](https://protobuf.dev/)
- [gRPC-Go](https://github.com/grpc/grpc-go)
- [gRPC最佳实践](https://grpc.io/docs/guides/performance/)

---

@author erik.zhou
