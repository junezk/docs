# 使用模式构建：多态模式（The Polymorphic Pattern）

[二月 7, 2019](http://mongoing.com/archives/20007)[官网翻译文章](http://mongoing.com/translation_blogs)[牟天垒](http://mongoing.com/archives/author/sh5dragon5)

当涉及MongoDB时，一个经常被问到的问题是“我如何在MongoDB中为我的应用程序构造模式（schema）？”老实说，这要看情况而定。你的应用程序读操作比写操作多吗？从数据库中读取时需要将哪些数据放在一起？有哪些性能因素需要考虑？文档有多大？它们今后会变成多大？你预计数据会如何增长和扩展？

所有这些以及更多的问题，都涉及到如何在MongoDB中设计数据库模式（schema）。有人说MongoDB是无模式的，而实际上模式设计在MongoDB中非常重要。有一个严峻的现实，我们发现的大多数性能问题都可以追溯到糟糕的模式设计。

在本系列文章“使用模式构建”中，我们将了解在MongoDB中行之有效的十二种常见的模式设计方式（Schema Design Patterns）。我们希望本系列文章能够建立一种在设计模式时可以使用的通用方法和词汇表。利用这些模式（patterns）可以在模式（schema）规划中使用“构建基块（building blocks）”，从而使这个过程更多地成为一种方法论而不是艺术。

MongoDB使用[文档数据模型](https://docs.mongodb.com/manual/core/data-model-design/?_ga=2.150316978.548613641.1548994675-1197570158.1502203710)。此模型具有内在的灵活性，允许数据模型支持你的应用程序需求。灵活性也可能导致模式比它们应有样子的更复杂。在考虑模式设计时，我们应该考虑性能、可伸缩性和简单性。

让我们开始探索模式设计时先看一下被认为是所有模式基础的模式——*多态模式*。当我们的文档具有比差异更多的相似性时，就会使用这种模式。它也同样适合于当我们希望将文档保存在单一集合中的场景。

## 多态模式

当集合中的所有文档都具有相似但不相同的结构时，我们将其称为多态模式。如前所述，当我们希望从单个集合中访问（查询）信息时，多态模式非常有用。根据我们要运行的查询将文档分组在一起（而不是将其分散在多个表或集合中）有助于提高性能。

假设我们有一个应用程序用来跟踪所有不同运动项目的专业运动员。

我们仍然希望能够在应用程序中访问所有的运动员，但每个运动员的属性都不尽相同，这就是多态模式可以发挥作用的地方。在下面的示例中，我们将来自两个不同项目运动员的数据存储在同一个集合中。即使文档在同一集合中，存储的关于每个运动员的数据也不必须是相同的。

[![Polymorphic1-nanehssyv3](http://mongoing.com/wp-content/uploads/2019/02/Polymorphic1-nanehssyv3-1024x493.png)](http://mongoing.com/wp-content/uploads/2019/02/Polymorphic1-nanehssyv3.png)
*Polymorphic Design Pattern with Common Fields*

对于职业运动员的记录既有相似之处也有不同之处。使用多态模式，我们可以很容易地适应这些差异。如果不使用多态模式，我们可能会有一个保龄球运动员的集合和一个网球运动员的集合。当我们想询问所有运动员时，我们需要进行耗时且复杂的连接操作（join）。相反，由于我们使用了多态模式，我们所有的数据都存储在一个运动员集合中，通过一个简单的语句就可以完成对所有运动员的查询。

这种设计模式也可以使用在嵌入式子文档中。在上面的例子中，Martina Navratilova不仅仅是作为一名单独的选手参加比赛，所以我们可能希望她的记录结构如下：

[![Polymorphic2-bpk0w3ig5y](http://mongoing.com/wp-content/uploads/2019/02/Polymorphic2-bpk0w3ig5y-1024x676.gif)](http://mongoing.com/wp-content/uploads/2019/02/Polymorphic2-bpk0w3ig5y.gif)
*Polymorphic Design Pattern with sub-documents*

从应用程序开发的角度来看，当使用多态模式时，我们将查看文档或子文档中的特定字段，以便能够跟踪差异。例如，我们知道一个网球运动员可能参加不同的项目，而另一个运动员可能不参加。这通常需要应用程序基于给定文档中的信息选择不同的代码路径。或者，可能会编写不同的类或子类来处理网球、保龄球、足球和橄榄球运动员之间的差异。

## 应用场景示例

多态模式的一个示例用例是[单一视图应用程序](https://www.mongodb.com/use-cases/single-view)。假设你在一家公司工作，随着时间的推移，这家公司以其技术和数据模式收购了其它公司。假如每家公司都有许多数据库，每个都以不同的方式为“向客户提供的保险”建模。然后你购买了这些公司，并希望将所有这些系统集成到一起。而将这些不同的系统合并到一个统一的SQL模式中是一项既昂贵又费时的工作。

[Metlife](https://www.metlife.com/)能够在几个月内利用MongoDB和多态模式构建他们的单一视图应用程序。他们的单一视图应用程序将来自多个来源的数据聚合到一个中央存储库中，从而使客户服务、保险代理、计费还有其它部门能够360°了解一个客户。这使得他们能够以较低的成本为客户提供更好的服务。此外，利用MongoDB的灵活数据模型和多态模式，开发团队能够快速创新，使其产品上线。

单一视图应用程序是多态模式的一个用例。它也适用于产品目录，例如自行车和鱼竿具有不同的属性。我们的运动员示例可以很容易地扩展到一个更完善的内容管理系统中，并在其中使用多态模式。

## 结论

当文档具有更多的相似性而不是差异性时，就会使用多态模式。这种模式设计的典型用例是：

- 单一视图应用程序
- 内容管理
- 移动应用程序
- 产品目录

多态模式提供了一个易于实现的设计，允许在单个集合中进行查询，并且它还是我们接下来文章中探讨的许多设计模式的起点。我们将讨论的下一个模式是[属性模式（Attribute Pattern）](http://mongoing.com/archives/22881)。

如果你有任何问题，请在下面留言。

## 使用模式构建系列：

- [多态模式（The Polymorphic Pattern）](http://mongoing.com/archives/20007)
- [属性模式（The Attribute Pattern）](http://mongoing.com/archives/22881)
- [桶模式（The Bucket Pattern）](http://mongoing.com/archives/24190)
- [异常值模式（The Outlier Pattern）](http://mongoing.com/archives/24757)
- [计算模式（The Computed Pattern）](http://mongoing.com/archives/24825)
- [子集模式（The Subset Pattern）](http://mongoing.com/archives/25244)
- [扩展引用模式（The Extended Reference Pattern）](http://mongoing.com/archives/25410)
- [近似值模式（The Approximation Pattern）](http://mongoing.com/archives/25484)
- [树形模式（The Tree Pattern）](http://mongoing.com/archives/25491)

*本文译自：Building with Patterns: The Preallocation Pattern*