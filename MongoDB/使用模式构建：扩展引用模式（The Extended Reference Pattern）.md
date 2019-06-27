# 使用模式构建：扩展引用模式（The Extended Reference Pattern）

[三月 31, 2019](http://mongoing.com/archives/25410)[官网翻译文章](http://mongoing.com/translation_blogs)[牟天垒](http://mongoing.com/archives/author/sh5dragon5)

贯穿整个《使用模式构建》，我希望你已经了解到一件事，即模式是什么样子取决于数据的访问方式。如果我们有许多相似的字段，*属性模式*可能是一个很好的选择。为了适配一小部分数据的访问会极大地改变我们的应用程序吗？也许*异常值模式*是值得考虑的。还有一些模式，例如*子集模式*，会引用额外的集合，并依赖JOIN操作将每个数据块重新组合在一起。如果需要大量的JOIN操作来聚集那些需要频繁访问的数据，这时要怎么办呢？这就是我们可以使用*扩展引用模式*的地方。

## 扩展引用模式

有时将数据放置在一个单独的集合中是有道理的。如果一个实体可以被认为是一个单独的“事物”，那么使其拥有单独的集合通常是有意义的。例如在一个电子商务应用中，存在订单的概念，就像客户和库存一样，它们都是独立的逻辑实体。

[![extended_reference2](http://mongoing.com/wp-content/uploads/2019/03/extended_reference2.png)](http://mongoing.com/wp-content/uploads/2019/03/extended_reference2.png)

然而从性能的角度来看，这就成了问题，因为我们需要为特定的订单将信息拼凑起来。一个客户可以有N个订单，创建一个1-N关系。如果我们反过来从订单的角度看，它们与客户之间有一种N-1的关系。仅仅是为了减少JOIN操作而为每个订单嵌入关于客户的**所有**信息，会导致大量的信息重复。此外，对于订单来说，并非所有的客户信息都是必须的。

扩展引用模式提供了一种很好的方法来处理这类情况。我们只复制经常访问的字段，而不是复制**全部**的客户信息。我们只嵌入那些优先级最高、访问最频率的字段，例如名称和地址，而不是嵌入所有信息或包含一个引用来JOIN信息。

[![extended_reference1-o1xbjhqpca](http://mongoing.com/wp-content/uploads/2019/03/extended_reference1-o1xbjhqpca.png)](http://mongoing.com/wp-content/uploads/2019/03/extended_reference1-o1xbjhqpca.png)

使用此模式时需要考虑的一点是，数据是重复的。因此最好存储在主文档中的数据是不会经常更改的字段。像user_id和人名之类的东西是不错的选择，这些很少改变。

此外，要注意只引入和复制所需的数据。想象一下订单发票，如果我们在发票上输入客户的姓名，我们是否在那个时间点会需要他们的第二个电话号码和非送货地址？可能不会，因此我们可以将该数据从**invoice**集合中删除，并添加一个**custormer**集合的引用。

当信息被更新时，我们同样需要考虑如何处理。哪些扩展引用发生了更改？应该什么时候进行更新？如果该信息是账单地址，我们是否需要出于历史目的维护该地址，还是可以直接更新？有时使数据重复会更好，因为你可以保留历史值，这可能更有意义。我们发货时客户所居住的地址在订单文档中更有意义，然后可以通过客户集合来获取现在的地址。

## 应用场景示例

订单管理应用是此模式的经典用例。在考虑订单到客户的N-1关系时，我们希望减少信息的连接以提高性能。通过包含对需要频繁连接数据的一个简单引用，我们在处理过程中省掉了一个步骤。

我们继续使用订单管理系统的作为例子。在发票上，Acme公司可能被列为一个铁砧的供应商。从发票的角度来看，拥有Acme公司的联系信息可能并不重要。例如，这些信息最好保存在单独的**supplier**集合中。在**invoice**集合中，我们会保留有关供应商的必要信息，作为对供应商信息的扩展引用。

## 结论

当应用程序中有许多重复的JOIN操作时，*扩展引用模式*是一个很好的解决方案。通过识别查找端（lookup side）的字段并将那些经常访问的字段引入主文档，可以提高性能。这是通过更快的读取和减少JOIN的总数来实现的。但是请注意，重复数据是这种设计模式的一个副作用。

本系列的下一篇文章将介绍*近似值模式（Approximation Pattern）*。

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

*本文译自：Building with Patterns: The Extended Reference Pattern*