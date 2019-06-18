# 使用模式构建：异常值模式（The Outlier Pattern）

[三月 2, 2019](http://mongoing.com/archives/24757)[官网翻译文章](http://mongoing.com/translation_blogs)[牟天垒](http://mongoing.com/archives/author/sh5dragon5)

到目前为止，在《使用模式构建》系列中，我们已经研究了[多态模式](http://mongoing.com/archives/20007)、[属性模式](http://mongoing.com/archives/22881)和[桶模式](http://mongoing.com/archives/24190)。其中，尽管文档的模式略有不同，但从应用程序和查询的角度来看，文档的结构基本上是一致的。然而，如果情况并非如此会怎么样？当有数据不属于“正常”模式时会发生什么？如果有异常值怎么办？

假设你正在搭建一个出售图书的电子商务网站，你可能会想查询“有哪些人购买了某本特定的书”。这对于一个可以向顾客展示他感兴趣书籍的推荐系统来说会很有用。你决定将顾客的**user_id**存储在每本书的一个数组中。很简单，对吧？

这可能确实适用于99.99%的情况，但是当J.K.罗琳发行了一本新的哈利波特书籍，并且销量以百万计激增时，会发生什么呢？16MB的[BSON文档](https://www.mongodb.com/json-and-bson)大小限制很容易达到。针对这种异常情况重新设计整个应用程序可能会降低典型书籍的性能，但我们确实需要考虑这一点。

## 异常值模式

使用异常值模式就是在防止一些少数的查询或文档将我们推向对大多数用例来说都不佳的解决方案。并非每本书都能卖出数百万册。

一个存有**user_id**的典型**book**文档可能看起来像这样：

```
{
    "_id": ObjectID("507f1f77bcf86cd799439011")
    "title": "A Genealogical Record of a Line of Alger",
    "author": "Ken W. Alger",
    …,
    "customers_purchased": ["user00", "user01", "user02"]

}
```

对于绝大多数不太可能登上“畅销书”排行榜的书来说，这可以工作得很好。尽管将异常值考虑进来后导致了**customers_purchased**数组超出了我们设置的1000个条目的限制，但我们可以添加一个新字段将这本书“标记”为异常值。

```
{
    "_id": ObjectID("507f191e810c19729de860ea"),
    "title": "Harry Potter, the Next Chapter",
    "author": "J.K. Rowling",
    …,
   "customers_purchased": ["user00", "user01", "user02", …, "user999"],
   "has_extras": "true"
}
```

然后，我们将多出的信息移动到与书籍的**id**相关联的单独文档中。在应用程序中，我们可以看文档是否有值为**true**的**has_extras**字段。如果是，那么应用程序将会检索额外的信息。这样处理可以使其对大多数应用程序代码来说是基本透明的。

许多设计决策都基于应用程序的工作负载，因此这个解决方案旨在展示一个异常值模式的示例。这里要理解的重要概念是，异常值在其数据中有足够大的差异，如果它们被当作“正常值”对待，那么为它们更改应用程序设计将降低其它更典型查询和文档的性能。

## 应用场景示例

异常模式是一种高级模式, 但可以带来较大的性能改进。它经常在受欢迎程度可以作为一个因素的情况下使用，例如社交网络关系、图书销售、电影评论等。互联网已经大幅缩小了我们的世界，当某个东西变得受欢迎时，它改变了我们需要对数据建模的方式。

一个例子是拥有视频会议产品的客户。大多数视频会议的被授权的与会者列表可以和会议保存在同一文档中。然而，也有一些活动预计会有数千的参加者，比如一家公司的全体员工。对于那些“异常”会议，这个客户使用“overflow”文档来记录那些长长的与会者列表。

## 结论

异常值模式所要解决的问题是防止以少量文档或查询来确定应用程序的解决方案，尤其是当该解决方案对大多数用例来说不是最佳的时候。我们可以利用MongoDB的灵活数据模型在文档中添加一个字段来将其标记为异常值。然后在应用程序内部，我们对异常值的处理会略有不同。通过为典型的文档或查询定制模式，应用程序的性能将会针对那些正常的用例进行优化，而那些异常值仍将得到处理。

这个模式需要考虑的一点是，它通常是为特定的查询和情况而定制的。因此，一些临时产生的查询可能会导致性能不理想。此外，由于大部分工作是在应用程序代码本身内完成的，因此随着时间的推移可能需要进行额外的代码维护。

在我们的下一篇《使用模式构建》中，我们将了解计算模式（Computed Pattern）以及如何为那些可能会导致不必要资源浪费的应用程序进行模式优化。如果你有任何问题，请在下面留言。

*## 使用模式构建系列：

- [多态模式（The Polymorphic Pattern）](http://mongoing.com/archives/20007)
- [属性模式（The Attribute Pattern）](http://mongoing.com/archives/22881)
- [桶模式（The Bucket Pattern）](http://mongoing.com/archives/24190)
- [异常值模式（The Outlier Pattern）](http://mongoing.com/archives/24757)
- [计算模式（The Computed Pattern）](http://mongoing.com/archives/24825)
- [子集模式（The Subset Pattern）](http://mongoing.com/archives/25244)
- [扩展引用模式（The Extended Reference Pattern）](http://mongoing.com/archives/25410)
- [近似值模式（The Approximation Pattern）](http://mongoing.com/archives/25484)
- [树形模式（The Tree Pattern）](http://mongoing.com/archives/25491)

*本文译自：Building with Patterns: The Preallocation Pattern**