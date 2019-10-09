---
title: Python双下方法，特殊方法
date: 2019-10-09 14:18:33
tags: 
- Python 
- magic method
---
在 Python 中，我们可以经常看到以双下划线 __ 包裹起来的方法，这些方法被称为魔法方法（magic method）或特殊方法（special method）。简单地说，这些方法可以给 Python 的类提供特殊功能，方便我们定制一个类
<!-- more -->

#### `__len__`  
内置 `len()`函数实现该方法。该方法返回对象的长度（大于等于0的整数）
```python
class Demo(object):
    def __len__(self): 
        return 666

demo = Demo()
print(len(demo))   # len -- 调用对象的 __len__ 方法

# 只要对象实现了__len__方法，该对象就可以使用len()方法
```

#### `__getitem__`, `__setitem`, `__delitem__`
self[key]调用的就是调用`__getitem__`方法。对于序列类型，key应该为整数或者切片对象。
self[key]赋值调用的是`__setitem__`方法。
del 操作调用`__delitem__`方法

```python
class dictDemo(object):
    def __init__(self):
        self._values = {"a": 1}
        
    def __getitem__(self, key):
        return self._values[key]
        
    def __setitem__(self, key, value):
        self._values[key] = value
    
    def __delitem__(self, key):
        del self._values[key]
        
demo = dictDemo()
print(demo["a"])  # 1  调用__getitem__
demo["b"] = 2     # 调用__setitem__
print(demo["b"])  # 2

del demo["a"]     # 调用__delitem__
print(demo["a"])  # KeyError

```

#### `__getattr__`, `__getattribute__`, `__setattr__`, `__delattr__` 
自定义以上方法来定义类实例属性的使用，赋值，删除。

`__getattr__`: 当一个实例的属性可以正常的访问，`__getattr__`方法将不会调用。换句话说：假设有一个对象x, 它有一个属性`name`, `x.name = "dli"`, 那么在访问x.name时将不会调用`__getattr__`方法， 而是直接返回这个属性。这样做是为了提高效率。

`__getattribute__`: 无条件调用以实现类实例的属性访问。此调用在`__getattr__`之前。如果类还定义了`__getattr__`，则不会调用后者，除非`__getattribute__`显式调用它或引发`attributerror`。为了避免此方法中的无限递归，它的实现始终调用基本方法来访问它的任何属性。eg: `object.__getattribute__(self, item)`

```python
class A(object):
    def __init__(self):
        self.name = "dli"

    def __getattr__(self, item):
        print("__getattr__")
        return self[item]

    def __getattribute__(self, item):
        print("__getattribute__")
        return object.__getattribute__(self, item)

a = A()
print(a.name)  

# output #
# __getattribute__
# dli

```

`__getattribute__`抛出异常

```python
class A(object):
    def __init__(self):
        self.name = "dli"

    def __getattr__(self, item):
        print("__getattr__")
        return 123

    def __getattribute__(self, item):
        print("__getattribute__")
        raise AttributeError

a = A()
print(a.name)

# output #
# __getattribute__
# __getattr__
# 123
```

`__setattr__`: 对实例对象的属性赋值时将会被调用。它就应该用相同的名称调用基类方法`object.__setattr__(self, name, value)`  

`__delattr__`: del obj.name 时将会被调用

