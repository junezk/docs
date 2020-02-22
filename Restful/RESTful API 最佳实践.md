# RESTful API 最佳实践

`RESTful`  是目前最流行的 API 规范，适用于 Web 接口规范的设计。让接口易读，且含义清晰。本文将介绍如何设计易于理解和使用的 API，并且借助 Docker api 的实践说明。

## URL 设计

### 1.1 动词 + 宾语

它的核心思想就是客户端发出的数据操作指令都是「动词 + 宾语」的结构，比如 GET /articles这个命令，GET是动词，/articles是宾语。

动词通常来说就是五种 HTTP 方法，对应我们业务接口的 CRUD 操作。而宾语就是我们要操作的资源，可以理解成面向资源设计。我们所关注的数据就是资源。

- GET: 读取资源
- POST：新建资源
- PUT：更新资源
- PATCH：资源部分数据更新
- DELETE：删除资源

正确的例子

- GET /zoos：列出所有动物园
- POST /zoos：新建一个动物园
- GET /zoos/ID：获取某个指定动物园的信息
- PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
- PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
- DELETE /zoos/ID：删除某个动物园
- GET /zoos/ID/animals：列出某个指定动物园的所有动物
- DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物

### 1.2 动词的覆盖

有些客户端只能使用GET和POST这两种方法。服务器必须接受 POST 模拟其他三个方法（PUT、PATCH、DELETE）。

这时，客户端发出的 HTTP 请求，要加上 `X-HTTP-Method-Override` 属性，告诉服务器应该使用哪一个动词，覆盖 POST 方法。

### 1.3 宾语必须是名词

就是 API 的url ，是 HTTP 动词作用的对象，所以应该是名词。例如 /books 这个 URL 就是正确的，而下面的 URL 不是名词，都是错误的写法。

错误示范：

```
GET /getAllUsers?name=jl
POST /createUser
POST /deleteUSer
复制代码
```

### 1.4 复数名词

URL 是名词，那么是使用复数还是单数？

没有统一的规定，但是我们通常操作的数据多数是一个集合，比如 `GET /books` ,所以我们就使用复数。

统一规范，建议都使用复数 URL， 比如 获取 id = 2 的书 `GET /books/2` 要好于 `GET /book/2`。

### 1.5 避免出现多级 URL

有时候我们要操作的资源可能是有多个层级，因此很容易写多级 URL，比如获取某个作者某种分类的文章。

> `GET /authors/2/`categories/2 获取作者ID = 2 分类 = 2 的文章

这种 URL 不利于拓展，语义 也不清晰。

更好的方式就是 除了第一级，其他级别都是通过查询字符串表达。

> 正确方式： `GET /authors/12?categories=2`

查询已发布的文章

> 错误 写法： GET  /artichels/published
>
> 正确写法： GET /artichels?published=true

## 过滤信息（Filtering）

## 状态码如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。

下面是一些常见的参数。

> - ?limit=10：指定返回记录的数量
> - ?offset=10：指定返回记录的开始位置。
> - ?page=2&per_page=100：指定第几页，以及每页的记录数。
> - ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
> - ?animal_type_id=1：指定筛选条件

参数的设计允许存在冗余，即允许API路径和URL参数偶尔有重复。比如，GET /zoo/ID/animals 与 GET /animals?zoo-id=ID 的含义是相同的。推荐后者，避免出现多级URL。

### 2.1 状态码必须精确

客户端的请求，服务求都必须响应，包含 HTTP 状态码和数据。

HTTP 状态码就是一个三位数，分成五个类别。

- 1xx：相关信息
- 2xx：操作成功
- 3xx：重定向
- 4xx：客户端错误
- 5xx：服务器错误

### 2.2  2xx 状态码

200状态码表示操作成功，但是不同的方法可以返回更精确的状态码。

- GET: 200 OK
- POST: 201 Created
- PUT: 200 OK
- PATCH: 200 OK
- DELETE: 204 No Content

### 2.3 4xx 状态码

4xx状态码表示客户端错误，主要有下面几种。

- 400 Bad Request：服务器不理解客户端的请求，未做任何处理。
- 401 Unauthorized：用户未提供身份验证凭据，或者没有通过身份验证。
- 403 Forbidden：用户通过了身份验证，但是不具有访问资源所需的权限。
- 404 Not Found：所请求的资源不存在，或不可用。
- 405 Method Not Allowed：用户已经通过身份验证，但是所用的 HTTP 方法不在他的权限之内。
- 410 Gone：所请求的资源已从这个地址转移，不再可用。
- 415 Unsupported Media Type：客户端要求的返回格式不支持。比如，API 只能返回 JSON 格式，但是客户端要求返回 XML 格式。
- 422 Unprocessable Entity ：客户端上传的附件无法处理，导致请求失败。
- 429 Too Many Requests：客户端的请求次数超过限额。

### 2.4 5xx 状态码

5xx状态码表示服务端错误。一般来说，API 不会向用户透露服务器的详细信息，所以只要两个状态码就够了。

- 500 Internal Server Error：客户端请求有效，服务器处理时发生了意外。
- 503 Service Unavailable：服务器无法处理请求，一般用于网站维护状态。

## 服务器响应

### 3.1 不要返回纯文本

API 返回的数据格式，不应该是纯文本，而应该是一个 JSON 对象，因为这样才能返回标准的结构化数据。所以，服务器回应的 HTTP 头的 Content-Type 属性要设为 application/json 。

客户端请求时，也要明确告诉服务器，可以接受 JSON 格式，即请求的 HTTP 头的ACCEPT 属性也要设成 application/json。下面是一个例子。

### 3.2 发生错误的时候，不要返回 200 状态码

有一种不恰当的做法是，即使发生错误，也返回200状态码，把错误信息放在数据体里面，就像下面这样。

错误例子：

```
HTTP/1.1 200 OK
ConteNTP-Type: application/json

{
	"status": "fail",
	"msg": "错误"
}
复制代码
```

上面代码中，解析数据体以后，才能得知操作失败。

这张做法实际上取消了状态码，这是完全不可取的。正确的做法是，状态码反映发生的错误，具体的错误信息放在数据体里面返回。下面是一个例子。

正确方式：

```
HTTP/1.1 400 Bad Request
ConteNTP-Type: application/json

{
	"status": "fail",
	"msg": "错误"
}
复制代码
```

## docker RESTful 规范解析

接下来我们分析 docker api 对于 restful 的使用，助于我们在实际工作中合理设计。

docker 文档 url ：[docs.docker.com/engine/api/…](https://docs.docker.com/engine/api/v1.19/)

### 4.1 GET 获取容器列表

```
GET /v1.19/containers/json?all=1&before=8dfafdbc3a40&size=1 HTTP/1.1
```

通过 all=1&before=8dfafdbc3a40&size=1 过滤容器数据

### 4.2 GET 获取指定ID或者名字容器

```
GET /containers/(id or name)/json
GET /v1.19/containers/4fa6e0f0c678/json HTTP/1.1
```

### 4.3 GET 获取某个容器的进程

```
GET /v1.19/containers/4fa6e0f0c678/top HTTP/1.1
```

假如想过滤进程等可以通过查询字符串实现

```
GET /v1.19/containers/4fa6e0f0c678/top?ps_args=aux HTTP/1.1
```

返回的数据

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "Titles" : [
    "USER","PID","%CPU","%MEM","VSZ","RSS","TTY","STAT","START","TIME","COMMAND"
  ]
  "Processes" : [
    [
      "root","13642","0.0","0.1","18172","3184","pts/0","Ss","17:03","0:00","/bin/bash"
    ],
    [
      "root","13895","0.0","0.0","4348","692","pts/0","S+","17:15","0:00","sleep 10"
    ]
  ],
}
复制代码
```

### 4.4 POST 创建容器

```
POST /containers/create
POST /v1.19/containers/create HTTP/1.1
Content-Type: application/json
Content-Length: 12345

{
       "Hostname": "",
       "Domainname": "",
       "User": "",
       "AttachStdin": false,
       "AttachStdout": true,
       "AttachStderr": true,
       "Tty": false,
       "OpenStdin": false,
       "StdinOnce": false,
       "Env": [
               "FOO=bar",
               "BAZ=quux"
       ],
       "Cmd": [
               "date"
       ],
       "Entrypoint": null,
       "Image": "ubuntu",
       "Labels": {
               "com.example.vendor": "Acme",
               "com.example.license": "GPL",
               "com.example.version": "1.0"
       },
       "Volumes": {
         "/volumes/data": {}
       }
   
  }
```

## 4.5 DELETE 删除容器

根据容器 id 删除一个容器，v是请求是否删除 容器 volumes

```
DELETE /v1.19/containers/16253994b7c4?v=1 HTTP/1.1
```

**Query parameters**:

- **v** – 1/True/true or 0/False/false, Remove the volumes associated to the container. Default `false`.
- **force** - 1/True/true or 0/False/false, Kill then remove the container. Default `false`.
- **link** - 1/True/true or 0/False/false, Remove the specified link associated to the container. Default `false`.