---
title: Python魔术方法，特殊方法
date: 2019-10-09 14:18:33
tags: 
- Python 
- magic method
---
在 Python 中，我们可以经常看到以双下划线 __ 包裹起来的方法，这些方法被称为魔法方法（magic method）或特殊方法（special method）。

简单地说，这些方法可以给 Python 的类提供特殊功能，方便我们定制一个类。

这篇文章主要是总结了在我们开发中，经常遇到的那些“魔法”方法，如何使用以及它们的使用场景。
<!-- more -->

# 基本定制

## `__new__`
构造方法，通过调用该方法创建一个`cls`的实例。`__new__`是一个静态方法（特例，不用显示声明），它会将所属请求的类作为第一个参数。

- 如果`__new__()`返回`cls`实例， 则新实例的`__init__`方法会执行
- 如果`__new__()`没有返回`cls`实例，则`__init__`方法不会执行。

```python
class Person(object):

    def __init__(self, name):
        print("__init__")
        self.name = name

    def __new__(cls, *args, **kwargs):
        print("__new__")

p = Person("dli")  # __new__ 
# output
# __new__
```

```python
class Person(object):

    def __init__(self, name):
        print("__init__")
        self.name = name

    def __new__(cls, *args, **kwargs):
        print("__new__")
        return super().__new__(cls)

p = Person("dli")  # __new__ -> __init__
# output
# __new__
# __init__
```

`__new__()` 的目的主要是允许不可变类型的子类 (例如 int, str 或 tuple) 定制实例创建过程。它也常会在自定义[元类](https://stackoverflow.com/questions/100003/what-are-metaclasses-in-python)中被重载以便定制类创建过程。

## `__init__`
初始化一个类实例。
```python
class Person(object):

    def __init__(self, name):
        print("__init__")
        self.name = name

p = Person("dli")  # __init__  

```

## `__del__`
`__del__()` 方法可以 (但不推荐!) 通过创建一个该实例的新引用来推迟其销毁。这被称为对象重生。

`del x` 并不直接调用 `x.__del__()` --- 前者会将 `x`的引用计数减一，而后者仅会在 `x` 的引用计数变为零时被调用。

## `__str__`/`__repr__`

- `__str__`强调可读性。str(), print()都调用该方法。
- `__repr__`强调准确性。内置类型 `object` 所定义的默认实现会调用 `object.__repr__()`。

```python
class A(object):

    def __init__(self):
        self.name = "dli"

    def __str__(self):
        print("__str__")
        return self.name

    def __repr__(self):
        print("__repr__")
        return repr(self.name)

a = A()
print(a)         # __str__
print("%r" % a)  # __repr__

```

如果一个类定义了 `__repr__()` 但未定义 `__str__()`，则在需要该类实例的“非正式”字符串表示时也会使用 `__repr__()`。

## `__hash__`
内置函数`hash()`。哈希集：`set`，`forzenset`，`dict`。

>If a class does not define an __eq__() method it should not define a __hash__() operation either; if it defines __eq__() but not __hash__(), its instances will not be usable as items in hashable collections. If a class defines mutable objects and implements an __eq__() method, it should not implement __hash__(), since the implementation of hashable collections requires that a key’s hash value is immutable (if the object’s hash value changes, it will be in the wrong hash bucket).


用户定义的类默认带有 `__eq__()` 和 `__hash__()` 方法
一个类如果重载了 `__eq__()` 且没有定义 `__hash__()` 则会将其 `__hash__()` 隐式地设为 `None`。当一个类的 `__hash__()` 方法为 `None` 时，该类的实例将在一个程序尝试获取其哈希值时正确地引发 `TypeError`，并会在检测 `isinstance(obj, collections.abc.Hashable)` 时被正确地识别为不可哈希对象。

## `__bool__`
内置的 `bool()` 函数，调用此方法以实现真值检测，应该返回 `False` 或 `True`。如果未定义此方法，则会查找并调用 `__len__()` 并在其返回非零值时视对象的逻辑值为真。如果一个类既未定义 `__len__()` 也未定义 `__bool__()` 则视其所有实例的逻辑值为真。

# 自定义属性访问
自定义以下方法来定义类实例属性的使用，赋值，删除。

## `__getattr__`

当一个实例的属性可以正常的访问，`__getattr__`方法将不会调用。换句话说：假设有一个对象x, 它有一个属性`name`, `x.name = "dli"`, 那么在访问x.name时将不会调用`__getattr__`方法， 而是直接返回这个属性。这样做是为了提高效率。

## `__getattribute__`
无条件调用以实现类实例的属性访问。此调用在`__getattr__`之前。

如果类还定义了`__getattr__`，则不会调用后者，除非`__getattribute__`显式调用它或引发`attributerror`。

为了避免此方法中的无限递归，它的实现始终调用基本方法来访问它的任何属性。eg: `object.__getattribute__(self, item)`

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

我们可以改写`__getattribute__`实现查看权限， 打印log日志等

## `__setattr__`
对实例对象的属性赋值时将会被调用。它应该用相同的名称调用基类方法`object.__setattr__(self, name, value)`  

## `__delattr__`
`del obj.name`。删除某对象的属性时，该方法将会被调用

# 实现描述器Descriptors
只有当包含该方法的类的实例(instance)（所谓的描述符类）出现在所有者（ower）类中（描述符必须位于所有者类字典中，或者位于其父类之一的类字典中）时，才应用以下方法。

- `__get__(self, instance, ower)`
- `__set__(self, instance, value)`
- `__delete__(self, instance)`

```python
class Age(object):
    """描述器"""
    def __init__(self, age=18):
        self.age = age

    def __get__(self, instance, owner):
        print(type(self), type(instance), type(owner))
        return self.age

    def __set__(self, instance, value):
        self.age = value

    def __delete__(self, instance):
        pass


class Person(object):
    age = Age()


p = Person()
print(p.age)
p.age = 10
print(p.age)

print(Person.age)
# output
# <class '__main__.Age'> <class '__main__.Person'> <class 'type'>
# 18
# <class '__main__.Age'> <class '__main__.Person'> <class 'type'>
# 10
# <class '__main__.Age'> <class 'NoneType'> <class 'type'>
# 10
```
描述器发起调用的开始点是一个绑定 `p.age`。参数的组合方式依 `p` 而定:

- p是一个实例对象。`p.age` 将调用 `type(p).__dict__["age].__get__(p, type(p))`
- p是一个类。`p.age`将调用 `type(p).__dict__["age].__get__(None, type(p))`


# 容器类操作
容器通常属于序列（如列表或元组）或映射（如字典），但也存在其他形式的容器。  
`Containers usually are sequences (such as lists or tuples) or mappings (like dictionaries), but can represent other containers as well. `

## `__len__`  
内置 `len()`函数实现该方法。该方法返回对象的长度（大于等于0的整数）
```python
class Demo(object):
    def __len__(self): 
        return 666

demo = Demo()
print(len(demo))   # len -- 调用对象的 __len__ 方法

# 只要对象实现了__len__方法，该对象就可以使用len()方法
```

## `__getitem__`
self[key]调用的就是调用`__getitem__`方法。对于序列类型，key应该为整数或者切片对象。

## `__setitem`
self[key]赋值调用的是`__setitem__`方法。

## `__delitem__`
del self[key]操作调用`__delitem__`方法

## `__missing__`
当使用`__getitem__`来访问一个不存在的`key`的时候，会调用`__miss__`方法获取默认值，并将该值添加到字典中去

Python中的`defaultdict`给所有的key赋一个默认的value就是通过该方法实现的。

## `__iter__`
此方法在需要为容器创建迭代器时被调用。此方法应该返回一个新的迭代器对象，它能够逐个迭代容器中的所有对象。

对于映射(dict)，它应该逐个迭代容器中的键。

迭代器对象也需要实现此方法；它们需要返回对象自身

## `__contains__`
`item in obj` 该方法将被执行，用来判断某个元素是否存在于容器中。

如果 `item` 是 `obj` 的成员则应返回真，否则返回假。对于映射类型，此检测应基于映射的键而不是值或者键值对。

```python
class dictDemo(object):
    def __init__(self, defaul=None):
        self._values = {"a": 1}
        self.defaul = defaul

    def __getitem__(self, key):
        try:
            return self._values[key]
        except KeyError:
            return self.__missing__(key)

    def __setitem__(self, key, value):
        self._values[key] = value

    def __delitem__(self, key):
        del self._values[key]

    def __missing__(self, key):
        self._values[key] = value = self.defaul
        return value

    def __iter__(self):
        return iter(self._values)

    def __next__(self):
        # 如果__iter__返回self，则必须实现此方法
        pass

    def __contains__(self, item):
        return item in self._values

my_dict = dictDemo()
print(my_dict["a"])  # __getitem__
my_dict["b"] = 2     # __setitem__
print(my_dict["b"])  #

print(my_dict["c"])  # __getitem__  捕获异常，调用__missing__

del my_dict["a"]     # __delitem__
print(my_dict["a"])  # None 没实现__missing__就会抛出KeyError

for i in my_dict:    # __iter__
    print(i)


print("c" in my_dict)  # __contains__


```

