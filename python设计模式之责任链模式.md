### 责任链模式（python精通设计模式）
责任链(Chain of Responsibility)模式用于让多个对象来处理单个请求时，或者用于预先不知道应该由哪个对象(来自某个对象链)来处理某个特定请求时，其原则：
(1) 存在一个对象链(链表、树或任何其他便捷的数据结构)。 
(2) 我们一开始将请求发送给链中的第一个对象。
(3) 对象决定其是否要处理该请求。
(4) 对象将请求转发给下一个对象。
(5) 重复该过程，直到到达链尾。



### 举例请假
if n <= 2:
    组长处理
elif 2 < n <=5:
    BLM处理
elif 5 < n <= 15:
    部门总监
elif 15 < n <= 30:
    CTO
elif 30 < n:
    离职


### 松耦合设计
不通过继承，而是通过组合来实现，
在对象初始化时，把下一跳添加进来，一环接一环，但又随时可以拆开重新组合
添加一个新步骤而不用影响到其他已添加的步骤


```python
from abc import ABCMeta, abstractmethod
from dataclasses import dataclass


class Staff(metaclass=ABCMeta):
    def __init__(self, parent=None):
        self.parent = parent
        
    @abstractmethod
    def approve(self, days):
        pass

    
class GroupLeader(Staff):
    def approve(self, days):
        if days <= 2:
            print("GroupLeader approve")
        else:
            print("--> BsLineManager")
            return self.parent.approve(days)
    
    
class BsLineManager(Staff):
    def approve(self, days):
        if days <= 5:
            print("BsLineManager approve")
        else:
            print("--> Director")
            return self.parent.approve(days)
    

class Director(Staff):
    def approve(self, days):
        if days <= 5:
            print("Director approve")
        else:
            print("--> CTO")
            return self.parent.approve(days)
        
class CTO(Staff):
    def approve(self, days):
        if days <= 15:
            print("CTO approve")
        else:
            print("--> CEO")
            return self.parent.approve(days)
    
class CEO(Staff):
    def approve(self, days):
        if days <= 30:
            print("CEO approve")
        else:
            print("dimission")

            
ceo = CEO()
cto = CTO(ceo)
dr = Director(cto)
blm = BsLineManager(dr)
gl = GroupLeader(blm)

gl.approve(10)
```

    --> BsLineManager
    --> Director
    --> CTO
    CTO approve



```python

```
