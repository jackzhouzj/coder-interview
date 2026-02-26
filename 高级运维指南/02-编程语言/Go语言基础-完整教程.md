# Go语言基础完整教程

> Go是云原生时代的运维开发语言，高性能、简洁、并发能力强
>
> @author erik.zhou

## 📋 目录

- [Go语言简介](#go语言简介)
- [环境搭建](#环境搭建)
- [基础语法](#基础语法)
- [并发编程](#并发编程)
- [网络编程](#网络编程)
- [运维工具开发](#运维工具开发)

## 🎯 学习目标

- [ ] 理解Go语言特性和优势
- [ ] 掌握Go基础语法
- [ ] 熟练使用goroutine和channel
- [ ] 能够开发网络应用
- [ ] 开发运维自动化工具
- [ ] 理解Go的最佳实践

## Go语言简介

### 为什么选择Go

**优势**
- 编译速度快，执行效率高
- 语法简洁，学习曲线平缓
- 原生支持并发（goroutine）
- 丰富的标准库
- 静态类型，类型安全
- 自动垃圾回收
- 跨平台编译

**适用场景**
- 云原生应用开发
- 微服务架构
- 运维工具开发
- 网络服务器
- 分布式系统
- DevOps工具链

**知名项目**
- Docker - 容器引擎
- Kubernetes - 容器编排
- Prometheus - 监控系统
- Etcd - 分布式键值存储
- Consul - 服务发现
- Terraform - 基础设施即代码

## 环境搭建

### 安装Go

**Linux安装**
```bash
# 下载Go
wget https://go.dev/dl/go1.21.5.linux-amd64.tar.gz

# 解压
sudo tar -C /usr/local -xzf go1.21.5.linux-amd64.tar.gz

# 配置环境变量
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bashrc
source ~/.bashrc

# 验证安装
go version
```

**配置Go模块代理（国内）**
```bash
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

### 第一个Go程序

```go
// hello.go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

```bash
# 运行
go run hello.go

# 编译
go build hello.go
./hello

# 编译并安装
go install hello.go
```

## 基础语法

### 变量和常量

```go
package main

import "fmt"

func main() {
    // 变量声明
    var name string = "张三"
    var age int = 30
    var isAdmin bool = true
    
    // 类型推导
    var city = "北京"
    
    // 短变量声明（只能在函数内）
    email := "zhangsan@example.com"
    
    // 多变量声明
    var (
        username = "admin"
        password = "123456"
    )
    
    // 常量
    const PI = 3.14159
    const (
        StatusOK = 200
        StatusNotFound = 404
    )
    
    fmt.Println(name, age, isAdmin, city, email)
}
```

### 数据类型

```go
package main

import "fmt"

func main() {
    // 基本类型
    var i int = 42
    var f float64 = 3.14
    var b bool = true
    var s string = "hello"
    
    // 数组
    var arr [5]int = [5]int{1, 2, 3, 4, 5}
    
    // 切片（动态数组）
    slice := []int{1, 2, 3, 4, 5}
    slice = append(slice, 6)
    
    // 映射（字典）
    m := make(map[string]int)
    m["age"] = 30
    m["score"] = 95
    
    // 结构体
    type Person struct {
        Name string
        Age  int
    }
    p := Person{Name: "张三", Age: 30}
    
    // 指针
    var ptr *int = &i
    
    fmt.Println(i, f, b, s)
    fmt.Println(arr, slice, m, p, *ptr)
}
```

### 控制流程

```go
package main

import "fmt"

func main() {
    // if语句
    age := 18
    if age >= 18 {
        fmt.Println("成年人")
    } else {
        fmt.Println("未成年人")
    }
    
    // if with初始化
    if score := 85; score >= 90 {
        fmt.Println("优秀")
    } else if score >= 60 {
        fmt.Println("及格")
    } else {
        fmt.Println("不及格")
    }
    
    // switch语句
    day := 3
    switch day {
    case 1:
        fmt.Println("星期一")
    case 2:
        fmt.Println("星期二")
    case 3:
        fmt.Println("星期三")
    default:
        fmt.Println("其他")
    }
    
    // for循环
    for i := 0; i < 5; i++ {
        fmt.Println(i)
    }
    
    // while风格
    count := 0
    for count < 5 {
        fmt.Println(count)
        count++
    }
    
    // 遍历切片
    nums := []int{1, 2, 3, 4, 5}
    for index, value := range nums {
        fmt.Printf("索引: %d, 值: %d\n", index, value)
    }
    
    // 遍历映射
    m := map[string]int{"a": 1, "b": 2}
    for key, value := range m {
        fmt.Printf("键: %s, 值: %d\n", key, value)
    }
}
```

### 函数

```go
package main

import "fmt"

// 基本函数
func add(a int, b int) int {
    return a + b
}

// 多返回值
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("除数不能为0")
    }
    return a / b, nil
}

// 命名返回值
func calculate(a, b int) (sum int, product int) {
    sum = a + b
    product = a * b
    return // 自动返回sum和product
}

// 可变参数
func sum(nums ...int) int {
    total := 0
    for _, num := range nums {
        total += num
    }
    return total
}

// 闭包
func counter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}

func main() {
    fmt.Println(add(3, 5))
    
    result, err := divide(10, 2)
    if err != nil {
        fmt.Println("错误:", err)
    } else {
        fmt.Println("结果:", result)
    }
    
    s, p := calculate(3, 4)
    fmt.Printf("和: %d, 积: %d\n", s, p)
    
    fmt.Println(sum(1, 2, 3, 4, 5))
    
    c := counter()
    fmt.Println(c()) // 1
    fmt.Println(c()) // 2
}
```

### 结构体和方法

```go
package main

import "fmt"

// 定义结构体
type Server struct {
    Hostname string
    IP       string
    Port     int
    Status   string
}

// 值接收者方法
func (s Server) GetInfo() string {
    return fmt.Sprintf("%s (%s:%d) - %s", 
        s.Hostname, s.IP, s.Port, s.Status)
}

// 指针接收者方法（可以修改结构体）
func (s *Server) Start() {
    s.Status = "running"
}

func (s *Server) Stop() {
    s.Status = "stopped"
}

// 构造函数
func NewServer(hostname, ip string, port int) *Server {
    return &Server{
        Hostname: hostname,
        IP:       ip,
        Port:     port,
        Status:   "stopped",
    }
}

func main() {
    server := NewServer("web01", "192.168.1.10", 8080)
    fmt.Println(server.GetInfo())
    
    server.Start()
    fmt.Println(server.GetInfo())
    
    server.Stop()
    fmt.Println(server.GetInfo())
}
```

### 接口

```go
package main

import "fmt"

// 定义接口
type Storage interface {
    Save(key string, value string) error
    Load(key string) (string, error)
    Delete(key string) error
}

// 内存存储实现
type MemoryStorage struct {
    data map[string]string
}

func NewMemoryStorage() *MemoryStorage {
    return &MemoryStorage{
        data: make(map[string]string),
    }
}

func (m *MemoryStorage) Save(key string, value string) error {
    m.data[key] = value
    return nil
}

func (m *MemoryStorage) Load(key string) (string, error) {
    value, ok := m.data[key]
    if !ok {
        return "", fmt.Errorf("键不存在: %s", key)
    }
    return value, nil
}

func (m *MemoryStorage) Delete(key string) error {
    delete(m.data, key)
    return nil
}

// 使用接口
func processStorage(storage Storage) {
    storage.Save("name", "张三")
    value, _ := storage.Load("name")
    fmt.Println("值:", value)
    storage.Delete("name")
}

func main() {
    storage := NewMemoryStorage()
    processStorage(storage)
}
```

### 错误处理

```go
package main

import (
    "errors"
    "fmt"
)

// 自定义错误
type ConfigError struct {
    Field string
    Value string
}

func (e *ConfigError) Error() string {
    return fmt.Sprintf("配置错误: 字段 %s 的值 %s 无效", e.Field, e.Value)
}

// 返回错误
func validateConfig(port int) error {
    if port < 1 || port > 65535 {
        return &ConfigError{
            Field: "port",
            Value: fmt.Sprintf("%d", port),
        }
    }
    return nil
}

// 错误包装
func readConfig() error {
    err := validateConfig(70000)
    if err != nil {
        return fmt.Errorf("读取配置失败: %w", err)
    }
    return nil
}

func main() {
    // 基本错误处理
    err := validateConfig(8080)
    if err != nil {
        fmt.Println("错误:", err)
    } else {
        fmt.Println("配置有效")
    }
    
    // 错误类型断言
    err = validateConfig(70000)
    if err != nil {
        if configErr, ok := err.(*ConfigError); ok {
            fmt.Printf("配置错误: %s = %s\n", 
                configErr.Field, configErr.Value)
        }
    }
    
    // 错误包装和解包
    err = readConfig()
    if err != nil {
        fmt.Println("错误:", err)
        // 解包错误
        if errors.Is(err, &ConfigError{}) {
            fmt.Println("这是一个配置错误")
        }
    }
}
```

## 并发编程

### Goroutine

```go
package main

import (
    "fmt"
    "time"
)

func task(id int) {
    for i := 0; i < 3; i++ {
        fmt.Printf("任务 %d: 执行 %d\n", id, i)
        time.Sleep(time.Millisecond * 100)
    }
}

func main() {
    // 启动goroutine
    go task(1)
    go task(2)
    go task(3)
    
    // 等待goroutine完成
    time.Sleep(time.Second)
    fmt.Println("主程序结束")
}
```

### Channel

```go
package main

import "fmt"

func main() {
    // 创建channel
    ch := make(chan int)
    
    // 发送数据（在goroutine中）
    go func() {
        ch <- 42
    }()
    
    // 接收数据
    value := <-ch
    fmt.Println("接收到:", value)
    
    // 带缓冲的channel
    buffered := make(chan string, 2)
    buffered <- "hello"
    buffered <- "world"
    fmt.Println(<-buffered)
    fmt.Println(<-buffered)
    
    // 关闭channel
    ch2 := make(chan int, 3)
    ch2 <- 1
    ch2 <- 2
    ch2 <- 3
    close(ch2)
    
    // 遍历channel
    for value := range ch2 {
        fmt.Println(value)
    }
}
```

### Select

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)
    
    go func() {
        time.Sleep(time.Second)
        ch1 <- "来自ch1"
    }()
    
    go func() {
        time.Sleep(time.Second * 2)
        ch2 <- "来自ch2"
    }()
    
    // select等待多个channel
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println(msg1)
        case msg2 := <-ch2:
            fmt.Println(msg2)
        case <-time.After(time.Second * 3):
            fmt.Println("超时")
        }
    }
}
```

### WaitGroup

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    fmt.Printf("工作者 %d 开始\n", id)
    time.Sleep(time.Second)
    fmt.Printf("工作者 %d 完成\n", id)
}

func main() {
    var wg sync.WaitGroup
    
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go worker(i, &wg)
    }
    
    wg.Wait()
    fmt.Println("所有工作者完成")
}
```

### Mutex

```go
package main

import (
    "fmt"
    "sync"
)

type Counter struct {
    mu    sync.Mutex
    value int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}

func main() {
    counter := &Counter{}
    var wg sync.WaitGroup
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            counter.Increment()
        }()
    }
    
    wg.Wait()
    fmt.Println("计数器值:", counter.Value())
}
```

## 网络编程

### HTTP服务器

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
)

type Response struct {
    Message string `json:"message"`
    Status  int    `json:"status"`
}

func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
}

func jsonHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    
    response := Response{
        Message: "成功",
        Status:  200,
    }
    
    json.NewEncoder(w).Encode(response)
}

func main() {
    http.HandleFunc("/", helloHandler)
    http.HandleFunc("/api/status", jsonHandler)
    
    fmt.Println("服务器启动在 :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### HTTP客户端

```go
package main

import (
    "fmt"
    "io"
    "net/http"
    "time"
)

func main() {
    // GET请求
    resp, err := http.Get("https://api.github.com")
    if err != nil {
        fmt.Println("错误:", err)
        return
    }
    defer resp.Body.Close()
    
    body, _ := io.ReadAll(resp.Body)
    fmt.Println("响应:", string(body))
    
    // 自定义客户端
    client := &http.Client{
        Timeout: time.Second * 10,
    }
    
    resp, err = client.Get("https://api.github.com")
    if err != nil {
        fmt.Println("错误:", err)
        return
    }
    defer resp.Body.Close()
    
    fmt.Println("状态码:", resp.StatusCode)
}
```

## 运维工具开发

### 系统信息采集工具

```go
package main

import (
    "fmt"
    "os"
    "os/exec"
    "runtime"
    "strings"
)

type SystemInfo struct {
    OS       string
    Arch     string
    CPUs     int
    Hostname string
    Uptime   string
}

func getSystemInfo() (*SystemInfo, error) {
    hostname, err := os.Hostname()
    if err != nil {
        return nil, err
    }
    
    // 获取系统运行时间
    cmd := exec.Command("uptime")
    output, err := cmd.Output()
    if err != nil {
        return nil, err
    }
    
    return &SystemInfo{
        OS:       runtime.GOOS,
        Arch:     runtime.GOARCH,
        CPUs:     runtime.NumCPU(),
        Hostname: hostname,
        Uptime:   strings.TrimSpace(string(output)),
    }, nil
}

func main() {
    info, err := getSystemInfo()
    if err != nil {
        fmt.Println("错误:", err)
        return
    }
    
    fmt.Printf("操作系统: %s\n", info.OS)
    fmt.Printf("架构: %s\n", info.Arch)
    fmt.Printf("CPU核心数: %d\n", info.CPUs)
    fmt.Printf("主机名: %s\n", info.Hostname)
    fmt.Printf("运行时间: %s\n", info.Uptime)
}
```

### 日志监控工具

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strings"
    "time"
)

func monitorLog(filename string, keyword string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    
    // 移动到文件末尾
    file.Seek(0, os.SEEK_END)
    
    scanner := bufio.NewScanner(file)
    
    fmt.Printf("开始监控日志文件: %s\n", filename)
    fmt.Printf("关键词: %s\n", keyword)
    
    for {
        if scanner.Scan() {
            line := scanner.Text()
            if strings.Contains(line, keyword) {
                fmt.Printf("[%s] 发现匹配: %s\n", 
                    time.Now().Format("2006-01-02 15:04:05"), line)
            }
        }
        time.Sleep(time.Millisecond * 100)
    }
}

func main() {
    if len(os.Args) < 3 {
        fmt.Println("用法: logmonitor <文件> <关键词>")
        return
    }
    
    filename := os.Args[1]
    keyword := os.Args[2]
    
    if err := monitorLog(filename, keyword); err != nil {
        fmt.Println("错误:", err)
    }
}
```

### 批量SSH执行工具

```go
package main

import (
    "fmt"
    "golang.org/x/crypto/ssh"
    "sync"
)

type Server struct {
    Host     string
    Port     string
    User     string
    Password string
}

func executeCommand(server Server, command string) (string, error) {
    config := &ssh.ClientConfig{
        User: server.User,
        Auth: []ssh.AuthMethod{
            ssh.Password(server.Password),
        },
        HostKeyCallback: ssh.InsecureIgnoreHostKey(),
    }
    
    client, err := ssh.Dial("tcp", 
        fmt.Sprintf("%s:%s", server.Host, server.Port), config)
    if err != nil {
        return "", err
    }
    defer client.Close()
    
    session, err := client.NewSession()
    if err != nil {
        return "", err
    }
    defer session.Close()
    
    output, err := session.CombinedOutput(command)
    if err != nil {
        return "", err
    }
    
    return string(output), nil
}

func main() {
    servers := []Server{
        {Host: "192.168.1.10", Port: "22", User: "root", Password: "password"},
        {Host: "192.168.1.11", Port: "22", User: "root", Password: "password"},
    }
    
    command := "uptime"
    
    var wg sync.WaitGroup
    for _, server := range servers {
        wg.Add(1)
        go func(s Server) {
            defer wg.Done()
            
            output, err := executeCommand(s, command)
            if err != nil {
                fmt.Printf("[%s] 错误: %v\n", s.Host, err)
                return
            }
            
            fmt.Printf("[%s] 输出:\n%s\n", s.Host, output)
        }(server)
    }
    
    wg.Wait()
}
```

## 📚 学习检查清单

- [ ] 理解Go语言特性
- [ ] 掌握基础语法
- [ ] 熟练使用goroutine和channel
- [ ] 能够开发HTTP服务
- [ ] 开发运维自动化工具
- [ ] 理解Go的最佳实践

## 🔗 参考资源

- [Go官方文档](https://go.dev/doc/)
- [Go by Example](https://gobyexample.com/)
- [Effective Go](https://go.dev/doc/effective_go)
- [Go语言圣经](https://gopl-zh.github.io/)

---

@author erik.zhou
