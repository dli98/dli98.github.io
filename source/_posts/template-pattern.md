---
title: template pattern
date: 2019-10-10 19:30:07
tags: design-patterns
---

定义一个操作中算法的骨架，而将一些步骤延迟到子类中，模板方法使得子类可以不改变算法的结构即可重定义该算法的某些特定步骤。

通俗点的理解就是 ：完成一件事情，有固定的数个步骤，但是每个步骤根据对象的不同，而实现细节不同；就可以在父类中定义一个完成该事情的总方法，按照完成事件需要的步骤去调用其每个步骤的实现方法。每个步骤的具体实现，由子类完成。

<!-- more -->

父类定义模板方法，子类实现具体细节。

# Python实例说明

```python
class Download(object):

    def process_item(self):
        raise NotImplementedError

    def download(self):
        self.process_item()


class CsvDownload(Download):

    def process_item(self):
        print("Got csv data")


class PdfDownload(Download):

    def process_item(self):
        print("Got pdf data")


downloader = CsvDownload()
downloader.download()

downloader = PdfDownload()
downloader.download()

```

## 模板模式的优点
- 封装不变部分，扩展可变部分
- 提取公共代码，便于维护
- 通过父类调用子类的操作，通过子类扩展新的行为。符合开闭原则（对扩展开放，对修改封闭）

## 不足
- 复杂的项目会增加阅读难度。
- 大量的继承会产生大量冗余代码，也加重了系统负担
