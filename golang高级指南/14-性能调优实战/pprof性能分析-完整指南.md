# pprof性能分析 - 完整指南

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **Go版本**：1.21+
- **pprof工具**：runtime/pprof、net/http/pprof
- **可视化工具**：go tool pprof

### 学习难度
- **难度等级**：⭐⭐⭐⭐ (较难)
- **预计学习时间**：15-20小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Go语言基础
- 并发编程
- HTTP服务开发
- 性能分析基础

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 掌握pprof的使用方法
- [ ] 能够进行CPU性能分析
- [ ] 能够进行内存性能分析
- [ ] 能够分析Goroutine泄漏
- [ ] 能够定位性能瓶颈

## 📖 目录

1. [pprof简介](#1-pprof简介)
2. [CPU性能分析](#2-cpu性能分析)
3. [内存性能分析](#3-内存性能分析)
4. [Goroutine分析](#4-goroutine分析)
5. [阻塞分析](#5-阻塞分析)
6. [互斥锁分析](#6-互斥锁分析)
7. [实战案例](#7-实战案例)
8. [最佳实践](#8-最佳实践)

---

## 1. pprof简介

### 1.1 什么是pprof

pprof是Go语言内置的性能分析工具，可以分析CPU、内存、Goroutine等性能指标。

**核心特点**：
- 🔥 **内置支持**：无需安装第三方工具
- 🔥 **多维度分析**：CPU、内存、Goroutine、阻塞、锁
- 🔥 **可视化**：支持火焰图、调用图等
- 🔥 **实时分析**：支持在线分析运行中的程序

### 1.2 pprof的类型

```go
package main

import (
    "fmt"
    "net/http"
    _ "net/http/pprof"
)

func main() {
    // 🔥 启动pprof HTTP服务
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    
    fmt.Println("pprof服务启动: http://localhost:6060/debug/pprof/")
    fmt.Println("可用的profile类型:")
    fmt.Println("- /debug/pprof/profile     CPU性能分析")
    fmt.Println("- /debug/pprof/heap        堆内存分析")
    fmt.Println("- /debug/pprof/goroutine   Goroutine分析")
    fmt.Println("- /debug/pprof/block       阻塞分析")
    fmt.Println("- /debug/pprof/mutex       互斥锁分析")
    fmt.Println("- /debug/pprof/threadcreate 线程创建分析")
    fmt.Println("- /debug/pprof/allocs      内存分配分析")
    
    select {}
}
```

---

## 2. CPU性能分析

### 2.1 采集CPU Profile

```go
package main

import (
    "fmt"
    "os"
    "runtime/pprof"
    "time"
)

func main() {
    // 🔥 方法1：使用runtime/pprof
    f, err := os.Create("cpu.prof")
    if err != nil {
        panic(err)
    }
    defer f.Close()
    
    // 开始CPU性能分析
    pprof.StartCPUProfile(f)
    defer pprof.StopCPUProfile()
    
    // 执行需要分析的代码
    heavyComputation()
    
    fmt.Println("CPU profile已保存到 cpu.prof")
}

func heavyComputation() {
    // 模拟CPU密集型计算
    for i := 0; i < 1000000; i++ {
        _ = fibonacci(30)
    }
}

func fibonacci(n int) int {
    if n <= 1 {
        return n
    }
    return fibonacci(n-1) + fibonacci(n-2)
}

// 分析方法：
// go tool pprof cpu.prof
// (pprof) top        # 查看CPU占用最多的函数
// (pprof) list main  # 查看main包的详细信息
// (pprof) web        # 生成调用图（需要安装graphviz）
```

### 2.2 HTTP方式采集

```go
package main

import (
    "fmt"
    "net/http"
    _ "net/http/pprof"
    "time"
)

func main() {
    // 🔥 方法2：使用HTTP接口
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    
    fmt.Println("pprof服务启动: http://localhost:6060/debug/pprof/")
    fmt.Println("采集CPU profile:")
    fmt.Println("curl http://localhost:6060/debug/pprof/profile?seconds=30 > cpu.prof")
    fmt.Println("分析profile:")
    fmt.Println("go tool pprof cpu.prof")
    
    // 模拟业务逻辑
    for {
        heavyComputation()
        time.Sleep(time.Second)
    }
}

func heavyComputation() {
    for i := 0; i < 100000; i++ {
        _ = fibonacci(25)
    }
}

func fibonacci(n int) int {
    if n <= 1 {
        return n
    }
    return fibonacci(n-1) + fibonacci(n-2)
}
```

### 2.3 分析CPU Profile

```bash
# 🔥 采集30秒的CPU profile
curl http://localhost:6060/debug/pprof/profile?seconds=30 > cpu.prof

# 🔥 交互式分析
go tool pprof cpu.prof

# 常用命令：
# top        - 显示CPU占用最多的函数
# top10      - 显示前10个函数
# list func  - 显示函数的详细代码
# web        - 生成调用图（需要graphviz）
# pdf        - 生成PDF报告
# png        - 生成PNG图片

# 🔥 直接分析在线服务
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# 🔥 生成火焰图
go tool pprof -http=:8080 cpu.prof
```

---

## 3. 内存性能分析

### 3.1 采集Heap Profile

```go
package main

import (
    "fmt"
    "os"
    "runtime"
    "runtime/pprof"
)

func main() {
    // 🔥 采集堆内存profile
    
    // 执行需要分析的代码
    memoryIntensive()
    
    // 手动触发GC
    runtime.GC()
    
    // 保存heap profile
    f, err := os.Create("heap.prof")
    if err != nil {
        panic(err)
    }
    defer f.Close()
    
    pprof.WriteHeapProfile(f)
    fmt.Println("Heap profile已保存到 heap.prof")
}

func memoryIntensive() {
    // 模拟内存密集型操作
    data := make([][]byte, 1000)
    for i := range data {
        data[i] = make([]byte, 1024*1024) // 1MB
    }
    
    // 保持引用，防止被GC
    _ = data
}

// 分析方法：
// go tool pprof heap.prof
// (pprof) top        # 查看内存占用最多的函数
// (pprof) list main  # 查看详细信息
```

### 3.2 内存分配分析

```go
package main

import (
    "fmt"
    "net/http"
    _ "net/http/pprof"
    "time"
)

func main() {
    // 🔥 启动pprof服务
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    
    fmt.Println("内存分析:")
    fmt.Println("- 堆内存: http://localhost:6060/debug/pprof/heap")
    fmt.Println("- 内存分配: http://localhost:6060/debug/pprof/allocs")
    fmt.Println("- 采集命令: curl http://localhost:6060/debug/pprof/heap > heap.prof")
    
    // 模拟内存分配
    ticker := time.NewTicker(time.Second)
    for range ticker.C {
        allocateMemory()
    }
}

func allocateMemory() {
    // 频繁分配内存
    data := make([][]byte, 100)
    for i := range data {
        data[i] = make([]byte, 1024*1024)
    }
}

// 分析内存分配：
// go tool pprof http://localhost:6060/debug/pprof/allocs
// (pprof) top        # 查看分配最多的函数
// (pprof) list func  # 查看函数详情
```

### 3.3 内存泄漏检测

```go
package main

import (
    "fmt"
    "net/http"
    _ "net/http/pprof"
    "runtime"
    "time"
)

var leakedData [][]byte

func main() {
    // 🔥 内存泄漏检测
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    
    fmt.Println("内存泄漏检测:")
    fmt.Println("1. 采集初始heap: curl http://localhost:6060/debug/pprof/heap > heap1.prof")
    fmt.Println("2. 等待一段时间")
    fmt.Println("3. 采集第二次heap: curl http://localhost:6060/debug/pprof/heap > heap2.prof")
    fmt.Println("4. 对比分析: go tool pprof -base heap1.prof heap2.prof")
    
    // 模拟内存泄漏
    go memoryLeak()
    
    // 定期打印内存使用
    ticker := time.NewTicker(5 * time.Second)
    for range ticker.C {
        var m runtime.MemStats
        runtime.ReadMemStats(&m)
        fmt.Printf("堆内存: %d MB, Goroutine: %d\n", 
            m.HeapAlloc/1024/1024, runtime.NumGoroutine())
    }
}

func memoryLeak() {
    ticker := time.NewTicker(time.Second)
    for range ticker.C {
        // 持续分配内存但不释放
        leakedData = append(leakedData, make([]byte, 1024*1024))
    }
}
```

---

## 4. Goroutine分析

### 4.1 Goroutine泄漏检测

```go
package main

import (
    "fmt"
    "net/http"
    _ "net/http/pprof"
    "runtime"
    "time"
)

func main() {
    // 🔥 Goroutine分析
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    
    fmt.Println("Goroutine分析:")
    fmt.Println("- 查看Goroutine: http://localhost:6060/debug/pprof/goroutine")
    fmt.Println("- 采集命令: curl http://localhost:6060/debug/pprof/goroutine > goroutine.prof")
    fmt.Println("- 分析命令: go tool pprof goroutine.prof")
    
    // 模拟Goroutine泄漏
    go goroutineLeak()
    
    // 定期打印Goroutine数量
    ticker := time.NewTicker(5 * time.Second)
    for range ticker.C {
        fmt.Printf("Goroutine数量: %d\n", runtime.NumGoroutine())
    }
}

func goroutineLeak() {
    ticker := time.NewTicker(time.Second)
    for range ticker.C {
        // 创建永远阻塞的Goroutine
        ch := make(chan int)
        go func() {
            <-ch // 永远阻塞
        }()
    }
}

// 分析Goroutine：
// go tool pprof http://localhost:6060/debug/pprof/goroutine
// (pprof) top        # 查看Goroutine最多的函数
// (pprof) traces     # 查看Goroutine的调用栈
```

---

## 5. 阻塞分析

### 5.1 阻塞性能分析

```go
package main

import (
    "fmt"
    "net/http"
    _ "net/http/pprof"
    "runtime"
    "time"
)

func main() {
    // 🔥 启用阻塞分析
    runtime.SetBlockProfileRate(1)
    
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    
    fmt.Println("阻塞分析:")
    fmt.Println("- 查看阻塞: http://localhost:6060/debug/pprof/block")
    fmt.Println("- 采集命令: curl http://localhost:6060/debug/pprof/block > block.prof")
    
    // 模拟阻塞
    go blockingOperation()
    
    select {}
}

func blockingOperation() {
    ch := make(chan int)
    
    // 创建多个Goroutine竞争channel
    for i := 0; i < 10; i++ {
        go func() {
            for {
                ch <- 1
                time.Sleep(100 * time.Millisecond)
            }
        }()
    }
    
    // 慢速消费
    for range ch {
        time.Sleep(time.Second)
    }
}
```

---

## 6. 互斥锁分析

### 6.1 锁竞争分析

```go
package main

import (
    "fmt"
    "net/http"
    _ "net/http/pprof"
    "runtime"
    "sync"
    "time"
)

func main() {
    // 🔥 启用互斥锁分析
    runtime.SetMutexProfileFraction(1)
    
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    
    fmt.Println("互斥锁分析:")
    fmt.Println("- 查看锁竞争: http://localhost:6060/debug/pprof/mutex")
    fmt.Println("- 采集命令: curl http://localhost:6060/debug/pprof/mutex > mutex.prof")
    
    // 模拟锁竞争
    go lockContention()
    
    select {}
}

func lockContention() {
    var mu sync.Mutex
    counter := 0
    
    // 创建多个Goroutine竞争锁
    for i := 0; i < 100; i++ {
        go func() {
            for {
                mu.Lock()
                counter++
                time.Sleep(10 * time.Millisecond) // 持有锁的时间
                mu.Unlock()
            }
        }()
    }
}
```

---

## 7. 实战案例

### 7.1 优化CPU密集型程序

```go
package main

import (
    "fmt"
    "time"
)

// ❌ 优化前：使用递归计算斐波那契数列
func fibonacciSlow(n int) int {
    if n <= 1 {
        return n
    }
    return fibonacciSlow(n-1) + fibonacciSlow(n-2)
}

// ✅ 优化后：使用动态规划
func fibonacciFast(n int) int {
    if n <= 1 {
        return n
    }
    
    a, b := 0, 1
    for i := 2; i <= n; i++ {
        a, b = b, a+b
    }
    return b
}

func main() {
    // 测试优化前
    start := time.Now()
    result := fibonacciSlow(40)
    fmt.Printf("优化前: 结果=%d, 耗时=%v\n", result, time.Since(start))
    
    // 测试优化后
    start = time.Now()
    result = fibonacciFast(40)
    fmt.Printf("优化后: 结果=%d, 耗时=%v\n", result, time.Since(start))
}

// 性能对比：
// 优化前: 结果=102334155, 耗时=1.2s
// 优化后: 结果=102334155, 耗时=0.000001s
```

---

## 8. 最佳实践

### 8.1 性能分析流程

```go
package main

// 🔥 性能分析最佳实践

// 1. 确定性能目标
// - 响应时间目标
// - 吞吐量目标
// - 资源使用目标

// 2. 建立性能基准
// - 使用benchmark测试
// - 记录初始性能指标

// 3. 采集性能数据
// - CPU profile
// - Memory profile
// - Goroutine profile

// 4. 分析性能瓶颈
// - 使用pprof分析
// - 生成火焰图
// - 定位热点函数

// 5. 优化代码
// - 算法优化
// - 数据结构优化
// - 并发优化

// 6. 验证优化效果
// - 重新测试
// - 对比性能指标
// - 确认达到目标

// 7. 持续监控
// - 生产环境监控
// - 定期性能分析
// - 及时发现问题
```

---

## 📝 学习检查清单

- [ ] 掌握pprof的基本使用
- [ ] 能够进行CPU性能分析
- [ ] 能够进行内存性能分析
- [ ] 能够检测Goroutine泄漏
- [ ] 能够分析阻塞和锁竞争
- [ ] 能够定位性能瓶颈
- [ ] 能够进行性能优化

---

## 🔗 相关资源

- [pprof文档](https://pkg.go.dev/runtime/pprof)
- [pprof使用指南](https://go.dev/blog/pprof)
- [性能分析工具](https://github.com/google/pprof)
- [火焰图工具](https://github.com/uber/go-torch)

---

@author erik.zhou
