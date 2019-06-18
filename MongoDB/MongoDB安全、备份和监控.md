# MongoDB安全、备份和监控

[四月 25, 2019](http://www.mongoing.com/archives/25687)[其他](http://www.mongoing.com/archives/category/uncategorized)[backup](http://www.mongoing.com/archives/tag/backup)、[Monitoring](http://www.mongoing.com/archives/tag/monitoring)、[Security](http://www.mongoing.com/archives/tag/security)[xica](http://www.mongoing.com/archives/author/xica)

MongoDB是一个键值对的数据库管理系统。当涉及到数据库管理时，诸如安全性、备份、对数据库的访问等重要方面都是重要的概念。

本教程中包含如下几个部分：
1、 MongoDB安全概述
2、 MongoDB备份程序
3、 MongoDB监控
4、 MongoDB索引和性能考虑

### MongoDB安全概述

MongoDB能够为数据库定义安全机制。在默认情况下，我们不希望每个人都可以访问MongoDB中的每个数据库，因此在MongoDB中需要某种安全机制是很重要的。以下是在数据库中实现安全性的最佳实践

1、启用访问控制——创建用户，以便在访问MongoDB上的数据库时强制所有应用程序和用户具有某种身份验证机制。
2、配置基于角色的访问控制——有时可能需要对权限进行逻辑分组，可以在角色中进行分组。然后可以将用户分配给这些角色。
3、尝试将MongoDB配置为某种加密协议，如TLS或SSL。这些协议可用于加密客户机和mongoDB环境之间的流量。
4、配置审计——管理员通常需要知道谁在做什么，这有助于稍后分析问题。最好的方法是在MongoDB中启用审计。
5、使用一个单独的用户id运行MongoDB服务器实例，该用户id可以访问服务器环境上所需的资源。

### MongoDB备份程序

在使用MongoDB时，一定要确保有一个备份过程，以防MongoDB中的数据由于任何原因损坏。下面是MongoDB中可用的备份机制：

1、**通过复制底层数据文件进行备份**——这可能是最简单的机制，所需要做的就是复制MongoDB所在的数据文件，并将其复制到另一个位置，理想情况下应该是另一个服务器。
2、**用mongodump备份数据库**——mongodump工具从MongoDB数据库中读取数据并创建高保真的BSON文件。需要考虑的是，如果数据集的容量很大，那么mongodump可能非常消耗资源，因此为了缓解这个问题，应该在辅助服务器上运行这个实用程序。
3、**MongoDB Cloud Manager备份**——通过从MongoDB环境中读取oplog数据，MongoDB Cloud Manager不断备份MongoDB副本集和分片集群。MongoDB Cloud Manager可以通过存储oplog数据来创建时间恢复点，这样它就可以在任何时刻为特定的复制集或分片集群创建恢复。

### MongoDB监控

监视是MongoDB中最关键的管理活动之一。这是因为您可以更主动地监视环境，以发现可能出现的问题。下面是一些实现监视的方法：

1、mongostat将告诉您在服务器上实际执行了多少次数据库操作，例如插入、查询、更新、删除等。这将很好地了解服务器正在处理多少负载，并预示您是否需要服务器上的额外资源，或者是否需要额外的服务器来分配负载。
2、mongotop跟踪并报告MongoDB实例的当前读写活动，并根据每个集合报告这些统计信息。
3、MongoDB提供了一个web接口，可以在一个简单的web页面中诊断和监视信息。您可以通过本地服务器打开浏览器程序，通过如下URL进行访问： http://localhost:28017
4、通过shell的serverStatus命令或db.serverStatus()返回数据库状态的概述，其中包含磁盘使用、内存使用、与MongoDB环境建立的连接等详细信息。

### MongoDB索引和性能考虑

1、索引在任何数据库中都非常重要，可以用来提高MongoDB中搜索查询的效率。如果您一直在文档中执行搜索，那么最好在搜索条件中使用的文档字段上添加索引。
2、尝试限制返回的查询结果的数量。假设您只想从文档中看到2个字段。然后确保您的查询只针对显示您需要的2个字段，而不是所有字段。
3、如果希望查看某些字段值，则只查询使用的这些字段，不要查询集合中的所有字段。

### 总结

 为了保证数据库中的数据安全，在数据库中使用安全机制是非常重要的。
 使用createUser命令在数据库中创建用户。将特定的角色分配给用户，让他们对数据库本身具有特定的权限。
 针对特定数据库的所有数据库添加管理员。这就是给用户赋予既不是userAdmin角色和userAdminAnyDatabase角色的用户。
 始终备份MongoDB数据，以便在发生任何灾难时，数据可以轻松恢复。
 始终监控MongoDB环境，以便更加主动地查看问题，在问题发生前。

注解：本文章翻译于guru99之MongoDB教程模块

译者：管祥青