###  Spring IOC 理解 
来源： https://www.zhihu.com/question/23277575/answer/169698662?utm_source=wechat_session&utm_medium=social&utm_oi=1268335754431877120


控制反转( Inversion of Control ), 我觉得有必要先了解软件设计的一个重要思想：依赖倒置原则（Dependency Inversion Principle ）

正常的一个顺序：

1、汽车 -> 车身  -> 底盘 -> 轮胎

当我们要改轮胎尺寸的时候， 就会变得非常麻烦， 因为前者包含后者，所有轮胎尺寸参数要不停的往后传；


2、轮胎 -> 底盘 -> 车身 -> 汽车

size = 17

tier = Tier(size)
bot = Bottom(tier)
fr = Frame(bot)
car = car(fr)

这样如果我们要修改轮胎的尺寸只要改前面两行的代码即可
这里把底层类作为参数传入上层类，实现上层类对下层类的“控制”，所谓依赖注入。


因为采用了依赖注入，在初始化的过程中就不可避免的会写大量的对象初始化。这里IoC容器就解决了这个问题。这个容器可以自动对你的代码进行初始化，
你只需要维护一个Configuration（可以是xml可以是一段代码），而不用每次初始化一辆车都要亲手去写那一大段初始化的代码。

实现的时候无需注意细节， 配置化即可，实现的时候，先从第一步开始查找配置项，走到底部后，开始走第二步，也就是反转了，初始化轮胎，一步一步注入。


```python

```
