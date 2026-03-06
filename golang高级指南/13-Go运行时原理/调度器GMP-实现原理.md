# 调度器GMP - 实现原理

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **Go版本**：1.21+
- **调度模型**：GMP模型
- **调度策略**：抢占式调度

### 学习难度
- **难度等级**：⭐⭐⭐⭐⭐ (非常难)
- **预计学习时间**：20-25小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Go语言基础
- 并发编程
- 操作系统原理
- 线程调度

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 深入理解GMP调度模型
- [ ] 掌握Goroutine调度原理
- [ ] 理解抢占式调度机制
- [ ] 能够优化并发性能
- [ ] 理解调度器演进历史

## 📖 目录

1. [调度器概述](#1-调度器概述)
2. [GMP模型详解](#2-gmp模型详解)
3. [调度流程](#3-调度流程)
4. [抢占式调度](#4-抢占式调度)
5. [系统调用处理](#5-系统调用处理)
6. [调度器优化](#6-调度器优化)
7. [源码分析](#7-源码分析)
8. [最佳实践](#8-最佳实践)

---

## 1. 调度器概述

### 1.1 为什么需要调度器

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    // 🔥 Go调度器的目标
    
    // 1. 高并发
    // - 支持百万级Goroutine
    // - 轻量级线程切换
    
    // 2. 低延迟
    // - 快速响应
    // - 公平调度
    
    // 3. 高效利用CPU
    // - 多核并行
    // - 负载均衡
    
    fmt.Printf("CPU核心数: %d\n", runtime.NumCPU())
    fmt.Printf("GOMAXPROCS: %d\n", runtime.GOMAXPROCS(0))
    fmt.Printf("Goroutine数: %d\n", runtime.NumGoroutine())
}
```

### 1.2 调度器演进历史

```go
package main

import "fmt"

func main() {
    // 🔥 Go调度器演进
    
    // Go 1.0: GM模型
    // - 全局队列
    // - 单一锁
    // - 性能瓶颈
    
    // Go 1.1: GMP模型
    // - 引入P（Processor）
    // - 本地队列
    // - 减少锁竞争
    
    // Go 1.2-1.13: 持续优化
    // - 抢占式调度
    // - 工作窃取
    // - 系统调用优化
    
    // Go 1.14+: 异步抢占
    // - 基于信号的抢占
    // - 解决长时间运行问题
    
    fmt.Println("调度器演进:")
    fmt.Println("GM -> GMP -> 抢占式 -> 异步抢占")
}
```

---

## 2. GMP模型详解

### 2.1 G（Goroutine）

```go
package main

// 🔥 G代表Goroutine
// 源码位置：runtime/runtime2.go

type g struct {
    // 栈信息
    stack       stack   // 栈的范围 [stack.lo, stack.hi)
    stackguard0 uintptr // 栈保护，用于检测栈溢出
    
    // 调度信息
    sched     gobuf   // 调度上下文
    goid      int64   // Goroutine ID
    gopc      uintptr // 创建该Goroutine的PC
    startpc   uintptr // Goroutine函数的PC
    
    // 状态信息
    atomicstatus uint32  // Goroutine状态
    preempt      bool    // 抢占标记
    
    // 关联的M和P
    m *m  // 当前运行的M
    lockedm muintptr  // 锁定的M
}

// Goroutine状态
const (
    _Gidle      = iota // 0: 刚分配，未初始化
    _Grunnable         // 1: 可运行，在运行队列中
    _Grunning          // 2: 正在运行
    _Gsyscall          // 3: 系统调用中
    _Gwaiting          // 4: 等待中（IO、Channel等）
    _Gdead             // 6: 已结束
    _Gcopystack        // 8: 栈复制中
    _Gpreempted        // 9: 被抢占
)

func main() {
    // G的特点：
    // - 初始栈2KB，可动态增长
    // - 保存执行上下文
    // - 轻量级，创建成本低
}
```

### 2.2 M（Machine）

```go
package main

// 🔥 M代表操作系统线程
// 源码位置：runtime/runtime2.go

type m struct {
    // 特殊的Goroutine
    g0      *g  // 用于执行调度代码的g
    gsignal *g  // 用于处理信号的g
    
    // 当前运行的G
    curg *g  // 当前正在运行的G
    
    // 关联的P
    p       puintptr  // 当前关联的P
    nextp   puintptr  // 下一个要关联的P
    oldp    puintptr  // 执行系统调用前的P
    
    // 线程信息
    id      int64     // 线程ID
    spinning bool     // 是否在寻找G
    blocked  bool     // 是否被阻塞
    
    // 锁定信息
    lockedg guintptr  // 锁定的G
}

func main() {
    // M的特点：
    // - 对应操作系统线程
    // - 数量可动态调整
    // - 执行G的代码
}
```

### 2.3 P（Processor）

```go
package main

// 🔥 P代表处理器（逻辑处理器）
// 源码位置：runtime/runtime2.go

type p struct {
    // P的ID
    id int32
    
    // P的状态
    status uint32  // P的状态
    
    // 关联的M
    m muintptr  // 当前关联的M
    
    // 本地运行队列
    runqhead uint32  // 队列头
    runqtail uint32  // 队列尾
    runq     [256]guintptr  // 本地队列，最多256个G
    
    // runnext是下一个要运行的G
    runnext guintptr
    
    // 空闲G列表
    gFree struct {
        gList
        n int32
    }
    
    // mcache
    mcache *mcache
}

// P的状态
const (
    _Pidle    = iota // 0: 空闲
    _Prunning        // 1: 运行中
    _Psyscall        // 2: 系统调用中
    _Pgcstop         // 3: GC停止
    _Pdead           // 4: 已死亡
)

func main() {
    // P的特点：
    // - 数量等于GOMAXPROCS
    // - 拥有本地运行队列
    // - 拥有mcache
    // - 减少锁竞争
}
```

### 2.4 GMP关系图

```go
package main

import "fmt"

func main() {
    // 🔥 GMP关系
    
    // G（Goroutine）
    // - 用户创建的协程
    // - 需要被调度执行
    
    // M（Machine）
    // - 操作系统线程
    // - 执行G的代码
    
    // P（Processor）
    // - 逻辑处理器
    // - 管理G的队列
    // - 关联M和G
    
    // 关系：
    // - M必须关联P才能执行G
    // - P拥有本地G队列
    // - G在P的队列中等待M执行
    
    fmt.Println("GMP关系:")
    fmt.Println("G -> P -> M")
    fmt.Println("G: 任务")
    fmt.Println("P: 调度器")
    fmt.Println("M: 执行者")
}
```

---

## 3. 调度流程

### 3.1 Goroutine创建

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    // 🔥 Goroutine创建流程
    
    // 1. 调用go关键字
    go func() {
        fmt.Println("新Goroutine")
    }()
    
    // 2. 编译器转换为newproc调用
    // newproc(siz, fn)
    
    // 3. 创建G结构
    // - 分配栈空间（初始2KB）
    // - 初始化G结构
    // - 设置G的状态为_Grunnable
    
    // 4. 将G加入P的本地队列
    // - 优先加入runnext
    // - 如果runnext已满，加入runq
    // - 如果runq已满，加入全局队列
    
    runtime.Gosched()  // 让出CPU
}
```

### 3.2 调度循环

```go
package main

// 🔥 调度循环（schedule函数）
// 源码位置：runtime/proc.go

func schedule() {
    // 调度循环，永不返回
    for {
        // 1. 检查是否需要GC
        if gcwaiting != 0 {
            gcstopm()
        }
        
        // 2. 获取下一个要运行的G
        gp := findrunnable()
        
        // 3. 执行G
        execute(gp)
    }
}

func findrunnable() *g {
    // 查找可运行的G
    
    // 1. 每61次调度，从全局队列获取G
    if schedtick%61 == 0 {
        gp := globrunqget(_p_, 1)
        if gp != nil {
            return gp
        }
    }
    
    // 2. 从P的本地队列获取G
    gp := runqget(_p_)
    if gp != nil {
        return gp
    }
    
    // 3. 从全局队列获取G
    gp = globrunqget(_p_, 0)
    if gp != nil {
        return gp
    }
    
    // 4. 从其他P窃取G（工作窃取）
    gp = stealWork()
    if gp != nil {
        return gp
    }
    
    // 5. 没有G可运行，进入休眠
    return nil
}
```

### 3.3 工作窃取

```go
package main

import "fmt"

func main() {
    // 🔥 工作窃取（Work Stealing）
    
    // 场景：某个P的本地队列为空
    
    // 步骤1：随机选择一个P
    // 步骤2：从该P的本地队列尾部窃取一半的G
    // 步骤3：将窃取的G加入自己的本地队列
    
    // 优点：
    // - 负载均衡
    // - 提高CPU利用率
    // - 减少全局队列竞争
    
    fmt.Println("工作窃取:")
    fmt.Println("1. 本地队列为空")
    fmt.Println("2. 从其他P窃取G")
    fmt.Println("3. 实现负载均衡")
}
```

---

## 4. 抢占式调度

### 4.1 协作式抢占

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    // 🔥 协作式抢占（Go 1.13及之前）
    
    // 抢占时机：
    // 1. 函数调用时检查栈
    // 2. Channel操作
    // 3. 系统调用
    // 4. 垃圾回收
    
    // 问题：长时间运行的循环无法被抢占
    go func() {
        for {
            // 没有函数调用，无法被抢占
        }
    }()
    
    runtime.Gosched()
}
```

### 4.2 异步抢占

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    // 🔥 异步抢占（Go 1.14+）
    
    // 基于信号的抢占
    // - 使用SIGURG信号
    // - 可以抢占任意代码
    // - 解决长时间运行问题
    
    go func() {
        start := time.Now()
        for time.Since(start) < time.Second {
            // 长时间运行的循环
            // Go 1.14+可以被抢占
        }
        fmt.Println("循环结束")
    }()
    
    time.Sleep(100 * time.Millisecond)
    fmt.Printf("Goroutine数: %d\n", runtime.NumGoroutine())
    
    time.Sleep(2 * time.Second)
}
```

---

## 5. 系统调用处理

### 5.1 阻塞系统调用

```go
package main

import (
    "fmt"
    "os"
    "time"
)

func main() {
    // 🔥 阻塞系统调用处理
    
    // 场景：文件IO、网络IO等
    
    // 步骤1：G进入系统调用
    // - G的状态变为_Gsyscall
    // - M和P解绑
    
    // 步骤2：P寻找新的M
    // - P可以关联其他M
    // - 继续执行其他G
    
    // 步骤3：系统调用返回
    // - G尝试重新获取P
    // - 如果获取失败，G加入全局队列
    
    go func() {
        // 阻塞系统调用
        file, _ := os.Open("test.txt")
        defer file.Close()
        
        buf := make([]byte, 1024)
        file.Read(buf)  // 阻塞IO
        
        fmt.Println("读取完成")
    }()
    
    time.Sleep(time.Second)
}
```

### 5.2 非阻塞系统调用

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    // 🔥 非阻塞系统调用处理
    
    // 场景：快速系统调用（如获取时间）
    
    // 步骤1：G进入系统调用
    // - M和P不解绑
    // - 快速返回
    
    // 步骤2：系统调用返回
    // - 继续执行G
    
    go func() {
        // 非阻塞系统调用
        _ = runtime.NumCPU()
        fmt.Println("快速系统调用")
    }()
    
    runtime.Gosched()
}
```

---

## 6. 调度器优化

### 6.1 调整GOMAXPROCS

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    // 🔥 调整GOMAXPROCS
    
    // 默认值：CPU核心数
    fmt.Printf("默认GOMAXPROCS: %d\n", runtime.GOMAXPROCS(0))
    
    // CPU密集型：设置为CPU核心数
    runtime.GOMAXPROCS(runtime.NumCPU())
    
    // IO密集型：可以设置为CPU核心数的2倍
    // runtime.GOMAXPROCS(runtime.NumCPU() * 2)
    
    // 测试不同GOMAXPROCS的性能
    testPerformance(1)
    testPerformance(2)
    testPerformance(4)
    testPerformance(8)
}

func testPerformance(procs int) {
    runtime.GOMAXPROCS(procs)
    
    start := time.Now()
    done := make(chan bool)
    
    for i := 0; i < 4; i++ {
        go func() {
            sum := 0
            for j := 0; j < 100000000; j++ {
                sum += j
            }
            done <- true
        }()
    }
    
    for i := 0; i < 4; i++ {
        <-done
    }
    
    fmt.Printf("GOMAXPROCS=%d, 耗时=%v\n", procs, time.Since(start))
}
```

### 6.2 减少Goroutine创建

```go
package main

import (
    "fmt"
    "sync"
)

// ❌ 频繁创建Goroutine
func badPattern() {
    for i := 0; i < 10000; i++ {
        go func(n int) {
            // 处理任务
            _ = n * 2
        }(i)
    }
}

// ✅ 使用Worker Pool
func goodPattern() {
    const numWorkers = 10
    jobs := make(chan int, 100)
    var wg sync.WaitGroup
    
    // 创建Worker Pool
    for w := 0; w < numWorkers; w++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for job := range jobs {
                // 处理任务
                _ = job * 2
            }
        }()
    }
    
    // 发送任务
    for i := 0; i < 10000; i++ {
        jobs <- i
    }
    close(jobs)
    
    wg.Wait()
}

func main() {
    fmt.Println("Goroutine优化:")
    fmt.Println("1. 使用Worker Pool")
    fmt.Println("2. 控制并发数量")
    fmt.Println("3. 复用Goroutine")
}
```

---

## 7. 源码分析

### 7.1 schedule函数

```go
package main

// 🔥 schedule函数是调度的核心
// 源码位置：runtime/proc.go

func schedule() {
    _g_ := getg()
    
top:
    // 1. 检查GC
    if sched.gcwaiting != 0 {
        gcstopm()
        goto top
    }
    
    // 2. 检查定时器
    checkTimers(_g_.m.p.ptr(), 0)
    
    // 3. 获取G
    var gp *g
    
    // 每61次从全局队列获取
    if _g_.m.p.ptr().schedtick%61 == 0 && sched.runqsize > 0 {
        lock(&sched.lock)
        gp = globrunqget(_g_.m.p.ptr(), 1)
        unlock(&sched.lock)
    }
    
    // 从本地队列获取
    if gp == nil {
        gp, inheritTime = runqget(_g_.m.p.ptr())
    }
    
    // 从全局队列或其他P窃取
    if gp == nil {
        gp, inheritTime = findrunnable()
    }
    
    // 4. 执行G
    execute(gp, inheritTime)
}
```

---

## 8. 最佳实践

### 8.1 调度器优化建议

```go
package main

// ✅ 最佳实践

// 1. 合理设置GOMAXPROCS
func setGOMAXPROCS() {
    // CPU密集型：CPU核心数
    // IO密集型：CPU核心数 * 2
    runtime.GOMAXPROCS(runtime.NumCPU())
}

// 2. 使用Worker Pool
func useWorkerPool() {
    // 控制并发数量
    // 复用Goroutine
}

// 3. 避免长时间运行
func avoidLongRunning() {
    // 定期调用runtime.Gosched()
    // 或使用Channel通信
}

// 4. 合理使用锁
func useLockProperly() {
    // 减少锁持有时间
    // 使用读写锁
    // 考虑使用原子操作
}
```

---

## 📝 学习检查清单

- [ ] 深入理解GMP模型
- [ ] 掌握调度流程
- [ ] 理解抢占式调度
- [ ] 掌握系统调用处理
- [ ] 能够优化调度性能
- [ ] 理解工作窃取机制
- [ ] 能够阅读调度器源码

---

## 🔗 相关资源

- [Go调度器设计](https://golang.org/s/go11sched)
- [Go调度器源码](https://github.com/golang/go/blob/master/src/runtime/proc.go)
- [调度器分析](https://golang.design/under-the-hood/zh-cn/part2runtime/ch06sched/)
- [GMP模型详解](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html)

---

@author erik.zhou
