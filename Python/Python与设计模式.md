# Python与设计模式

## 策略模式

完成一项任务往往有多种方式，我们将其称之为策略。

### 前言

比如，超市做活动，如果你的购物积分满1000，就可以按兑换现金抵用券10元，如果购买同一商品满10件，就可以打9折，如果如果购买的金额超过500，就可以享受满减50元的优惠。这是三个不同的促销策略。

再比如，联系朋友、同学，可以打电话，也可以发短信，可以发微信，也可以发邮件，这是四个不同的联系策略。

再比如，去外出旅游，我们可以选择火车，也可以选择公共汽车，可以选择飞机，也可以选择自驾游。这又是四个不同的出行策略。

以上这些真实场景，都有策略选择模型的影子，可以考虑使用策略模式。

经典的策略模式，是由三部分组成

- Context：上下文环境类
- Stragety：策略基类
- ConcreteStragety：具体策略

![img](Python与设计模式.assets/v2-621155311d0484f056587ebe39bb5289_hd.jpg)

### 举例

以第一个超市做活动的场景来举个例子。

- Context：Order类，订单信息，包括商品，价格和数量，以为购买者等
- Stragety：Promotion类，抽象基类，包含一个抽象方法（计算折扣）
- ContreteStragety：分三个类，FidelityPromo，BulkItemPromo，LargeOrderPromo，实现具体的折扣计算方法。

首先是 Order 类：

```python
class Item:
    def __init__(self, issue, price, quantity):
        self.issue = issue
        self.price = price
        self.quantity = quantity

    def total(self):
        return self.price * self.quantity

class Order:
    def __init__(self, customer, promotion=None):
        self.cart = []
        self.customer = customer
        self.promotion = promotion

    def add_to_cart(self, *items):
        for item in items:
            self.cart.append(item)

    def total(self):
        total = 0
        for item in self.cart:
            total += item.total()

        return total

    def due(self):
        if not self.promotion:
            discount = 0
        else:
            discount  = self.promotion.discount(self)
        return (self.total() - discount)
```

然后是积分兑换现金券的策略，为了保证我们的代码具有良好的可扩展性及维护性，我会先写一个策略类，它是一个抽象基类，它的子类都是一个具体的策略，都必须实现 `discount` 方法，就比如咱们的积分兑换现金策略。

```python
from abc import ABC, abstractmethod

class Promotion(ABC):
    @abstractmethod
    def discount(self, order):
        pass


class FidelityPromo(Promotion):
    '''
    如果积分满1000分，就可以兑换10元现金券
    '''
    def discount(self, order):
        return 10 if order.customer.fidelity >1000 else 0
```

假设现在小明去商场买了一件衣服（600块），两双鞋子（200*2），他的购物积分有1500点。

在平时，商场一般都没有活动，但是长年都有积分换现金抵用券的活动。

```python
>>> from collections import namedtuple

# 定义两个字段：名字，购物积分
>>> Customer = namedtuple('Customer', 'name fidelity')
>>> xm = Customer('小明', 1500)
>>> item1 = Item('鞋子', 200, 3)
>>> item2 = Item('衣服', 600, 1)
>>> order = Order(xm, FidelityPromo())
>>> order.add_to_cart(item1, item2)

# 原价 1200，用上积分后，只要1190
>>> order
<Order Total:1200 due:1190>
```

眼看着，五一节也快了，商场准备大搞促销

- 只要单项商品购买10件，即可9折。
- 如果订单总金额大于等于500，就可以立减50。

有了此前我们使用 `策略模式` 打下的基础，我们并不是使用硬编码的方式来配置策略，所以不需要改动太多的源码，只要直接定义五一节的两个促销策略类即可（同样继承自 Promotion 抽象基类），就像插件一样，即插即用。

```python
class BulkItemPromo(Promotion):
    '''
    如果单项商品购买10件，即可9折。
    '''
    def discount(self, order):
        discount = 0
        for item in order.cart:
            if item.quantity >= 10:
                discount += item.total() * 0.1
        return discount

class LargeOrderPromo(Promotion):
    '''
    如果订单总金额大于等于500，就可以立减50
    '''
    def discount(self, order):
        discount = 0
        if order.total() >= 500:
            discount = 50

        return discount
```

看到商场活动如此给力，小明的钱包也鼓了起来，开始屯起了生活用品。

如果使用了第一个策略，原价600，只需要花 580

```python
>>> from collections import namedtuple
>>> Customer = namedtuple('Customer', 'name fidelity')

>>> xm = Customer('小明', 300)

>>> item1 = Item('纸巾', 20, 10)
>>> item2 = Item('食用油', 50, 4)
>>> item3 = Item('牛奶', 50, 4)


>>> order = Order(xm, BulkItemPromo())
>>> order.add_to_cart(item1, item2, item3)

>>> order
<Order Total:600 due:580.0>
```

如果使用了第二个策略，原价600，只需要花550

```python
>>> from collections import namedtuple
>>> Customer = namedtuple('Customer', 'name fidelity')

>>> xm = Customer('小明', 300)

>>> item1 = Item('纸巾', 20, 10)
>>> item2 = Item('食用油', 50, 4)
>>> item3 = Item('牛奶', 50, 4)


>>> order = Order(xm, LargeOrderPromo())
>>> order.add_to_cart(item1, item2, item3)

>>> order
<Order Total:600 due:550>
```

两个策略即插即用，只需要在前台下订单时，选择对应的策略即可，原业务逻辑无需改动。

```python
>>> order = Order(xm, BulkItemPromo())
>>> order = Order(xm, LargeOrderPromo())
```

但是问题很快又来了，商场搞活动，却让顾客手动选择使用哪个优惠策略，作为一个良心的商家，应该要能自动对比所有策略得出最优惠的价格来给到顾客。这就要求后台代码要能够找出当前可用的全部策略，并一一比对折扣。

```python
# 找出所有的促销策略
all_promotion = [globals()[name] for name in globals() if name.endswith('Promo') and name != 'BestPromo']

# 实现一个最优策略类
class BestPromo(Promotion):
    def discount(self, order):
        # 找出当前文件中所有的策略
        all_promotion = [globals()[name] for name in globals() if name.endswith('Promo') and name != 'BestPromo']

        # 计算最大折扣
        return max([promo().discount(order) for promo in all_promotion])
```

在前台下订单的时候，就会自动计算所有的优惠策略，直接告诉顾客最便宜的价格。

```python
# 直接选择这个最优策略
>>> order = Order(xm, BestPromo())
>>> order.add_to_cart(item1, item2, item3)

>>> order
<Order Total:600 due:550>
```

通过以上例子，可以总结出使用`策略模式`的好处

1. 扩展性优秀，移植方便，使用灵活。可以很方便扩展策略；
2. 各个策略可以自由切换。这也是依赖抽象类设计接口的好处之一；

但同时，策略模式 也会带来一些弊端。

1. 项目比较庞大时，策略可能比较多，不便于维护；
2. 策略的使用方必须知道有哪些策略，才能决定使用哪一个策略，这与迪米特法则是相违背的。

对于以上的例子，仔细一想，其实还有不少可以优化的地方。

比如，为了实现经典的模式，我们先要定义一个抽象基类，再实现具体的策略类。对于上面这样一个简单的计算折扣价格逻辑来说，其实可以用函数来实现，然后在实例化 Order 类时指定这个策略函数即可，大可不必将类给搬出来。这样就可以避免在下订单时，不断的创建策略对象，减少多余的运行时消耗。这里就不具体写出代码了。

所以学习设计模式，不仅要知道如何利用这样的模式组织代码，更要领会其思想，活学活用，灵活变通。

以上，就是今天关于 `策略模式` 的一些个人分享，如有讲得不到位的，还请后台留言指正！

### 参考文章

- 《流畅的Python》

## 单例模式

> 前段时间，有朋友在我的读者群里问了几个关于单例模式的问题。
>
> 为了回答他的问题，我整理了单例模式的知识点，正好我也在写设计模式的系列。

之前在另一篇公众号文章看到一个挺搞笑的例子：

大意是讲，老婆在中国其实就是一个很形象的单例，你要娶一个老婆需要去民政局注册登记（要对类进行实例化），当你想再娶一个老婆时，这时民政局会说，不行，你已经有一个老婆了，并且它还会告诉你的老婆是谁。

然后有个朋友，还很生趣地评论说

> **单例模式** 允许你讨无数个老婆，但最终你会发现你讨来的老婆都是同一个人

玩笑之后，再回到我们的话题，先举几类我们经常见到的例子：

1、大家在解释单例模式时，经常要提到的一个例子是 Windows 的任务管理器。如果我们打开多个任务管理器窗口。显示的内容完全一致，如果在内部是两个一模一样的对象，那就是重复对象，就造成了内存的浪费；相反，如果两个窗口的内容不一致，那就会至少有一个窗口展示的内容是错误的，会给用户造成误解，到底哪个才是当前真实的状态呢？

2、一个项目中多个地方需要读取同一份配置文件，如果每次使用都是导入重新创建实例，读取文件，用完后再销毁，这样做的话，就造成不必要的IO浪费，可以使用单例模式只生成一份配置在内存中。

3、还有一个常见的例子是，一个网站的访问量、在线人数，在项目中是全局唯一（不考虑分布式），在这种情况下，使用单例模式是一种很好的方式。

从上面看来，在系统中确保某个对象的唯一性即一个类只能有一个实例有时是非常重要的。

按照惯例，我们先来用代码实践一下，看看如何用 Python 写单例模式。

这里介绍了三个较为常用的。

### 1、使用 __new__

```python
class User:
    _instance = None
    def __new__(cls, *args, **kwargs):
        print('===== 1 ====')
        if not cls._instance:
            print("===== 2 ====")
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self, name):
        print('===== 3 ====')
        self.name = name
```

验证结果

![img](Python与设计模式.assets/v2-41ced58d196cff17fa30a36719fdd2ab_hd.jpg)

### 2、使用装饰器

```python
instances = {}

def singleton(cls):
    def get_instance(*args, **kw):
        cls_name = cls.__name__
        print('===== 1 ====')
        if not cls_name in instances:
            print('===== 2 ====')
            instance = cls(*args, **kw)
            instances[cls_name] = instance
        return instances[cls_name]
    return get_instance

@singleton
class User:
    _instance = None

    def __init__(self, name):
        print('===== 3 ====')
        self.name = name
```

验证结果

![img](Python与设计模式.assets/v2-f5c62b90e9a0c16a65369c7816818273_hd.jpg)

### 3、使用元类

```python
class MetaSingleton(type):
    def __call__(cls, *args, **kwargs):
        print("cls:{}".format(cls.__name__))
        print("====1====")
        if not hasattr(cls, "_instance"):
            print("====2====")
            cls._instance = type.__call__(cls, *args, **kwargs)
        return cls._instance

class User(metaclass=MetaSingleton):
    def __init__(self, *args, **kw):
        print("====3====")
        for k,v in kw:
            setattr(self, k, v)
```

验证结果

![img](Python与设计模式.assets/v2-352536de9ef889db67f0a0035a404007_hd.jpg)

以上的代码，一般情况下没有问题，但在并发场景中，就会出现线程安全的问题。

如下这段代码我开启10个线程来模拟

```python
import time
import threading

class User:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            time.sleep(1)
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self, name):
        self.name = name

def task():
    u = User("wangbm")
    print(u)

for i in range(10):
    t = threading.Thread(target=task)
    t.start()
```

从结果来观察，很容易就发现，单例模式失效了，在10个线程下，并发创建实例，并不能保证一个类只有一个实例。

```python
<__main__.User object at 0x1050563c8>
<__main__.User object at 0x10551a208>
<__main__.User object at 0x1050563c8>
<__main__.User object at 0x1055a93c8>
<__main__.User object at 0x1050563c8>
<__main__.User object at 0x105527160>
<__main__.User object at 0x1055f4e48>
<__main__.User object at 0x1055e6c88>
<__main__.User object at 0x1055afcf8>
<__main__.User object at 0x105605940>
```

### 4、加锁方式

这在 Java 中，是可以使用饿汉模式来避免这个问题，在 Python 中我想到的办法是**加锁**。

首先实现一个给函数加锁的装饰器

```python
import threading

def synchronized(func):

    func.__lock__ = threading.Lock()

    def lock_func(*args, **kwargs):
        with func.__lock__:
            return func(*args, **kwargs)
    return lock_func
```

然后在实例化对象的函数上，使用这个装饰函数。

```python
import time
import threading

class User:
    _instance = None

    @synchronized
    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            time.sleep(1)
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self, name):
        self.name = name

def task():
    u = User("wangbm")
    print(u)

for i in range(10):
    t = threading.Thread(target=task)
    t.start()
```

结果如下，如预期只生成了一个实例。

```python
<__main__.User object at 0x10ff503c8>
<__main__.User object at 0x10ff503c8>
<__main__.User object at 0x10ff503c8>
<__main__.User object at 0x10ff503c8>
<__main__.User object at 0x10ff503c8>
<__main__.User object at 0x10ff503c8>
<__main__.User object at 0x10ff503c8>
<__main__.User object at 0x10ff503c8>
<__main__.User object at 0x10ff503c8>
<__main__.User object at 0x10ff503c8>
```

学会写只是第一步，还有一点，相当重要，要知道为何会有这个设计模式，它有什么优势，有什么局限性？

### 小节

总结一下，单例模式有如下优点：

1. 全局只有一个接入点，可以更好地进行数据同步控制，避免多重占用；
2. 由于单例模式要求在全局内只有一个实例，因而可以节省比较多的内存空间；
3. 单例可长驻内存，减少系统开销。

和其他设计模式一样，单例模式有一定的适用场景，但同时它也会给我们带来一些问题。

1. 由于单例对象是全局共享，所以其状态维护需要特别小心。一处修改，全局都会受到影响。
2. 单例对象没有抽象层，扩展不便。
3. 赋于了单例以太多的职责，某种程度上违反单一职责原则（六大原则后面会讲到）;
4. 单例模式是并发协作软件模块中需要最先完成的，因而其不利于测试；
5. 单例模式在某种情况下会导致“资源瓶颈”。

**参考文章**

- [Python与设计模式--单例模式](https://link.zhihu.com/?target=https%3A//yq.aliyun.com/articles/70418%3Futm_content%3Dm_14908%23comment)
- [Python线程安全的单例模式](https://link.zhihu.com/?target=https%3A//blog.csdn.net/lucky404/article/details/79668131)