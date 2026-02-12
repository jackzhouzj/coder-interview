# 并发编程

## 📋 模块概述

本模块涵盖Go语言最核心的特性——并发编程。Go的并发模型基于CSP（Communicating Sequential Processes），通过Goroutine和Channel实现高效的并发处理。

## 📚 学习内容

### 1. Goroutine
- Goroutine原理
- 创建和管理
- Goroutine调度
- 性能优化
- 常见陷阱

### 2. Channel
- Channel基础
- 有缓冲和无缓冲Channel
- Channel方向
- select语句
- Channel关闭

### 3. 并发模式
- Worker Pool
- Pipeline
- Fan-out/Fan-in
- 超时控制
- 取消机制

### 4. 同步原语
- Mutex互斥锁
- RWMutex读写锁
- WaitGroup
- Once
- Cond条件变量

### 5. Context
- Context接口
- WithCancel
- WithTimeout
- WithDeadline
- WithValue

## 🎯 学习目标

- [ ] 理解Goroutine的工作原理
- [ ] 掌握Channel的使用
- [ ] 能够实现常见的并发模式
- [ ] 理解并发安全
- [ ] 掌握Context的使用
- [ ] 能够进行并发性能优化

## 📖 子模块

1. [Goroutine完整教程](Goroutine-完整教程.md) ✅
2. [Channel完整教程](Channel-完整教程.md) ✅
3. [并发模式完整教程](并发模式-完整教程.md) ⭕ 待补充
4. [同步原语完整教程](同步原语-完整教程.md) ⭕ 待补充
5. [Context完整教程](Context-完整教程.md) ✅

## ⏱️ 预计学习时长

- Goroutine：10-15小时
- Channel：10-15小时
- 并发模式：15-20小时
- 同步原语：10-15小时
- Context：8-12小时
- 总计：53-77小时

## 📝 前置知识

- Go语言基础
- 函数和方法
- 接口
- 基本的并发概念

## 🔗 相关资源

- [Go并发编程](https://go.dev/doc/effective_go#concurrency)
- [Go并发模式](https://go.dev/blog/pipelines)
- [Go内存模型](https://go.dev/ref/mem)
- [Context包文档](https://pkg.go.dev/context)

---

**@author erik.zhou**
