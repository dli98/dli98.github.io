---
title: strategy pattern
date: 2019-10-11 10:05:18
tags: design-patterns
---

策略模式是一种比较简单的模式，也叫做政策模式（Policy Pattern）。

定义: `Define a family of algorithms, encapsulate each one, and make them interchangeable`.定义一系列算法，封装每一个算法，并使他们可以互换。

策略使算法独立于使用该算法的客户端而变化。

<!-- more -->

# python 实例讲解

- 背景：我们有一件商品
- 冲突：如果不同的节日都是相同促销活动，用户没太多吸引力
- 解决方法：我们需要针对不同的节目使用不同的促销活动。

```python
class Order:
    def __init__(self, price, discount_strategy=None):
        self.price = price
        self.discount_strategy = discount_strategy

    def price_after_discount(self):
        if self.discount_strategy:
            discount = self.discount_strategy(self)
        else:
            discount = 0
        return self.price - discount

    def __repr__(self):
        fmt = "<Price: {}, price after discount: {}>"
        return fmt.format(self.price, self.price_after_discount())


def ten_percent_discount(order):
    return order.price * 0.10


def on_sale_discount(order):
    return order.price * 0.25 + 20


if __name__ == '__main__':
    print(Order(100)) 
    # <Price: 100, price after discount: 100>
    print(Order(100, discount_strategy=ten_percent_discount)) 
    # <Price: 100, price after discount: 90.0>
    print(Order(100, discount_strategy=on_sale_discount))  
    # <Price: 100, price after discount: 55.0>

```

# 策略模式优缺点
## 优点

- 策略类之间可以自由切换。
- 易于扩展。符合"开闭原则"（OCP）
- 避免多重条件判断。

## 缺点

- 策略数量增多。每一个策略都是一个类，复用的可能性小。通过享元模式一定程度上减少对象的数量。
- 所有策略客户端都必须知道，并自行决定使用哪一个策略类。