# 🧩 Python 类（Class）超详细教程

## 1. 🔍 类是什么？

简单说：
 **类是对象的模板，对象是类的实例。**

```python
class Dog:
    pass

d = Dog()   # d 是一个对象（实例）
```

类定义了对象“应该长什么样”、有什么属性和方法。

------

## 2. 📝 如何定义类

```python
class Person:
    def __init__(self, name, age):
        self.name = name    # 实例属性
        self.age = age

    def say_hello(self):
        print(f"Hello, I'm {self.name}")
```

使用：

```python
p = Person("Tom", 18)
p.say_hello()
```

------

## 3. 🚪 构造函数 `__init__()` 完全说明

`__init__()` 在创建对象时自动调用，用来初始化属性。

```python
class Car:
    def __init__(self, brand, price=100):
        self.brand = brand
        self.price = price
```

注意：

- `self` 必须是第一个参数
- 构造函数**不能**有返回值（不能写 `return`）

------

## 4. 🎒 实例属性 vs 类属性

### ✔ 实例属性：属于对象

每个对象都有自己独立的一份：

```python
class A:
    def __init__(self):
        self.x = 1
```

### ✔ 类属性：属于类（所有实例共享）

```python
class A:
    count = 0   # 类属性

    def __init__(self):
        A.count += 1
```

类属性读取方式：

```python
print(A.count)     # 推荐
print(obj.count)   # 也可以，但不推荐
```

------

## 5. 🔧 实例方法、类方法、静态方法

### ✔ 实例方法（最常见）

第一个参数必须是 `self`：

```python
def foo(self):
    pass
```

### ✔ 类方法（@classmethod）

第一个参数是类本身（cls）：

```python
class A:
    total = 0

    @classmethod
    def show_total(cls):
        print(cls.total)
```

用途：

- 操作类属性
- 工厂方法（另一种创建对象的方式）

### ✔ 静态方法（@staticmethod）

不自动传入 `self` 或 `cls`：

```python
class Math:
    @staticmethod
    def add(a, b):
        return a + b
```

用途：

- 工具函数（逻辑相关但不需要类或实例参与）

------

## 6. 🛡 封装：私有变量（伪私有）

Python 没有真正私有，但以 `__` 开头的属性会变“名称改写”（Name Mangling）：

```python
class A:
    def __init__(self):
        self.__secret = 123
```

访问：

```python
a._A__secret
```

不建议频繁使用双下划线，一般用单下划线 `_name` 表示“内部使用”。

------

## 7. 🧬 继承与多态

### ✔ 继承

```python
class Animal:
    def eat(self):
        print("eat")

class Dog(Animal):
    def bark(self):
        print("wang")
```

### ✔ 方法重写（Override）

```python
class Dog(Animal):
    def eat(self):
        print("Dog eating")
```

### ✔ super() 使用

```python
class Child(Parent):
    def __init__(self):
        super().__init__()
```

------

## 8. ✨ 魔术方法（Magic Methods）

最常用的：

| 方法                     | 用途                 |
| ------------------------ | -------------------- |
| `__str__`                | 可读字符串           |
| `__repr__`               | 调试字符串           |
| `__len__`                | 让对象可使用 `len()` |
| `__getitem__`            | 索引访问             |
| `__iter__`               | 可迭代               |
| `__enter__` / `__exit__` | 上下文管理器（with） |

例子：

```python
class MyList:
    def __init__(self, items):
        self.items = items

    def __len__(self):
        return len(self.items)
```

------

## 9. 🧩 property 属性控制

允许把方法当作属性访问：

```python
class Person:
    def __init__(self, age):
        self._age = age

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, value):
        if value < 0:
            raise ValueError("negative age")
        self._age = value
```

使用：

```python
p.age = 10
print(p.age)
```

------

## 10. 🧭 最佳实践 & 常见坑点

#### ✔ 最佳实践

- 属性不要随便公开，内部用 `_name` 表示
- 多用 `property` 做数据校验
- 父类使用 `super()` 调用
- 使用类方法实现工厂模式
- 使用 `__repr__` 让调试更容易

### ❌ 坑点

- **类属性是共享的**
   如果是可变对象（list, dict）要特别注意
- 构造函数不能使用 `return`
- 静态方法无法访问 `self` 或 `cls`

# 