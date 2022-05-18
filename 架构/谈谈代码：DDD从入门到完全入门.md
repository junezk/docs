# [谈谈代码：DDD从入门到完全入门](https://segmentfault.com/a/1190000040556033)

之前的DDD文章——[`谈谈代码：降低复杂度，从放弃三层架构到DDD入门`](https://link.segmentfault.com/?enc=6s0%2FRj%2FB2GlDuLUflAXrgQ%3D%3D.0BMsZmu3N%2FjqBHSr0QK2k%2F%2BmZtF7h9x8%2BM4LzxxERwjHfYIOhsNMUmYHrqy623zp)，通篇下来像 是简单的讲了一些概念，然后快速的实战一下——很多同学反馈感觉就是入门了，但没有完全入门，因此我们再加一篇。

## 1.什么是DDD

先看下万能的维基百科：**Domain-driven design** (**DDD**) is the concept that the structure and language of software code (class names, [class methods](https://link.segmentfault.com/?enc=HpC8qf79t8y1czFaLyebPg%3D%3D.aYmx6aKyLRUjdojCsvzXNPzWApUFPa1pxl4WSipyiEroOaTrEMIlJ%2Bm688d15R1R), [class variables](https://link.segmentfault.com/?enc=lFlm5ACPCgrp0EktHkUb0w%3D%3D.uqwGri6IE7jbO1P%2Fb0ptiziliF1BRwGaJKDCxwBcuiTtsZ9q1OBOcLn5vDNPhrQn)) should match the [business domain](https://link.segmentfault.com/?enc=lR5RHZhuq38DAGwESjyx%2Bw%3D%3D.ItcCWeT%2B3ylQ7LZhSp9arXlvHUNfVM9WjvqgbYE77aSqoMTZVUjtfgKd0hmVsNlQ). For example, if a software processes loan applications, it might have classes such as LoanApplication and Customer, and methods such as AcceptOffer and Withdraw.

这边将其称为了一个概念。在我看来DDD是设计模式的超集，一种指导思想——用来指导如何解耦业务系统，划分业务模块，定义业务领域模型及其交互。

## 2. DDD诞生的背景

领域驱动设计这个概念并不新颖，早在 2004 年就被提出了，到现在已经有十几年的历史了。不过，它被大众熟知，还是基于另一个概念的兴起，那就是微服务。因此在实现的时候样子不一定是一模一样的，更多时候还是根据业务场景来因材施教。

## 3. DDD的核心思想

核心思想是让技术复杂度与业务复杂度隔离，并通过统一语言组织业务逻辑，降低认知成本。

具体主要体现在：

- 基础设施层：它负责隔离技术复杂度，通过抽象封装来对内提供服务，而不是让内部服务直接使用它。这意味着当外部基础设施变化时，业务并不会被迫进行变更。比如项目中的数据总线是Kafka，之后替换成了Pulsar，业务对其应该是无感知的。
- 厚领域层：同一领域的知识聚合在一个领域中，领域知识不再被割裂。这是单一职责原则的一种体现。
- 实体：用充血模型代替贫血模型，完全符合面向对象的思想。将业务中的对象完全投射到实体中，从面向资源转换成面向过程和面向对象。

## 4. DDD能解决什么样的问题

一般软件会经历几个不同的周期：

1. 大烟囱：每个应用各自为政，类似的需求重复开发，浪费人力
2. 服务化：根据不同的业务属性拆分服务。关注服务拆分、服务治理、模型抽象
3. 平台化：将相关的微服务同一成一个平台暴露出来。关注领域收敛、领域自治、能力沉淀
4. 中台化：将通用能力下沉至中台，快速响应前端服务。需要关注数据打通、能力串联、业务响应力

DDD主要在**技术密集型应用里有较大的作用**，尤其是当该应用进入服务化、平台化时，可以在：“服务拆分”、“服务治理”、“领域收敛”、“领域自治”发挥。在中台化中的“数据打通”也有一定的作用。

而微观来说，DDD可以有效减少代码的冗余程度以及需求响应的速度。

## 5. DDD实践中要注意的

### 5.1 使用IOC来保证层次之间的隔离

![img](谈谈代码：DDD从入门到完全入门.assets/1460000040556035.png)

经常有小伙伴问我，分层之间该怎么做？因为分层的边界没做好，代码会再度耦合再一起。对此我给出的答案是参考`inversion of control`。其常见实现有：

- Object Dependency Inject
- Service Provider Interface
- Strategy
- Abstract Factory

也可以参考我之前写的文章：[技巧：遵循Clean Architecture写好白盒测试](https://link.segmentfault.com/?enc=E1b0iJGjsgnFoJ35e7k59w%3D%3D.GEGllN9IvnkecwNNN3wpw3pVCa6fBVrVVPe0I47RReKSa1znKaQkYw2hRIV8BYLI)。

### 5.2 模块分离

模块分离是一种较为“硬”的手段，它让分层不再是一个约定，而是强制执行的规则。这样当我们拆分微服务时候，也可以较快的完成拆分。

### 5.3 DDD并不是只有三层到四层

也有小伙伴问过我，转DDD的是否只有三层过来的？其实并非如此。我这边可以举两个例子：

#### 5.3.1 流计算处理

我们以面向在线数据加密应用为例子：当一条数据流过我们的应用时，我们需要根据一些条件对其加密。
![img](谈谈代码：DDD从入门到完全入门.assets/1460000040629457.png)

- Source算子：它会从kafka拉取消息，并转换成我们需要的pojo对象
- Map算子：它的数据来源于流处理框架，并将pojo的某个属性转换成对象
- Sink算子：将pojo对象转换成二进制数据，发往kafka

那么代码中，kafka其实是基础层的代码。而`encryption`属于领域层，map(框架)和encryption之间的胶水代码则属于基础层。

![img](谈谈代码：DDD从入门到完全入门.assets/1460000040629458.png)

#### 5.3.2 GUI应用

![img](谈谈代码：DDD从入门到完全入门.assets/1460000040629459.png)

相信大家都在学生时代学过GUI or HTML 编程。那么按照DDD的做法来，业务逻辑应该与具体的界面无关——比如界面上的一个按钮（数据模型）会触发一种事件，当后台的事件接受者收到这个事件时，则会寻找相应的执行者，执行对应的逻辑。

在这里面：

- 界面可以是Qt，可以是Flex，可以是Ios，可以是Android，也可以是Vue。其本质是用户接口层。
- 后台的事件接受者是基础层，相对于界面和业务逻辑起到了一个承上启下的作用。
- 具体的执行逻辑则放在领域层，这是纯粹的逻辑，和UI界面无关。

![img](谈谈代码：DDD从入门到完全入门.assets/1460000040629460.png)