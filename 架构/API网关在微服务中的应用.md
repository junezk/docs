# API网关在微服务中的应用

# **前言**

现在的互联网产品技术架构，**如果没有上微服务架构**，都感觉被同行鄙视，太low了。在微服务架构中，**不同的微服务有不同的请求地址**，各个微服务之间通过**互相调用**完成用户请求。**客户端要完成用户请求，需要调用很多微服务接口。**

比如：

> 用户查看一个商品详情页，详情页包含了商品基本信息，商品价格，库存信息，评论信息，促销活动信息等，**而这些信息是不同的微服务提供的**；如：库存服务，促销服务，评论系统等。

![img](API网关在微服务中的应用.assets/16cf663ed71c1a5c)

用户要查看商品详情页，需要**让客户端调用多个微服务**，且**客户端直接与各个微服务通信**，会有以下的问题：

> 1、客户端**多次请求不同的微服务**，增加了客户端的复杂度。
>
> 2、**多次网络请求**，耗时增加。
>
> 3、微服务的请求**地址不同，很容易引起跨域问题。**
>
> 4、客户端请求微服务需要保证安全认证，但**每个服务都要进行认证。**
>
> 5、将来的项目重构，微服务的变化，如：把多个服务合并一个服务，或一个服务拆分多个服务；这样会**导致客户端请求需要重构**。
>
> 6、**限流、降级、监控等需求**，会导致实现复杂，每个服务都要实现，重复代码。

> 遇到这些问题，**我们怎么去解决呢**？我们只要增加一个API网关。

# **什么是API网关？**

API网关是一个服务器，是**系统的唯一入口**。从面向对象设计的角度看，**它与外观模式类似**。API网关封装了系统内部架构，为每个客户端**提供一个API入口**。

> API拥有一些职责，如**身份验证、监控、负载均衡、缓存、流控**。API网关方式的核心要点是，所有**客户端和消费端都通过统一的网关接入微服务**，在网关层处理所有的非业务功能。

简洁图：

![img](API网关在微服务中的应用.assets/16cf663ee7197789)



生产图：



![img](API网关在微服务中的应用.assets/16cf663ee178ca08)

# **网关优点**

通过上图中API网关做为**系统统一入口**，实现了对**各个微服务间的整合**，同时又做到了对客户端友好，**屏蔽系统的复杂性和差异性**。对比之前无API网关模式，API网关具有几个比较重要的优点：

> 1、网关可以和微服务注册中心连接，动态增加微服务应用，进行服务扩容
>
> 2、网关对于无法访问的服务，做到自动熔断
>
> 3、网关可以方便实现蓝绿部署、金丝雀发布
>
> 4、网关可处理微服务公共需求，简化微服务职责
>
> 5、网关可帮助客户端实现负载均衡

下面老顾就用图解的方式进行说明

**接口优化**

![img](API网关在微服务中的应用.assets/16cf663ee55cb8c9)

我们发现各个微服务的接口返回体是不一样的，在之前没有网关的时候，客户端获取商品id为1的商品信息时，需要**分别请求商品信息服务、价格服务、库存服务等**，才能获得完整的商品信息。客户端自行进行数据组装处理。有了网关这个多次接口请求就**完全的交给网关进行处理**，客户端**只要请求/good/1一个接口就行了，网关会请求多个微服务，把多个微服务返回的值进行合并**，一次性返回完整的信息。简化了客户端请求接口的复杂性。

注意：有些业务数据，**不一定要同步返回给客户端**，也许异步会更好，根据用户体验，业务需求而定。如：促销价格的话，**也许就需要客户端单独去请求了**，而**不是跟商品基础信息一起同步返回**。具体要看业务哦！

# **中心化**

![img](API网关在微服务中的应用.assets/16cf663eeb6271e8)

1、在实际业务中，很多提供的微服务接口，是需要身份认证的；

2、还有就是对外部的流量控制，防止流量过大，把整体系统搞崩溃；需要预估系统能够支撑的流量。

3、访问日志，监控分析

以上的需求，**每个微服务都需要，且跟具体业务无关**；这种类似的需求，交给网关处理，再适合不过了。由**网关统一处理，因为网关是入口，统一处理更加可控，简洁**。让微服务接口做业务相关的事情上面。这就是**中心化的思想，集中处理**。

# **负载均衡**

![img](API网关在微服务中的应用.assets/16cf663ee959bb32)

在实际的部署应用中，**当应用系统面临大量访问，负载过高时**，通常我们会增加服务数量来进行**横向扩展**，使用集群来提高系统的处理能力。此时多个服务通过某种负载算法分摊了系统的压力，我们将这种多节点分摊压力的行为称为**负载均衡**。

> 网关为入口，由网关与微服务进行交互，所以**网关必须要实现负载均衡的功能**；网关会获取**微服务注册中心里面的服务连接地址**，再配合一些算法选择其中一个服务地址，进行处理业务。这个属于客户端侧的负载均衡，由调用方去实现负载均衡逻辑。

主流注册中心Eureka、Zookeeper等； 负载均衡的算法一般有随机，轮询，权重，负载等。

# **服务熔断**

![img](API网关在微服务中的应用.assets/16cf663f05e653bf)

在现实生产环境中，会经常遇到**某个服务突然停止了工作**，然后返回了大量的错误。这个时间API网关可以实现断路器(circuit breakers)的能力,也就是说**超过了指定的阈值**，API网关就会**停止发送请求到那些失败的服务。**

> 这样就给了我们时间来**分析日志，实现修复以及发包更新**。通常当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会**发生级联故障，或者叫雪崩**。

断路器通过简单的断开流量的方式，这样就**不会有新的请求到达那些有问题的实例**，这时候我们就有相对**充分的时间来修复和解决问题**。

# **灰度发布**

又称金丝雀发布，起源是，矿井工人发现，金丝雀对瓦斯气体很敏感，矿工会在下井之前，先放一只金丝雀到井中，如果金丝雀不叫了，就代表瓦斯浓度高。

![img](https://user-gold-cdn.xitu.io/2019/9/3/16cf663f09a42a01?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

看看灰度发布逻辑

> 在灰度发布开始后，先启动一个新版本应用，但是**并不直接将流量切过来**，而是测试人员对新版本进行线上测试，启动的这个新版本应用，就是我们的金丝雀。如果没有问题，那么可以将**少量的用户流量导入到新版本上**，然后再对新版本做运行状态观察，收集各种运行时数据，如果此时**对新旧版本做各种数据对比**，就是所谓的 A/B测试。
>
> 新版本没什么问题，那么逐步扩大范围、流量，把**所有用户都迁移到新版本上面来**。

这种发布，通过网关就可以很好的实现，网关通过流控模块，进行控制分流。

**现有网关框架**

**1、Tyk**：Tyk是一个开放源码的API网关，它是快速、可扩展和现代的。Tyk提供了一个API管理平台，其中包括API网关、API分析、开发人员门户和API管理面板。Trk 是一个基于Go实现的网关服务。

**2、Kong**：Kong是一个可扩展的开放源码API Layer(也称为API网关或API中间件)。Kong 在任何RESTful API的前面运行，通过插件扩展，它提供了超越核心平台的额外功能和服务。

**3、Orange**：和Kong类似也是基于OpenResty的一个API网关程序，是由国人开发的。

**4、Netflix zuul**：Zuul是一种提供动态路由、监视、弹性、安全性等功能的边缘服务。Zuul是Netflix出品的一个基于JVM路由和服务端的负载均衡器。

**5、Spring Cloud Gateway**：Spring Cloud Gateway是基于Spring 框架5.0版本和Spring Boot 2.0的版本构建，提供路由等功能。

**6、apiaxle**：Nodejs 实现的一个 API 网关。

**7、api-umbrella**：Ruby 实现的一个 API 网关。

# **总结**

今天老顾有关网关的功能，就介绍到这里；网关是个非常核心组件，小伙伴们要多多了解，实战哦。