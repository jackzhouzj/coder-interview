# Python 编码规范完整教程

> 基于 PEP 8 和阿里巴巴开发规范
> 
> @author erik.zhou

## 📋 目录

- [技术概述](#技术概述)
- [命名规范](#命名规范)
- [代码格式](#代码格式)
- [类型提示](#类型提示)
- [文档字符串](#文档字符串)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)

## 📚 技术概述

### 版本信息
- **Python 版本**：3.9+
- **PEP 8 版本**：最新版
- **学习难度**：⭐⭐ (2/5)
- **重要程度**：⭐⭐⭐⭐⭐ (5/5)

### 学习目标
- [ ] 掌握 PEP 8 编码规范
- [ ] 理解命名规范和代码风格
- [ ] 掌握类型提示的使用
- [ ] 能够编写规范的文档字符串
- [ ] 养成良好的编码习惯

## 🎯 命名规范

### 1. 模块和包命名

**规则**：全小写，使用下划线分隔

```python
# ✅ 正确示例
import user_service
from data_processing import clean_data

# ❌ 错误示例
import UserService  # 不要使用大写
import user-service  # 不要使用连字符
```

### 2. 类命名

**规则**：PascalCase（大驼峰命名法）

```python
# ✅ 正确示例
class UserService:
    pass

class OrderManager:
    pass

class HTTPClient:  # 缩写词全大写
    pass

# 异常类以 Exception 或 Error 结尾
class ValidationException(Exception):
    pass

class DatabaseError(Exception):
    pass

# ❌ 错误示例
class user_service:  # 不要使用下划线
    pass

class orderManager:  # 不要使用小驼峰
    pass
```

### 3. 函数和方法命名

**规则**：snake_case（小写+下划线）

```python
# ✅ 正确示例
def get_user_by_id(user_id: int) -> dict:
    """根据用户ID获取用户信息"""
    pass

def calculate_total_price(items: list) -> float:
    """计算订单总价"""
    pass

# 私有方法以单下划线开头
def _validate_input(data: dict) -> bool:
    """验证输入数据（内部使用）"""
    pass

# ❌ 错误示例
def GetUserById(userId):  # 不要使用大驼峰
    pass

def calculateTotalPrice(items):  # 不要使用小驼峰
    pass
```

### 4. 变量命名

**规则**：snake_case（小写+下划线）

```python
# ✅ 正确示例
user_name = "张三"
order_id = 12345
total_count = 100

# 布尔变量使用 is_、has_、can_ 等前缀
is_active = True
has_permission = False
can_edit = True

# ❌ 错误示例
userName = "张三"  # 不要使用小驼峰
OrderID = 12345  # 不要使用大驼峰
totalCount = 100
```

### 5. 常量命名

**规则**：全大写+下划线分隔

```python
# ✅ 正确示例
MAX_RETRY_COUNT = 3
DEFAULT_TIMEOUT = 30
API_BASE_URL = "https://api.example.com"

# 常量应该集中定义在模块顶部或配置文件中
class Config:
    DATABASE_HOST = "localhost"
    DATABASE_PORT = 5432
    MAX_CONNECTIONS = 100

# ❌ 错误示例
max_retry_count = 3  # 常量应该全大写
MaxRetryCount = 3  # 不要使用大驼峰
```

### 6. 私有属性和方法

**规则**：单下划线开头（约定私有），双下划线开头（名称改写）

```python
class UserService:
    def __init__(self):
        self.public_attr = "公开属性"
        self._protected_attr = "受保护属性"  # 约定私有
        self.__private_attr = "私有属性"  # 名称改写
    
    def public_method(self):
        """公开方法"""
        pass
    
    def _protected_method(self):
        """受保护方法（约定私有）"""
        pass
    
    def __private_method(self):
        """私有方法（名称改写）"""
        pass
```

## 📐 代码格式

### 1. 缩进

**规则**：使用 4 个空格，禁止使用 Tab

```python
# ✅ 正确示例
def calculate_sum(numbers: list) -> int:
    total = 0
    for num in numbers:
        if num > 0:
            total += num
    return total

# ❌ 错误示例
def calculate_sum(numbers):
  total = 0  # 2个空格，不符合规范
	for num in numbers:  # Tab缩进，不符合规范
		total += num
  return total
```

### 2. 行长度

**规则**：每行不超过 79 字符（文档字符串不超过 72 字符）

```python
# ✅ 正确示例 - 使用括号换行
result = some_function(
    arg1, arg2, arg3,
    arg4, arg5
)

# 使用反斜杠换行（不推荐，优先使用括号）
total = first_variable + second_variable + \
        third_variable + fourth_variable

# ❌ 错误示例 - 超长行
result = some_function(arg1, arg2, arg3, arg4, arg5, arg6, arg7, arg8, arg9, arg10)
```

### 3. 空行

**规则**：
- 顶层函数和类定义之间空 2 行
- 类内方法之间空 1 行
- 函数内逻辑块之间空 1 行

```python
# ✅ 正确示例
import os
import sys


def top_level_function():
    """顶层函数"""
    pass


class MyClass:
    """类定义"""
    
    def __init__(self):
        self.value = 0
    
    def method_one(self):
        """方法一"""
        # 逻辑块1
        data = self._fetch_data()
        
        # 逻辑块2
        result = self._process_data(data)
        
        return result
    
    def method_two(self):
        """方法二"""
        pass


class AnotherClass:
    """另一个类"""
    pass
```

### 4. 导入语句

**规则**：
- 每个导入独占一行
- 按标准库、第三方库、本地模块分组
- 每组之间空一行

```python
# ✅ 正确示例
# 标准库
import os
import sys
from typing import List, Dict, Optional

# 第三方库
import numpy as np
import pandas as pd
from fastapi import FastAPI, HTTPException

# 本地模块
from app.models import User
from app.services import UserService
from app.utils import validate_email

# ❌ 错误示例
import os, sys  # 不要在一行导入多个模块
from typing import *  # 不要使用 * 导入
import pandas as pd, numpy as np  # 不要在一行导入多个模块
```

### 5. 空格使用

**规则**：
- 运算符两侧各一个空格
- 逗号后面一个空格
- 函数参数等号两侧不加空格

```python
# ✅ 正确示例
# 运算符
x = 1 + 2
y = x * 3
result = (x + y) / 2

# 列表、字典
numbers = [1, 2, 3, 4, 5]
user = {"name": "张三", "age": 30}

# 函数调用
result = calculate_sum(1, 2, 3)

# 函数定义（默认参数）
def greet(name: str, greeting: str = "你好") -> str:
    return f"{greeting}, {name}"

# ❌ 错误示例
x=1+2  # 运算符两侧缺少空格
numbers=[1,2,3,4,5]  # 逗号后缺少空格
result=calculate_sum(1,2,3)  # 等号两侧缺少空格
def greet(name: str, greeting: str="你好"):  # 默认参数等号两侧不应有空格
    pass
```

## 🏷️ 类型提示

### 1. 基础类型提示

```python
from typing import List, Dict, Tuple, Set, Optional, Union

# 基础类型
def greet(name: str) -> str:
    return f"你好, {name}"

def add(a: int, b: int) -> int:
    return a + b

def calculate_average(numbers: List[float]) -> float:
    return sum(numbers) / len(numbers)

# 可选类型
def find_user(user_id: int) -> Optional[Dict[str, str]]:
    """返回用户信息，如果不存在返回 None"""
    if user_id > 0:
        return {"name": "张三", "email": "zhangsan@example.com"}
    return None

# 联合类型
def process_data(data: Union[str, int, float]) -> str:
    """处理多种类型的数据"""
    return str(data)
```

### 2. 复杂类型提示

```python
from typing import List, Dict, Tuple, Callable, Any, TypeVar, Generic

# 字典类型
def get_user_scores() -> Dict[str, int]:
    return {"张三": 95, "李四": 88}

# 元组类型
def get_coordinates() -> Tuple[float, float]:
    return (39.9042, 116.4074)

# 可调用类型
def apply_function(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)

# 泛型
T = TypeVar('T')

def get_first_element(items: List[T]) -> Optional[T]:
    """获取列表第一个元素"""
    return items[0] if items else None

# 类型别名
UserId = int
UserData = Dict[str, Any]

def get_user(user_id: UserId) -> UserData:
    return {"id": user_id, "name": "张三"}
```

### 3. 类的类型提示

```python
from typing import List, Optional
from dataclasses import dataclass

@dataclass
class User:
    """用户数据类"""
    id: int
    name: str
    email: str
    age: Optional[int] = None
    tags: List[str] = None
    
    def __post_init__(self):
        if self.tags is None:
            self.tags = []

class UserService:
    """用户服务类"""
    
    def __init__(self, db_connection: Any):
        self.db = db_connection
        self._cache: Dict[int, User] = {}
    
    def get_user(self, user_id: int) -> Optional[User]:
        """获取用户信息"""
        if user_id in self._cache:
            return self._cache[user_id]
        
        # 从数据库获取
        user_data = self.db.query(user_id)
        if user_data:
            user = User(**user_data)
            self._cache[user_id] = user
            return user
        return None
    
    def get_all_users(self) -> List[User]:
        """获取所有用户"""
        return list(self._cache.values())
```

## 📝 文档字符串

### 1. 模块文档字符串

```python
"""
用户服务模块

本模块提供用户管理相关的功能，包括：
- 用户信息查询
- 用户创建和更新
- 用户权限管理

示例:
    from app.services import UserService
    
    service = UserService()
    user = service.get_user(123)

@author erik.zhou
"""

import logging
from typing import Optional, List
```

### 2. 类文档字符串

```python
class UserService:
    """
    用户服务类
    
    提供用户管理的核心功能，包括用户的增删改查操作。
    
    Attributes:
        db: 数据库连接对象
        cache: 用户缓存字典
        logger: 日志记录器
    
    Example:
        >>> service = UserService(db_connection)
        >>> user = service.get_user(123)
        >>> print(user.name)
        '张三'
    """
    
    def __init__(self, db_connection):
        """
        初始化用户服务
        
        Args:
            db_connection: 数据库连接对象
        """
        self.db = db_connection
        self.cache = {}
        self.logger = logging.getLogger(__name__)
```

### 3. 函数文档字符串（Google 风格）

```python
def calculate_discount(
    price: float,
    discount_rate: float,
    min_price: float = 0.0
) -> float:
    """
    计算折扣后的价格
    
    根据原价和折扣率计算折扣后的价格，并确保不低于最低价格。
    
    Args:
        price: 商品原价，必须大于0
        discount_rate: 折扣率，范围0-1之间
        min_price: 最低价格，默认为0
    
    Returns:
        折扣后的价格
    
    Raises:
        ValueError: 当价格小于等于0或折扣率不在0-1之间时
    
    Example:
        >>> calculate_discount(100.0, 0.2)
        80.0
        >>> calculate_discount(100.0, 0.5, min_price=60.0)
        60.0
    """
    if price <= 0:
        raise ValueError("价格必须大于0")
    if not 0 <= discount_rate <= 1:
        raise ValueError("折扣率必须在0-1之间")
    
    discounted_price = price * (1 - discount_rate)
    return max(discounted_price, min_price)
```

### 4. 函数文档字符串（NumPy 风格）

```python
def process_user_data(
    users: List[Dict],
    filter_active: bool = True,
    sort_by: str = "name"
) -> List[Dict]:
    """
    处理用户数据
    
    对用户列表进行过滤和排序处理。
    
    Parameters
    ----------
    users : List[Dict]
        用户数据列表，每个用户是一个字典
    filter_active : bool, optional
        是否只保留活跃用户，默认为 True
    sort_by : str, optional
        排序字段，可选值：'name', 'age', 'created_at'
        默认为 'name'
    
    Returns
    -------
    List[Dict]
        处理后的用户列表
    
    Raises
    ------
    ValueError
        当 sort_by 参数不是有效字段时
    
    Examples
    --------
    >>> users = [
    ...     {"name": "张三", "age": 30, "active": True},
    ...     {"name": "李四", "age": 25, "active": False}
    ... ]
    >>> result = process_user_data(users, filter_active=True)
    >>> len(result)
    1
    """
    valid_sort_fields = ["name", "age", "created_at"]
    if sort_by not in valid_sort_fields:
        raise ValueError(f"sort_by 必须是以下之一: {valid_sort_fields}")
    
    # 过滤
    if filter_active:
        users = [u for u in users if u.get("active", False)]
    
    # 排序
    users.sort(key=lambda x: x.get(sort_by, ""))
    
    return users
```

## ✨ 最佳实践

### 1. 字符串比较

```python
# ✅ 正确示例 - 使用 equals 方法
name = "admin"
if name == "admin":  # 对于字面量可以使用 ==
    print("管理员")

# 对于变量比较，推荐使用这种方式避免 None 问题
if "admin" == name:  # 字面量在前，避免 name 为 None 时报错
    print("管理员")

# ❌ 错误示例 - 使用 is 比较字符串
if name is "admin":  # 不要使用 is 比较字符串
    print("管理员")
```

### 2. 集合初始化指定容量

```python
# ✅ 正确示例 - 预估容量
users = []
users_dict = {}

# 如果知道大概数量，可以预分配
if expected_count > 0:
    users = [None] * expected_count

# ❌ 错误示例 - Python 的 list 和 dict 会自动扩容，无需像 Java 那样指定初始容量
# 但要注意避免在循环中频繁创建新对象
```

### 3. 避免循环内创建对象

```python
# ✅ 正确示例 - 对象在循环外创建
result = []
temp_dict = {}

for i in range(1000):
    temp_dict.clear()  # 复用对象
    temp_dict["index"] = i
    result.append(temp_dict.copy())

# ❌ 错误示例 - 循环内创建对象
result = []
for i in range(1000):
    temp_dict = {}  # 每次循环都创建新对象
    temp_dict["index"] = i
    result.append(temp_dict)
```

### 4. 异常处理

```python
import logging

logger = logging.getLogger(__name__)

# ✅ 正确示例 - 明确捕获异常并记录日志
def process_data(data: dict) -> dict:
    """处理数据"""
    try:
        result = expensive_operation(data)
        return result
    except ValueError as e:
        logger.error(f"数据验证失败: {e}", exc_info=True)
        return {"error": "数据格式错误"}
    except Exception as e:
        logger.error(f"处理数据时发生未知错误: {e}", exc_info=True)
        raise

# ❌ 错误示例 - 空 catch 块
def process_data_bad(data):
    try:
        result = expensive_operation(data)
        return result
    except Exception:  # 捕获所有异常但不处理
        pass  # 不要使用空 catch
```

### 5. 使用 @override 装饰器（Python 3.12+）

```python
from typing import override

class Animal:
    def make_sound(self) -> str:
        return "Some sound"

class Dog(Animal):
    @override  # Python 3.12+ 支持
    def make_sound(self) -> str:
        """重写父类方法"""
        return "Woof!"
    
# 对于 Python 3.11 及以下版本，可以使用注释
class Cat(Animal):
    def make_sound(self) -> str:  # type: ignore[override]
        """重写父类方法"""
        return "Meow!"
```

## 📋 代码审查清单

### 命名规范检查
- [ ] 类名使用 PascalCase
- [ ] 函数/方法名使用 snake_case
- [ ] 常量使用全大写+下划线
- [ ] 私有属性/方法以下划线开头
- [ ] 布尔变量使用 is_/has_/can_ 前缀

### 代码格式检查
- [ ] 使用 4 个空格缩进
- [ ] 每行不超过 79 字符
- [ ] 顶层函数/类之间空 2 行
- [ ] 类内方法之间空 1 行
- [ ] 导入语句按标准库、第三方库、本地模块分组

### 类型提示检查
- [ ] 函数参数添加类型提示
- [ ] 函数返回值添加类型提示
- [ ] 类属性添加类型提示
- [ ] 使用 Optional 表示可选类型

### 文档字符串检查
- [ ] 模块有文档字符串
- [ ] 类有文档字符串
- [ ] 公共函数/方法有文档字符串
- [ ] 文档字符串包含参数说明
- [ ] 文档字符串包含返回值说明
- [ ] 文档字符串包含异常说明

### 最佳实践检查
- [ ] 字符串比较使用 ==，不使用 is
- [ ] 异常处理不使用空 catch
- [ ] 避免循环内创建对象
- [ ] 使用 logging 记录日志
- [ ] 敏感信息不打印到日志

## ❓ 常见问题

### Q1: 为什么要遵循编码规范？
A: 编码规范的好处：
- 提高代码可读性和可维护性
- 减少代码审查时间
- 降低 bug 率
- 便于团队协作
- 符合行业标准

### Q2: 如何自动检查代码规范？
A: 使用以下工具：
- Black：自动格式化代码
- Flake8：检查代码风格
- mypy：检查类型提示
- pylint：代码质量分析
- pre-commit：提交前自动检查

### Q3: 类型提示是必须的吗？
A: 不是必须的，但强烈推荐：
- 提高代码可读性
- IDE 可以提供更好的代码补全
- 可以使用 mypy 进行静态类型检查
- 便于代码维护和重构

### Q4: 如何处理超长行？
A: 推荐方法：
1. 使用括号换行（推荐）
2. 使用反斜杠换行
3. 将长表达式拆分成多个变量
4. 重构代码，简化逻辑

### Q5: 私有方法一定要用下划线开头吗？
A: 这是 Python 的约定：
- 单下划线：约定私有（不强制）
- 双下划线：名称改写（强制）
- 遵循约定可以提高代码可读性

## 🔗 相关资源

- [PEP 8 官方文档](https://pep8.org/)
- [Google Python 风格指南](https://google.github.io/styleguide/pyguide.html)
- [Python 类型提示](https://docs.python.org/zh-cn/3/library/typing.html)
- [Real Python - Code Style](https://realpython.com/python-pep8/)

---

**记住：良好的编码规范是成为资深工程师的第一步！** 💪

@author erik.zhou
