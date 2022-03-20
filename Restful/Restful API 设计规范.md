# Restful API 设计规范

#### 使用的名词而不是动词

不应该使用动词：

```
/getAllResources`
`/createNewResources`
`/deleteAllResources
```

#### GET方法和查询参数不能改变资源状态:

如果要改变资源的状态，使用PUT、POST、DELETE。下面是错误的用GET方法来修改user的状态：

```
GET /users/711?activate
GET /users/711/activate
```

#### Rest的核心原则是将你的API拆分为逻辑上的资源。这些资源通过HTTP被操作(GET,POST,PUT,DELETE)

我们定义资源ticket、user、group:

- GET /tickets # 获取ticket列表
- GET /tickets/12 # 查看某个具体的ticket
- POST /tickets # 新建一个ticket
- PUT /tickets/12 #新建ticket 12
- DELETE /tickets/12 # 删除ticket 12

只需要一个endpoint：/tickets，再也没有其他什么命名规则和url规则了。

一个可以遵循的规则是：虽然看起来使用复数来描述某一个资源看起来特别扭，但是统一所有的endpoint，使用复数使得你的URL更加规整。这让API使用者更加容易理解，对开发者来说也更容易实现。

**处理关联：**

- GET /tickets/12/messages # 获取ticket 12的message列表
- GET /tickets/12/messages/5 #获取ticket 12的message 5
- POST /tickets/12/messages 创建ticket 12的一个message
- PUT /tickets/12/messages/5 更新ticket 12的message 5
- DELETE /tickets/12/messages/5 删除ticket 12的message 5

#### 避免层级过深的URI

`/` 在url中表达层级，用于按实体关联关系进行对象导航，一般根据id导航。

过深的导航容易导致url膨胀，不易维护，如 `GET /zoos/1/areas/3/animals/4`，尽量使用查询参数代替路劲中的实体导航，如`GET /animals?zoo=1&area=3`。

#### 结果过滤，排序，搜索

url最好越简短越好，对结果过滤、排序、搜索相关的功能都应该通过参数实现。

**过滤**：例如你想限制`GET /tickets` 的返回结果：只返回那些open状态的ticket， `GET /tickets?state=open` 这里的state就是过滤参数。

**排序**：和过滤一样，一个好的排序参数应该能够描述排序规则，而不和业务相关。复杂的排序规则应该通过组合实现。排序参数通过 `,` 分隔，排序参数前加 `-` 表示降序排列。

- GET /tickets?sort=-priority #获取按优先级降序排列的ticket列表
- GET /tickets?sort=-priority,created_at #获取按优先级降序排列的ticket列表，在同一个优先级内，先创建的ticket排列在前面。

**搜索**：有些时候简单的排序是不够的。我们可以使用搜索技术来实现

- GET /tickets?q=return&state=open&sort=-priority,create_at # 获取优先级最高且打开状态的ticket，而且包含单词return的ticket列表。

#### 限制API返回值的域

有时候API使用者不需要所有的结果，在进行横向限制的同时（例如值返回API结果的前十个），还应该可以进行纵向限制，并且这个功能能有效的提高网络带宽使用率和速度。可以使用fields查询参数来限制返回的域例如：

- GET /tickets?fields=id,subject,customer_name,updated_at&state=open&sort=-updated_at

#### Response不要包装

response 的 body直接就是数据，不要做多余的包装。错误实例：

```
{
    "success":true,
    "data":{"id":1, "name":"xiaotuan"}
}
```

#### 更新和创建操作应该返回资源

在POST操作以后，返回201created 状态码，并且包含一个指向新资源的url作为返回头。

#### 命名方式

是蛇形命名还是驼峰命名？如果使用json那么最好的应该是遵守JavaScript的命名方法-驼峰命名法。Java、C# 使用驼峰，python、ruby使用蛇形。

#### 默认使用pretty print格式，开启gzip

开启pretty print返回结果会更加友好易读，而且额外的传输也可以忽略不计。如果忘了使用gzip那么传输效率将会大大减少，损失大大增加。