# 使用模式构建：子集模式（The Subset Pattern）

[三月 24, 2019](http://mongoing.com/archives/25244)[官网翻译文章](http://mongoing.com/translation_blogs)[牟天垒](http://mongoing.com/archives/author/sh5dragon5)

在多年前，第一代PC拥有高达256KB的RAM和两个5.25英寸的软盘驱动器。没有硬盘，因为在当时它们极为昂贵。这些限制导致在处理大量（对那时来说）数据时由于内存不足，必须在物理上交换软盘。如果当时有办法只把我经常使用的数据（如同整体数据的一个子集）放入内存就好了。

现代应用程序也无法幸免于资源消耗的影响。MongoDB将频繁访问的数据（称为[工作集](https://docs.mongodb.com/manual/reference/glossary/?_ga=2.60772906.627645384.1553429492-1197570158.1502203710#term-working-set)）保存在RAM中。当数据和索引的工作集超过分配的物理RAM时，随着磁盘访问的发生以及数据从RAM中转出，性能会开始下降。

我们如何解决这个问题？首先，我们可以向服务器添加更多的RAM，不过也就只能扩展这么多。我们也可以考虑对集合进行[分片](https://docs.mongodb.com/manual/reference/glossary/?_ga=2.223911452.627645384.1553429492-1197570158.1502203710#term-shard)，但这会带来额外的成本和复杂性，而我们的应用程序可能还没有准备好来应对这些。另一种选择是减小工作集的大小，这就是我们可以利用子集模式的地方。

## 子集模式

此模式用来解决工作集超出RAM，从而导致信息从内存中被删除的问题。这通常是由拥有大量数据的大型文档引起的，这些数据实际上并没有被应用程序使用。我这么说到底是什么意思呢？

假设一个电子商务网站有一个产品评论列表。当访问该产品的数据时，我们很可能只需要最近10个左右的评论。将整个产品数据与所有评论一起读入，很容易导致工作集的膨胀。

[![fulldocsubset-0jesu52gzs](http://mongoing.com/wp-content/uploads/2019/03/fulldocsubset-0jesu52gzs.png)](http://mongoing.com/wp-content/uploads/2019/03/fulldocsubset-0jesu52gzs.png)

相比于将所有的评论与产品存储在一起，我们可以将其分为两个集合。一个集合具有最常用的数据，例如当前的评论；另一个集合具有不太常用的数据，例如旧的评论、产品历史记录等。我们可以复制在一对多或多对多关系中最常用的那部分数据。

[![docsubset2-ncq6t9lt01](http://mongoing.com/wp-content/uploads/2019/03/docsubset2-ncq6t9lt01-618x1024.png)](http://mongoing.com/wp-content/uploads/2019/03/docsubset2-ncq6t9lt01.png)

在*Product*集合中，我们只保留最近十次的评论。这允许通过只引入整体数据的一部分或子集来减少工作集。附加信息（本例中的reviews）存储在单独的*Reviews*集合中，如果用户希望查看更多的评论，则可以访问该集合。在考虑将数据拆分到何处时，文档中使用最多的部分应放入“主”集合，而使用频率较低的数据应放入另一个集合。对于我们例子中的评论，这个分割点可能是产品页面上可见的评论数。

## 应用场景示例

当我们的文档拥有大量数据而其并不常用时，子集模式就非常有用。产品评论、文章评论、电影中的演员信息都是这个模式的应用场景案例。每当文档大小对工作集的大小产生压力并导致工作集超过计算机的RAM容量时，子集模式便成为一个可以考虑的选项。

## 结论

通过使用包含有频繁访问数据的较小文档，我们减少了工作集的总体大小。这使得应用程序所需要的最常用信息的磁盘访问时间更短。在使用子集模式时必须做的一个权衡是，我们必须管理子集，而且如果我们需要引入更旧的评论或所有信息，则需要额外的数据库访问才能做到这一点。

本系列的下一篇文章将介绍*扩展引用模式（Extended Reference Pattern）*的特性和优点。

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

*本文译自：Building with Patterns: The Subset Pattern*