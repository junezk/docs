# [30分钟理解GraphQL核心概念](https://segmentfault.com/a/1190000014131950)



写在前面
----

在上一篇文章[RPC vs REST vs GraphQL](https://segmentfault.com/a/1190000013961872)中，对于这三者的优缺点进行了比较宏观的对比，而且我们也会发现，一般比较简单的项目其实并不需要GraphQL，但是我们仍然需要对新的技术有一定的了解和掌握，在新技术普及时才不会措手不及。

这篇文章主要介绍一些我接触GraphQL的这段时间，觉得需要了解的比较核心的概念，比较适合一下人群：

*   听说过GraphQL的读者，想深入了解一下
*   想系统地学习GraphQL的读者
*   正在调研GraphQL技术的读者

这些概念并不局限于服务端或者是客户端，如果你熟悉这些概念，在接触任意使用GraphQL作为技术背景的库或者框架时，都可以通过文档很快的上手。

如果你已经GraphQL应用于了实际项目中，那么这篇文章可能不适合你，因为其中并没有包含一些实践中的总结和经验，关于实践的东西我会在之后再单另写一篇文章总结。

什么是GraphQL
----------

介绍GraphQL是什么的文章网上一搜一大把，篇幅有长有短，但是从最核心上讲，它是一种查询语言，再进一步说，是一种API查询语言。

这里可能有的人就会说，什么？API还能查？API不是用来调用的吗？是的，这正是GraphQL的强大之处，引用官方文档的一句话：

> ask exactly what you want.

我们在使用REST接口时，接口返回的数据格式、数据类型都是后端预先定义好的，如果返回的数据格式并不是调用者所期望的，作为前端的我们可以通过以下两种方式来解决问题：

*   和后端沟通，改接口（更改数据源）
*   自己做一些适配工作（处理数据源）

一般如果是个人项目，改后端接口这种事情可以随意搞，但是如果是公司项目，改后端接口往往是一件比较敏感的事情，尤其是对于三端（web、andriod、ios）公用同一套后端接口的情况。大部分情况下，均是按第二种方式来解决问题的。

因此如果接口的返回值，可以通过某种手段，从静态变为动态，即调用者来声明接口返回什么数据，很大程度上可以进一步解耦前后端的关联。

在GraphQL中，我们通过预先定义一张`Schema`和声明一些`Type`来达到上面提及的效果，我们需要知道：

*   对于数据模型的抽象是通过Type来描述的
*   对于接口获取数据的逻辑是通过Schema来描述的

这么说可能比较抽象，我们一个一个来说明。

Type
----

对于数据模型的抽象是通过Type来描述的，每一个Type有若干Field组成，每个Field又分别指向某个Type。

GraphQL的Type简单可以分为两种，一种叫做`Scalar Type(标量类型)`，另一种叫做`Object Type(对象类型)`。

### Scalar Type

GraphQL中的内建的标量包含，`String`、`Int`、`Float`、`Boolean`、`Enum`，对于熟悉编程语言的人来说，这些都应该很好理解。

值得注意的是，GraphQL中可以通过`Scalar`声明一个新的标量，比如：

*   prisma（一个使用GraphQL来抽象数据库操作的库）中，还有`DateTime`和`ID`这两个标量分别代表日期格式和主键
*   在使用GraphQL实现文件上传接口时，需要声明一个`Upload`标量来代表要上传的文件

总之，我们只需要记住，标量是GraphQL类型系统中最小的颗粒，关于它在GraphQL解析查询结果时，我们还会再提及它。

### Object Type

仅有标量是不够的抽象一些复杂的数据模型的，这时候我们需要使用对象类型，举个例子(先忽略语法，仅从字面上看)：

    type Article {
      id: ID
      text: String
      isPublished: Boolean
    }

上面的代码，就声明了一个`Article`类型，它有3个Field，分别是`ID`类型的id，`String`类型的text和`Boolean`类型的isPublished。

对于对象类型的Field的声明，我们一般使用标量，但是我们也可以使用另外一个对象类型，比如如果我们再声明一个新的`User`类型，如下：

    type User {
      id: ID
      name: String
    }

这时我们就可以稍微的更改一下关于`Article`类型的声明代码，如下：

    type Article {
      id: ID
      text: String
      isPublished: Boolean
      author: User
    }

`Article`新增的`author`的Field是`User`类型, 代表这篇文章的作者。

总之，我们通过对象模型来构建GraphQL中关于一个数据模型的形状，同时还可以声明各个模型之间的内在关联（一对多、一对一或多对多）。

### Type Modifier

关于类型，还有一个较重要的概念，即类型修饰符，当前的类型修饰符有两种，分别是`List`和`Required` ，它们的语法分别为`[Type]`和`Type!`, 同时这两者可以互相组合，比如`[Type]!`或者`[Type!]`或者`[Type!]!`(请仔细看这里`!`的位置)，它们的含义分别为：

*   列表本身为必填项，但其内部元素可以为空
*   列表本身可以为空，但是其内部元素为必填
*   列表本身和内部元素均为必填

我们进一步来更改上面的例子，假如我们又声明了一个新的`Comment`类型，如下：

    type Comment {
      id: ID!
      desc: String,
      author: User!
    }

你会发现这里的`ID`有一个`!`，它代表这个Field是必填的，再来更新`Article`对象，如下：

    type Article {
      id: ID!
      text: String
      isPublished: Boolean
      author: User!
      comments: [Comment!]
    }

我们这里的作出的更改如下：

*   id字段改为必填
*   author字段改为必填
*   新增了comments字段，它的类型是一个元素为Comment类型的List类型

最终的`Article`类型，就是GraphQL中关于文章这个数据模型，一个比较简单的类型声明。

Schema
------

现在我们开始介绍`Schema`，我们之前简单描述了它的作用，即它是用来描述`对于接口获取数据逻辑`的，但这样描述仍然是有些抽象的，我们其实不妨把它当做REST架构中每个独立资源的`uri`来理解它，只不过在GraphQL中，我们用Query来描述资源的获取方式。因此，我们可以将`Schema`理解为多个Query组成的一张表。

这里又涉及一个新的概念`Query`，GraphQL中使用`Query`来抽象数据的查询逻辑，当前标准下，有三种查询类型，分别是_query（查询）_、_mutation（更改）_和_subscription（订阅）_。

Note: 为了方便区分，`Query`特指GraphQL中的查询（包含三种类型），`query`指GraphQL中的查询类型（仅指查询类型）

### Query

上面所提及的3中基本查询类型是作为`Root Query（根查询）`存在的，对于传统的CRUD项目，我们只需要前两种类型就足够了，第三种是针对当前日趋流行的`real-time`应用提出的。

我们按照字面意思来理解它们就好，如下：

*   query（查询）：当获取数据时，应当选取Query类型
*   mutation（更改）：当尝试修改数据时，应当使用mutation类型
*   subscription（订阅）：当希望数据更改时，可以进行消息推送，使用subscription类型

仍然以一个例子来说明。

首先，我们分别以REST和GraphQL的角度，以`Article`为数据模型，编写一系列CRUD的接口，如下：

Rest 接口

    GET /api/v1/articles/
    GET /api/v1/article/:id/
    POST /api/v1/article/
    DELETE /api/v1/article/:id/
    PATCH /api/v1/article/:id/


GraphQL Query

    query {
      articles(): [Article!]!
      article(id: Int): Article!
    }
    
    mutation {
      createArticle(): Article!
      updateArticle(id: Int): Article!
      deleteArticle(id: Int): Article!
    }

对比我们较熟悉的REST的接口我们可以发现，GraphQL中是按根查询的类型来划分Query职能的，同时还会明确的声明每个Query所返回的数据类型，这里的关于类型的语法和上一章节中是一样的。需要注意的是，我们所声明的任何`Query`都必须是`Root Query`的子集，这和GraphQL内部的运行机制有关。

例子中我们仅仅声明了Query类型和Mutation类型，如果我们的应用中对于评论列表有`real-time`的需求的话，在REST中，我们可能会直接通过长连接或者通过提供一些带验证的获取长连接url的接口，比如：

    POST /api/v1/messages/

之后长连接会将新的数据推送给我们，在GraphQL中，我们则会以更加声明式的方式进行声明，如下

    subscription {
      updatedArticle() {
        mutation
        node {
            comments: [Comment!]!
        }
      }
    }

我们不必纠结于这里的语法，因为这篇文章的目的不是让你在30分钟内学会GraphQL的语法，而是理解的它的一些核心概念，比如这里，我们就声明了一个订阅Query，这个Query会在有新的Article被创建或者更新时，推送新的数据对象。当然，在实际运行中，其内部实现仍然是建立于长连接之上的，但是我们能够以更加声明式的方式来进行声明它。

### Resolver

如果我们仅仅在Schema中声明了若干Query，那么我们只进行了一半的工作，因为我们并没有提供相关Query所返回数据的逻辑。为了能够使GraphQL正常工作，我们还需要再了解一个核心概念，`Resolver（解析函数）`。

GraphQL中，我们会有这样一个约定，Query和与之对应的Resolver是同名的，这样在GraphQL才能把它们对应起来，举个例子，比如关于`articles(): [Article!]!`这个Query, 它的Resolver的名字必然叫做`articles`。

在介绍Resolver之前，是时候从整体上了解下GraphQL的内部工作机制了，假设现在我们要对使用我们已经声明的`articles`的Query，我们可能会写以下查询语句（同样暂时忽略语法）：

    Query {
      articles {
           id
           author {
               name
           }
           comments {
          id
          desc
          author
        }
      }
    }

GraphQL在解析这段查询语句时会按如下步骤（简略版）：

*   首先进行第一层解析，当前`Query`的`Root Query`类型是`query`，同时需要它的名字是`articles`
*   之后会尝试使用`articles`的`Resolver`获取解析数据，第一层解析完毕
*   之后对第一层解析的返回值，进行第二层解析，当前`articles`还包含三个子`Query`，分别是`id`、`author`和`comments`

    *   id在Author类型中为标量类型，解析结束
    *   author在Author类型中为对象类型User，尝试使用`User`的`Resolver`获取数据，当前field解析完毕
    *   之后对第二层解析的返回值，进行第三层解析，当前`author`还包含一个`Query`, `name`，由于它是标量类型，解析结束
    *   comments同上...

我们可以发现，GraphQL大体的解析流程就是遇到一个Query之后，尝试使用它的Resolver取值，之后再对返回值进行解析，这个过程是递归的，直到所解析Field的类型是`Scalar Type（标量类型）`为止。解析的整个过程我们可以把它想象成一个很长的Resolver Chain（解析链）。

这里对于GraphQL的解析过程只是很简单的概括，其内部运行机制远比这个复杂，当然这些对于使用者是黑盒的，我们只需要大概了解它的过程即可。

Resolver本身的声明在各个语言中是不一样的，因为它代表数据获取的具体逻辑。它的函数签名(以js为例子)如下：

    function(parent, args, ctx, info) {
        ...
    }

其中的参数的意义如下：

*   parent: 当前上一个Resolver的返回值
*   args: 传入某个Query中的函数（比如上面例子中`article(id: Int)`中的`id`）
*   ctx: 在Resolver解析链中不断传递的中间变量（类似中间件架构中的context）
*   info: 当前Query的AST对象

值得注意的是，Resolver内部实现对于GraphQL完全是黑盒状态。这意味着Resolver如何返回数据、返回什么样的数据、从哪返回数据，完全取决于Resolver本身，基于这一点，在实际中，很多人往往把GraphQL作为一个中间层来使用，数据的获取通过Resolver来封装，内部数据获取的实现可能基于RPC、REST、WS、SQL等多种不同的方式。同时，基于这一点，当你在对一些未使用GraphQL的系统进行迁移时（比如REST），可以很好的进行增量式迁移。

总结
--

大概就这么多，首先感谢你耐心的读到这里，虽然题目是30分钟熟悉GraphQL核心概念，但是可能已经超时了，不过我相信你对GraphQL中的核心概念已经比较熟悉了。但是它本身所涉及的东西远远比这个丰富，同时它还处于飞速的发展中。

最后我尝试根据这段时间的学习GraphQL的经验，提供一些进一步学习和了解GraphQL的方向和建议，仅供参考：

### 想进一步了解GraphQL本身

我建议再仔细去官网，读一下官方文档，如果有兴趣的话，看看GraphQL的spec也是极好的。这篇文章虽然介绍了核心概念，但是其他一些概念没有涉及，比如Union、Interface、Fragment等等，这些概念均是基于核心概念之上的，在了解核心概念后，应当会很容易理解。

### 偏向服务端

偏向服务端方向的话，除了需要进一步了解GraphQL在某个语言的具体生态外，还需要了解一些关于缓存、上传文件等特定方向的东西。如果是想做系统迁移，还需要对特定的框架做一些调研，比如graphene-django。

如果是想使用GraphQL本身做系统开发，这里推荐了解一个叫做[prisma](https://www.prisma.io/docs/)的框架，它本身是在GraphQL的基础上构建的，并且与一些GraphQL的生态框架兼容性也较好，在各大编程语言也均有适配，它本身可以当做一个ORM来使用，也可以当做一个与数据库交互的中间层来使用。

### 偏向客户端

偏向客户端方向的话，需要进一步了解关于graphql-client的相关知识，我这段时间了解的是apollo，一个开源的grapql-client框架，并且与各个主流前端技术栈如Angular、React等均有适配版本，使用感觉良好。

同时，还需要了解一些额外的查询概念，比如分页查询中涉及的Connection、Edge等。

大概就这么多，如有错误，还望指正。