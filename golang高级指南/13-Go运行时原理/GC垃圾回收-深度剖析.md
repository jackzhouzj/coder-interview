# GC垃圾回收 - 深度剖析

> @author erik.zhou

## 📚 技术概述

### 版本信息
- **Go版本**：1.21+
- **GC算法**：三色标记清除
- **GC模式**：并发标记清除

### 学习难度
- **难度等级**：⭐⭐⭐⭐⭐ (非常难)
- **预计学习时间**：20-25小时
- **重要程度**：⭐⭐⭐⭐⭐ (必学)

### 前置知识
- Go语言基础
- 并发编程
- 操作系统原理
- 数据结构

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 深入理解Go的GC机制
- [ ] 掌握三色标记算法
- [ ] 理解写屏障机制
- [ ] 能够进行GC调优
- [ ] 能够排查内存泄漏

## 📖 目录

1. [GC基础概念](#1-gc基础概念)
2. [三色标记算法](#2-三色标记算法)
3. [写屏障机制](#3-写屏障机制)
4. [GC触发时机](#4-gc触发时机)
5. [GC性能分析](#5-gc性能分析)
6. [GC调优技巧](#6-gc调优技巧)
7. [内存泄漏排查](#7-内存泄漏排查)
8. [最佳实践](#8-最佳实践)

---

## 1. GC基础概念

### 1.1 什么是GC

垃圾回收（Garbage Collection）是自动内存管理机制，负责回收不再使用的内存。

**Go GC特点**：
- 🔥 **并发标记**：GC与用户程序并发执行
- 🔥 **三色标记**：使用三色标记算法
- 🔥 **写屏障**：保证并发标记的正确性
- 🔥 **低延迟**：目标是STW时间<1ms


### 1.2 GC演进历史

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    // 🔥 Go GC演进历史
    // Go 1.0-1.3: 标记-清除，STW时间长
    // Go 1.4: 精确GC，减少STW时间
    // Go 1.5: 三色标记，并发GC，STW<10ms
    // Go 1.6-1.7: 优化写屏障，STW<5ms
    // Go 1.8-1.12: 持续优化，STW<1ms
    // Go 1.13+: 进一步优化，引入Scavenger
    
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    
    fmt.Printf("GC次数: %d\n", m.NumGC)
    fmt.Printf("GC暂停时间: %v\n", m.PauseNs[(m.NumGC+255)%256])
    fmt.Printf("堆内存: %d MB\n", m.HeapAlloc/1024/1024)
}
```

### 1.3 GC的目标

```go
// 🔥 Go GC的三个目标
// 1. 低延迟：STW时间尽可能短
// 2. 高吞吐：GC占用CPU时间尽可能少
// 3. 内存利用率：减少内存碎片

// ⚠️ 这三个目标是相互矛盾的，需要权衡
```

---

## 2. 三色标记算法

### 2.1 三色标记原理

```go
package main

import "fmt"

// 🔥 三色标记算法
// 白色：未被访问的对象（可能是垃圾）
// 灰色：已被访问但其引用的对象未被访问
// 黑色：已被访问且其引用的对象也已被访问

type Object struct {
    name  string
    refs  []*Object
    color string // white, gray, black
}

func triColorMarking() {
    // 初始状态：所有对象都是白色
    root := &Object{name: "root", color: "white"}
    obj1 := &Object{name: "obj1", color: "white"}
    obj2 := &Object{name: "obj2", color: "white"}
    obj3 := &Object{name: "obj3", color: "white"}
    
    root.refs = []*Object{obj1, obj2}
    obj1.refs = []*Object{obj3}
    
    // 第一步：将根对象标记为灰色
    root.color = "gray"
    graySet := []*Object{root}
    
    // 第二步：遍历灰色对象
    for len(graySet) > 0 {
        // 取出一个灰色对象
        current := graySet[0]
        graySet = graySet[1:]
        
        // 将其引用的白色对象标记为灰色
        for _, ref := range current.refs {
            if ref.color == "white" {
                ref.color = "gray"
                graySet = append(graySet, ref)
            }
        }
        
        // 将当前对象标记为黑色
        current.color = "black"
    }
    
    // 第三步：清除所有白色对象（垃圾）
    fmt.Println("标记完成，白色对象将被回收")
}
```

### 2.2 并发标记的问题

```go
package main

// ⚠️ 并发标记的两个问题

// 问题1：对象丢失
// 场景：GC标记时，用户程序修改了对象引用
// 黑色对象引用了白色对象，但GC已经扫描过黑色对象
// 导致白色对象被错误回收

// 问题2：浮动垃圾
// 场景：GC标记时，用户程序删除了对象引用
// 灰色对象的引用被删除，但GC还会继续标记
// 导致垃圾对象被保留到下一轮GC

// 🔥 解决方案：写屏障
```

---

## 3. 写屏障机制

### 3.1 什么是写屏障

```go
package main

// 🔥 写屏障（Write Barrier）
// 在对象引用修改时插入的一段代码
// 用于保证并发标记的正确性

// Go使用混合写屏障（Hybrid Write Barrier）
// 结合了Dijkstra插入屏障和Yuasa删除屏障

// 伪代码
func writeBarrier(slot *Object, ptr *Object) {
    // 1. 记录被覆盖的旧值（删除屏障）
    shade(slot)
    
    // 2. 记录新值（插入屏障）
    shade(ptr)
    
    // 3. 更新引用
    *slot = ptr
}

func shade(obj *Object) {
    if obj != nil && obj.color == "white" {
        obj.color = "gray"
        // 加入灰色队列
    }
}
```

### 3.2 写屏障的开销

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    // 🔥 写屏障的性能影响
    
    // 禁用GC
    runtime.SetGCPercent(-1)
    
    start := time.Now()
    testWithoutGC()
    fmt.Printf("无GC耗时: %v\n", time.Since(start))
    
    // 启用GC
    runtime.SetGCPercent(100)
    
    start = time.Now()
    testWithGC()
    fmt.Printf("有GC耗时: %v\n", time.Since(start))
}

func testWithoutGC() {
    for i := 0; i < 1000000; i++ {
        _ = make([]byte, 1024)
    }
}

func testWithGC() {
    for i := 0; i < 1000000; i++ {
        _ = make([]byte, 1024)
    }
}
```

---

## 4. GC触发时机

### 4.1 GC触发条件

```go
package main

import (
    "fmt"
    "runtime"
    "runtime/debug"
)

func main() {
    // 🔥 GC触发条件
    
    // 1. 内存分配达到阈值
    // 当前堆内存 >= 上次GC后堆内存 * GOGC/100
    // 默认GOGC=100，即堆内存翻倍时触发GC
    
    // 2. 定时触发
    // 2分钟未触发GC，强制触发
    
    // 3. 手动触发
    runtime.GC()
    
    // 查看GC配置
    gcPercent := debug.SetGCPercent(-1)
    debug.SetGCPercent(gcPercent)
    fmt.Printf("GOGC: %d\n", gcPercent)
    
    // 设置GC百分比
    debug.SetGCPercent(200) // 堆内存3倍时触发GC
}
```

### 4.2 GC的四个阶段

```go
package main

// 🔥 GC的四个阶段

// 阶段1：标记准备（Mark Setup）- STW
// - 开启写屏障
// - 启动标记Worker
// - 扫描栈上的根对象
// STW时间：10-30微秒

// 阶段2：并发标记（Marking）- 并发
// - 标记所有可达对象
// - 用户程序和GC并发执行
// - 写屏障保证正确性

// 阶段3：标记终止（Mark Termination）- STW
// - 关闭写屏障
// - 清理标记队列
// - 计算下次GC阈值
// STW时间：60-90微秒

// 阶段4：清除（Sweeping）- 并发
// - 回收白色对象
// - 与用户程序并发执行
```

---

## 5. GC性能分析

### 5.1 查看GC统计信息

```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func main() {
    // 🔥 查看GC统计信息
    
    // 方法1：runtime.MemStats
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    
    fmt.Printf("=== GC统计信息 ===\n")
    fmt.Printf("GC次数: %d\n", m.NumGC)
    fmt.Printf("GC总暂停时间: %v\n", time.Duration(m.PauseTotalNs))
    fmt.Printf("上次GC时间: %v\n", time.Unix(0, int64(m.LastGC)))
    fmt.Printf("堆分配: %d MB\n", m.HeapAlloc/1024/1024)
    fmt.Printf("堆使用: %d MB\n", m.HeapInuse/1024/1024)
    fmt.Printf("堆对象数: %d\n", m.HeapObjects)
    fmt.Printf("栈使用: %d MB\n", m.StackInuse/1024/1024)
    
    // 最近256次GC的暂停时间
    fmt.Printf("\n=== 最近GC暂停时间 ===\n")
    for i := 0; i < 10 && i < int(m.NumGC); i++ {
        idx := (m.NumGC - uint32(i) + 255) % 256
        fmt.Printf("GC #%d: %v\n", m.NumGC-uint32(i), time.Duration(m.PauseNs[idx]))
    }
}
```

### 5.2 使用GODEBUG追踪GC

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    // 🔥 使用GODEBUG=gctrace=1追踪GC
    // 运行: GODEBUG=gctrace=1 go run main.go
    
    // 输出格式：
    // gc # @#s #%: #+#+# ms clock, #+#/#/#+# ms cpu, #->#-># MB, # MB goal, # P
    
    // 示例输出：
    // gc 1 @0.001s 0%: 0.018+0.23+0.004 ms clock, 0.14+0.076/0.19/0.23+0.032 ms cpu, 4->4->0 MB, 5 MB goal, 8 P
    
    // 解释：
    // gc 1: 第1次GC
    // @0.001s: 程序启动0.001秒后
    // 0%: GC占用CPU时间百分比
    // 0.018+0.23+0.004 ms clock: STW时间+并发标记时间+STW时间
    // 4->4->0 MB: GC前堆大小->GC后堆大小->存活对象大小
    // 5 MB goal: 下次GC触发阈值
    // 8 P: 使用8个P
    
    for i := 0; i < 5; i++ {
        allocateMemory()
        runtime.GC()
    }
}

func allocateMemory() {
    _ = make([]byte, 10*1024*1024) // 10MB
    fmt.Println("分配10MB内存")
}
```

### 5.3 使用pprof分析内存

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
    // 🔥 启动pprof服务
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    
    fmt.Println("pprof服务启动: http://localhost:6060/debug/pprof/")
    fmt.Println("查看堆内存: http://localhost:6060/debug/pprof/heap")
    fmt.Println("查看GC: http://localhost:6060/debug/pprof/heap?gc=1")
    
    // 模拟内存分配
    ticker := time.NewTicker(time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        allocateMemory()
        
        var m runtime.MemStats
        runtime.ReadMemStats(&m)
        fmt.Printf("堆内存: %d MB, GC次数: %d\n", 
            m.HeapAlloc/1024/1024, m.NumGC)
    }
}

func allocateMemory() {
    data := make([][]byte, 100)
    for i := range data {
        data[i] = make([]byte, 1024*1024) // 1MB
    }
}

// 使用方法：
// 1. 运行程序
// 2. 浏览器访问 http://localhost:6060/debug/pprof/heap
// 3. 或使用命令行：go tool pprof http://localhost:6060/debug/pprof/heap
```

---

## 6. GC调优技巧

### 6.1 调整GOGC参数

```go
package main

import (
    "fmt"
    "runtime"
    "runtime/debug"
)

func main() {
    // 🔥 调整GOGC参数
    
    // GOGC默认值为100
    // 表示当堆内存增长100%时触发GC
    
    // 场景1：内存充足，追求性能
    // 增大GOGC，减少GC频率
    debug.SetGCPercent(200) // 堆内存增长200%时触发GC
    
    // 场景2：内存紧张，追求内存利用率
    // 减小GOGC，增加GC频率
    debug.SetGCPercent(50) // 堆内存增长50%时触发GC
    
    // 场景3：禁用GC（不推荐）
    debug.SetGCPercent(-1)
    
    // 查看当前GOGC
    gcPercent := debug.SetGCPercent(-1)
    debug.SetGCPercent(gcPercent)
    fmt.Printf("当前GOGC: %d\n", gcPercent)
}
```

### 6.2 对象池复用

```go
package main

import (
    "fmt"
    "sync"
)

// 🔥 使用sync.Pool减少GC压力

var bufferPool = sync.Pool{
    New: func() interface{} {
        return make([]byte, 1024)
    },
}

func processData() {
    // 从池中获取对象
    buffer := bufferPool.Get().([]byte)
    defer bufferPool.Put(buffer) // 使用完放回池中
    
    // 使用buffer
    _ = buffer
}

func main() {
    for i := 0; i < 1000000; i++ {
        processData()
    }
    fmt.Println("处理完成")
}
```

### 6.3 减少指针使用

```go
package main

import (
    "fmt"
    "runtime"
)

// ❌ 大量指针增加GC扫描负担
type BadStruct struct {
    data []*int // 指针数组
}

// ✅ 使用值类型减少GC扫描
type GoodStruct struct {
    data []int // 值数组
}

func main() {
    // 🔥 减少指针使用可以显著降低GC压力
    
    // 测试1：使用指针
    start := runtime.NumGC()
    bad := make([]*BadStruct, 10000)
    for i := range bad {
        bad[i] = &BadStruct{
            data: make([]*int, 100),
        }
    }
    runtime.GC()
    fmt.Printf("使用指针，GC次数增加: %d\n", runtime.NumGC()-start)
    
    // 测试2：使用值类型
    start = runtime.NumGC()
    good := make([]GoodStruct, 10000)
    for i := range good {
        good[i] = GoodStruct{
            data: make([]int, 100),
        }
    }
    runtime.GC()
    fmt.Printf("使用值类型，GC次数增加: %d\n", runtime.NumGC()-start)
}
```

---

## 7. 内存泄漏排查

### 7.1 常见内存泄漏场景

```go
package main

import (
    "fmt"
    "time"
)

// ❌ 场景1：Goroutine泄漏
func goroutineLeak() {
    ch := make(chan int)
    go func() {
        val := <-ch // 永远阻塞，Goroutine泄漏
        fmt.Println(val)
    }()
    // 没有发送数据，Goroutine永远不会退出
}

// ❌ 场景2：定时器泄漏
func timerLeak() {
    for i := 0; i < 1000; i++ {
        time.AfterFunc(time.Hour, func() {
            fmt.Println("执行")
        })
        // 没有Stop定时器，导致泄漏
    }
}

// ❌ 场景3：全局变量持有引用
var globalCache = make(map[string][]byte)

func cacheLeak() {
    for i := 0; i < 10000; i++ {
        key := fmt.Sprintf("key_%d", i)
        globalCache[key] = make([]byte, 1024*1024) // 1MB
    }
    // 全局变量持有引用，无法被GC回收
}

// ✅ 正确做法：及时清理
func correctUsage() {
    // 1. 使用context控制Goroutine生命周期
    // 2. 及时Stop定时器
    // 3. 定期清理缓存
}
```

### 7.2 使用pprof排查内存泄漏

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
    // 🔥 启动pprof
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    
    fmt.Println("pprof: http://localhost:6060/debug/pprof/")
    
    // 模拟内存泄漏
    go memoryLeak()
    
    // 定期打印内存使用
    ticker := time.NewTicker(5 * time.Second)
    for range ticker.C {
        var m runtime.MemStats
        runtime.ReadMemStats(&m)
        fmt.Printf("堆内存: %d MB\n", m.HeapAlloc/1024/1024)
    }
}

var leakedData [][]byte

func memoryLeak() {
    ticker := time.NewTicker(time.Second)
    for range ticker.C {
        // 持续分配内存但不释放
        leakedData = append(leakedData, make([]byte, 1024*1024))
    }
}

// 排查步骤：
// 1. 运行程序
// 2. 等待一段时间，让内存泄漏积累
// 3. 获取heap profile：
//    curl http://localhost:6060/debug/pprof/heap > heap.prof
// 4. 分析profile：
//    go tool pprof heap.prof
// 5. 使用top命令查看内存占用最多的函数
// 6. 使用list命令查看具体代码
```

---

## 8. 最佳实践

### 8.1 GC友好的代码

```go
package main

// ✅ 最佳实践

// 1. 预分配切片容量
func goodSlice() []int {
    s := make([]int, 0, 100) // 预分配容量
    for i := 0; i < 100; i++ {
        s = append(s, i)
    }
    return s
}

// 2. 使用对象池
// 见前面的sync.Pool示例

// 3. 避免频繁创建小对象
func goodString() string {
    // 使用strings.Builder
    var builder strings.Builder
    for i := 0; i < 100; i++ {
        builder.WriteString("hello")
    }
    return builder.String()
}

// 4. 及时释放大对象
func goodRelease() {
    data := make([]byte, 100*1024*1024) // 100MB
    // 使用data
    _ = data
    data = nil // 及时释放
    runtime.GC() // 可选：手动触发GC
}
```

---

## 📝 学习检查清单

- [ ] 深入理解三色标记算法
- [ ] 掌握写屏障机制
- [ ] 了解GC触发时机
- [ ] 能够使用pprof分析内存
- [ ] 掌握GC调优技巧
- [ ] 能够排查内存泄漏
- [ ] 编写GC友好的代码

---

## 🔗 相关资源

- [Go GC设计文档](https://go.dev/doc/gc-guide)
- [Go运行时源码](https://github.com/golang/go/tree/master/src/runtime)
- [GC可视化工具](https://github.com/davecheney/gcvis)
- [内存分析工具pprof](https://github.com/google/pprof)

---

@author erik.zhou
