---
title: Python装饰器简单记录
date: 2025-02-25
tags:
  - Python
  - 面向对象
categories:
  - Python
description: 装饰器@也许算是Python开发的一个转折点，学了这个才发现Python确实很有意思，在装饰器之前Python是一门简单语言，之后就是一门多变灵活的简洁语言了.
---

#装饰器
# 定义
装饰器是一个 函数（或类），
可以包装另一个函数/类，在不修改原代码情况下增强功能

**几个关键点**
- 通过“装饰”动态扩展对象行为，符合OOP开闭原则（扩展开放，修改关闭）
- Python万物皆对象，万物可传参
- `func`是一个约定俗成的名称，可以更换

## 简单示例
```python
# 定义一个装饰器函数
def logger(func):  # 2
    def wrapper(*args, **kwargs):
        print(f"开始执行函数: {func.__name__}")    # 3
        result = func(*args, **kwargs)  # 4
        print(f"函数执行完毕")  # 6
        return result  # 7
    return wrapper

# 使用装饰器
@logger
def add(a, b):  # 5
    return a + b

print(add(2, 3))  # 1
# 输出：
# 开始执行函数: add
# 函数执行完毕
# 5
```

## 为什么需要装饰器

考虑现有多个写好的函数，现在想进行一个函数运行时间的统计
- 最原始方案：
	挨着在每个函数的头尾添加time并计算
	或者 在调用的位置计算（更麻烦）
原始方案有大面积重复工作，且需要修改封装好的内容；

- 将**函数作为参数**传参到新的函数

```python
import time
def calculate_time(func_name):
	t1 = time.time()
	func_name()
	print(f"Use time: {time.time() - t1}")

def funcA():
	print("this is func A")

def funcB():
	print("this is func B")

calculate_time(funcA())
```
这样可以不用修改原本函数，算是一个装饰器的构想；
但是仍然需要修改调用位置；

而装饰器，就通过特有的语法，不需要修改现有代码（封装好的函数，和调用位置）

## 语法糖 @

使用装饰器`@`，**默认传入参数是被装饰的函数**

```python
import time
def calculate_time(func):
	def wrapper():
		t1 = time.time()
		func()
		print(f"Use time: {time.time() - t1}")
	return wrapper()

@calculate_time
def funcA():
	print("this is func A")

@calculate_time
def funcB():
	print("this is func B")

funcA()
```

当然目前的装饰器函数没有接收传参，实际上大部分时候，需要被装饰的函数，很可能都是有参数的

```python
def calculate_time(func):
	def wrapper(*args, **kwargs):
		t1 = time.time()
		func(*args, **kwargs)
		print(f"Use time: {time.time() - t1}")
	return wrapper()

@calculate_time
def funcC(name):
	print(f"this is func C : {name}")

funcC("test")
```

# 使用

## 类装饰器

```python
class ClassDecorator1:
	def __init__(self, func):
		self.func = func
		print(" 构造函数init ")
	def __call__(self, *args, **kwargs):
		print(" 进入__call__ ")
		t1 = time.time()
		self.func(*args, **kwargs)
		print(f"执行时间{time.time() - t1}")

class ClassDecorator2:
	def __init__(self, arg1, arg2):
		self.arg1 = arg1
		self.arg2 = arg2
	def __call__(self, func):
		print(" 执行call")
		def wrapper(*args, **kwargs):
			t1 = time.time()
			self.func(*args, **kwargs)

		return wrapper

@ClassDecorator1
def funcA():
	pass

@ClassDecorator('hello', 'test')
def funcB():
	pass
```

## 多个装饰器
有里到外执行

```python
@decorator1
@decorator2
@decorator3
def func():
	pass
```

最先执行的`@decorator3`

# 内置装饰器
#装饰器 


| **装饰器**            | **核心用途**  | **典型场景**    |
| ------------------ | --------- | ----------- |
| `@property`        | 属性访问控制    | 数据校验、惰性计算   |
| `@classmethod`     | 类方法       | 工厂方法、替代构造函数 |
| `@staticmethod`    | 静态方法      | 工具函数        |
| `@functools.wraps` | 保留元数据     | 自定义装饰器      |
| `@contextmanager`  | 简化上下文管理器  | 资源管理（文件、锁）  |
| `@atexit.register` | 注册退出函数    | 清理临时资源      |
| `@lru_cache`       | 缓存函数结果    | 计算密集型函数     |
| `@abstractmethod`  | 强制子类实现方法  | 接口定义        |
| `@dataclass`       | 自动生成数据类方法 | 数据容器类       |
| `@cache`           |           |             |

## @property
将类方法转换为属性，控制属性的访问和修改

## @classmethod
定义类方法，方法第一个参数是类本身

```python
class DateUtils:
    @classmethod
    def from_string(cls, date_str): # 通过字符串进行构造
        # 返回一个类实例
        year, month, day = map(int, date_str.split("-"))
        return cls(year, month, day)  # 假设类有 __init__(year, month, day)

    def __init__(self, year, month, day): # 正常构造函数
        self.year = year
        self.month = month
        self.day = day

date = DateUtils.from_string("2023-10-01")
print(date.year)  # 输出 2023
```
可以被子类继承

**优势**：
	普通类方法需要创建实例才能调用，`@classmethod`**无需实例化**即可调用，直接操作类的属性和逻辑；
	Python不支持传统的构造重载，可以通过`@classmethod`实现同样目的 #重载
	构造逻辑也就从`__init__`解耦了，更加简洁

## @staticmethod
定义静态方法，不依赖实例和类，类似普通函数，但是**逻辑上属于类**

```python
class MathUtils:
    @staticmethod
    def add(a, b):
        return a + b

print(MathUtils.add(3, 5))  # 输出 8
```
- **不能访问类的属性**
- 常用于工具函数

### 优势

- 封装在类中，避免命名冲突
- 逻辑上关联的代码方便集中管理
- 无需实例化，直接通过类名调用

## @cache
python3.9新增的装饰器
`from functools import cache`
**缓存函数的所有调用结果，避免重复计算**，非常使用递归和记忆化搜索

在一次运行过程中，缓存@cache所装饰函数的结果
例如计算了`f(13)`，然后在一百次递归后再次用到了`f(13)`，这次就不再计算直接从缓存中读取值

```python
from functools import cache

@cache
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(100))  # 快速计算，缓存所有中间结果
```

但是也会占用大量的缓存