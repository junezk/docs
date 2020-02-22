# RESTful API设计实践参考手册

写这篇文章的主要目的其实是希望把自己多年关于RESTful API的设计经验总结梳理一下，顺便跟大家做个分享，独乐乐，不如众乐乐么。

为了更好的把设计上的一些问题和注意的点说明白，这篇文章从始至终都会围绕着同一个例子来说明。当然，这个例子是一个yy的REST架构的软件系统。我们就叫他GoodHR好了，而这个系统的url就是[www.goodhr.com](https://link.zhihu.com/?target=http%3A//www.goodhr.com)（如有雷同，纯属巧合）。

简单的来讲RESTful API其实是REST系统架构设计理念之下基于web环境的http协议来描述系统接口。所以，整个RESTful API设计主要分为两个大的部分，描述资源（resource）的URI和对资源进行行操作的方法（method）。

文章会分为三个主要部分：1）对于资源的URI（Uniform Resource Identifier）设计实践；2）对于方法的设计实践，这里，我只是把最常用的方法（POST、PUT、GET和DELETE）进行了说明；3）对于一些比较特殊的场景，比如，设计支持复杂query/search能力的RESTful API，支持异步处理的RESTful API等，进行一个case by case的说明。

## URI设计

在RESTful API设计中，最主要的目的其实是让整个接口更具有自描述性，这样在接口的易用性上和可维护性上才能有更好的表现。

关于资源的URI，简单来讲就是用来在web环境下如何定位一个资源的描述标准。RESTful API的资源URI一般由两个部分组成的：Path和Query Parameters，下面我来分别介绍和说明一下。

## Path

主要是描述一个资源的访问路径的，而Path的一般要用名词来组成。

在RESTful API设计中Path内容还包含有Path Parameter，也就是动态的参数部分，比如，我们如果有一个Path来指向一个具体的公司资源对象，我们会有如下的Path设计

```
http://www.goodhr.com/api/v1/companies/{公司id}
```

这里{公司id}就是Path parameter，他可以被替换为具体的id值用来表明，那个一个公司资源对象。Path Parameter在整个RESTful API的URI设计理念里面是代表的**必选参数**，也就是说，Path Parameter的值一定要给的，否则这个RESTful API将无法使用，或者会访问到另外一个API的情况。

其实，在面对不同的场景下，Path的设计上是有不同的实践方式的，下面我把常用的列举出来：

### 1. 资源对象集

如果用来描述一种资源（一个资源的聚合），那么需要用复数形式表示，比如下面的例子：

```
http://www.goodhr.com/api/v1/companies/66/employees
```

### 2. 单独资源对象

如果用来描述一个资源，那么，这个资源肯定是可以有一个唯一标示的来确定这个资源，比如下面的例子：

```
http://www.goodhr.com/api/v1/companies/66/employees/{员工id}
```

### 3. 资源从属关系

从属资源关系，比如，员工会从属于一个公司，那么这种uri的设计就应该用下面的方式表达：

```
http://www.goodhr.com/api/v1/companies/66/employees/1001
```

这种方式的表达另外一个作用就是代表了其资源的生命周期的强依赖，简单来讲如果一个公司被删除了，那么其下面的员工也应该不能再被访问或者修改。

### 4. 资源索引关系

可索引关系的资源，比如，一个员工可以在一个公司里面加入多个部门，那么，部门和员工的关系并不是一个强从属关系，只是说，通过一个部门可以反查出这个部门里面有哪些员工。

```
http://www.goodhr.com/api/v1/companies/66/departments/{部门id}/employees
```

上面这种表达方式，其实和从属关系的资源uri没有太大的不同，只是通过增加了departments一级的路径来描述是某一个部门里面的一群员工。

## Query Parameter

Query parameter是说在URI中问号（？）之后出现的key-value的参数，在RSETful API的设计中其实提供的是一个**可选参数**的作用 —— 不会出现无访问其他资源的情况，并且，也不会造成这个API不能被执行。比如，我们需要获取一个公司下的所有员工资源对象，那么，我们的API可以设计如下：

```
http://www.goodhr.com/api/v1/companies/66/employees?pageStart=1&pageSize=100
```

那么上面出现的pageStart和pageSize其实就是query parameter，用来作为翻页的参数，如果，在实现这个RESTful API的时候，一定要对于当缺少pageStart或pageSize的时候，也会有一个缺省的逻辑实现，在这里，我们一般会说在pageStart不给的时候，我们默认是从1开始，如果pageSize不给与的时候，我们用缺省的页面大小，如：20来代替。

## 方法设计

在RESTful API中对于资源的操作的方法最常用的就是POST、PUT、GET、和DELETE。下面，我将一次来对于每一个方法进行具体的说明如何设计。

在http的官方协议规格说明中GET、PUT和DELETE方法是幂等的，而POST不是。

## 1. POST方法

POST方法代表对uri所描述的资源进行创建一个新的资源。POST方法是可以携带请求体的（request body），RESTful API的body一般使用的是JSON类型。而POST方法的返回类型一般也是JSON格式，并且，HTTP的status code应该是201（Created）

定义一个POST RESTful API一般有两种场景：

### 1. 资源Id不确定

如果，你要创建的资源的Id是由Server来分发的，那么一般，URI应该是按照**资源集**的设计风格。比如，创建一个新的职员，其URI如下

```
http://www.goodhr.com/api/v1/companies/66/employees
```

请求体

```
{
    "firstname": "Steve",
    "lastname": "Bill",
    "birthDate": "1982-01-01",
    "gender": "male",
    "hiredDate": "2019-01-01",  
    "socialSecurityNumber": "1234567890123456789"
}
```

返回结果

```
{
    "companyId": 66,
    "id": 1001,
    "firstname": "Steve",
    "lastname": "Bill",
    "birthDate": "1982-01-01",
    "gender": "male",    
    "hiredDate": "2019-01-01",
    "socialSecurityNumberMask": "123************789",
    "creationTime": "2019-01-01 09:00:01",
    "updatedTime": "2019-01-01 09:00:01"
}
```

这里“id”字段的值是由server端在创建一个employee记录的时候分发的。

### 2. 资源Id可确定

有一种情况是，你对于要创建的资源的Id是预先确定的，而不需要创建的时候由Server来分发。那么这种情况下，URI应该是按照**单独资源**的设计风格。

比如，创建一个新的部门（department），其URI如下

```
http://www.goodhr.com/api/v1/companies/66/departments/dept-acct
```

请求体

```
{
    "name": "财会",
    "introduction": "公司的财务部门"
}
```

返回结果

```
{
    "companyId": 66,
    "code": "dept-acct",
    "name": "财会",
    "introduction": "公司的财务部门"
    "creationTime": "2019-01-01 09:00:01"
}
```

## PUT方法

简单来讲PUT方法一般是用来更新一个资源相关信息项目的，根据http的规范定义，PUT应该是幂等的操作，所以，在PUT的接口设计上一定是对资源的信息进行全量的更新，而不是部分更新。这里我们用员工做例子：

URI应该是按照**单独资源**的设计风格，而返回的http的status code在成功的情况下应该是200

### 1. 资源对象属性相对简单

一般来讲，一些资源对象所包含的属性信息其实相对简单，那么，这种情况，就用标准的PUT的RESTful API来设计就可以了，比如，员工信息更新：

```
http://www.goodhr.com/api/v1/companies/66/employees/1001
```

而请求体的设计在这里就要对全量的员工信息进行更新（如下），而不是部分更新 请求体

```
{
    "firstname": "Kevin",
    "lastname": "Bill",
    "birthDate": "1982-01-01",
    "gender": "male",    
    "hiredDate": "2018-03-01",  
    "socialSecurityNumber": "1234567890123456789"
}
```

返回结果

```
{
    "companyId": 66,
    "id": 1001,
    "firstname": "Kevin",
    "lastname": "Bill",
    "birthDate": "1982-01-01",
    "gender": "male",    
    "hiredDate": "2018-03-01",
    "socialSecurityNumberMask": "123************789",
    "creationTime": "2019-01-01 09:00:01",
    "updatedTime": "2019-01-05 14:56:12"
}
```

### 2. 资源对象有很多属性

有些时候，我们会出现一种场景就是，一个资源所包含的属性很多，而且，这些属性是有一些类型上的归类的，比如，公司资源对象上，会有基本信息，税务信息，董事会信息，营业状态等等，那么在这种情况下，我们如果用一个RESTful API来做全量的更新的话，其实，不论是从这个API的易于使用上，还是以后如果对能访问资源的权限管理上都有一定的问题。这种情况下，我会这么设计这个API，对于不同类型的属性给独立的URI来做，如下：

公司基本信息

```
http://www.goodhr.com/api/v1/companies/66/profile
```

公司税负信息

```
http://www.goodhr.com/api/v1/companies/66/tax-info
```

公司董事会信息

```
http://www.goodhr.com/api/v1/companies/66/executive-info
```

公司营业状态

```
http://www.goodhr.com/api/v1/companies/66/status
```

## DELETE方法

这个方法顾名思义，就是删除资源，此方法本身也是幂等的，所以，这种接口的实现规则就是，不论是第一次删除一个存在的资源，还是之后删除已经不存在的资源，都应该是返回同样的结果 —— http的status code = 204。

### 1.删除单独资源

URI应该是按照**单独资源**的设计风格，不需要给请求消息体比如，我们删除一个员工。

```
http://www.goodhr.com/api/v1/companies/66/employees/1001
```

返回消息有两种方式：1. 返回一个空内容，只是http status code为204即可。2. 可以返回一个结果，其内容可以包含被删除员工的关键Id，比如，上面的URI中的公司Id和1001。

```
{
    "companyId": 66,
    "employeeId": 1001
}
```

### 2.删除一批资源

这种情况其实不常出现，但是，是有可能有应用场景的，比如，删除所有离职时间超过3年以上的员工记录，这种URI可以用下面的设计风格：

```
http://www.goodhr.com/api/v1/companies/66/employees/_resigned-exceed-years/3
```

”_resigned-exceed-years”在这里是一个对于employees资源集的一个刷选条件，而其后面的3就是这个刷选条件的值。

这里我建议要给一个返回值，主要的作用是用来对于这次删除的一个结果报告，如下：

```
{
    "companyId": 66,
    "numberOfDeletedEmployees": 132
}
```

## GET

这个方法的使用场景很容易理解，就是获取资源，但是，在实际的实践过程中我发现，GET方法所面对的场景反而是最复杂的。比如，复杂的多条件搜索。

### 1. 获取单独资源对象

获取一个指定的资源对象，比如，通过给定员工id获取一个员工

```
http://www.goodhr.com/api/v1/companies/66/employees/1001
```

返回值

```
{
    "companyId": 66,
    "id": 1001,
    "firstname": "Steve",
    "lastname": "Bill",
    "birthDate": "1982-01-01",
    "gender": "male",    
    "hiredDate": "2019-01-01",
    "socialSecurityNumberMask": "123************789",
    "creationTime": "2019-01-01 09:00:01",
    "updatedTime": "2019-01-01 09:00:01"
}
```

### 2. 获取资源对象集

比如，获取一个公司下的所有员工

```
http://www.goodhr.com/api/v1/companies/66/employees?pageStart=1&pageSize=100&orderBy=creationTime&order=DESC
```

对于一个资源集的返回数据设计上，要尽量用加一层字段来进行更好地表现一组数据以及展示是通过什么条件获得的这组数据，设计可以考虑如下方式：

```
{    
    "queryConditions": {},
    "pagination": {
        "pageStart": 1,
        "pageSize": 100,
        "sorting": {
            "orderBy": "creationTime",
            "order": "DESC"
        }
    },
    "data": {
        "size": 100,
        "records": [
            {
                "companyId": 66,
                "id": 1001,
                "firstname": "Steve",
                "lastname": "Bill",
                "birthDate": "1982-01-01",
                "gender": "male",                
                "hiredDate": "2019-01-01",
                "socialSecurityNumberMask": "123************789",
                "creationTime": "2019-01-01 09:00:01",
                "updatedTime": "2019-01-01 09:00:01"
            }
            ...
        ] 
    }   
}
```

在对于一组资源对象进行获取操作的时候，尽量在返回对象中包含获取时候所使用的参数信息，比如，上面，包含了queryConditions用来表示资源获取的时候搜索条件，这里没有，所以是空，pagination用来说明data里面的资源的分页信息，而data字段中不只是包含了返回的资源集合records还包含了，其实际records的大小，以增加这个接口的易用性。

### 3. 通过搜索获取资源集对象

提供对资源的搜索能力其实在很多系统中都是常见的能力，比如，通过搜索名称获得员工名称，或者是多种条件的组合搜索，比如，姓名加年龄等等。这种RESTful API在设计上，如果用GET方法的话，最大的问题在于GET本身不可以提供request body，所以，一般都会以Query Parameter的方式进行支持。

比如，我们要搜索firstname是steve的，年龄小于60岁的员工，并且，以入职时间倒序来进行返回：

```
http://www.goodhr.com/api/v1/companies/66/employees?firstname=Steve&age=%3C60
pageStart=1&pageSize=100&orderBy=hiredDate&order=DESC

{    
    "queryConditions": {
        "firstname":"steve",
        "age": "<60"
    },
    "pagination": {
        "pageStart": 1,
        "pageSize": 100,
        "sorting": {
            "orderBy": "creationTime",
            "order": "DESC"
        }
    },
    "data": {
        "size": 13,
        "records": [
            {
                "companyId": 66,
                "id": 1001,
                "firstname": "Steve",
                "lastname": "Bill",
                "birthDate": "1982-01-01",
                "gender": "male",                
                "hiredDate": "2019-01-01",
                "socialSecurityNumberMask": "123************789",
                "creationTime": "2019-01-01 09:00:01",
                "updatedTime": "2019-01-01 09:00:01"
            }
            ...
        ] 
    }   
}
```

很多人其实很害怕使用GET来做复杂搜索的话，因为URL的长度限制会有问题，不过，根据http的标准规定2048个字符以内都是没有问题的，所以，一般系统提供的对于资源的搜索能力应该是足够满足了。当然，我们不可否认其实，还是有一些复杂的搜索场景出现的，如果，是这样，可以看后面的三部分的关于特殊场景的一些设计实践介绍。

## 特殊场景

基于2/8理论来说，上面的URI和方法两部分中的设计实践整理你可以理解为覆盖了80%的场景，而这部分，我更多关注的是这个20%的复杂场景，当然，我总结的也只是我过往10年中在为不同行业的系统设计RESTful API的时候遇到情况，并不是包含所有的情况，所以，如果，你发现我讲述的场景里面并没有覆盖到你的，你可以选择私信我，我们可以一起来探讨一下如何设计 ：）。

### 1. 超复杂搜索

在不常见的场景中，相对常见的是这种过于复杂的搜索RESTful API的提供。这种情况下如果按照标准的GET+query paramters的方式来进行设计，其实，会有出现超出url标准长度要求的可能性，所以，特殊情况，我们就只能特殊处理，一般来说，我们会使用POST方法来代替，然后把搜索条件放到request body里以json的方式提供。

比如，我们需要对一个员工的请假记录进行搜索：

```
http://www.goodhr.com/api/v1/companies/66/timeoff-records?pageStart=1&pageSize=100&orderBy=creationTime&order=DESC
```

使用POST方法来发送搜索条件信息：

```
{    
    "type": [
        "vocation",
        "sick"
    ],
    "appliedDateRange": {
        "startDate": "2019-01-01",
        "endDate": null
    },
    "departments":[
        "dept-acct",
        "dept-tech",
        "dept-marketing"
    ]   
}
```

返回对象

```
{    
    "queryConditions": {    
        "type": [
            "vocation",
            "sick"
        ],
        "appliedDateRange": {
            "startDate": "2019-06-01",
            "endDate": null
        },
        "departments":[
            "dept-acct",
            "dept-tech",
            "dept-marketing"
        ]   
    },
    "pagination": {
        "pageStart": 1,
        "pageSize": 100,
        "sorting": {
            "orderBy": "creationTime",
            "order": "DESC"
        }
    },
    "data": {
        "size": 13,
        "records": [
            {
                "companyId": 66,
                "id": 10293,
                "applicantId": 1002,
                "applicationDateTime": "2019-01-01 09:00:01",
                "approverId": 98,
                "type": "vocation",
                "timeoffBegin": "2019-06-01 AM",
                "timeoffEnd": "2019-06-01 PM",
                "creationTime": "2019-01-01 09:00:01",
                "updatedTime": "2019-01-01 09:00:01"
            }
            ...
        ] 
    }   
}
```

### 2. 异步处理

如果是需要提供比如，备份数据，产生报告结果，复杂的计算任务，或者是需要人工处理的，一般会用异步处理方式，在RESTful API的设计中，所谓的异步和系统中的异步API不太一样在于，他并不是一个基于线程的异步方案，而更多的是，通过一个RESTful API出发请求处理，而这个请求的处理之后的结果则是需要通过另一个API来进行获取。这种异步处理的设计有两种方式，我在下面分别介绍一下。

### a. 执行和获取分离

简单的来讲这种设计风格就是，你需要触发API A来创建一个处理请求任务，再通过API B进行轮训获得结果。比如，我们要生成，所有员工的请加统计报告：

第一个RESTful API，我们用来创建生成报告，这里我们用POST方法，因为，每调用一次这个接口，就会创建一个新的报告生成任务

```
http://www.goodhr.com/api/v1/companies/66/statistics/timeoff
```

请求体

```
{    
    "contentType": "excel",
    "dateRange": {
        "startDate": "2018-01-01",
        "endDate": "2018-12-31",
    }
}
```

返回对象

```
{  
    "jobId": "timeoff-100202",
    "creationTime": "2019-01-01 09:00:01",
    "contentType": "excel",
    "dateRange": {
        "startDate": "2018-01-01",
        "endDate": "2018-12-31",
    }
}
```

这里，jobId就是用来从第二个API来获得这个异步处理任务的结果的。

第二个RESTful API用来获取这个统计报告的，这里我们使用GET方法。

```
http://www.goodhr.com/api/v1/companies/66/statistics/timeoff/{jobId}
```

返回对象

```
{  
    "jobId": "timeoff-100202",
    "creationTime": "2019-01-01 09:00:01",
    "status": "finished",
    "retrivalLocation": "https://static.goodhr.com/reports/timeoff-100202.xls"
}
```

当然，也有可能出现报告还在生成中的情况，那样的话，返回对象中的status有可能就是“processing”，而retrievalLocaiton就是null。

### b. 执行完了主动推送

这个设计方案和a对比来说就是，a方案是pull的方式获取结果，而这个方案是push的方式。如果用这个方式设计上面产生timeoff的报告的RESTful API的话，在整体设计上会有一些复杂度，当然，好处就是push的设计方式，系统所受到的系统压力会小很多。

在这个设计方案里，我们只需要提供a里面的第一个POST的RESTful API来做同样的产生报告的任务生成，请求体也没有变化，返回对象中需要增加一个字段”callbackUrl，如下：

```
{  
    "jobId": "timeoff-100202",
    "creationTime": "2019-01-01 09:00:01",
    "callbackUrl": "https://another-system.somewhere.com/receive-reports/timeoff",
    "contentType": "excel",
    "dateRange": {
        "startDate": "2018-01-01",
        "endDate": "2018-12-31",
    }
}
```

这里的callbackUrl其实是一个webhook设计，对于webhook，如果是内部系统，建议简单的加一个host白名单校验，而如果是以一种SaaS的Open API的方式向第三方提供扩展能力，那么就需要额外的搭建一个webhook的管理功能来保障足够的安全性，比如跨域攻击或请求拦截等。

### 3. 超多字段资源对象

有些时候，一些资源对象包含了很多属性，几十个，甚至上百，而这个时候，我们通过一个RESTful API来获取这个资源，如果，只是通过Id获取确定的一个资源对象，那其实还是可以接受的，但是，如果是获取资源对象集，那么不论是系统开销还是网络开销，都会产生不必要的浪费。

对于这种资源的相关RESTful API的设计上可以考虑在其URI的设计上上提供一个query parameter作为告诉服务端这个资源的那些属性需要返回，比如，对于公司资源的访问上。

```
http://www.goodhr.com/api/v1/companies/66?fields={属性类型}
```

这里fields的值可以按照一个公司对象的属性分类进行提供，比如，添加tax-info代表需要返回税务信息，添加executive-info代表需要返回管理层信息。当然，如果fields没有给的话，我们应该有一个缺省的返回内容逻辑，比如，返回公司基本信息作为缺省的返回属性。

这种设计方案中fields的力度最好是一种属性，而不是一个属性，这样在代码的可读性上，以及服务端的代码实现复杂度和灵活性上会有一个相对不错的平衡，假设，如果你需要这个接口的调用者把具体的每一个属性名称都要给的话，那么，这将是一个灾难。

### 4. 幂等操作设计

这种RESTful API的设计其实很简单，一般都是在URI上提供一个query parameter用来代表一个事务id，一次可以让你实现不断的对次事务进行安全的重试操作。

假设，我们的系统对于创建一个员工记录的逻辑中，包含了和其他系统的交互以及数据的更新，那么，我们就需要改造一下之前POST方法中创建员工记录的例子，让其可支持幂等操作。

```
http://www.goodhr.com/api/v1/companies/66/employees
```

请求体

```
{
    "firstname": "Steve",
    "lastname": "Bill",
    "birthDate": "1982-01-01",
    "gender": "male",
    "hiredDate": "2019-01-01",  
    "socialSecurityNumber": "1234567890123456789"
}
```

这里，我们对于返回结果增加一个字段transactionId，用来表示本次创建请求的事务。

返回结果

```
{
    "transactionId": "e721ac103ckc910ck20c",
    ...
}
```

如果，创建过程中出现异常（非204 status code），造成你的创建员工记录失败，那么，你可以在第二次重试的时候把这个transactionId加上，以保证服务端可以实现一个幂等操作，如下：

```
http://www.goodhr.com/api/v1/companies/66/employees?transactionId=e721ac103ckc910ck20c
```

## 最后的最后

这里有几个基本的设计实践原则，我觉得不只是对于RESTful API，其实只要是设计系统API都有帮助。

## Tell, don’t ask原则

这个原则主要强调的是，每一个接口都要可以完成一整个业务逻辑，而不是通过调用方组合多个接口完成一个业务逻辑的。

比如，创建一个员工记录的时候，需要调用政府的一个系统接口对其SIN（社会保险号）进行校验，那么最佳的实现方式是，要有一个创建员工记录接口一次性完成所有的SIN校验以及记录创建，如下，

**good**

```
POST: http://www.goodhr.com/api/v1/companies/66/employees
```

请求体

```
{
    "firstname": "Steve",
    "lastname": "Bill",
    "birthDate": "1982-01-01",
    "gender": "male",
    "hiredDate": "2019-01-01",  
    "socialSecurityNumber": "1234567890123456789"
}
```

相反，有的时候会有人把这两个逻辑拆分为两个独立的接口来提供，就会变成下面的效果

**bad**

校验社会保险号

```
GET: http://www.goodhr.com/api/v1/sin-record/1234567890123456789
```

创建员工记录

```
POST: http://www.goodhr.com/api/v1/companies/66/employees
```

请求体

```
{
    "firstname": "Steve",
    "lastname": "Bill",
    "birthDate": "1982-01-01",
    "gender": "male",
    "hiredDate": "2019-01-01",  
    "socialSecurityNumber": "1234567890123456789"
    "sinVerfiied": true
}
```

## 版本

一定要有版本化管理，这样不仅仅是便于维护，更为了让你的系统更容易地进行升级和重构。

在RESTful API的中，版本的实践方式一般是在uri中Path中添加版本信息

```
http://www.goodhr.com/api/v{版本号}
复制代码
```

版本号部分可以是数字，也可以是其他你希望的，比如，有的会使用日期如20190101这种。