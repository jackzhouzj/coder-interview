# 协程与异步IO完整教程

> 掌握Python异步编程，构建高性能IO密集型应用
> 
> @author erik.zhou

## 📚 教程概述

**版本信息**: Python 3.11+  
**学习难度**: ⭐⭐⭐⭐  
**重要程度**: ⭐⭐⭐⭐⭐  
**预计学习时长**: 30-40小时

## 🎯 学习目标

- [ ] 理解协程和异步IO的原理
- [ ] 掌握asyncio核心API
- [ ] 能够使用async/await编写异步代码
- [ ] 掌握异步HTTP客户端和服务器
- [ ] 理解事件循环机制
- [ ] 能够处理异步异常和取消
- [ ] 掌握异步数据库操作
- [ ] 了解异步编程最佳实践

## 📖 目录

1. [协程基础](#1-协程基础)
2. [asyncio核心](#2-asyncio核心)
3. [async/await语法](#3-asyncawait语法)
4. [事件循环](#4-事件循环)
5. [异步HTTP](#5-异步http)
6. [异步数据库](#6-异步数据库)
7. [任务和Future](#7-任务和future)
8. [异步上下文管理器](#8-异步上下文管理器)
9. [异步生成器](#9-异步生成器)
10. [最佳实践](#10-最佳实践)

---

## 1. 协程基础

### 1.1 什么是协程

```python
"""
协程概念
@author erik.zhou
"""
import asyncio

print("""
协程（Coroutine）：

定义：
- 可以暂停和恢复执行的函数
- 使用async def定义
- 使用await调用

与线程的区别：
- 协程：单线程，协作式调度，轻量级
- 线程：多线程，抢占式调度，重量级

优势：
1. 高并发：可以创建成千上万个协程
2. 低开销：切换成本极低
3. 无需锁：单线程执行，避免竞态条件
4. 易于理解：代码看起来像同步代码

适用场景：
- IO密集型任务
- 网络请求
- 文件操作
- 数据库查询

不适用场景：
- CPU密集型任务（使用多进程）
- 需要真正并行执行的任务
""")

# 第一个协程
async def hello():
    """简单的协程函数"""
    print("Hello")
    await asyncio.sleep(1)  # 模拟IO操作
    print("World")

# 运行协程
asyncio.run(hello())
```

### 1.2 协程的执行

```python
"""
协程执行方式
@author erik.zhou
"""
import asyncio

async def say_hello(name):
    """打招呼的协程"""
    print(f"Hello, {name}!")
    await asyncio.sleep(1)
    print(f"Goodbye, {name}!")
    return f"Done with {name}"

# 方式1：使用asyncio.run()（推荐）
result = asyncio.run(say_hello("Alice"))
print(f"Result: {result}")

# 方式2：手动管理事件循环
async def main():
    """主协程"""
    result = await say_hello("Bob")
    print(f"Result: {result}")

asyncio.run(main())

# 方式3：并发执行多个协程
async def run_multiple():
    """并发执行多个协程"""
    # 使用gather
    results = await asyncio.gather(
        say_hello("Alice"),
        say_hello("Bob"),
        say_hello("Charlie")
    )
    print(f"All results: {results}")

asyncio.run(run_multiple())

print("""
执行方式对比：

1. asyncio.run()：
   - Python 3.7+
   - 自动创建和关闭事件循环
   - 推荐用于主入口

2. await：
   - 在协程内部调用其他协程
   - 会暂停当前协程

3. asyncio.gather()：
   - 并发执行多个协程
   - 等待所有协程完成
   - 返回结果列表

4. asyncio.create_task()：
   - 创建任务并发执行
   - 不等待完成
""")
```

### 1.3 协程的状态

```python
"""
协程状态
@author erik.zhou
"""
import asyncio
import inspect

async def example_coroutine():
    """示例协程"""
    print("协程开始")
    await asyncio.sleep(1)
    print("协程结束")
    return "完成"

# 创建协程对象（但不执行）
coro = example_coroutine()

print(f"协程对象: {coro}")
print(f"是否为协程: {inspect.iscoroutine(coro)}")
print(f"协程状态: {inspect.getcoroutinestate(coro)}")

# 执行协程
result = asyncio.run(coro)
print(f"结果: {result}")

print("""
协程状态：

1. CORO_CREATED：已创建，未执行
2. CORO_RUNNING：正在执行
3. CORO_SUSPENDED：已暂停（await）
4. CORO_CLOSED：已完成或关闭

注意：
- 协程对象只能运行一次
- 未运行的协程会产生警告
- 使用await或asyncio.run()执行
""")
```

---

## 2. asyncio核心

### 2.1 事件循环基础

```python
"""
事件循环（Event Loop）
@author erik.zhou
"""
import asyncio

print("""
事件循环：

概念：
- 异步编程的核心
- 管理和调度协程执行
- 处理IO事件

工作原理：
1. 注册协程到事件循环
2. 循环检查可执行的协程
3. 执行协程直到遇到await
4. 切换到其他协程
5. 重复直到所有协程完成

关键API：
- asyncio.run()：运行主协程
- asyncio.get_event_loop()：获取当前循环
- loop.run_until_complete()：运行协程
- loop.create_task()：创建任务
""")

async def task1():
    """任务1"""
    print("任务1开始")
    await asyncio.sleep(2)
    print("任务1完成")

async def task2():
    """任务2"""
    print("任务2开始")
    await asyncio.sleep(1)
    print("任务2完成")

async def main():
    """主函数"""
    # 并发执行
    await asyncio.gather(task1(), task2())

# 运行
asyncio.run(main())
```

### 2.2 创建和管理任务

```python
"""
任务管理
@author erik.zhou
"""
import asyncio

async def fetch_data(id, delay):
    """模拟获取数据"""
    print(f"开始获取数据 {id}")
    await asyncio.sleep(delay)
    print(f"完成获取数据 {id}")
    return f"数据 {id}"

async def main():
    """主函数"""
    # 方式1：使用create_task创建任务
    task1 = asyncio.create_task(fetch_data(1, 2))
    task2 = asyncio.create_task(fetch_data(2, 1))
    
    # 等待任务完成
    result1 = await task1
    result2 = await task2
    
    print(f"结果: {result1}, {result2}")
    
    # 方式2：使用gather并发执行
    results = await asyncio.gather(
        fetch_data(3, 1),
        fetch_data(4, 2),
        fetch_data(5, 1.5)
    )
    print(f"所有结果: {results}")
    
    # 方式3：使用TaskGroup（Python 3.11+）
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(fetch_data(6, 1))
        t2 = tg.create_task(fetch_data(7, 2))
    
    print("TaskGroup完成")

asyncio.run(main())

print("""
任务管理方式对比：

1. create_task()：
   - 立即开始执行
   - 返回Task对象
   - 需要手动await

2. gather()：
   - 并发执行多个协程
   - 等待所有完成
   - 返回结果列表
   - 可以设置return_exceptions=True

3. TaskGroup（3.11+）：
   - 结构化并发
   - 自动等待所有任务
   - 异常会取消其他任务
""")
```

### 2.3 超时和取消

```python
"""
超时和取消
@author erik.zhou
"""
import asyncio

async def long_running_task():
    """长时间运行的任务"""
    try:
        print("任务开始")
        await asyncio.sleep(10)
        print("任务完成")
        return "成功"
    except asyncio.CancelledError:
        print("任务被取消")
        raise

async def timeout_example():
    """超时示例"""
    try:
        # 设置超时
        result = await asyncio.wait_for(
            long_running_task(),
            timeout=2.0
        )
        print(f"结果: {result}")
    except asyncio.TimeoutError:
        print("任务超时")

async def cancel_example():
    """取消示例"""
    task = asyncio.create_task(long_running_task())
    
    # 等待1秒后取消
    await asyncio.sleep(1)
    task.cancel()
    
    try:
        await task
    except asyncio.CancelledError:
        print("任务已取消")

# 运行示例
asyncio.run(timeout_example())
asyncio.run(cancel_example())

print("""
超时和取消：

1. wait_for()：
   - 设置超时时间
   - 超时抛出TimeoutError
   - 自动取消任务

2. cancel()：
   - 请求取消任务
   - 抛出CancelledError
   - 需要在协程中处理

3. 最佳实践：
   - 总是处理CancelledError
   - 清理资源
   - 使用try/finally
""")
```

---

## 3. async/await语法

### 3.1 async def定义协程

```python
"""
async def语法
@author erik.zhou
"""
import asyncio

# 普通函数
def sync_function():
    """同步函数"""
    return "同步结果"

# 协程函数
async def async_function():
    """异步函数"""
    await asyncio.sleep(1)
    return "异步结果"

# 错误示例：不能在同步函数中使用await
def wrong_function():
    # result = await async_function()  # SyntaxError
    pass

# 正确：在协程中使用await
async def correct_function():
    """正确使用await"""
    result = await async_function()
    return result

# 协程可以调用同步函数
async def mixed_function():
    """混合调用"""
    sync_result = sync_function()  # 直接调用
    async_result = await async_function()  # await调用
    return sync_result, async_result

asyncio.run(mixed_function())

print("""
async/await规则：

1. async def：
   - 定义协程函数
   - 返回协程对象
   - 必须被await或run

2. await：
   - 只能在async函数中使用
   - 暂停当前协程
   - 等待异步操作完成

3. 可await对象：
   - 协程
   - Task
   - Future
   - 实现__await__的对象
""")
```

### 3.2 await表达式

```python
"""
await表达式详解
@author erik.zhou
"""
import asyncio

async def compute(x, y):
    """计算函数"""
    await asyncio.sleep(1)
    return x + y

async def main():
    """主函数"""
    # 基本用法
    result = await compute(1, 2)
    print(f"结果: {result}")
    
    # 链式await
    result = await compute(await compute(1, 2), 3)
    print(f"链式结果: {result}")
    
    # 在表达式中使用
    total = await compute(1, 2) + await compute(3, 4)
    print(f"总和: {total}")
    
    # 条件表达式
    x = await compute(1, 2) if True else await compute(3, 4)
    print(f"条件结果: {x}")

asyncio.run(main())

print("""
await使用场景：

1. 调用协程：
   result = await coroutine()

2. 等待Task：
   result = await task

3. 等待Future：
   result = await future

4. 表达式中：
   x = await func1() + await func2()

注意事项：
- await会暂停当前协程
- 不要在循环中串行await（应该并发）
- 使用gather或create_task并发执行
""")
```

---

## 4. 事件循环

### 4.1 事件循环API

```python
"""
事件循环API
@author erik.zhou
"""
import asyncio

async def task():
    """示例任务"""
    print("任务执行")
    await asyncio.sleep(1)
    return "完成"

# 获取事件循环
loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)

try:
    # 运行协程
    result = loop.run_until_complete(task())
    print(f"结果: {result}")
    
    # 运行多个任务
    tasks = [task() for _ in range(3)]
    results = loop.run_until_complete(asyncio.gather(*tasks))
    print(f"多个结果: {results}")
    
finally:
    # 关闭循环
    loop.close()

print("""
事件循环API：

创建和获取：
- asyncio.new_event_loop()：创建新循环
- asyncio.get_event_loop()：获取当前循环
- asyncio.set_event_loop()：设置当前循环

运行：
- loop.run_until_complete()：运行协程
- loop.run_forever()：永久运行
- loop.stop()：停止循环
- loop.close()：关闭循环

任务管理：
- loop.create_task()：创建任务
- loop.call_soon()：调度回调
- loop.call_later()：延迟调度
- loop.call_at()：定时调度

推荐：
- 使用asyncio.run()代替手动管理
- 每个线程只有一个事件循环
""")
```

### 4.2 回调函数

```python
"""
事件循环回调
@author erik.zhou
"""
import asyncio

def callback(arg):
    """回调函数"""
    print(f"回调执行: {arg}")

async def main():
    """主函数"""
    loop = asyncio.get_running_loop()
    
    # 立即调度
    loop.call_soon(callback, "立即执行")
    
    # 延迟调度
    loop.call_later(1, callback, "1秒后执行")
    
    # 定时调度
    loop.call_at(loop.time() + 2, callback, "2秒后执行")
    
    # 等待回调执行
    await asyncio.sleep(3)

asyncio.run(main())

print("""
回调函数：

类型：
1. call_soon()：下一次迭代执行
2. call_later(delay, callback)：延迟执行
3. call_at(when, callback)：指定时间执行

特点：
- 回调在事件循环中执行
- 不能是协程函数
- 适合简单的同步操作

推荐：
- 优先使用协程而不是回调
- 回调适合与同步代码集成
""")
```

---

## 5. 异步HTTP

### 5.1 aiohttp客户端

```python
"""
aiohttp HTTP客户端
@author erik.zhou
"""
import asyncio
import aiohttp

async def fetch_url(session, url):
    """获取URL内容"""
    async with session.get(url) as response:
        return await response.text()

async def fetch_multiple_urls():
    """并发获取多个URL"""
    urls = [
        'https://api.github.com',
        'https://httpbin.org/get',
        'https://jsonplaceholder.typicode.com/posts/1'
    ]
    
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        for url, result in zip(urls, results):
            if isinstance(result, Exception):
                print(f"{url}: 错误 - {result}")
            else:
                print(f"{url}: 成功 - {len(result)} 字节")

# asyncio.run(fetch_multiple_urls())

print("""
aiohttp客户端特点：

1. 异步HTTP请求：
   - 支持GET、POST等方法
   - 自动处理连接池
   - 支持超时设置

2. Session管理：
   - 复用连接
   - 自动处理Cookie
   - 支持认证

3. 最佳实践：
   - 使用async with管理Session
   - 设置合理的超时
   - 处理异常
   - 限制并发数量

示例：
async with aiohttp.ClientSession() as session:
    async with session.get(url, timeout=10) as resp:
        data = await resp.json()
""")
```

### 5.2 aiohttp服务器

```python
"""
aiohttp Web服务器
@author erik.zhou
"""
from aiohttp import web
import asyncio

async def handle_hello(request):
    """处理hello请求"""
    name = request.match_info.get('name', 'World')
    return web.Response(text=f"Hello, {name}!")

async def handle_json(request):
    """处理JSON请求"""
    data = await request.json()
    response_data = {
        'received': data,
        'status': 'success'
    }
    return web.json_response(response_data)

async def handle_post(request):
    """处理POST请求"""
    data = await request.post()
    return web.Response(text=f"Received: {dict(data)}")

# 创建应用
app = web.Application()

# 添加路由
app.router.add_get('/', handle_hello)
app.router.add_get('/hello/{name}', handle_hello)
app.router.add_post('/json', handle_json)
app.router.add_post('/post', handle_post)

# 运行服务器
# web.run_app(app, host='127.0.0.1', port=8080)

print("""
aiohttp服务器特点：

1. 异步Web框架：
   - 高性能
   - 支持WebSocket
   - 中间件支持

2. 路由系统：
   - 路径参数
   - 查询参数
   - 请求体解析

3. 最佳实践：
   - 使用中间件处理通用逻辑
   - 异步数据库操作
   - 合理设置超时
   - 错误处理

示例：
app = web.Application()
app.router.add_get('/api/users', get_users)
web.run_app(app)
""")
```

---

## 6. 异步数据库

### 6.1 异步数据库操作

```python
"""
异步数据库操作
@author erik.zhou
"""
import asyncio
# import asyncpg  # PostgreSQL
# import aiomysql  # MySQL
# import aiosqlite  # SQLite

async def async_database_example():
    """异步数据库示例"""
    print("""
    异步数据库库：
    
    1. asyncpg（PostgreSQL）：
       conn = await asyncpg.connect(
           host='localhost',
           database='mydb',
           user='user',
           password='password'
       )
       rows = await conn.fetch('SELECT * FROM users')
       await conn.close()
    
    2. aiomysql（MySQL）：
       conn = await aiomysql.connect(
           host='localhost',
           user='user',
           password='password',
           db='mydb'
       )
       async with conn.cursor() as cursor:
           await cursor.execute('SELECT * FROM users')
           rows = await cursor.fetchall()
       conn.close()
    
    3. aiosqlite（SQLite）：
       async with aiosqlite.connect('database.db') as db:
           async with db.execute('SELECT * FROM users') as cursor:
               rows = await cursor.fetchall()
    
    4. Motor（MongoDB）：
       from motor.motor_asyncio import AsyncIOMotorClient
       client = AsyncIOMotorClient('mongodb://localhost:27017')
       db = client.mydb
       users = await db.users.find().to_list(length=100)
    
    最佳实践：
    - 使用连接池
    - 异步事务处理
    - 批量操作
    - 错误处理和重试
    """)

asyncio.run(async_database_example())
```

### 6.2 SQLAlchemy异步

```python
"""
SQLAlchemy异步ORM
@author erik.zhou
"""
import asyncio
# from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
# from sqlalchemy.orm import sessionmaker

async def sqlalchemy_async_example():
    """SQLAlchemy异步示例"""
    print("""
    SQLAlchemy异步ORM：
    
    1. 创建异步引擎：
       engine = create_async_engine(
           'postgresql+asyncpg://user:pass@localhost/dbname',
           echo=True
       )
    
    2. 创建异步Session：
       async_session = sessionmaker(
           engine,
           class_=AsyncSession,
           expire_on_commit=False
       )
    
    3. 使用Session：
       async with async_session() as session:
           result = await session.execute(
               select(User).where(User.name == 'Alice')
           )
           users = result.scalars().all()
    
    4. 事务处理：
       async with async_session() as session:
           async with session.begin():
               session.add(new_user)
               await session.commit()
    
    优势：
    - ORM功能完整
    - 类型提示支持
    - 迁移工具（Alembic）
    - 与同步代码兼容
    """)

asyncio.run(sqlalchemy_async_example())
```

---

## 7. 任务和Future

### 7.1 Task详解

```python
"""
Task对象详解
@author erik.zhou
"""
import asyncio

async def work(name, duration):
    """工作任务"""
    print(f"{name} 开始")
    await asyncio.sleep(duration)
    print(f"{name} 完成")
    return f"{name} 结果"

async def task_management():
    """任务管理"""
    # 创建任务
    task1 = asyncio.create_task(work("任务1", 2))
    task2 = asyncio.create_task(work("任务2", 1))
    
    # 检查任务状态
    print(f"任务1完成: {task1.done()}")
    print(f"任务2取消: {task2.cancelled()}")
    
    # 等待任务
    result1 = await task1
    result2 = await task2
    
    print(f"结果: {result1}, {result2}")
    
    # 任务异常处理
    async def failing_task():
        await asyncio.sleep(1)
        raise ValueError("任务失败")
    
    task3 = asyncio.create_task(failing_task())
    
    try:
        await task3
    except ValueError as e:
        print(f"捕获异常: {e}")
    
    # 获取所有任务
    all_tasks = asyncio.all_tasks()
    print(f"当前任务数: {len(all_tasks)}")

asyncio.run(task_management())

print("""
Task对象：

属性和方法：
- done()：是否完成
- cancelled()：是否取消
- cancel()：请求取消
- result()：获取结果
- exception()：获取异常
- add_done_callback()：添加回调

生命周期：
1. 创建：create_task()
2. 运行：自动调度
3. 完成：正常或异常
4. 取消：cancel()

注意事项：
- Task是Future的子类
- 自动调度执行
- 异常会被保存
- 需要await或获取结果
""")
```

### 7.2 Future对象

```python
"""
Future对象
@author erik.zhou
"""
import asyncio

async def future_example():
    """Future示例"""
    # 创建Future
    future = asyncio.Future()
    
    async def set_result():
        """设置结果"""
        await asyncio.sleep(1)
        future.set_result("Future结果")
    
    # 创建任务设置结果
    asyncio.create_task(set_result())
    
    # 等待Future完成
    result = await future
    print(f"Future结果: {result}")
    
    # Future异常
    future2 = asyncio.Future()
    future2.set_exception(ValueError("Future异常"))
    
    try:
        await future2
    except ValueError as e:
        print(f"捕获Future异常: {e}")

asyncio.run(future_example())

print("""
Future对象：

概念：
- 表示异步操作的最终结果
- 可以设置结果或异常
- 可以被await

方法：
- set_result(value)：设置结果
- set_exception(exc)：设置异常
- result()：获取结果
- exception()：获取异常
- done()：是否完成
- cancelled()：是否取消

使用场景：
- 与回调代码集成
- 手动控制异步流程
- 实现自定义异步对象

注意：
- Task是Future的子类
- 通常不需要直接创建Future
- 使用Task更方便
""")
```

---

## 8. 异步上下文管理器

### 8.1 async with语法

```python
"""
异步上下文管理器
@author erik.zhou
"""
import asyncio

class AsyncResource:
    """异步资源"""
    
    async def __aenter__(self):
        """进入上下文"""
        print("获取资源")
        await asyncio.sleep(1)
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """退出上下文"""
        print("释放资源")
        await asyncio.sleep(1)
        return False
    
    async def do_something(self):
        """执行操作"""
        print("使用资源")
        await asyncio.sleep(1)

async def use_async_resource():
    """使用异步资源"""
    async with AsyncResource() as resource:
        await resource.do_something()
    print("资源已释放")

asyncio.run(use_async_resource())

# 使用asynccontextmanager装饰器
from contextlib import asynccontextmanager

@asynccontextmanager
async def async_resource():
    """异步资源管理器"""
    print("设置资源")
    await asyncio.sleep(1)
    try:
        yield "资源"
    finally:
        print("清理资源")
        await asyncio.sleep(1)

async def use_decorator():
    """使用装饰器"""
    async with async_resource() as res:
        print(f"使用 {res}")

asyncio.run(use_decorator())

print("""
异步上下文管理器：

实现方式：
1. 实现__aenter__和__aexit__
2. 使用@asynccontextmanager装饰器

使用场景：
- 异步数据库连接
- 异步文件操作
- 异步网络连接
- 异步锁

示例：
async with aiohttp.ClientSession() as session:
    async with session.get(url) as response:
        data = await response.text()

优势：
- 自动资源管理
- 异常安全
- 代码简洁
""")
```

---

## 9. 异步生成器

### 9.1 async for语法

```python
"""
异步生成器
@author erik.zhou
"""
import asyncio

async def async_range(n):
    """异步range生成器"""
    for i in range(n):
        await asyncio.sleep(0.1)
        yield i

async def use_async_generator():
    """使用异步生成器"""
    async for i in async_range(5):
        print(f"值: {i}")

asyncio.run(use_async_generator())

# 异步生成器表达式
async def async_comprehension():
    """异步推导式"""
    # 异步列表推导
    results = [i async for i in async_range(5)]
    print(f"结果: {results}")
    
    # 异步生成器表达式
    gen = (i * 2 async for i in async_range(5))
    async for value in gen:
        print(f"双倍: {value}")

asyncio.run(async_comprehension())

print("""
异步生成器：

定义：
async def generator():
    for item in items:
        await asyncio.sleep(1)
        yield item

使用：
async for item in generator():
    process(item)

异步推导：
- [x async for x in gen]
- {x async for x in gen}
- (x async for x in gen)

应用场景：
- 流式数据处理
- 异步迭代大数据集
- 实时数据流
- 分页API调用
""")
```

---

## 10. 最佳实践

```python
"""
异步编程最佳实践
@author erik.zhou
"""
import asyncio

print("""
异步编程最佳实践：

1. 何时使用异步：
   ✅ IO密集型任务（网络、文件、数据库）
   ✅ 高并发场景
   ✅ 需要等待外部资源
   ❌ CPU密集型任务
   ❌ 简单的同步操作

2. 并发控制：
   - 使用Semaphore限制并发数
   - 避免创建过多任务
   - 合理设置超时

3. 异常处理：
   - 总是处理异常
   - 使用try/except包装await
   - gather()使用return_exceptions=True

4. 资源管理：
   - 使用async with管理资源
   - 及时关闭连接
   - 避免资源泄漏

5. 性能优化：
   - 批量操作代替循环
   - 使用连接池
   - 缓存频繁访问的数据
   - 避免阻塞操作

6. 调试技巧：
   - 使用asyncio.run(debug=True)
   - 检查未await的协程
   - 使用日志记录
   - 性能分析工具

7. 常见陷阱：
   ❌ 忘记await
   ❌ 在循环中串行await
   ❌ 阻塞事件循环
   ❌ 不处理CancelledError
   ❌ 混用同步和异步代码

8. 代码组织：
   - 异步函数命名清晰
   - 分离同步和异步代码
   - 使用类型提示
   - 编写异步测试
""")

# 并发控制示例
async def controlled_concurrency():
    """控制并发数量"""
    semaphore = asyncio.Semaphore(5)  # 最多5个并发
    
    async def limited_task(i):
        async with semaphore:
            print(f"任务 {i} 开始")
            await asyncio.sleep(1)
            print(f"任务 {i} 完成")
    
    tasks = [limited_task(i) for i in range(20)]
    await asyncio.gather(*tasks)

asyncio.run(controlled_concurrency())
```

---

## 📝 学习检查清单

- [ ] 理解协程和异步IO的原理
- [ ] 掌握asyncio核心API
- [ ] 能够使用async/await编写异步代码
- [ ] 掌握异步HTTP客户端和服务器
- [ ] 理解事件循环机制
- [ ] 能够处理异步异常和取消
- [ ] 掌握异步数据库操作
- [ ] 了解异步编程最佳实践
- [ ] 能够进行异步性能优化
- [ ] 掌握异步调试技巧

## 🔗 相关资源

- [Python asyncio文档](https://docs.python.org/zh-cn/3/library/asyncio.html)
- [aiohttp文档](https://docs.aiohttp.org/)
- [Real Python - Async IO](https://realpython.com/async-io-python/)
- [asyncpg文档](https://magicstack.github.io/asyncpg/)
- [Motor文档](https://motor.readthedocs.io/)

---

**@author erik.zhou**  
**最后更新**: 2026-02-13
