# [领域驱动设计(DDD)实践之路(四)：领域驱动在微服务设计中的应用](https://my.oschina.net/vivotech/blog/4807494)

这是“领域驱动设计实践之路”系列的第四篇文章，从单体架构的弊端引入微服务，结合领域驱动的概念介绍了如何做微服务划分、设计领域模型并展示了整体的微服务化的系统架构设计。结合分层架构、六边形架构和整洁架构的思想，以实际使用场景为背景，展示了一个微服务的程序结构设计。



# 一、单体架构的弊端

![img](领域驱动设计(DDD)实践之路(四)：领域驱动在微服务设计中的应用.assets/9aa38a23b25eac5a5cfe787e7d85f17c.webp)

单体结构示例(引用自互联网)

一般在业务发展的初期，整个应用涉及的功能需求较少，相对比较简单，单体架构的应用比较容易部署、测试，横向扩展也比较易实现。

然而，随着需求的不断增加， 越来越多的人加入开发团队，代码库也在飞速地膨胀。慢慢地，单体应用变得越来越臃肿，可维护性、灵活性逐渐降低，维护成本越来越高。

下面分析下单体架构应用存在的一些弊端：



## **1、复杂性高**

在项目初期应该有人可以做到对应用各个功能和实现了如指掌，随着业务需求的增多，各种业务流程错综复杂的揉在一起，整个系统变得庞大且复杂，以至于很少有开发者清楚每一个功能和业务流程细节。

这样会使得新业务的需求评估或者异常问题定位会占用较多的时间，同时也蕴含着未知风险。更糟糕的是，这种极度的复杂性会形成一种恶性循环，每一次更改都会使得系统变得更复杂，更难懂。



## **2.技术债务多**

随着时间推移、需求变更和人员更迭，会逐渐形成应用程序的技术债务，并且越积越多。比如，团队必须长期使用一套相同的技术栈，很难采用新的框架和编程语言。有时候想引入一些新的工具时，就会使得项目中需要同时维护多套技术框架，比如同时维护Hibernate和Mybatis，使得成本变高。



## **3.错误难隔离**

由于业务项目的所有功能模块都在一个应用上承担，包括核心和非核心模块，任何一个模块或者一个小细节的地方，因为设计不合理、代码质量差等原因，都有可能造成应用实例的崩溃，从而使得业务全面受到影响。其根本原因就是核心和非核心功能的代码都运行在同一个环境中。



## **4. 项目团队间协同成本高，业务响应越来越慢**

多个类似的业务项目之间势必会存在类似的功能模块，如果都采用单体模式，就会带来重复功能建设和维护。而且，有时候还需要互相产生交互，打通单体系统之间的交互集成和协作的成本也需要额外付出。

再者，当项目大到一定程度，不同的模块可能是不同的团队来维护，迭代联调的冲突，代码合并分支的冲突都会影响整个开发进度，从而使得业务响应速度越来越慢。



## **5.扩展成本高**

随着业务的发展，系统在出现业务处理瓶颈的时候，往往是由于某一个或几个功能模块负载较高造成的，但因为所有功能都打包在一起，在出现此类问题时，只能通过增加应用实例的方式分担负载，没办法对单独的几个功能模块进行服务能力的扩展，从而带来资源额外配置的消耗，成本较高。

针对以上痛点，近年来越来越多的互联网公司采用“微服务”架构构建自身的业务平台，而“微服务”也获得了越来越多技术人员的肯定。

微服务其实是SOA的一种演变后的形态，与SOA的方法和原则没有本质区别。SOA理念的核心价值是，松耦合的服务带来业务的复用，按照业务而不是技术的维度，结合高内聚、低耦合的原则来划分微服务，这正好与领域驱动设计所倡导的理念相契合。



# 二、微服务设计



## **1. 微服务划分**

从广义上讲，领域即是一个组织所做的事情以及其中包含的一切。每个组织都有它自己的业务范围和做事方式，这个业务范围以及在其中所进行的活动便是领域。

DDD的子域和限界上下文的概念，可以很好地跟微服务架构中的服务进行匹配。而且，微服务架构中的自治化团队负责服务开发的概念，也与DDD中每个领域模型都由一个独立团队负责开发的概念吻合。DDD倡导按业务领域来划分系统，微服务架构更强调从业务维度去做分治来应对系统复杂度，跳过业务架构设计出来的架构关注点不在业务响应上，可能就是个大泥球，在面临需求迭代或响应市场变化时就很痛苦。

DDD的核心诉求就是将业务架构映射到系统架构上，在响应业务变化调整业务架构时，也随之变化系统架构。而微服务追求业务层面的复用，设计出来的系统架构和业务一致；在技术架构上则系统模块之间充分解耦，可以自由地选择合适的技术架构，去中心化地治理技术和数据。

![img](领域驱动设计(DDD)实践之路(四)：领域驱动在微服务设计中的应用.assets/e72794794c16d58c2cef73f46d0dc25a.webp)

以电商的资源订购系统为例，典型业务用例场景包括查看资源，购买资源，查询用户已购资源等。

领域驱动为每一个子域定义单独的领域模型，子域是领域的一部分，从业务的角度分析我们需要覆盖的业务用例场景，以高内聚低耦合的思想，结合单一职责原则(SRP)和闭包原则(CCP)，从业务领域的角度，划分出用户管理子域，资源管理子域，订单子域和支付子域共四个子域。

每个子域对应一个限界上下文。限界上下文是一种概念上的边界，领域模型便工作于其中，每个限界上下文都有自己的通用语言。限界上下文使得你在领域模型周围加上了一个显式的、清晰的边界。当然，限界上下文不仅仅包含领域模型。当使用微服务架构时，每个限界上下文对应一个微服务。



## **2.** **领域模型**

![img](领域驱动设计(DDD)实践之路(四)：领域驱动在微服务设计中的应用.assets/080361bf43b01a3e9e6ae57c2563d686.webp)

聚合是一个边界内领域对象的集群，可以将其视为一个单元，它由根实体和可能的一个或多个其他实体和值对象组成。聚合将领域模型分解为**块**，每个聚合都可以作为一个单元进行处理。

聚合根是聚合中唯一可以由外部类引用的部分，客户端只能通过调用聚合根上的方法来更新聚合。

聚合代表了一致的边界，对于一个设计良好的聚合来说，无论由于何种业务需求而发生改变，在单个事务中，聚合中的所有不变条件都是一致的。聚合的一个很重要的经验设计原则是，一个事务中只修改一个聚合实例。更新聚合时需要更新整个聚合而不是聚合中的一部分，否则容易产生一致性问题。

比如A和B同时在网上购买东西，使用同一张订单，同时意识到自己购买的东西超过预算，此时A减少点心数量，B减少面包数量，两个消费者并发执行事务，那么订单总额可能会低于最低订单限额要求，但对于一个消费者来说是满足最低限额要求的。所以应该站在聚合根的角度执行更新操作，这会强制执行一致性业务规则。

另外，我们不应该设计过大的聚合，处理大聚合构成的"巨无霸"对象时，容易出现不同用例同时需要修改其中的某个部分，因为聚合设计时考虑的一致性约束是对整个聚合产生作用的，所以对聚合的修改会造成对聚合整体的变更，如果采用乐观并发，这样就容易产生某些用例会被拒绝的场景，而且还会影响系统的性能和可伸缩性。

使用大聚合时，往往为了完成一项基本操作，需要将成百上千个对象一同加载到内存中，造成资源的浪费。所以应尽量采用小聚合，一方面使用根实体来表示聚合，其中只包含最小数量的属性或值类型属性，这里的最小数量表示所需的最小属性集合，不多也不少。必须与其他属性保持一致的属性是所需的属性。

在聚合中，如果你认为有些被包含部分应该建模成一个实体，此时，思考下这个部分是否会随着时间而改变，或者该部分是否能被全部替换。如果可以全部替换，那么可以建模成值对象，而非实体。因为值对象本身是不可变的，只能进行全部替换，使用起来更安全，所以，一般情况下优先使用值对象。很多情况下，许多建模成实体的概念都可以重构成值对象。小聚合还有助于事务的成功执行，即它可以减少事务提交冲突，这样不仅可以提升系统的性能和可伸缩性，另外系统的可用性也得到了增强。

另外聚合直接的引用通过唯一标识实现，而不是通过对象引用，这样不仅减少聚合的使用空间，更重要的是可以实现聚合直接的松耦合。如果聚合是另一个服务的一部分，则不会出现跨服务的对象引用问题，当然在聚合内部对象之间是可以相互引用的。

上述关于聚合的主要使用原则总结起来可以归纳为以下几点：

1. 只引用聚合根。
2.  通过唯一标识引用其他聚合。
3. 一个事务中只能创建或修改一个聚合。
4. 聚合边界之外使用最终一致性。

当然在实际使用的过程中，比如某一个业务用例需要获取到聚合中的某个领域对象，但该领域对象的获取路径较繁琐，为了兼容该特殊场景，可以将聚合中的属性(实体或值对象)直接返回给应用层，使得应用层直接操作该领域对象。

我们经常会遇到在一个聚合上执行命令方法时，还需要在其他聚合上执行额外的业务规则，尽量使用最终一致性，因为最终一致性可以按聚合维度分步骤处理各个环节，从而提升系统的吞吐量。对于一个业务用例，如果应该由执行该用例的用户来保证数据的一致性，那么可以考虑使用事务一致性，当然此时依然需要遵循其他聚合原则。如果需要其他用户或者系统来保证数据一致性，那么使用最终一致性。实际上，最终一致性可以支持绝大部分的业务场景。

基于上面对电商的资源订购系统业务子域的划分，设计出资源聚合，订单聚合，支付聚合和用户聚合，资源聚合与订单聚合之间通过资源ID进行关联，订单聚合与支付聚合之间通过订单ID和用户ID进行关联，支付聚合和用户聚合之间通过用户ID进行关联。资源聚合根中包含多个资源包值对象，一个资源包值对象又包含多个预览图值对象。当然在实际开发的过程中，根据实际情况聚合根中也可以包含实体对象。每个聚合对应一个微服务，对于特别复杂的系统，一个子域可能包含多个聚合，也就包含多个微服务。



## **3.** **微服务系统架构设计**

![img](领域驱动设计(DDD)实践之路(四)：领域驱动在微服务设计中的应用.assets/af4e8c8bf3358e8ceb9f75f3e38400ad.webp)

 

基于上面对电商的资源订购系统子域的分析，服务器后台使用用户服务，资源服务，订单服务和支付服务四个微服务实现。上图中的API Gateway也是一种服务，同时可以看成是DDD中的应用层，类似面向对象设计中的外观(Facade)模式。

作为整个后端架构的统一门面，封装了应用程序内部架构，负责业务用例的任务协调，每个用例对应了一个服务方法，调用多个微服务并将聚合结果返回给客户端。它还可能有其他职责，比如身份验证，访问授权，缓存，速率限制等。以查询已购资源为例，API Gateway需要查询订单服务获取当前用户已购的资源ID列表，然后根据资源ID列表查询资源服务获取已购资源的详细信息，最终将聚合结果返回给客户端。

当然在实际应用的过程中，我们也可以根据API请求的复杂度，从业务角度，将API Gateway划分为多个不同的服务，防止又回归到API Gateway的单体瓶颈。

另外，有时候从业务领域角度划分出来的某些子域比较小，从资源利用率的角度，单独放到一个微服务中有点单薄。这个时候我们可以打破一个限界上下文对应一个微服务的理念，将多个子域合并到同一个微服务中，由微服务自己的应用层实现多子域任务的协调。

所以，在我们的系统架构中可能会出现微服务级别的小应用层和API Gateway级别的大应用层使用场景，理论固然是理论，还是需要结合实际情况灵活应用。



# 三、领域驱动概念在单个微服务设计中的应用



## **1.** **架构选择分析**

![img](领域驱动设计(DDD)实践之路(四)：领域驱动在微服务设计中的应用.assets/2da278c8ab901df2acd9f17f6aaabd3d.webp)

分层架构图(引用自互联网)

![img](领域驱动设计(DDD)实践之路(四)：领域驱动在微服务设计中的应用.assets/471729841a56c221f01956c3b7114984.webp)

六边形架构图(引用自互联网)

![img](领域驱动设计(DDD)实践之路(四)：领域驱动在微服务设计中的应用.assets/ba1c46b6c37ffcbc4902e12dff2ff3a5.webp)

整洁架构图(引用自互联网)

上面整洁架构图中的同心圆分别代表了软件系统中的不同层次，通常越靠近中心，其所在的软件层次就越高。

整洁架构的依赖关系规则告诉我们，源码中的依赖关系必须只指向同心圆的内层，即由低层机制指向高层策略。换句话说，任何属于内层圆中的代码都不应该牵涉外层圆中的代码，尤其是内层圆中的代码不应该引用外层圆中代码所声明的名字，包括函数、类、变量以及一切其他有命名的软件实体。同样，外层圆使用的数据格式也不应该被内层圆中的代码所使用，尤其是当数据格式由外层圆的框架所生成时。

总之，不应该让外层圆中发生的任何变更影响到内层圆的代码。业务实体这一层封装的是整个业务领域中最通用、最高层的业务逻辑，它们应该属于系统中最不容易受外界影响而变动的部分，也就是说一般情况下我们的核心领域模型部分是比较稳定的，不应该因为外层的基础设施比如数据存储技术选型的变化，或者UI展示方式等的变化受影响，从而需要做相应的改动。

在以往的项目经验中，大多数同学习惯也比较熟悉分层架构，一般包括展示层、应用层，领域层和基础设施层。六边形架构的一个重要好处是它将业务逻辑与适配器中包含的表示层和数据访问层的逻辑分离开来，业务逻辑不依赖于表示层逻辑或数据访问层逻辑，由于这种分离，单独测试业务逻辑要容易得多。

另一个好处是，可以通过多个适配器调用业务逻辑，每个适配器实现特定的API或用户界面。业务逻辑还可以调用多个适配器，每个适配器调用不同的外部系统。所以六边形架构是描述微服务架构中每个服务的架构的好方法。

根据我们具体的实践经验，比如在我们平时的项目中最常见的就是MySQL和Redis存储，而且也很少改变为其他存储结构。这里将分层架构和六边形架构进行思想融合，目的是一方面希望我们的微服务设计结构更优美，另一方面希望在已有编程习惯的基础上，更容易接受新的整洁架构思想。

我们项目中微服务的实现结合分层架构，六边形架构和整洁架构的思想，以实际使用场景为背景，采用的应用程序结构图如下。

![img](领域驱动设计(DDD)实践之路(四)：领域驱动在微服务设计中的应用.assets/47a2e52a47a96c55ef804b18bc895000.webp)

从上图可以看到，我们一个应用总共包含应用层application，领域层domain和基础设施层infrastructure。领域服务的facade接口需要暴露给其他三方系统，所以单独封装为一个模块。因为我们一般习惯于分层架构模式构建系统，所以按照分层架构给各层命名。

站在六边形架构的角度，应用层application等同于入站适配器，基础设施层infrastructure等同于出站适配器，所以实际上应用层和基础设施层同属外层，可以认为在同一层。

facade模块其实是从领域层domain剥离出来的，站在整洁架构的角度，领域层就是内核业务实体，这里封装的是整个业务领域中最通用、最高层的业务逻辑，一般情况下核心领域模型部分是比较稳定的，不受外界影响而变动。facade是微服务暴露给外界的领域服务能力，一般情况下接口的设定应符合当前领域服务的边界界定，所以facade模块属于内核领域层。

facade接口的实现在应用层application的impl部分，符合整洁架构外层依赖内层的思想，对于impl输入端口和入站适配器，可以采用不同的协议和技术框架实现，比如dubbo或HSF等。下面对各个模块的构成进行逐一解释。



## **2. 领域层Domain**



### **工厂Factory**

对象的创建本身是一个主要操作，但被创建的对象并不适合承担复杂的装配操作。将这些职责混在一起可能会产生难以理解的拙劣设计。让客户直接负责创建对象又会使客户的设计陷入混乱，并且破坏装配对象的封装，而且导致客户与被创建对象的实现之间产生过于紧密的耦合。

复杂对象的创建是领域层的职责，但这项任务并不属于那些用于表示模型的对象。所以一般使用一个单独的工厂类或者在领域服务中提供一个构造领域对象的接口来负责领域对象的创建。

这里，我们选择给领域服务增加一个领域对象创建接口来承担工厂的角色。

```
/**
 * description: 资源领域服务
 *
 * @author Gao Ju
 * @date 2020/7/27
 */
public class ResourceServiceImpl implements ResourceService {
 
    /**
     * 创建资源聚合模型
     *
     * @param resourceCreateCommand 创建资源命令
     * @return
     */
    @Override
    public ResourceModel createResourceModel(ResourceCreateCommand resourceCreateCommand) {
        ResourceModel resourceModel = new ResourceModel();
        Long resId = SequenceUtil.generateUuid();
        resourceModel.setResId(resId);
        resourceModel.setName(resourceCreateCommand .getName());
        resourceModel.setAuthor(resourceCreateCommand .getAuthor());
        List<PackageItem> packageItemList = new ArrayList<>();
        ...
        resourceModel.setPackageItemList(packageItemList);
        return resourceModel;
    }
}
```



### **资源库Repository**

通常将聚合实例存放在资源库中，之后再通过该资源库来获取相同的实例。

如果修改了某个聚合，那么这种改变将被资源库持久化，如果从资源库中移除了某个实例，则将无法从资源库中重新获取该实例。

资源库是针对聚合维度创建的，聚合类型与资源库存在一对一的关系。

简单来说，资源库是对聚合的CRUD操作的封装。资源库内部采用哪种存储设施MySQL，MongoDB或者Redis等，对领域层来说其实是不感知的。

![img](领域驱动设计(DDD)实践之路(四)：领域驱动在微服务设计中的应用.assets/3176ac604f9f13cad6cfb0e9d609ede7.webp)

资源repository构成图

在我们的项目中采用MySQL作为资源repository的持久化存储，上图中每个DO对应一个数据库表，当然你也可以采用其他存储结构或设计为其他表结构，具体的处理流程均由repository进行封装，对领域服务来说只感知Resource聚合维度的CRUD操作，示例代码如下。

```
/**
 * description: 资源仓储
 *
 * @author Gao Ju
 * @date 2020/08/23
 */
@Repository("resourceRepository")
public class ResourceRepositoryImpl implements ResourceRepository {
 
    /**
     * 资源Mapper
     */
    @Resource
    private ResourceMapper resourceMapper;
 
    /**
     * 资源包Mapper
     */
    @Resource
    private PackageMapper packageMapper;
 
    /**
     * 资源包预览图Mapper
     */
    @Resource
    private PackagePreviewMapper packagePreviewMapper;
 
    /**
     * 创建订单信息
     *
     * @param resourceModel 资源聚合模型
     * @return
     */
    @Override
    public void add(ResourceModel resourceModel) {
        ResourceDO resourceDO = new ResourceDO();
        resourceDO.setName(resourceModel.getName());
        resourceDO.setAuthor(resourceModel.getAuthor());
        List<PackageDO> packageDOList = new ArrayList<>();
        List<PackagePreviewDO> packagePreviewDOList = new ArrayList<>();
        for (PackageItem packageItem : resourceModel.getPackageItemList()) {
            PackageDO packageDO = new PackageDO();
            packageDO.setResId(resourceModel.getResId());
            Long packageId = SequenceUtil.generateUuid();
            packageDO.setPackageId(packageId);
            for (PreviewItem previewItem: packageItem.getPreviewItemList()) {
                PackagePreviewDO packagePreviewDO = new PackagePreviewDO();
                ...
                packagePreviewDOList.add(packagePreviewDO);
            }
            packageDOList.add(packageDO);
        }
 
        resourceMapper.insert(resourceDO);
        packageMapper.insertBatch(packageDOList);
        packagePreviewMapper.insertBatch(packagePreviewDOList);
    }
}
```

你可能有疑问，按照整洁架构的思想，repository的接口定义在领域层，repository的实现应该定义在基础设施层，这样就符合外层依赖稳定度较高的内层了。

结合我们实际开发过程，一般存储结构选定或者表结构设定后，一般不太容易做很大的调整，所以就按照习惯的分层结构使用，领域层直接依赖基础设施层实现，降低编码时带来的额外习惯上的成本。



### **领域服务Service**

领域驱动强调我们应该创建充血领域模型，将数据和行为封装在一起，将领域模型与现实世界中的业务对象相映射。各类具备明确的职责划分，将领域逻辑分散到各个领域对象中。

领域中的服务表示一个无状态的操作，它用于实现特定于某个领域的任务。当某个操作不适合放在领域对象上时，最好的方式是使用领域服务。

简单总结领域服务本身所承载的职责，就是通过串联领域对象、资源库，生成并发布领域事件，执行事务控制等一系列领域内的对象的行为，为上层应用层提供交互的接口。

```
/**
 * description: 订单领域服务
 *
 * @author Gao Ju
 * @date 2020/8/24
 */
public class UserOrderServiceImpl implements UserOrderService {
 
    /**
     * 订单仓储
     */
    @Autowired
    private OrderRepository orderRepository;
 
    /**
     * 消息发布器
     */
    @Autowired
    private MessagePublisher messagePublisher;
 
    /**
     * 订单逻辑处理
     *
     * @param userOrder 用户订单
     */
    @Override
    public void createOrder(UserOrder userOrder) {
        orderRepository.add(userOrder);
        OrderCreatedEvent orderCreatedEvent = new OrderCreatedEvent();
        orderCreatedEvent.setUserId(userOrder.getUserId());
        orderCreatedEvent.setOrderId(userOrder.getOrderId());
        orderCreatedEvent.setPayPrice(userOrder.getPayPrice());
        messagePublisher.send(orderCreatedEvent);
    }
}
```

在实践的过程中，为了简单方便，我们仍然采用贫血领域模型，将领域对象自身行为和不属于领域对象的行为都放在领域服务中实现。

大部分场景领域服务返回聚合根或者简单类型，某些特殊场景也可以将聚合根中包含的实体或值对象返回给调用方。领域服务也可以同时操作多个领域对象，多个聚合，将其转换为另外的输出。

介于我们实际的使用场景，领域比较简单，领域服务只操作一个领域的对象，只操作一个聚合，由应用服务来协调多个领域对象。



## **3. 领域事件DomainEvent**

在领域驱动设计的上下文中，聚合在被创建时，或发生其他重大更改时发布领域事件，领域事件是聚合状态更改时所触发的。

领域事件命名时，一般选择动词的过去分词，因为状态改变时就代表当前事件已经发生，领域事件的每个属性都是原始类型值或值对象，比如事件ID和创建时间等，事件ID也可以用来做幂等用。

从概念上讲，领域事件由聚合负责发布，聚合知道其状态何时发生变化，从而知道要发布的事件。

由于聚合不能使用依赖注入，需要通过方法参数的形式将消息发布器传递给聚合，但这将基础设施和业务逻辑交织在一起，有悖于我们解耦设计的原则。

更好的方法是将事件发布放到领域服务中，因为服务可以使用依赖注入来获取对消息发布器的引用，从而轻松发布事件。只要状态发生变化，聚合就会生成事件，聚合方法的返回值中包括一个事件列表，并将它们返回给领域服务。

Saga是一种在微服务架构中维护数据一致性的机制，Sage由一连串的本地事务组成，每一个本地事务负责更新它所在服务的私有数据库，通过异步消息的方式来协调一系列本地事务，从而维护多个服务之间数据的最终一致性。Saga包括协同式和编排式，

我们采用协同式来实现分布式事务，发布的领域事件以命令式消息的方式发送给Saga参与方。如果领域事件是自我发布自我消费，不依赖消息中间件实现，则可以使用事件总线模式来进行管理。下面以购买资源的过程为例进行说明。



### **购买资源的过程**

- 提交创建订单请求，OrderService创建一个处于PAYING状态的UserOrder，并发布OrderCreated事件。
- UserService消费OrderCreated事件，验证用户是否可以下单，并发布UserVerified事件。
- PaymentService消费UserVerified事件，进行实际的支付操作，并发布PaySuccess事件。
- OrderService接收PaySuccess事件，将UserOrder状态改为PAY_SUCCESS。

**补偿过程**

- PaymentService消费UserVerified事件，进行实际的支付操作，若支付失败，并发布PayFailed事件。
- OrderService接收PayFailed事件，将UserOrder状态改为PAY_FAILED。

在Saga的概念中，

第1步叫可补偿性事务，因为后面的步骤可能会失败。

第3步叫关键性事务，因为它后面跟着不可能失败的步骤。第4步叫可重复性事务，因为其总是会成功。

```
/**
 * description: 领域事件基类
 *
 * @author Gao Ju
 * @date 2020/7/27
 */
public class BaseEvent {
    /**
     * 消息唯一ID
     */
    private String messageId;
 
    /**
     * 事件类型
     */
    private Integer eventType;
 
    /**
     * 事件创建时间
     */
    private Date createTime;
 
    /**
     * 事件修改时间
     */
    private Date modifiedTime;
}
 
 
/**
 * description: 订单创建事件
 *
 * @author Gao Ju
 * @date 2020/8/24
 */
public class OrderCreatedEvent extends BaseEvent {
 
    /**
     * 用户ID
     */
    private String userId;
 
    /**
     * 订单ID
     */
    private String orderId;
 
    /**
     * 支付价格
     */
    private Integer payPrice;
}
```



## **4.Facade模块**

facade和domain属于同一层，某些提供给三方使用的类定义在facade，比如资源类型枚举CategoryEnum限制三方资源使用范围，然后domain依赖facade中enum定义。

另外，根据迪米特法则和告诉而非询问原则，客户端应该尽量少地知道服务对象内部结构，通过调用服务对象的公共接口的方式来告诉服务对象所要执行的操作。

所以，我们不应该把领域模型泄露到微服务之外，对外提供facade服务时，根据领域对象包装出一个数据传输对象DTO(Data Transfer Object)，来实现和外部三方系统的交互，比如上图中的ResourceDTO。



## **5.****应用层Application**

应用层是业务逻辑的入口，由入站适配器调用。facade的实现，定时任务的执行和消息监听处理器都属于入站适配器，所以他们都位于应用层。

正常情况下一个微服务对应一个聚合，实践过程中，某些场景下一个微服务可以包含多个聚合，应用层负责用例流的任务协调。领域服务依赖注入应用层，通过领域服务执行领域业务规则，应用层还会处理授权认证，缓存，DTO与领域对象之间的防腐层转换等非领域操作。

```
/**
 * description: 订单facade
 *
 * @author Gao Ju
 * @date 2020/8/24
 */
public class UserOrderFacadeImpl implements UserOrderFacade {
 
    /**
     * 订单服务
     */
    @Resource
    private UserOrderService userOrderService;
 
    /**
     * 创建订单信息
     *
     * @param orderPurchaseParam 订单交易参数
     * @return
     */
    @Override
    public FacadeResponse<UserOrderPurchase> createOrder(OrderPurchaseParam orderPurchaseParam ) {
        UserOrder userOrder = new UserOrder();
        userOrder.setUserId(request.getUserId());
        userOrder.setResId(request.getResId());
        userOrder.setPayPrice(request.getPayAmount());
        userOrder.setOrderStatus(OrderStatusEnum.Create.getCode());
        userOrderService.handleOrder(userOrder);
        userOrderPurchase.setOrderId(userOrderDO.getId());
        userOrderPurchase.setCreateTime(new Date());
        return FacadeResponseFactory.getSuccessInstance(userOrderPurchase);
    }
}
```



## **6.基础设施层 Infrastructure**

基础设施的职责是为应用程序的其他部分提供技术支持。与数据库的交互dao模块，与Redis缓存，本地缓存交互的cache模块，与参数中心，三方rpc服务的交互，消息框架消息发布者都封装在基础设施层。

另外，程序中用到的工具类util模块和异常类exception也统一封装在基础设施层。

从分层架构的角度，领域层可以依赖基础设施层实现与其他外设的交互。另外，无论从分层架构的上层application层还是从六边形架构的角度的输入端口和适配器application，都可以依赖作为底层或处于同层的输出端口和适配器的infrastructure层，比如调用util或者exception模块。



# 四、结束语

其实，无论是面向服务架构SOA，微服务，领域驱动，还是中台，其目的都是在说，我们做架构设计的时候，应该从业务视角出发，对所涉及的业务领域，基于高内聚、低耦合的思想进行划分，最大限度且合理的实现业务重用。

这样不仅方便提供专业且稳定的业务服务，更有利于业务的沉淀和可持续发展。业务之下是基于技术的系统实现，技术造就业务，业务引领技术，两者相辅相成，共同为社会进步做出贡献。



# 五、参考文献

- [1] 《领域驱动设计软件核心复杂性应对之道》Eric Evans著, 赵俐 盛海燕 刘霞等译，人民邮电出版社
- [2] 《实现领域驱动设计》Vaughn Vernon著, 滕云译, 张逸审，电子工业出版社
- [3] 《微服务架构设计模式》[美]克里斯.理查森(Chris Richardson) 著, 喻勇译，机械工业出版社
- [4] 《架构整洁之道》[美]Robert C.Martin 著，孙宇聪 译，电子工业出版社
- [5] 《企业IT架构转型之道阿里巴巴中台战略思想与架构实践》钟华编著，机械工业出版社
- [6] [领域驱动设计(DDD)实践之路(二)：事件驱动与CQRS](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI4NjY4MTU5Nw%3D%3D%26mid%3D2247486689%26idx%3D1%26sn%3D3d73f1a466aa8270db39a84244df3ac5%26chksm%3Debd87c73dcaff5655494060e1afe52256627a996b993329fcf7fc9341457d39afab7bfa9e3e7%26scene%3D21%23wechat_redirect)，vivo互联网技术
- [7][ 领域驱动设计在互联网业务开发中的实践](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fblog.csdn.net%2Fmeituantech%2Farticle%2Fdetails%2F80062248)，美团技术团队