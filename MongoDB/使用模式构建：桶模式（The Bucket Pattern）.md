# 使用模式构建：桶模式（The Bucket Pattern）

[二月 12, 2019](http://mongoing.com/archives/24190)[官网翻译文章](http://mongoing.com/translation_blogs)[牟天垒](http://mongoing.com/archives/author/sh5dragon5)

在本期《使用模式构建》中，我们将介绍桶模式。这种模式在处理物联网（IOT）、实时分析或通用时间序列数据时特别有效。通过将数据放在一起，我们可以更容易地将数据组织成特定的组，提高发现历史趋势或提供未来预测的能力，同时还能对存储进行优化。

## 桶模式

随着数据在一段时间内持续流入（时间序列数据），我们可能倾向于将每个测量值存储在自己的文档中。然而，这种倾向是一种非常偏向于关系型数据处理的方式。如果我们有一个传感器每分钟测量温度并将其保存到数据库中，我们的数据流可能看起来像这样：

```
{
   sensor_id: 12345,
   timestamp: ISODate("2019-01-31T10:00:00.000Z"),
   temperature: 40
}

{
   sensor_id: 12345,
   timestamp: ISODate("2019-01-31T10:01:00.000Z"),
   temperature: 40
}

{
   sensor_id: 12345,
   timestamp: ISODate("2019-01-31T10:02:00.000Z"),
   temperature: 41
}
```

随着我们的应用程序在数据和索引大小上的扩展，这可能会带来一些问题。例如，我们可能最终不得不对每次测量的**sensor_id**和**timestamp**进行索引，实现以内存为代价的快速访问。但利用文档数据模型，我们可以按时间将这些数据“以桶的方式”储存到特定时间片测量值的文档中。我们还可以通过编程方式向每一个“桶”中添加附加信息。

通过将桶模式应用于数据模型，我们可以在节省索引大小、简化潜在的查询以及在文档中使用预聚合数据的能力等方面获得一些收益。获取上面的数据流并对其应用桶模式，我们可以得到：

```
{
    sensor_id: 12345,
    start_date: ISODate("2019-01-31T10:00:00.000Z"),
    end_date: ISODate("2019-01-31T10:59:59.000Z"),
    measurements: [
       {
       timestamp: ISODate("2019-01-31T10:00:00.000Z"),
       temperature: 40
       },
       {
       timestamp: ISODate("2019-01-31T10:01:00.000Z"),
       temperature: 40
       },
       … 
       {
       timestamp: ISODate("2019-01-31T10:42:00.000Z"),
       temperature: 42
       }
    ],
   transaction_count: 42,
   sum_temperature: 2413
}
```

使用桶模式，我们将数据“封装”到一个小时的桶中。这个特定的数据流仍然在增长，因为它目前只有42个测量值；这个小时还有更多的测量值要添加到“桶”中。当它们添加到**measurements**数组中时，**transaction_count**将增加，并且**sum_temperature**也将更新。

有了预先聚合的**sum_temperature**值，就可以很容易拉出一个特定的存储桶并确定该桶的平均温度（**sum_temperature** / **transaction_count**）。在处理时间序列数据时，知道2018年7月13日加利福尼亚州康宁市下午2:00至3:00的平均温度通常比知道下午2:03那一时刻的温度更有意义也更重要。通过用桶组织数据并进行预聚合，我们可以更轻松地提供这些信息。

此外，随着我们收集的信息越来越多，为了更高效我们可能决定将源数据进行归档。你想我们多久才会需要访问从1948年开始康宁市的温度？能够将这些数据桶移动到数据存档中是一项很大的收益。

## 应用场景示例

有一个[Bosch的物联网实现](https://www.mongodb.com/customers/bosch)可以成为时间序列数据在现实世界中体现价值的一个例子。他们将MongoDB和时间序列数据应用于一个汽车业的数据程序中。该应用程序从整个车辆的各种传感器中获取数据，从而提高车辆本身的诊断能力和部件性能。

其它一些例子还包括在银行的金融程序中使用这种模式将交易进行分组。

## 结论

处理时间序列数据时，在MongoDB中使用桶模式是一个很好的选择。它减少了集合中的文档总数，提高了索引性能，并且通过预聚合简化了数据访问。

桶模式在许多情况下都非常有效，但是如果我们的数据中有异常值呢？这就是我们要讨论的下一个模式——[异常值设计模式（Outlier Design Pattern）](http://mongoing.com/archives/24757)的作用所在。

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