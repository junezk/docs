# 使用模式构建：计算模式（The Computed Pattern）

[三月 15, 2019](http://mongoing.com/archives/24825)[官网翻译文章](http://mongoing.com/translation_blogs)[牟天垒](http://mongoing.com/archives/author/sh5dragon5)

我们已经在*使用模式构建系列*研究了各种优化存储数据的方法。现在，我们从另一个角度来看看模式设计。通常，仅仅存储数据并使其可用还不够。当我们可以从数据中计算出值时，数据会变得有用的多。最新Amazon Alexa的总销售收入是多少？有多少观众看了这部最新的大片？这类问题可以从数据库中存储的数据那里得到答案，但必须进行计算。

每次在请求时运行这些计算都会是一个极其消耗资源的过程，特别是在大型数据集上。CPU周期、磁盘访问、内存都会被牵涉进来。

假设现在有一个关于电影信息的Web应用程序。每次我们访问应用查找电影时，页面都会提供有关播放这部电影的影院数量、观看电影的总人数以及总收入的信息。如果应用必须不断地为每次页面访问计算这些值，那么当碰上那些很受欢迎的电影时会使用掉大量的处理资源。

然而，大多数时候我们不需要知道确切的数字。我们可以在后台进行计算，然后每隔一段时间更新一次电影信息的主文档。这些计算允许我们在显示有效数据的同时无需给CPU带来额外的负担。

## 计算模式

当有在应用程序中需要重复计算的数据时，我们可以使用计算模式。当数据访问模式为读取密集型时，也会使用计算模式；例如，如果每小时有1000000次读取而只有1000次写入，则在写入时进行计算会使计算次数减少1000倍。

[![cpucomputed-fbcpkmvbsy](http://mongoing.com/wp-content/uploads/2019/03/cpucomputed-fbcpkmvbsy-1024x640.png)](http://mongoing.com/wp-content/uploads/2019/03/cpucomputed-fbcpkmvbsy.png)

在我们的电影数据库示例中，我们可以根据特定电影上的所有放映信息进行计算，并将计算结果与电影本身的信息存储在一起。在低写负载的环境中，这个计算可以与源数据的任意更新一起完成。如果有更多的常规写入，则可以按定义好的时间间隔（例如每小时）进行计算。因为不会对上映信息中的源数据做任何修改，所以我们可以继续运行现有的计算，或者在任何时间点运行新的计算，并且确定将得到正确的结果。

一些执行计算的其它策略可能会涉及例如向文档添加时间戳以指示文档上次的更新时间。之后，应用程序可以确定何时需要进行计算。另一种选择是可以生成一个需要完成的计算队列。使用何种更新策略最好留给应用开发人员去选择。

## 应用场景示例

只要有对数据进行计算的需求，就可以使用计算模式。一个很好的例子是需要求和的数据集（如收入或观影者），但时间序列数据、产品目录、单视图应用程序和事件源也同样很适合这种模式。

这是许多客户已经实现的模式。例如，一个客户对车辆数据进行了大量的聚合查询，并将结果存储在服务器上，以在接下来的几个小时显示这些信息。

一家出版公司将所有类型的数据进行编制来创建像“100个最佳的……”这样的有序列表。这些列表一段时间只需要重新生成一次，而底层数据可能在其它时间更新。

## 结论

这一强大的设计模式可以减少CPU工作负载并提高应用程序性能。它可以用于对集合中的数据进行计算或操作，并将结果存储在文档中，以避免重复进行相同的计算。当你的系统在重复执行相同的计算，并且具有较高的读写比时，请考虑使用计算模式。

我们已经完成了*使用模式构建*这个系列的三分之一了。下一次我们将讨论*子集模式（Subset Pattern）*的特性和优点，以及它如何帮助解决内存不足问题。如果你有任何问题，请在下面留言。

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