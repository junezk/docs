# 对API进行版本控制的重要性和实现方式

我在API设计中收到的最常见问题之一就是如何对API进行版本控制。虽然并非所有API都完全相同，但我发现在API版本控制方面，某些模式和实践适用于大多数团队。我已经将这些内容收集起来，下面将提供一些关于版本控制策略的建议，该策略将帮助大多数API提供商，无论他们是向内部署API，还是对外的API。

## API版本真的那么重要吗？

API是你与API使用者之间建立的纽带。正常情况下，你们之间的纽带不会轻易的断开。纽带包括URI模式，有效负载结构，字段和参数名称，预期行为以及其他内容。这种方法的最大好处是显而易见的：API使用者的理解不会变更，应用程序可以保证持续有效。

但是，永久不变是不现实的。有时因为业务变化，你可能需要对API进行重大改变。发生这种情况时，最好的方式是，你确保不会做任何会导致API使用者修复代码的事情。



![img](https://user-gold-cdn.xitu.io/2019/8/7/16c69d8809ad2792?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 打破与不间断的变化

非破坏性变更往往就像是“添加剂”，一般是添加新字段或嵌套资源到资源陈述，又或是增加新的端点，如PUT或PATCH。API使用者从一开始应该构建能够适应这些非破坏性更改的客户端代码。

**突破性变化包括：**

1.名字段或资源路径，通常在API发布后为了统一命名规范。

2.更改有效负载结构，一般是适应以下内容：

a.重命名或删除字段

b.将字段从单个值更改为一对多关系（比如从一个帐户的一个电子邮件地址移动到这个帐户的电子邮件地址列表）。

c.修改了API的URL，导致返回结果不一致的情况。

简而言之，一旦你将API对外公开发布，你就必须保持它是可用的并且不会影响到使用者的。如果您遇到多个项目，则需要对API进行版本控制，以防止破坏现有的API使用者。



![img](https://user-gold-cdn.xitu.io/2019/8/7/16c69d8eaa6065a8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 定义API版本策略

任何不断发展变化的API都需要API版本控制策略。你的API版本可以适应根据API使用者的期望而切换不同版本变得有所不同。我通常建议将以下API版本控制策略作为整体API管理系统的一部分。

1.如果你的API处于早期测试版本中，为了获得消费者的反馈，请建立您的API可能会发生变化的正确期望。在此阶段内，你会保留这个版本一段时间，因为你的API设计可能还会更改。作为消费者，API是不稳定的，因此他们应该预期到可能会发生的变化。

2.一旦发布，你的API应被视为契约，如果没有新版本，则不能被替换。

3.不间断的更改会导致次要版本出现问题，客户端会自动迁移到最新版本，不会出现任何负面副作用。

4.突破性的变化意味着客户必须迁移到此新版本，因为它包含一个或多个重大更改。你必须与API使用者建立适当的时间表并定期沟通，以确保他们能方便地迁移到新版本。但在某些情况下，这可能无法马上实现，所以你的团队将会被要求暂时性支持以前的API版本。

## 何时必须实现API版本控制

一旦确定需要新版本的API，就需要知道如何处理它。实现API版本控制有三种常用方法。

**1.资源版本控制**

该版本是HTTP请求中Accept标头的一部分，例如，Accept:application/vnd.github.v3+json 发送到GET /customers。这考虑了许多版本控制的首选形式，因为资源在版本化的同时需要保持URI相同。如果未在Accept标头中提供，某些API会选择提供最新版本作为默认版本。

**2.URI版本控制**

该版本是URI的一部分，作为前缀或后缀，例如，/v1/customers 或/customers/v1。虽然URI版本控制不如基于内容的版本控制那么精确，但它往往是最常见的，因为它适用于不支持自定义标头的各种工具。缺点是资源URI随每个新版本而变化，有些人认为这与拥有永不改变的URI的意图背道而驰。

**3.主机名版本控制**

该版本是主机名的一部分而不是URI，例如，[v2.api.myapp.com/customers。当…](https://link.juejin.im/?target=https%3A%2F%2Fv2.api.myapp.com%2Fcustomers%E3%80%82%E5%BD%93%E6%8A%80%E6%9C%AF%E9%99%90%E5%88%B6%E9%98%BB%E6%AD%A2%E5%9F%BA%E4%BA%8EURI%E6%88%96Accept%E6%A0%87%E5%A4%B4%E5%88%B0API%E7%9A%84%E6%AD%A3%E7%A1%AE%E5%90%8E%E7%AB%AF%E7%89%88%E6%9C%AC%E6%97%B6%EF%BC%8C%E5%B0%86%E4%BD%BF%E7%94%A8%E6%AD%A4%E6%96%B9%E6%B3%95%E3%80%82)

**备注**：无论您选择哪个选项，API版本都只应包含主要编号。不需要小数字（即/v1/customers，不是/v1.1/customers）。



![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1002" height="174"></svg>)



## 实现版本控制的工具

使用工具和技术可以从根本上实现API版本控制过程。用市场上优秀的API编辑器将使技术开发团队能够在更短的时间内生成并切换更多的API版本，从而不断改进设计决策。

结合工具进行版本控制是大多数开发过程的重要组成部分。API设计领域中也有这种能版本控制的工具，实际上，全球范围内API服务领域中已经存在一些优秀的Web API设计工具。

现在，如EOLINKER、RAML、Swagger，都提供了出色的编辑工具来支持他们的语言。EOLINKER采用的是版本对比和重点标注提示，可以清晰的切换、对比。RAML、Swagger采用的是版本切换，方便程度可能略逊一点。而且只有前者是支持中文的，后两种只支持英文语言。这些API编辑器都能轻松地实现API版本的控制，使得更容易在更短的时间内切换运行版本。



![img](https://user-gold-cdn.xitu.io/2019/8/7/16c6a1e8b466bee3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



附上EOLINKER的官方网址：[www.eolinker.com](https://link.juejin.im/?target=https%3A%2F%2Fgio.ren%2Fw%2FnPNXnW9W)

附上Swagger的官方网址：[swagger.io/](https://link.juejin.im/?target=https%3A%2F%2Fswagger.io%2F)

附上RAML的官方网址：[raml.org/](https://link.juejin.im/?target=https%3A%2F%2Framl.org%2F)

## 最后的想法

请记住，API是与你的消费者链接的枢纽。打破旧的连接，就需要新版本。选择策略，制定计划，并与API使用者沟通该计划，这才是版本控制的最终目的。

*参考资料：James Higginbotham，When and How Do You Version Your API?*

*原文地址：dzone.com/articles/wh…*