# 在Python中搭建有效的微服务架构

API(应用编程接口)优先方法是Connexion（联接）搭建有效的微服务的有力工具。使用API的好处是满足了客户期望。除此之外，使用Connexion将使你在Python中开发API变得更加流畅。

Google在收购Apigee时强调了应用程式界面在当今应用程序架构中的重要性。使用具有精心设计的API微服务对于保持业务成功至关重要，因为它简化了复杂软件解决方案的开发。成长型公司历经企业复杂性而自然提高，这种复杂性来自市场变化和为了迎合客户的需求。像Zalando公司，面临这样的挑战，选择采用微服务。 微服务的采用旨在使其更易于构建和维护其应用程序。

微服务是一种将复杂的软件解决方案分解成更小的可组合服务的风格，这些小型服务可以独立维护和部署。每个服务都围绕通常提供REST API(应用编程接口)的业务功能开发。清楚地说明每个服务提供什么功能可以对团队构建和维护其微服务的生产力产生巨大影响。微服务开发人员应向客户端开发人员提供构建和维护网络，移动或其他服务，描述如何使用他们的REST API。未能清楚地描述一个API和维护一个最新的文档会导致系统设计的崩坏并造成团队成员的挫败感。API优先方法正在成为解决这些问题的解决方案。

## API优先方法

API方法最近已经添加到应用程序开发方法的最初12个因素中 - 一组规则和指导方针用来开发强大的应用程序，一群经验丰富的开发人员想出了构建Heroku平台。它是专注于API规范开发过程中的第一级神器，并用它们作为软件开发者和团队之间共享合同的方法。甚至在编写第一行代码之前，构建规范的服务也可以促进API的潜在用户的审核，创建API的模拟，以及生成文档。

Google，IBM，Microsoft和其他公司已经携手创建了开源API计划（OAI），以支持定义和建立一个独立于供应商的格式，用于描述称为开放API规范（以前称为Swagger 2.0）的REST API。 还有其他可用于描述API的格式，可以与API优先方法一起使用，最著名的是API Blueprint和RAML，但是开源API格式目前拥有更大的用户和支持者社区。 从使用开源API格式编写的规范，可以容易地生成用于实现的初始代码。 支持许多语言和框架：Ruby，Java，Node，C＃等。对于Python，Connexion是API优先最佳选择，因为它既不依赖代码生成也不需要样板代码。

Connexion是一个建立在Flask之上的开源框架，它根据API First方法促进了Python中的Microservices的开发。 它是由Zalando创建的，以满足这种解决方案的内部需求。 Connexion正在积极发展，我是该项目的主要维护者之一，还有Joao Santos和Henning Jacobs。

## 建立在Python中一个简单的服务

在Python中构建有效微服务的第一步是在我们的API中使用开源API规范。我们将专注于描述什么路线，参数，负载和响应代码我们的API产品。我们从一个简单的端点示例开始，该端点响应字符串“Hello API！”。

![img](http://www.uml.org.cn/wfw/images/2018071031.png)

在上面的代码块中，我们指定了我们的API有一个端点“/ greeting”，接受方法为“GET”的请求，并返回状态码200，表示成功。 注意，我们的规范中没有定义业务逻辑，这部分是由我们的端点实现的，这将在Python中完成。“operationId”是什么定义取决于你对你的的端点进行API调用时执行哪个Python函数。

![img](http://www.uml.org.cn/wfw/images/2018071032.png)

如你所见，Connexion API的处理程序是免费的样板代码。返回简单数据结构的普通Python函数用作API调用的处理程序。 处理程序返回的数据结构可以通过Connexion针对API规范进行验证（如果指定的话）。 默认情况下禁用此验证以在开发期间授予灵活性。 现在我们可以使用Connexion将我们的代码粘贴到API规范，并让服务器运行。 运行我们的API的最简方法是使用Connexion CLI工具：

![img](http://www.uml.org.cn/wfw/images/2018071033.png)

现在我们可以去浏览器并访问地址http：// localhost：5000 / greeting，我们应该能够看到消息：

![img](http://www.uml.org.cn/wfw/images/2018071034.png)

为了使其更具动态性，我们可以更改我们的规范，为用户名添加HTTP参数。 为此，我们的规范应该看起来像这样：

![img](http://www.uml.org.cn/wfw/images/2018071035.png)

现在Connexion将一个可选参数传递给我们的函数。 为了处理，我们还必须改变我们的Python函数。

![img](http://www.uml.org.cn/wfw/images/2018071036.png)

我们的Python代码现在匹配API规范。 现在我们可以重新启动我们的服务器，按Ctrl + C并再次运行以下命令来查看这些更改。

![img](http://www.uml.org.cn/wfw/images/2018071037.png)

我们的服务器再次监听http://localhost:5000/greeting。 此外，现在可以传递可选数“name”。 Connexion包括可在http://localhost:5000/ui上使用的API控制台界面。 默认情况下，启用此控制台UI可以轻松调用API的端点。 它还用作我们微服务的文档。

现在我们知道如何创建Connexion应用程序的基础知识，有关更详细的示例，请检查Github上提供的Connexion示例项目。Connexion的官方文档是一个完整的信息资源，在使用Connexion开发RESTful微服务时，应使用此文档作为参考。

## 向前进

公司们开始意识到API优先方法的好处。 Etsy发表了一篇博文，描述了API优先方法如何帮助他们通过提供一致的API来解决挑战。 在Zalando，API优先方法是他们的软件开发生命周期的核心，有一个同行评审反馈过程和创建RESTful微服务的指南，这些都是可用的开源。

我们可以发现许多支持API优先方法的开源工具，可用于多种语言。Connexion是在Python中干净实现API优先方法的完美框架，并且在积极开发中。Connexion不依赖代码生成。 这使得您可以自由地演变您的API规范，而不破坏已经实现的代码。