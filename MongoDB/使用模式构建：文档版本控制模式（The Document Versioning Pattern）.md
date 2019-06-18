# 使用模式构建：文档版本控制模式（The Document Versioning Pattern）

[五月 5, 2019](http://mongoing.com/archives/26025)[官网翻译文章](http://mongoing.com/translation_blogs)[牟天垒](http://mongoing.com/archives/author/sh5dragon5)

数据库，例如MongoDB，非常擅长查询大量数据并进行频繁更新。然而，在大多数情况下，我们只针对数据的最新状态执行查询。那如果有些场景下我们需要查询数据的以前状态呢？如果我们需要一些文档的版本控制功能怎么办？这就是我们可以使用文档版本控制模式的地方。

这个模式的关键是保持文档的版本历史记录处于可用状态。我们可以构建一个专用的版本控制系统和MongoDB配合使用。这个系统用于处理少数文档的更改，而MongoDB用于处理其它文档。这可能看起来有些笨。但是通过使用文档版本控制模式，我们可以避免使用多个系统来管理当前文档及其历史，方法是将它们保存在同一个数据库中。

## 文档版本控制模式

这种模式解决了这样一个问题：希望可以在不引入第二个管理系统的情况下保留MongoDB中某些文档的旧版本。为此，我们在每个文档中添加一个字段，以便跟踪文档版本。然后，数据库将会有两个集合：一个集合具有最新的（和查询最多的数据），另一个具有所有数据的修订版本。

文档版本控制模式对数据库以及应用程序中的数据访问模式做了一些假设。

1. 每个文档不会有太多的修订版本。
2. 需要做版本控制的文档不会太多。
3. 大多数的查询都是基于文档的最新版本。

如果你发现这些假设不适用于你的场景，那么这个模式也许不太合适。这需要你更改对于这一模式中版本的实现，或者你的用例可能需要换一个解决方案。

## 应用场景示例

文档版本控制模式在高度规范化的行业中非常有用，这些行业会要求一组数据的特定时间点版本。金融和医疗行业就是很好的例子，保险业和法律相关的行业也同样如此。有许多场景需要跟踪数据某些部分的历史记录。

我们来看看一个保险公司可能会如何使用这种模式。每个客户都有一个“标准”保单和一个（根据客户意愿增加的）该客户特有的保单附加条款。这附加的第二部分包括保险单附加条款列表和正在投保的特定项目列表。当客户更改了受保的具体项目时，这一信息需要随之更新，而同时之前的历史记录也需要保留。这在业主或承租人这样的保单中相当常见。例如，有人想要投保的特定项目超出了所提供的典型保险范围，那么这部分会作为附加条款单独列出。保险公司的另一个用例可能是保留他们随时间邮寄给客户的“标准保单”的所有版本。

根据文档版本控制模式的需求，这看起来是一个非常好的用例。保险公司可能有几百万个客户，对“附加”列表的修改可能不会太频繁，而且对保单的大多数搜索针对的都是最新版本。

在我们的数据库中，每个客户可能在**current_policies**集合中有一个包含客户特定信息的**current_policy**文档，以及在**policy_revisions**集合中有一个**policy_revision**文档。此外，还会有一个对于大多数客户来说相同的**standard_policy**集合。当客户购买新项目并希望将其添加到其保单中时，将使用**current_policy**文档创建一个新的**policy_revision**文档。随后，文档中的版本字段将会递增以标识其为最新版本，并将客户的更改添至其中。

[![documentversioning0-qex0ek1957](http://mongoing.com/wp-content/uploads/2019/05/documentversioning0-qex0ek1957-300x228.png)](http://mongoing.com/wp-content/uploads/2019/05/documentversioning0-qex0ek1957.png)

[![document_ver_2a-mw8mlpazzy](http://mongoing.com/wp-content/uploads/2019/05/document_ver_2a-mw8mlpazzy-1024x519.png)](http://mongoing.com/wp-content/uploads/2019/05/document_ver_2a-mw8mlpazzy.png)

最新版本存储在**current_policies**集合中，而旧版本将写入**policy_revisions**集合。通过在**current_policy**集合中保留最新版本，查询请求可以保持简单。根据对数据的需求，**policy_revisions**集合可能也只保留几个版本。

[![document_ver_3a-nqhnglt0yj](http://mongoing.com/wp-content/uploads/2019/05/document_ver_3a-nqhnglt0yj-1024x681.png)](http://mongoing.com/wp-content/uploads/2019/05/document_ver_3a-nqhnglt0yj.png)

在这个例子中，中土（Middle-earth）保险公司为其客户制定了一个**standard_policy**。夏尔（Shire）的所有居民都将共享这个保单文档。现在比尔博（Bilbo）还想在他正常的保险范围之外添加一些特别的保项：他的精灵宝剑（Elven Sword）以及，当然，还有至尊魔戒（the One Ring）。这些将保存在**current_policies**集合中，并且在进行更改时，**policy_revisions**集合将保留更改的历史记录。

文档版本控制模式相对容易实现。它可以在现有系统上实现，而不会对应用程序或现有文档进行太多的更改。此外，访问文档最新版本的查询仍然可以执行。

这种模式的一个缺点是对于历史信息需要访问不同的集合。此外，这种模式对数据库的总体写入量会更高。这就是为什么使用此模式的要求之一是数据的更改不会太频繁。

## 结论

当你需要跟踪文档的更改时，文档版本控制模式是一个很好的选择。它相对容易实现，并且可以应用于现有的一组文档。另一个好处是，对最新版本数据的查询仍然可以很好地执行。但是，它不能取代专用的版本控制系统。

本系列的下一篇文章将介绍最后一个模式*模式版本控制（Document Versioning）*。

如果你有任何问题，请在下面留言。

## 使用模式构建系列：

- [多态模式（The Polymorphic Pattern）](http://mongoing.com/archives/20007)
- [属性模式（The Attribute Pattern）](http://mongoing.com/archives/22881)
- [分桶模式（The Bucket Pattern）](http://mongoing.com/archives/24190)
- [异常值模式（The Outlier Pattern）](http://mongoing.com/archives/24757)
- [计算模式（The Computed Pattern）](http://mongoing.com/archives/24825)
- [子集模式（The Subset Pattern）](http://mongoing.com/archives/25244)
- [扩展引用模式（The Extended Reference Pattern）](http://mongoing.com/archives/25410)
- [近似值模式（The Approximation Pattern）](http://mongoing.com/archives/25484)
- [树形模式（The Tree Pattern）](http://mongoing.com/archives/25491)
- [预分配模式（The Preallocation Pattern）](http://mongoing.com/archives/25525)

*本文译自：Building with Patterns: The Document Versioning Pattern*