# Python进阶完整教程

> 深入掌握Python高级特性，提升编程能力
> 
> @author erik.zhou

## 📚 教程概述

**版本信息**: Python 3.9+  
**学习难度**: ⭐⭐⭐⭐  
**重要程度**: ⭐⭐⭐⭐⭐  
**预计学习时长**: 20-30小时

## 🎯 学习目标

- [ ] 掌握装饰器的原理和应用
- [ ] 理解生成器和迭代器
- [ ] 掌握上下文管理器
- [ ] 理解元类和反射
- [ ] 掌握函数式编程
- [ ] 理解描述符协议
- [ ] 掌握高级数据结构

## 📖 目录

1. [装饰器](#1-装饰器)
2. [生成器与迭代器](#2-生成器与迭代器)
3. [上下文管理器](#3-上下文管理器)
4. [元类与反射](#4-元类与反射)
5. [函数式编程](#5-函数式编程)
6. [描述符](#6-描述符)
7. [高级数据结构](#7-高级数据结构)
8. [最佳实践](#8-最佳实践)

---

## 1. 装饰器

### 1.1 函数装饰器

#### 基础装饰器

```python
"""
函数装饰器基础示例
@author erik.zhou
"""
import time
from functools import wraps

def timer(func):
    """计时装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} 执行时间: {end - start:.4f}秒")
        return result
    return wrapper

@timer
def slow_function():
    """模拟耗时操作"""
    time.sleep(1)
    return "完成"

# 使用
result = slow_function()
```

#### 带参数的装饰器

```python
"""
带参数的装饰器
@author erik.zhou
"""
def repeat(times):
    """重复执行装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(times):
                result = func(*args, **kwargs)
                results.append(result)
            return results
        return wrapper
    return decorator

@repeat(times=3)
def greet(name):
    """问候函数"""
    return f"Hello, {name}!"

# 使用
results = greet("Alice")
print(results)  # ['Hello, Alice!', 'Hello, Alice!', 'Hello, Alice!']
```

### 1.2 类装饰器

```python
"""
类装饰器示例
@author erik.zhou
"""
class CountCalls:
    """统计函数调用次数的装饰器"""
    
    def __init__(self, func):
        self.func = func
        self.count = 0
    
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"{self.func.__name__} 被调用 {self.count} 次")
        return self.func(*args, **kwargs)

@CountCalls
def say_hello():
    """问候函数"""
    print("Hello!")

# 使用
say_hello()  # say_hello 被调用 1 次
say_hello()  # say_hello 被调用 2 次
```

### 1.3 装饰器链

```python
"""
装饰器链示例
@author erik.zhou
"""
def bold(func):
    """加粗装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        return f"<b>{func(*args, **kwargs)}</b>"
    return wrapper

def italic(func):
    """斜体装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        return f"<i>{func(*args, **kwargs)}</i>"
    return wrapper

@bold
@italic
def greet(name):
    """问候函数"""
    return f"Hello, {name}"

# 使用
result = greet("World")
print(result)  # <b><i>Hello, World</i></b>
```

---

## 2. 生成器与迭代器

### 2.1 生成器基础

```python
"""
生成器基础示例
@author erik.zhou
"""
def fibonacci(n):
    """斐波那契数列生成器"""
    a, b = 0, 1
    count = 0
    while count < n:
        yield a
        a, b = b, a + b
        count += 1

# 使用
for num in fibonacci(10):
    print(num, end=' ')
# 输出: 0 1 1 2 3 5 8 13 21 34
```

### 2.2 生成器表达式

```python
"""
生成器表达式示例
@author erik.zhou
"""
# 列表推导式（占用内存）
squares_list = [x**2 for x in range(1000000)]

# 生成器表达式（节省内存）
squares_gen = (x**2 for x in range(1000000))

# 使用
for square in squares_gen:
    if square > 100:
        break
    print(square)
```

### 2.3 自定义迭代器

```python
"""
自定义迭代器示例
@author erik.zhou
"""
class Countdown:
    """倒计时迭代器"""
    
    def __init__(self, start):
        self.current = start
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

# 使用
for num in Countdown(5):
    print(num)  # 5, 4, 3, 2, 1
```

### 2.4 yield from

```python
"""
yield from 示例
@author erik.zhou
"""
def chain(*iterables):
    """连接多个可迭代对象"""
    for iterable in iterables:
        yield from iterable

# 使用
result = list(chain([1, 2, 3], ['a', 'b', 'c'], range(4, 7)))
print(result)  # [1, 2, 3, 'a', 'b', 'c', 4, 5, 6]
```

---

## 3. 上下文管理器

### 3.1 使用 with 语句

```python
"""
上下文管理器基础示例
@author erik.zhou
"""
# 文件操作
with open('data.txt', 'w') as f:
    f.write('Hello, World!')
# 文件自动关闭

# 多个上下文管理器
with open('input.txt', 'r') as fin, open('output.txt', 'w') as fout:
    content = fin.read()
    fout.write(content.upper())
```

### 3.2 自定义上下文管理器

```python
"""
自定义上下文管理器示例
@author erik.zhou
"""
class Timer:
    """计时上下文管理器"""
    
    def __enter__(self):
        self.start = time.time()
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.end = time.time()
        self.elapsed = self.end - self.start
        print(f"执行时间: {self.elapsed:.4f}秒")
        return False  # 不抑制异常

# 使用
with Timer():
    time.sleep(1)
    print("执行任务")
```

### 3.3 使用 contextlib

```python
"""
contextlib 模块示例
@author erik.zhou
"""
from contextlib import contextmanager

@contextmanager
def temporary_change(obj, attr, value):
    """临时修改对象属性"""
    original = getattr(obj, attr)
    setattr(obj, attr, value)
    try:
        yield
    finally:
        setattr(obj, attr, original)

# 使用
class Config:
    debug = False

config = Config()
print(config.debug)  # False

with temporary_change(config, 'debug', True):
    print(config.debug)  # True

print(config.debug)  # False
```

---

## 4. 元类与反射

### 4.1 type 动态创建类

```python
"""
动态创建类示例
@author erik.zhou
"""
# 使用 type 创建类
def init(self, name):
    self.name = name

def greet(self):
    return f"Hello, {self.name}"

Person = type('Person', (), {
    '__init__': init,
    'greet': greet
})

# 使用
person = Person("Alice")
print(person.greet())  # Hello, Alice
```

### 4.2 自定义元类

```python
"""
自定义元类示例
@author erik.zhou
"""
class SingletonMeta(type):
    """单例元类"""
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    """数据库连接类（单例）"""
    
    def __init__(self):
        print("初始化数据库连接")

# 使用
db1 = Database()  # 初始化数据库连接
db2 = Database()  # 不会再次初始化
print(db1 is db2)  # True
```

### 4.3 反射操作

```python
"""
反射操作示例
@author erik.zhou
"""
class Person:
    """人员类"""
    
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def greet(self):
        return f"Hello, I'm {self.name}"

person = Person("Alice", 30)

# hasattr - 检查属性是否存在
print(hasattr(person, 'name'))  # True
print(hasattr(person, 'email'))  # False

# getattr - 获取属性值
name = getattr(person, 'name')
email = getattr(person, 'email', 'no-email@example.com')
print(name)  # Alice
print(email)  # no-email@example.com

# setattr - 设置属性值
setattr(person, 'email', 'alice@example.com')
print(person.email)  # alice@example.com

# delattr - 删除属性
delattr(person, 'email')
print(hasattr(person, 'email'))  # False
```

---

## 5. 函数式编程

### 5.1 高阶函数

```python
"""
高阶函数示例
@author erik.zhou
"""
# map
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

# filter
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))
print(even_numbers)  # [2, 4]

# reduce
from functools import reduce
sum_all = reduce(lambda x, y: x + y, numbers)
print(sum_all)  # 15
```

### 5.2 偏函数

```python
"""
偏函数示例
@author erik.zhou
"""
from functools import partial

def power(base, exponent):
    """幂运算"""
    return base ** exponent

# 创建平方函数
square = partial(power, exponent=2)
print(square(5))  # 25

# 创建立方函数
cube = partial(power, exponent=3)
print(cube(5))  # 125
```

### 5.3 闭包

```python
"""
闭包示例
@author erik.zhou
"""
def make_multiplier(factor):
    """创建乘法器"""
    def multiplier(x):
        return x * factor
    return multiplier

# 使用
double = make_multiplier(2)
triple = make_multiplier(3)

print(double(5))  # 10
print(triple(5))  # 15
```

---

## 6. 描述符

### 6.1 描述符协议

```python
"""
描述符协议示例
@author erik.zhou
"""
class Validator:
    """验证描述符"""
    
    def __init__(self, min_value=None, max_value=None):
        self.min_value = min_value
        self.max_value = max_value
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__.get(self.name)
    
    def __set__(self, instance, value):
        if self.min_value is not None and value < self.min_value:
            raise ValueError(f"{self.name} 不能小于 {self.min_value}")
        if self.max_value is not None and value > self.max_value:
            raise ValueError(f"{self.name} 不能大于 {self.max_value}")
        instance.__dict__[self.name] = value

class Person:
    """人员类"""
    age = Validator(min_value=0, max_value=150)
    
    def __init__(self, age):
        self.age = age

# 使用
person = Person(30)
print(person.age)  # 30

try:
    person.age = 200  # 抛出 ValueError
except ValueError as e:
    print(e)
```

### 6.2 property 装饰器

```python
"""
property 装饰器示例
@author erik.zhou
"""
class Circle:
    """圆形类"""
    
    def __init__(self, radius):
        self._radius = radius
    
    @property
    def radius(self):
        """获取半径"""
        return self._radius
    
    @radius.setter
    def radius(self, value):
        """设置半径"""
        if value < 0:
            raise ValueError("半径不能为负数")
        self._radius = value
    
    @property
    def area(self):
        """计算面积"""
        return 3.14159 * self._radius ** 2

# 使用
circle = Circle(5)
print(circle.radius)  # 5
print(circle.area)  # 78.53975

circle.radius = 10
print(circle.area)  # 314.159
```

---

## 7. 高级数据结构

### 7.1 collections 模块

```python
"""
collections 模块示例
@author erik.zhou
"""
from collections import namedtuple, defaultdict, Counter, deque

# namedtuple - 命名元组
Point = namedtuple('Point', ['x', 'y'])
p = Point(10, 20)
print(p.x, p.y)  # 10 20

# defaultdict - 默认字典
word_count = defaultdict(int)
for word in ['apple', 'banana', 'apple']:
    word_count[word] += 1
print(dict(word_count))  # {'apple': 2, 'banana': 1}

# Counter - 计数器
counter = Counter(['apple', 'banana', 'apple', 'orange', 'banana', 'apple'])
print(counter.most_common(2))  # [('apple', 3), ('banana', 2)]

# deque - 双端队列
dq = deque([1, 2, 3])
dq.appendleft(0)
dq.append(4)
print(list(dq))  # [0, 1, 2, 3, 4]
```

### 7.2 heapq 模块

```python
"""
heapq 模块示例
@author erik.zhou
"""
import heapq

# 创建最小堆
heap = []
for num in [5, 2, 8, 1, 9]:
    heapq.heappush(heap, num)

# 弹出最小元素
print(heapq.heappop(heap))  # 1
print(heapq.heappop(heap))  # 2

# 获取最大的 n 个元素
numbers = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
print(heapq.nlargest(3, numbers))  # [42, 37, 23]
print(heapq.nsmallest(3, numbers))  # [-4, 1, 2]
```

---

## 8. 最佳实践

### 8.1 使用 dataclass

```python
"""
dataclass 示例
@author erik.zhou
"""
from dataclasses import dataclass, field
from typing import List

@dataclass
class Product:
    """产品类"""
    name: str
    price: float
    tags: List[str] = field(default_factory=list)
    
    def __post_init__(self):
        """初始化后处理"""
        if self.price < 0:
            raise ValueError("价格不能为负数")

# 使用
product = Product("Laptop", 999.99, ["electronics", "computer"])
print(product)
```

### 8.2 使用 Enum

```python
"""
枚举类型示例
@author erik.zhou
"""
from enum import Enum, auto

class Status(Enum):
    """状态枚举"""
    PENDING = auto()
    PROCESSING = auto()
    COMPLETED = auto()
    FAILED = auto()

# 使用
current_status = Status.PENDING
print(current_status)  # Status.PENDING
print(current_status.name)  # PENDING
print(current_status.value)  # 1
```

### 8.3 使用 typing

```python
"""
类型提示示例
@author erik.zhou
"""
from typing import List, Dict, Optional, Union, Callable

def process_items(
    items: List[str],
    config: Dict[str, int],
    callback: Optional[Callable[[str], None]] = None
) -> Union[List[str], None]:
    """
    处理项目列表
    
    Args:
        items: 项目列表
        config: 配置字典
        callback: 可选的回调函数
    
    Returns:
        处理后的项目列表或 None
    """
    if not items:
        return None
    
    result = []
    for item in items:
        if callback:
            callback(item)
        result.append(item.upper())
    
    return result
```

---

## 📝 学习检查清单

- [ ] 能够编写和使用装饰器
- [ ] 理解生成器和迭代器的区别
- [ ] 能够使用上下文管理器
- [ ] 理解元类的作用
- [ ] 掌握函数式编程技巧
- [ ] 能够使用描述符
- [ ] 熟悉常用的高级数据结构
- [ ] 能够使用 dataclass 和 Enum
- [ ] 掌握类型提示的使用

## 🔗 相关资源

- [Python官方文档 - 装饰器](https://docs.python.org/zh-cn/3/glossary.html#term-decorator)
- [Python官方文档 - 生成器](https://docs.python.org/zh-cn/3/glossary.html#term-generator)
- [Python官方文档 - 上下文管理器](https://docs.python.org/zh-cn/3/library/contextlib.html)
- [Python官方文档 - 元类](https://docs.python.org/zh-cn/3/reference/datamodel.html#metaclasses)
- [Fluent Python](https://www.oreilly.com/library/view/fluent-python/9781491946237/)

---

**@author erik.zhou**  
**最后更新**: 2025-02-12
