# 使用MongoDB图表对数据进行可视化

[四月 25, 2019](http://www.mongoing.com/archives/25674)[其他](http://www.mongoing.com/archives/category/uncategorized)[Data Visualization](http://www.mongoing.com/archives/tag/data-visualization)、[mongodb](http://www.mongoing.com/archives/tag/mongodb)、[MongoDB Charts](http://www.mongoing.com/archives/tag/mongodb-charts)[xica](http://www.mongoing.com/archives/author/xica)

将数据存储在数据库中对于当今的企业来说是一件很自然的事情。客户信息、历史订单、产品定价、物联网传感器数据，以及更多的正在被记录下来的信息，以备将来使用。然而，仅仅存储数据还不足以形成竞争市场优势。我们还必须能够分析数据，分析数据有很多方法可以选择。如果您想在MongoDB中进行可视化分析的数据，MongoDB图表是一个非常好的选择。

在使用MongoDB图表之前，实际上有三种方法可以可视化MongoDB数据。
1、 协同第三方BI工具，但需要充分利用 MongoDB 商业智能(BI)连接器
2、 利用第三方工具，执行Extract-Transform-Load (ETL)相关操作
3、 编写自定义代码并使用图表库，如D3.js或Bokeh。

### MongoDB图表的好处

目前处于Beta版的MongoDB图表提供了一种简单的方法来可视化MongoDB中的数据。您不需要将数据移动到不同的存储库、编写自己的代码或购买第三方工具。MongoDB图表知道并理解丰富性的文档数据模型，并非常容易地对数据可视化。

此外，MongoDB图表提供了一种安全的方法，可以创建可视化仪表板并与每个人共享，或者只与目标团队成员共享。类似地，后台使用的数据源也可以安全地共享。例如，销售部门的数据不需要提供给市场，除非有必要。非常强大的MongoDB图表将遵循MongoDB的安全设计作为首要任务。

下载完MongoDB图表Docker镜像，并按照如下安装说明操作，我们就可以连接到MongoDB Atlas中存储的数据源，并开始制作可视化仪表板。连接到MongoDB 图表服务器，我们需要采取以下三个步骤:
1、添加数据源
2、创建一个仪表板
3、创建图表

### 使用MongoDB图表分析Airbnb数据

我建立了一个数据库，里面有来自各个城市的Airbnb数据。我们将在这里探索来自华盛顿州西雅图的数据集，但是您可以自由地探索其他数据集。我们需要从拥有数据的Atlas集群中获取连接字符串，并以图表的形式连接到它。

[![1](http://mongoing.com/wp-content/uploads/2019/04/1.gif)](http://mongoing.com/wp-content/uploads/2019/04/1.gif)

该图是动态图：https://webassets.mongodb.com/_com_assets/cms/atlas-URI-6k09mb3bg3.gif

### 添加数据源

我们的MongoDB 图表服务器运行在localhost:80上，我们可以登录到Data Sources选项卡。我们使用Atlas (mongodb+srv://airbnbdemo:airbnb@airbnb-rgl39.mongodb.net/test?retryWrites=true)中的URI并选择Connect。接下来，我们被问到希望从集群中使用哪个数据源，在本例中，我将选择airbnb数据库中的seattlelistingandviews。对于权限，我只想保持所有内容都是私有的，因此我将接受缺省值并选择Publish Data Source。
一旦连接上，就可以给数据源添加别名。我把它叫做Airbnb西雅图。

**注意:上面的URI包含一个示例URI。您应该连接到自己的Atlas集群，并使用授权的用户名和密码。**

### 创建一个仪表板

下一步是创建一个实际的仪表板来装饰我们的可视化图表了。在Dashboards部分中，选择New Dashboard并填写一个名称和描述，就像Ken的Airbnb Dashboard一样。这里就是我们在仪表盘中添加图表的地方。

### 创建一个图表

单击Add Chart按钮后，我们可以开始构建可视化图表了。我们将从下拉框中选择Airbnb西雅图数据源。MongoDB图表自动确定哪些字段可用来进行探索。在这个练习中，我想看看西雅图的哪些社区拥有最多的Airbnb房产，并按房产类型进行划分。我们将对类型使用堆叠条形图。

1、对于x轴，我们需要id字段，根据count进行聚合。

[![2](http://mongoing.com/wp-content/uploads/2019/04/2.gif)](http://mongoing.com/wp-content/uploads/2019/04/2.gif)

该图是动态图：https://webassets.mongodb.com/_com_assets/cms/x-axis-value-cz2tkvt97r.gif

将x轴值赋给MongoDB图表，沿着y轴我们会看到地址和郊区。注意，address是这里的子文档，MongoDB图表天生知道如何处理这种类型的数据。我想按聚集值对郊区进行排序，按降序排列，并将结果限制在前20个郊区。

[![3](http://mongoing.com/wp-content/uploads/2019/04/3.gif)](http://mongoing.com/wp-content/uploads/2019/04/3.gif)

该图是动态图：https://webassets.mongodb.com/_com_assets/cms/y-axis-value-h1llqzam8w.gif

将y轴值赋给堆叠的条形图，让我们添加property_type字段作为我们的系列。

[![4](http://mongoing.com/wp-content/uploads/2019/04/4.gif)](http://mongoing.com/wp-content/uploads/2019/04/4.gif)

该图是动态图：https://webassets.mongodb.com/_com_assets/cms/series-value-b1gprdumq6.gif

将一个序列值赋给一个堆叠的条形图，现在我们可以根据位置命名图表、属性并保存它。
然后我们回到仪表板，在那里我们可以添加其他可视化，以便进一步研究。

看一下这个简短的视频，可以看到从这个数据源创建的其他一些可视化效果。
视频(youtube)地址：https://youtu.be/c7gIXDYDrNU

### 总 结：

MongoDB图表是一个非常好的新工具，可以直观地探索数据。它对于特定的用例有一些很好的特性，比如:
1、数据的临时分析
2、理解文档数据模型的优点
3、使用基于用户的共享和权限，项目协作非常容易
4、对于非开发人员来说，使用它进行自助数据分析已经足够直观了

MongoDB图表是在MongoDB数据上构建可视化的最快方法。我鼓励你今天就下载并试用它。让我知道你从Airbnb的数据集中得出了什么可视化结果。我总是喜欢看到人们如何探索他们的数据。

[MongoDB Charts下载地址：https://www.mongodb.com/download-center/charts](https://www.mongodb.com/download-center/charts)

译者：管祥青

[原文标题：Visualizing Your Data With MongoDB Charts](https://www.kenwalger.com/blog/nosql/mongodb/visualizing-data-mongodb-charts/)
原文地址：https://www.kenwalger.com/blog/nosql/mongodb/visualizing-data-mongodb-charts/