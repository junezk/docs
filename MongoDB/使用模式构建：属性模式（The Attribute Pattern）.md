# 使用模式构建：属性模式（The Attribute Pattern）

[二月 10, 2019](http://mongoing.com/archives/22881)[官网翻译文章](http://mongoing.com/translation_blogs)[牟天垒](http://mongoing.com/archives/author/sh5dragon5)

欢迎回到MongoDB模式设计系列。上一次我们研究了[多态模式](http://mongoing.com/archives/20007)，它涵盖了集合中所有文档具有相似但不相同结构的情况。在本文中，我们将了解一下属性模式。属性模式特别适用于以下情况：

- 我们有一些大文档，它们有很多相似的字段，而这些字段的一个子集具有共同的特征，我们希望对该子集字段进行排序或查询；
- 我们需要排序的字段只能在一小部分文档中找到；
- 上述两个条件均满足。

出于性能原因考虑，为了优化搜索我们可能需要许多索引以照顾到所有子集。创建所有这些索引可能会降低性能。属性模式为这种情况提供了一个很好的解决方案。

## 属性模式

假设现在有一个关于电影的集合。其中所有文档中可能都有类似的字段：标题、导演、制片人、演员等等。假如我们希望在上映日期这个字段进行搜索，这时面临的挑战是“哪个上映日期”？在不同的国家，电影通常在不同的日期上映。

```
{
    title: "Star Wars",
    director: "George Lucas",
    ...
    release_US: ISODate("1977-05-20T01:00:00+01:00"),
    release_France: ISODate("1977-10-19T01:00:00+01:00"),
    release_Italy: ISODate("1977-10-20T01:00:00+01:00"),
    release_UK: ISODate("1977-12-27T01:00:00+01:00"),
    ...
}
```

搜索上映日期需要同时查看多个字段。为了快速进行搜索，我们需要在电影集合中使用多个索引：

```
{release_US: 1}
{release_France: 1}
{release_Italy: 1}
...
```

使用属性模式，我们可以将此信息移至数组中并减少对索引需求。我们将这些信息转换成一个包含键值对的数组：

```
{
    title: "Star Wars",
    director: "George Lucas",
    …
    releases: [
        {
        location: "USA",
        date: ISODate("1977-05-20T01:00:00+01:00")
        },
        {
        location: "France",
        date: ISODate("1977-10-19T01:00:00+01:00")
        },
        {
        location: "Italy",
        date: ISODate("1977-10-20T01:00:00+01:00")
        },
        {
        location: "UK",
        date: ISODate("1977-12-27T01:00:00+01:00")
        },
        … 
    ],
    … 
}
```

通过在数组中的元素上创建一个这样的索引，索引变得更易于管理：{ “releases.location”: 1, “releases.date”: 1}

使用属性模式，我们可以将组织信息添加到文档中，在获取通用特征的同时以应对罕见的/不可预测的字段，比如在一个新节日或小节日里上映的电影。此外，使用键/值约定允许非确定性命名（non-deterministic naming）并且可以很容易地添加限定符（qualifiers）。假如我们有一个关于瓶装水的数据集合，那么它们的属性可能看起来是这样：

```
"specs": [
    { k: "volume", v: "500", u: "ml" },
    { k: "volume", v: "12", u: "ounces" }
]
```

这里我们将信息分为键和值“k”和“v”，并添加第三个字段“u”，允许度量单位单独存储。

## 应用场景示例

属性模式非常适合具有相同值类型的字段集（如日期列表）。它在处理产品特性时也能很好地工作。有些产品，如服装，可能具有以小、中、大来表示的尺码，同一集合中的其他产品可以用体积表示，其它的可以用实际尺寸或重量来表示。

一个资产管理领域的客户最近使用属性模式部署了他们的解决方案。客户使用该模式存储给定资产的所有特征。这些特征在资产中很少常见，或者在设计时很难预见到。关系模型通常使用复杂的设计过程以[用户定义字段](https://martinfowler.com/bliki/UserDefinedField.html)的形式表达这样的思想。

虽然产品目录中的许多字段类似，例如名称、供应商、制造商、原产地等，但产品的规格或属性可能有所不同。如果应用程序和数据访问模式依赖于需要同时搜索这些不同字段，那么属性模式为数据提供了一个良好的结构。

## 结论

属性模式针对每个文档中许多类似字段提供了更简单的文档索引。通过将这个数据子集移动到一个键值子文档中，我们可以使用不确定的字段名，为信息添加额外的限定符，并更清楚地说明原始字段和值的关系。当我们使用属性模式时，由于需要的索引更少，查询变得更简单更快。

我们将讨论的下一个模式是[桶模式（Bucket Pattern）](http://mongoing.com/archives/24190)。

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