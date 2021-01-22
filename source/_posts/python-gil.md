---
title: python-gil
date: 2021-01-21 17:30:22
tags:
- Python 
---

在 Cpython 中，全局解释器锁 GIL（Global Interpreter Lock）是一个 mutex, 它保证多线程环境下只有一个线程可以执行python字节码。

<!-- more -->

Q：GIL 为什么是 mutex？

A：因为 Cpython 的内存管理不是线程安全的，GIL保证了任何时刻都只有一个线程在运行。

Q：何时释放GIL？

A: 在进行IO操作时，python解析器会释放GIL，但IO操作结束之后，必须获取GIL才能继续运行。在python3.2之前， 基于tick计数器进行GIL，tick计数器表示当前线程在释放gil之前连续执行的多少个字节码，一个已经获取GIL的线程在tick到达100之后释放 GIL，给其它线程获取GIL的机会。python3.2之后，基于时间片释放 GIL, 默认间隔 `DEFAULT_INTERVAL=5000` 0.5秒

Q: 如何解决GIL？

A: C语言扩展，多进程，其它语言编译的解释器（`Jpython`, `LronPython`）。

Q: 为何不移除GIL

A: 早期python开发者利用GIL的特性（即默认python内部对象是thread-safe的，无需在实现时考虑额外的内存锁和同步操作），当大家试图去拆分和去除GIL的时候，发现大量库代码开发者已经重度依赖GIL而非常难以去除了。


### GIL的影响

下面我们就对比下Python在多线程，单线程和多进程下的效率，测试代码如下:

#### 顺序执行
```python
import time
count = 100000000

def foo(c):
    while c > 0:
        c -= 1

t1 = time.time()
foo(count)
print(time.time() - t1)  # 5.546911001205444

```
#### 多线程
```python
import time
import threading

count = 100000000

def foo(c):
    while c > 0:
        c -= 1
        
t1 = time.time()
thread1 = threading.Thread(target=foo, args=(count // 2,))
thread2 = threading.Thread(target=foo, args=(count // 2,))
thread1.start()
thread2.start()
thread1.join()
thread2.join()
print(time.time() - t1)  # 5.651936054229736
```
#### 多进程
```python
import time
import multiprocessing as mp

count = 100000000

def foo(c):
    while c > 0:
        c -= 1

t1 = time.time()
mp1 = mp.Process(target=foo, args=(count // 2,))
mp2 = mp.Process(target=foo, args=(count // 2,))
mp1.start()
mp2.start()
mp1.join()
mp2.join()

print(time.time() - t1)  # 2.8516108989715576
```

可以看到python在多线程的情况下居然比单线程整整慢了。因为CPU有多个核心的时候，从release GIL到acquire GIL之间几乎是没有间隙的，所以当其他在其他核心上的线程被唤醒时，大部分情况下主线程已经又再一次获取到GIL了，这个时候被唤醒执行的线程只能白白的浪费CPU时间，然后达到切换时间后进入待调度状态，再被唤醒，再等待，以此往复恶性循环。

### 结论
1. 多进程可以使用多核CPU
2. 多线程也可以使用多核CPU, 但是会引起thrashing, 多线程效率反而下降

### Reference
- https://github.com/zpoint/CPython-Internals/blob/master/Interpreter/gil/gil_cn.md
- https://wiki.python.org/moin/GlobalInterpreterLock
- https://speakerdeck.com/dabeaz/understanding-the-python-gil?slide=35