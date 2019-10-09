---
title: borg pattern
date: 2019-10-08 20:25:50
tags: design-patterns
---


## borg pattern
borg模式实现了单例行为，与单例模式有所不同，它在一个类中不是只有一个实例，
而是有多个实例共享同一状态。换一句话说，borg模式的重点是共享状态而不是共享实例身份。


### python `__dict__`
在理解此模式在Python中怎么样实现之前，我们应该知道：  
`Object.__dict__ `  用于存储对象（可写）属性的字典或其他映射对象。  
`A dictionary or other mapping object used to store an object’s (writable) attributes.`

在Python中，每一个实例都有一个它自己的字典，但是borg模式修改了它，使得所有实例都共享一个相同的字典

### Python实现
```python
class Borg(object):
    __shared_state = {}
    
    def __init__(self):
        self.__dict__ = self.__shared_state
        self.state = "init"
        
    def __str__(self):
        return self.state
        
        
if __name__ == '__main__':
    rm1 = Borg()
    rm2 = Borg()
    
    rm1.state = 'Idle'
    rm2.state = 'Running'
    
    print('rm1: {0}'.format(rm1))
    print('rm2: {0}'.format(rm2))
    
    rm2.state = 'Zombie'
    
    print('rm1: {0}'.format(rm1))
    print('rm2: {0}'.format(rm2))
    
    print('rm1 id: {0}'.format(id(rm1)))
    print('rm2 id: {0}'.format(id(rm2)))
    
    
### OUTPUT ###
# rm1: Running
# rm2: Running
# rm1: Zombie
# rm2: Zombie
# rm1 id: 140732837899224
# rm2 id: 140732837899296
```

`__shared_state`属性是所有实例之间共享的字典。实例的属性通常被添加到实例的属性字典中（`__dict__`），我们在初始化实例
即在`__init__`方法中，使实例的`__dict__`属性字典为`__shared_state`共享字典。这样就确保属性字典本身是共享的,
因此所有其它属性也将被共享。


##### borg故事：
Borg这个名字来源于 Star trek 剧集。Borg 是一个邪恶种族，所有的 Borg 成员，都随时听从一个中央命令机构，叫做 Collective (集体). 
这个集体, 随时能够和任何一个 Borg 成员通信，在 Borg 成员里共享一切信息.

##### 注：
代码来源 [代码](https://github.com/faif/python-patterns)