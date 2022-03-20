# [为什么有很多人说 *Go* 语言不需要依赖注入？](https://www.zhihu.com/question/265433666/answer/337599960)

作者：阿莱克西斯
链接：https://www.zhihu.com/question/265433666/answer/337599960
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



更新下：其实我觉得依赖倒置原则是思路，是系统构建的大方向和哲学，最接近这个哲学的工具是First Class Module和Existance Type。而很多主牛语言根本没这种东西。那么一个比较通用的工具就是DI.

------

可能文章写得太长了。。。 其实简化来说就两点：

建议把2个逻辑分开：

各个components自己的逻辑

怎么把多个小components组合起来形成高一层组件的组合逻辑(也就是控制组合流，你可以手动DI，或者用任何类型的controller，或者用DI Framework，或者你使用的语言直接支持，或者利用隐式环境，关键是单独来完成这件事，别把怎么组合自己和自己的dependency这件事情，让自己来决定；也就是别把components的逻辑和components怎么和别人的实例组合的逻辑放在一起)

做到这点需要component不能把自己的依赖写死(不能自己去创建自己需要的dependency的实例)，而是只是声明自己需要的spec(一般是interface；如果只存这个spec/interface的一种实例，那么懒得定义interface用实例的类型做依赖声明也可以，需要存在不同实例的时候再重构抽出interface就行了，但是别自己create自己的依赖。比如sort的实现把要sort的compareTo的实例固定，那么sort就只能sort一种T）这就是广义的DI。 **无法把广义DI和DI framework分开思考的人就别评论了。谢。**

只有很少几种情况components可以直接决定自己的依赖，比如这个依赖绝对不会更改(这意味着你想连UT都把这个components和依赖必须一起测，因为写死了)，或者这个实例在世界上只存在一种(比如你依赖于一些[数学函数](https://www.zhihu.com/search?q=数学函数&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"337599960"})，Math.abs) 这些东西只存在一种实现，绝对不会变，即使放在UT里。



注意：如果需要component控制dependency生成的时机timing，可以inject进来dependency的factory。那么component虽然控制dependency instance生成的时机，但是不知道生成的具体实例类型，同样松耦合了各个Components

**再次注意：我个人不喜欢[DI Framework](https://www.zhihu.com/search?q=DI+Framework&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"337599960"})，是手动DI，或者用[Reader Monad](https://www.zhihu.com/search?q=Reader+Monad&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"337599960"})一派的，友军请别乱开枪。**

------

看了很多答案，感觉都没提到依赖注入对系统设计的重要性，所以其实我更想说一下依赖倒置原则和系统设计…

不过先答正题：Go当然也可以做[依赖注入](https://www.zhihu.com/search?q=依赖注入&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"337599960"})：["Dependency Injection" in Golang](https://link.zhihu.com/?target=http%3A//openmymind.net/Dependency-Injection-In-Go/)

因为依赖注入也可以手动完成，而根本不需要DI framework。所以任何支持subtyping或者[first class function](https://www.zhihu.com/search?q=first+class+function&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"337599960"})的语言都完全可以做dependency injection；甚至在C里也可以用函数指针做DI。



有位答主问：

> 我更好奇的是，这世上有一离开“依赖注入”就玩不转的项目吗？

我想就这个问题讲一下依赖注入，依赖倒置，和系统设计。



## 依赖注入的一般应用

依赖注入可以让让“提供功能类A”和“使用功能类B”解耦合；A注入在B里，那么如果A有subtyping的变换A_x*，*A_y...时，而整体系统调整(比如新feature，测试环境和prod环境)需要B在不同的场景/时间注入其他具体功能实现类A_x...时； B都不需要改，且系统可以通过不同的配置轻松实现在不同场景的多版本系统。最常见的就是业务类使用Dao类，不要在业务类里new具体Dao类，这样就绑死了关系，而注入Dao类，这样则任何Dao的subtyping，业务类都可以接受。



## 依赖注入与依赖倒置

其实依赖注入最重要的作用之一是实现依赖倒置；（控制反转和依赖倒置不同，最后说）

**<划重点>让任何B调A的关系，不都产生B必须绑死依赖于A的效果</划重点>。**

当B必须用到A的功能的时候，控制流是从B到A，A不需要知道B(最直观的proof A不需要import B)，而B必须知道A，或者说B直接依赖于A(直观proof就是B需要import A才能用A)，依赖关系箭头从B到A；这样会导致**类依赖图**甚至**包依赖图**(如果A和B在不同包)是从B指向(依赖)A。

但是如果有一个interface类C在B的包里(甚至自已的一个interface包里)，让A实现C(**注意这样A import C依赖于C**)，让B使用C的interface(B也依赖C)来间接使用C的implementation A，A的实例通过依赖注入到B里供B使用；这样依赖关系就变成了A依赖C，而如果C在B的包里(为什么C放在B的包里合理请往下看)，依赖关系就变成了A所在的包依赖于B所在的包(注意本来是B包依赖A包)。从而实现了依赖的倒置。



## 依赖倒置原则与系统设计

为什么要这么麻烦做依赖倒置？如果你看过我写的这篇关于系统分层的文章：[用谁都能看懂的方式来解释系统设计中的分层](https://zhuanlan.zhihu.com/p/32844598)，那么应该明白一个稳定易拓展的系统应该让底层依赖高层，而不是相反(高层依赖底层)，一个合理的系统，应该由高层按照自己的需求，指定底层应该满足的specification(比如B包是一个高层业务包，那么B里的interface C就是B对于底层的命令式的Specification，然后B里的所有逻辑都应该基于这个specification来写，因为B和B的Specification, 也就是interface C都是在高层抽象讨论问题，所以B和Interface C放在一个高层包里是一种合理设计)，底层应该想方设法来满足这个specification(也就是底层类A需要实现interface C)；而不应该让业务类扭曲自己来迎合底层类的需要(非依赖倒置的情况)；最终实现整个系统的高层底层配合；

用DDD的话来说就是，**业务决定技术实现， 而不是相反**。

不使用依赖反转的系统构架，**控制流和依赖关系流的依赖箭头是一个方向的**，由高层指向底层，也就是高层依赖底层，最明显的特点就是高层包需要import很多底层包里的类，这样的话任何的底层小改动，都可能产生影响高层业务逻辑类的改动的[蝴蝶效应](https://www.zhihu.com/search?q=蝴蝶效应&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"337599960"})；这样的系统耦合严重，维护，模拟，测试，实验和拓展都很困难(想想你一个业务类绑死了数据库类，你怎么做UT？怎么在Continuous Delivery的pipeline里模拟数据库做[integration test](https://www.zhihu.com/search?q=integration+test&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"337599960"})？怎么能简单的把业务数据写替换成写到SQS里，换成写到S3里，写到Kinesis里？ 或者能同时都写)，可能动不动就要重构和[re-architecture](https://www.zhihu.com/search?q=re-architecture&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"337599960"})；让程序员苦不堪言。而使用依赖倒置，使得底层包依赖高层包。高层包里不会import任何一个底层类。只要interface设计的好，底层的任何变动，都不会影响高层一行code。

**严重注意**⚠️：一个系统绝不仅仅只有高层和底层2层，而是可以有很多层，业务类也可以并需要分为多层，同时保证依赖倒置！



## 关于控制反转

顺便提一下控制反转，比较容易和依赖倒置原则弄混(太早学的这些，我在开始写这个答案的时候也弄混了这些名词。。。)，控制反转是EJB，Spring这种框架平台兴起的时候提起的一个概念。相对于Lib来说，我们总是自己application的逻辑调用Lib的逻辑，控制流由我们的application规定，EJB，Spring是让Framework调用我们的逻辑。在这个控制流的方向上，体现了控制反转。这个原则也称为Hollywood Principle - "Don't call us, we'll call you"。

其实Template Method这个设计模式也体现了控制反转，父类实现一些框架方法并定义控制流如何走，控制流会按照定义好的顺序调用一些abstract方法，然后留给子类去实现这些abstract方法。JUnit框架就是一个好例子。

控制反转是当系统的很多东西都能够模版化的时候，而业务逻辑只是其中的某一步或者几步，那么framework一般需要DI的方式来管理你的领域(业务)类，来inject到自己的控制流里，成为framework规定的控制流的一部分，而你的业务类则不需要关心控制流。可以看出，当控制流可以framework化和模版化的时候，控制反转相当重要（EJB，Spring，JUnit都是例子）。



## DI Framework

Dependency Framework, 就是Spring，GoogleGuice，MacWire(一个scala DI framework), 这种通过config file或者annotation等标记来帮助你自动DI的框架，那么到底是手动wire手动DI好，还是用DI framework好呢？

要评价这个就不免带个人意见色彩了。。。 我个人不是很喜欢DI framework，因为我喜欢函数风格的程序，DI对于函数式来说不过是high order function罢了。我个人更倾向于[MacWire](https://link.zhihu.com/?target=https%3A//github.com/adamw/macwire)的作者的意见, [In today’s post-OO world, is dependency injection still relevant?](https://link.zhihu.com/?target=http%3A//www.warski.org/blog/2015/02/in-todays-post-oo-world-is-dependency-injection-still-relevant/)  就是DI以后可能会设计为语言自己的功能，由语言来提供一个隐式“environmental” parameter。

Java，Scala里的DI framework有两个好处：

可以把wiring各个组件，类的code变成config，这样严格区分了“组件功能逻辑”和“控制组合逻辑”，降低了功能逻辑 泄漏到 “控制组合逻辑”的类里的风险(其实这很经常发生，我review code就经常看到新手SDE在组合逻辑里放[业务逻辑](https://www.zhihu.com/search?q=业务逻辑&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"337599960"})...) 而使用DI framework，比如Spring配置文件，根本就没办法写逻辑，只能老老实实指定谁怎么实例化和谁跟谁组合，所以避开了这些风险。

可以用反射等实例化非public可见，而是包可见，甚至private的类，而手动wire则需要所有需要实例化的类是public可见的。比如我们的例子里底层的功能类A必须是public可见，我们的手动wire所有高层底层组件的逻辑才能够实例化A。使用DI framework使得我们可以把A设为包可见甚至private，这样除了DI的[wire logic](https://www.zhihu.com/search?q=wire+logic&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"337599960"})，没有任何类能够实例化A，保证了A不被滥用。当然，如果你上了java9，你就可以用modular功能指定A只能被wire logic所在的包看到，所以上了java9的项目，在这点上DI framework就毫无优势了。



**依赖注入，可说是任何程序员必掌握的技巧之一，系统设计入门基础技能之一。而依赖倒置原则作为一条原则，自从被人发现以来，从来都没被颠覆过，可以算得上软件设计艺术的真理之一。**

**学会一门语言的语法，并不代表你“真的会写程序”了。正如你学会了铅笔的构造原理，你也没法像画家一样画画。真正的画家用任何画笔都能画画，只是有些画笔用着更顺手罢了。**

（最后推荐Clean Architecture这本书，除了依赖倒置原则，老司机还介绍了好几种其他[系统构架](https://www.zhihu.com/search?q=系统构架&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A"337599960"})的小技巧，推荐阅读。）

最后：在container，虚拟化，微服务，甚至serverless，FaaS盛行的今天，几百行code就可以是一个service，遵循依赖倒置原则来设计稳定的易拓展的系统，跟代码行数超过多少行有什么必然联系么？？？ 