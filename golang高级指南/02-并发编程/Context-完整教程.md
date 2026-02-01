# Context - 完整教程

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **Go版本**：1.21+
- **Context包**：context标准库

### 学习难度
- **难度等级**：⭐⭐⭐⭐ (较难)
- **预计学习时间**：8-12小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Go语言基础
- Goroutine和Channel
- 并发编程基础

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解Context的作用和原理
- [ ] 掌握Context的创建和使用
- [ ] 理解Context的传递机制
- [ ] 掌握超时控制
- [ ] 掌握取消信号传播
- [ ] 理解Context的最佳实践

## 📖 目录

1. [Context简介](#1-context简介)
2. [Context创建](#2-context创建)
3. [Context传递](#3-context传递)
4. [超时控制](#4-超时控制)
5. [取消信号](#5-取消信号)
6. [Context值传递](#6-context值传递)
7. [实战应用](#7-实战应用)
8. [最佳实践](#8-最佳实践)

---

## 1. Context简介

### 1.1 什么是Context

Context是Go语言中用于在Goroutine之间传递截止时间、取消信号和请求范围值的标准方式。

**核心特点**：
- 🔥 **超时控制**：设置操作的截止时间
- 🔥 **取消信号**：在Goroutine树中传播取消信号
- 🔥 **值传递**：在请求范围内传递值
- 🔥 **线程安全**：可以被多个Goroutine同时使用

**使用场景**：
- HTTP请求处理
- 数据库查询
- RPC调用
- 长时间运行的任务

---

## 2. Context创建

### 2.1 Background和TODO

```go
package main

import (
    "context"
    "fmt"
)

func main() {
    // 🔥 Background：根Context，通常用于main函数、初始化和测试
    ctx1 := context.Background()
    fmt.Printf("Background: %v\n", ctx1)
    
    // 🔥 TODO：当不确定使用哪个Context时使用
    ctx2 := context.TODO()
    fmt.Printf("TODO: %v\n", ctx2)
}
```

### 2.2 WithCancel

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    // 🔥 WithCancel：创建可取消的Context
    ctx, cancel := context.WithCancel(context.Background())
    
    go func() {
        for {
            select {
            case <-ctx.Done():
                fmt.Println("Goroutine退出")
                return
            default:
                fmt.Println("工作中...")
                time.Sleep(500 * time.Millisecond)
            }
        }
    }()
    
    time.Sleep(2 * time.Second)
    cancel()  // 取消Context
    time.Sleep(time.Second)
}
```

### 2.3 WithTimeout

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    // 🔥 WithTimeout：创建带超时的Context
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    go func() {
        select {
        case <-time.After(3 * time.Second):
            fmt.Println("任务完成")
        case <-ctx.Done():
            fmt.Println("超时:", ctx.Err())
        }
    }()
    
    time.Sleep(3 * time.Second)
}
```

### 2.4 WithDeadline

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    // 🔥 WithDeadline：创建带截止时间的Context
    deadline := time.Now().Add(2 * time.Second)
    ctx, cancel := context.WithDeadline(context.Background(), deadline)
    defer cancel()
    
    go func() {
        select {
        case <-time.After(3 * time.Second):
            fmt.Println("任务完成")
        case <-ctx.Done():
            fmt.Println("截止时间到:", ctx.Err())
        }
    }()
    
    time.Sleep(3 * time.Second)
}
```

---

## 3. Context传递

### 3.1 函数间传递

```go
package main

import (
    "context"
    "fmt"
    "time"
)

// 🔥 Context作为第一个参数
func doSomething(ctx context.Context, name string) {
    select {
    case <-time.After(2 * time.Second):
        fmt.Printf("%s 完成\n", name)
    case <-ctx.Done():
        fmt.Printf("%s 被取消: %v\n", name, ctx.Err())
    }
}

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel()
    
    doSomething(ctx, "任务1")
}
```

### 3.2 Goroutine间传递

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

func worker(ctx context.Context, id int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("Worker %d 退出\n", id)
            return
        default:
            fmt.Printf("Worker %d 工作中\n", id)
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    var wg sync.WaitGroup
    
    // 🔥 启动多个Worker
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go worker(ctx, i, &wg)
    }
    
    time.Sleep(2 * time.Second)
    cancel()  // 取消所有Worker
    wg.Wait()
}
```

---

## 4. 超时控制

### 4.1 HTTP请求超时

```go
package main

import (
    "context"
    "fmt"
    "io"
    "net/http"
    "time"
)

func fetchURL(ctx context.Context, url string) (string, error) {
    // 🔥 创建带Context的请求
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return "", err
    }
    
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()
    
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return "", err
    }
    
    return string(body), nil
}

func main() {
    // 🔥 设置5秒超时
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    body, err := fetchURL(ctx, "https://www.google.com")
    if err != nil {
        fmt.Println("错误:", err)
        return
    }
    
    fmt.Println("响应长度:", len(body))
}
```

### 4.2 数据库查询超时

```go
package main

import (
    "context"
    "database/sql"
    "fmt"
    "time"
)

func queryWithTimeout(db *sql.DB) error {
    // 🔥 设置查询超时
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    rows, err := db.QueryContext(ctx, "SELECT * FROM users WHERE age > ?", 18)
    if err != nil {
        return err
    }
    defer rows.Close()
    
    for rows.Next() {
        var id int
        var name string
        if err := rows.Scan(&id, &name); err != nil {
            return err
        }
        fmt.Printf("ID: %d, Name: %s\n", id, name)
    }
    
    return rows.Err()
}
```

### 4.3 RPC调用超时

```go
package main

import (
    "context"
    "fmt"
    "time"
)

type RPCClient struct{}

func (c *RPCClient) Call(ctx context.Context, method string, args interface{}) (interface{}, error) {
    // 🔥 模拟RPC调用
    select {
    case <-time.After(3 * time.Second):
        return "success", nil
    case <-ctx.Done():
        return nil, ctx.Err()
    }
}

func main() {
    client := &RPCClient{}
    
    // 🔥 设置RPC超时
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    result, err := client.Call(ctx, "GetUser", map[string]interface{}{"id": 1})
    if err != nil {
        fmt.Println("RPC调用失败:", err)
        return
    }
    
    fmt.Println("RPC结果:", result)
}
```

---

## 5. 取消信号

### 5.1 手动取消

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func longRunningTask(ctx context.Context) {
    for i := 0; i < 10; i++ {
        select {
        case <-ctx.Done():
            fmt.Println("任务被取消:", ctx.Err())
            return
        default:
            fmt.Printf("执行步骤 %d\n", i+1)
            time.Sleep(500 * time.Millisecond)
        }
    }
    fmt.Println("任务完成")
}

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    
    go longRunningTask(ctx)
    
    // 🔥 2秒后手动取消
    time.Sleep(2 * time.Second)
    cancel()
    
    time.Sleep(time.Second)
}
```

### 5.2 级联取消

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func level1(ctx context.Context) {
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()
    
    go level2(ctx)
    
    select {
    case <-time.After(3 * time.Second):
        fmt.Println("Level 1 完成")
    case <-ctx.Done():
        fmt.Println("Level 1 被取消")
    }
}

func level2(ctx context.Context) {
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()
    
    go level3(ctx)
    
    select {
    case <-time.After(3 * time.Second):
        fmt.Println("Level 2 完成")
    case <-ctx.Done():
        fmt.Println("Level 2 被取消")
    }
}

func level3(ctx context.Context) {
    select {
    case <-time.After(3 * time.Second):
        fmt.Println("Level 3 完成")
    case <-ctx.Done():
        fmt.Println("Level 3 被取消")
    }
}

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    
    go level1(ctx)
    
    // 🔥 1秒后取消，所有层级都会被取消
    time.Sleep(1 * time.Second)
    cancel()
    
    time.Sleep(time.Second)
}
```

---

## 6. Context值传递

### 6.1 WithValue

```go
package main

import (
    "context"
    "fmt"
)

type key string

const (
    userIDKey key = "userID"
    traceIDKey key = "traceID"
)

func processRequest(ctx context.Context) {
    // 🔥 从Context获取值
    userID := ctx.Value(userIDKey)
    traceID := ctx.Value(traceIDKey)
    
    fmt.Printf("UserID: %v, TraceID: %v\n", userID, traceID)
}

func main() {
    // 🔥 使用WithValue传递值
    ctx := context.Background()
    ctx = context.WithValue(ctx, userIDKey, 123)
    ctx = context.WithValue(ctx, traceIDKey, "abc-123")
    
    processRequest(ctx)
}
```

### 6.2 类型安全的值传递

```go
package main

import (
    "context"
    "fmt"
)

type contextKey int

const (
    userKey contextKey = iota
    requestIDKey
)

type User struct {
    ID   int
    Name string
}

// 🔥 类型安全的设置和获取
func WithUser(ctx context.Context, user *User) context.Context {
    return context.WithValue(ctx, userKey, user)
}

func GetUser(ctx context.Context) (*User, bool) {
    user, ok := ctx.Value(userKey).(*User)
    return user, ok
}

func main() {
    ctx := context.Background()
    user := &User{ID: 1, Name: "张三"}
    
    ctx = WithUser(ctx, user)
    
    if u, ok := GetUser(ctx); ok {
        fmt.Printf("User: %+v\n", u)
    }
}
```

### 6.3 Context值传递注意事项

```go
package main

import (
    "context"
    "fmt"
)

func main() {
    // ⚠️ 注意1：不要传递可选参数
    // Context值应该用于请求范围的数据，不是函数参数的替代品
    
    // ❌ 错误用法
    ctx := context.WithValue(context.Background(), "config", map[string]string{})
    
    // ✅ 正确用法：使用函数参数
    processWithConfig(ctx, map[string]string{})
    
    // ⚠️ 注意2：使用类型安全的key
    // ❌ 错误：使用字符串作为key
    ctx = context.WithValue(ctx, "userID", 123)
    
    // ✅ 正确：使用自定义类型
    type userIDKey struct{}
    ctx = context.WithValue(ctx, userIDKey{}, 123)
    
    // ⚠️ 注意3：不要存储大量数据
    // Context值应该是轻量级的
}

func processWithConfig(ctx context.Context, config map[string]string) {
    fmt.Println("处理配置")
}
```

---

## 7. 实战应用

### 7.1 HTTP服务器中间件

```go
package main

import (
    "context"
    "fmt"
    "net/http"
    "time"
)

type contextKey string

const requestIDKey contextKey = "requestID"

// 🔥 添加请求ID中间件
func requestIDMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        requestID := fmt.Sprintf("%d", time.Now().UnixNano())
        ctx := context.WithValue(r.Context(), requestIDKey, requestID)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// 🔥 超时中间件
func timeoutMiddleware(timeout time.Duration) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            ctx, cancel := context.WithTimeout(r.Context(), timeout)
            defer cancel()
            
            next.ServeHTTP(w, r.WithContext(ctx))
        })
    }
}

func handler(w http.ResponseWriter, r *http.Request) {
    requestID := r.Context().Value(requestIDKey)
    
    select {
    case <-time.After(2 * time.Second):
        fmt.Fprintf(w, "Request ID: %v\n", requestID)
    case <-r.Context().Done():
        http.Error(w, "Request timeout", http.StatusRequestTimeout)
    }
}

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", handler)
    
    // 应用中间件
    handler := requestIDMiddleware(timeoutMiddleware(3 * time.Second)(mux))
    
    http.ListenAndServe(":8080", handler)
}
```

### 7.2 并发任务管理

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

type Task struct {
    ID   int
    Name string
}

func processTask(ctx context.Context, task Task) error {
    select {
    case <-time.After(time.Duration(task.ID) * time.Second):
        fmt.Printf("任务 %s 完成\n", task.Name)
        return nil
    case <-ctx.Done():
        fmt.Printf("任务 %s 被取消\n", task.Name)
        return ctx.Err()
    }
}

func processTasks(ctx context.Context, tasks []Task) error {
    var wg sync.WaitGroup
    errChan := make(chan error, len(tasks))
    
    for _, task := range tasks {
        wg.Add(1)
        go func(t Task) {
            defer wg.Done()
            if err := processTask(ctx, t); err != nil {
                errChan <- err
            }
        }(task)
    }
    
    // 等待所有任务完成
    go func() {
        wg.Wait()
        close(errChan)
    }()
    
    // 收集错误
    for err := range errChan {
        if err != nil {
            return err
        }
    }
    
    return nil
}

func main() {
    tasks := []Task{
        {ID: 1, Name: "任务1"},
        {ID: 2, Name: "任务2"},
        {ID: 3, Name: "任务3"},
    }
    
    // 🔥 设置总超时时间
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    if err := processTasks(ctx, tasks); err != nil {
        fmt.Println("处理失败:", err)
    }
}
```

---

## 8. 最佳实践

### 8.1 Context使用规范

```go
// ✅ 规范1：Context作为第一个参数
func DoSomething(ctx context.Context, arg string) error {
    // ...
}

// ✅ 规范2：不要存储Context
type Server struct {
    // ❌ 不要这样做
    // ctx context.Context
}

// ✅ 规范3：不要传递nil Context
func process(ctx context.Context) {
    if ctx == nil {
        ctx = context.Background()
    }
    // ...
}

// ✅ 规范4：Context值仅用于请求范围的数据
// 不要用于传递可选参数

// ✅ 规范5：使用defer cancel()
func example() {
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    defer cancel()  // 确保资源释放
    // ...
}
```

### 8.2 错误处理

```go
package main

import (
    "context"
    "errors"
    "fmt"
)

func doWork(ctx context.Context) error {
    select {
    case <-ctx.Done():
        // 🔥 检查取消原因
        err := ctx.Err()
        if errors.Is(err, context.Canceled) {
            return fmt.Errorf("工作被取消: %w", err)
        }
        if errors.Is(err, context.DeadlineExceeded) {
            return fmt.Errorf("工作超时: %w", err)
        }
        return err
    default:
        // 执行工作
        return nil
    }
}
```

### 8.3 性能优化

```go
package main

import (
    "context"
    "time"
)

// ✅ 复用Context
func processRequests(ctx context.Context, requests []Request) {
    for _, req := range requests {
        // 为每个请求创建子Context
        reqCtx, cancel := context.WithTimeout(ctx, 5*time.Second)
        processRequest(reqCtx, req)
        cancel()  // 及时释放资源
    }
}

// ✅ 避免Context值嵌套过深
// ❌ 不好
ctx = context.WithValue(ctx, key1, val1)
ctx = context.WithValue(ctx, key2, val2)
ctx = context.WithValue(ctx, key3, val3)

// ✅ 好：使用结构体
type RequestData struct {
    Val1 string
    Val2 string
    Val3 string
}
ctx = context.WithValue(ctx, requestDataKey, &RequestData{})
```

---

## 📝 学习检查清单

- [ ] 理解Context的作用和原理
- [ ] 掌握Context的创建方法
- [ ] 理解Context的传递机制
- [ ] 掌握超时控制
- [ ] 掌握取消信号传播
- [ ] 理解Context值传递
- [ ] 掌握Context的最佳实践
- [ ] 能够在实际项目中应用Context

---

## 🔗 相关资源

- [Go Context包](https://pkg.go.dev/context)
- [Go Concurrency Patterns: Context](https://go.dev/blog/context)
- [Context最佳实践](https://go.dev/blog/context-and-structs)

---

@author erik.zhou
