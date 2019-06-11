# 使用 JWT 让你的 RESTful API 更安全

传统的 cookie-session 机制可以保证的接口安全，在没有通过认证的情况下会跳转至登入界面或者调用失败。

在如今 RESTful 化的 API 接口下，cookie-session 已经不能很好发挥其余热保护好你的 API 。

更多的形式下采用的基于 Token 的验证机制，JWT 本质的也是一种 Token，但是其中又有些许不同。

## 什么是 JWT ？

JWT 及时 JSON Web Token，它是基于 [RFC 7519](javascript:void()) 所定义的一种在各个系统中传递**紧凑**和**自包含**的 JSON 数据形式。

- **紧凑（Compact）** ：由于传送的数据小，JWT 可以通过GET、POST 和 放在 HTTP 的 header 中，同时也是因为小也能传送的更快。
- **自包含（self-contained）** : Payload 中能够包含用户的信息，避免数据库的查询。

JSON Web Token 由三部分组成使用 `.` 分割开：

- Header
- Payload
- Signature

一个 JWT 形式上类似于下面的样子：

```
xxxxx.yyyy.zzzz
```

### Header

jwt的头部承载两部分信息：

- typ：声明类型，这里是jwt
- alg：声明加密的算法，通常直接使用 HMAC SHA256

alg 是所使用的 hash 算法例如 HMAC SHA256 或 RSA，typ 是 Token 的类型自然就是 JWT。

完整的头部就像下面这样的JSON：

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

然后使用 Base64Url 编码成第一部分。

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.<second part>.<third part>
```

### Payload

载荷就是JWT 主要的信息存储部分。这个名字像是特指飞机上承载的货品，其中包含了许多种的声明（claims）。

Claims 的实体一般包含用户和一些元数据，这些 claims 分成三种类型：reserved, public, 和 private claims。

- **（保留声明）reserved claims** ：预定义的 [一些声明](javascript:void())，并不是强制的但是推荐。

  > 这里都使用三个字母的原因是保证 JWT 的紧凑

- **（公有声明）public claims**：公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息。但不建议添加敏感信息，因为该部分在客户端可解码.。

- **（私有声明）private claims**：私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为是base64编码，意味着该部分信息可以归类为明文信息。

保留声明 (建议但不强制使用) ：

- iss: jwt签发者
- sub: jwt所面向的用户
- aud: 接收jwt的一方
- exp: jwt的过期时间，这个过期时间必须要大于签发时间
- nbf: 定义在什么时间之前，该jwt都是不可用的.
- iat: jwt的签发时间
- jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。

一个 Pyload 可以是这样子的：

```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

这部分同样使用 Base64Url 编码成第二部分。

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.<third part>
```

### Signature

在创建该部分时候你应该已经有了 编码后的 Header 和 Payload 还需要一个秘钥，这个加密的算法应该 Header 中指定。

一个使用 HMAC SHA256 的例子如下:

```
HMACSHA256(
  base64UrlEncode(header) + "." +  base64UrlEncode(payload),
  secret
)
```

signature 用于验证消息的发送者以及消息是没有经过篡改的。 完整的JWT 完整的JWT格式的输出是以`.`分隔的三段Base64编码，与SAML等基于XML的标准相比，JWT在HTTP和HTML环境中更容易传递。

所以，做后你的一个完整的 JWT 应该是如下形式：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

> 注意被 `.` 分割开的三个部分

## JSON Web Token 的工作流程

在用户使用证书或者账号密码登入的时候一个 JSON Web Token 将会返回，同时可以把这个 JWT 存储在local storage、或者 cookie 中，用来替代传统的在服务器端创建一个 session 返回一个 cookie。

![这里写图片描述](使用 JWT 让你的 RESTful API 更安全.assets/20180412115551744.png)

1. 用户使用用户名/密码，向认证服务器请求。
2. 认证服务器认证通过后，生成JWT token返回给用户
3. 用户向应用服务器调用API请求资源时都需添加JWT token
4. 应用服务器接收请求后先取出JWT token进行校验，通过后返回有效资源，否则返回错误。

当用户想要使用受保护的路由时候，应该要在请求得时候带上 JWT ，一般的是在 header 的 **Authorization** 使用 **Bearer** 的形式，一个包含的 JWT 的请求头的 Authorization 如下：

```
Authorization: Bearer <token>
```

这是一中无状态的认证机制，用户的状态从来不会存在服务端，在访问受保护的路由时候回校验 HTTP header 中 Authorization 的 JWT，同时 JWT 是会带上一些必要的信息，不需要多次的查询数据库。

这种无状态的操作可以充分的使用数据的 APIs，甚至是在下游服务上使用，这些 APIs 和哪服务器没有关系，因此，由于没有 cookie 的存在，所以在不存在跨域（CORS, Cross-Origin Resource Sharing）的问题。

### 什么时候使用JWT

- 身份验证：这是 JWT 最常用的场景。**一旦用户登录进系统，每一个后继的请求都会包含 JWT。**系统允许用户访问那些令牌有权限的路由，服务和资源。单点登录是一个在当代广泛使用 JWT 的特性，这是因为下面两点原因：其一是 JWT 的系统开销很小；其二是 JWT 能够轻而易举地应用到跨域的场景中。
- 信息交换：JWT 是一种在系统各个部分之间安全地传送信息的方法。这是因为只要 JWT 可以被数字签名，比如使用公私密钥对，你就可以信任 JWT 中包含的发送者身份信息。除此之外，当签名在计算的时候，使用了 JWT 的头部部分和负载部分，你也可以验证信息的内容有没有被篡改过。

### JWT何时下发

- 登录下发新token,原token实际上并没有失效,保证多端登录没问题.
- Token剩余有效时长大于可续期时长(根据业务平台自己定)时重新下发.举个栗子针对WEB端Token24小时有效,当有效时间小于12小时刷新Token,也就是当用户连续12小时没操作网站才会被退出.
- 修改或者重置密码时下发新Token,并吊销之前的Token

### JWT吊销

JWT不需要在服务端存储，因此吊销是个大问题，无法吊销的话就会出现用户密码被盗,即使用户修改了密码，其他人也并不会立即失效，这点在安全性很高的地方几乎是不允许的情况。因此吊销是必要的。

要完美地失效JWT是没办法做到的。有以下几个方法可以做到失效 JWT token：

1. 将 token 存入 DB（如 Redis）中，失效则删除；但增加了一个每次校验时候都要先从 DB 中查询 token 是否存在的步骤，而且违背了 JWT 的无状态原则（这不就和 session 一样了么？）。
2. 维护一个 token 黑名单，失效则加入黑名单中。
3. 在 JWT 中增加一个版本号字段，失效则改变该版本号。
4. 在服务端设置加密的 key 时，为每个用户生成唯一的 key，失效则改变该 key。

### JWT安全

- 不应该在jwt的payload部分存放敏感信息，因为该部分是客户端可解密的部分。
- 保护好secret私钥，该私钥非常重要。
- 建议的方式是通过SSL加密的传输（https协议），从而避免敏感信息被嗅探。

## 在 Flask 和 Express 中使用 JSON Web Token

JWT 在各个 Web 框架中都有 JWT 的包可以直接使用，下面使用 Flask 和 Express 作为例子演示。

- [Flask-JWT](javascript:void())
- [express-jwt](javascript:void())

下面会使用 [httpie](javascript:void()) 作为演示工具：

```
HTTPie: HTTP client, a user-friendly cURL replacement.

- Download a URL to a file:
    http -d example.org

- Send form-encoded data:
    http -f example.org name="bob" profile-picture@"bob.png"

- Send JSON object:
    http example.org name="bob"

- Specify an HTTP method:
    http HEAD example.org

- Include an extra header:
    http example.org X-MyHeader:123

- Pass a user name and password for server authentication:
    http -a username:password example.org

- Specify raw request body via stdin:
    cat data.txt | http PUT example.org
```

### Flask 中使用 JSON Web Token

这里的演示是 `Flask-JWT` 的 Quickstart内容。

安装必要的软件包：

```
pip install flask
pip install Flask-JWT
```

一个简单的 DEMO：

```
from flask import Flask
from flask_jwt import JWT, jwt_required, current_identity
from werkzeug.security import safe_str_cmp

class User(object):
    def __init__(self, id, username, password):
        self.id = id
        self.username = username
        self.password = password

    def __str__(self):
        return "User(id="%s")" % self.id

users = [
    User(1, "user1", "abcxyz"),
    User(2, "user2", "abcxyz"),
]

username_table = {u.username: u for u in users}
userid_table = {u.id: u for u in users}

def authenticate(username, password):
    user = username_table.get(username, None)
    if user and safe_str_cmp(user.password.encode("utf-8"), password.encode("utf-8")):
        return user

def identity(payload):
    user_id = payload["identity"]
    return userid_table.get(user_id, None)

app = Flask(__name__)
app.debug = True
app.config["SECRET_KEY"] = "super-secret"

jwt = JWT(app, authenticate, identity)

@app.route("/protected")
@jwt_required()
def protected():
    return "%s" % current_identity

if __name__ == "__main__":
    app.run()
```

首先需要获取用户的 JWT：

```
% http POST http://127.0.0.1:5000/auth username="user1" password="abcxyz"             ~
HTTP/1.0 200 OK
Content-Length: 193
Content-Type: application/json
Date: Sun, 21 Aug 2016 03:48:41 GMT
Server: Werkzeug/0.11.10 Python/2.7.10

{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZGVudGl0eSI6MSwiaWF0IjoxNDcxNzUxMzIxLCJuYmYiOjE0NzE3NTEzMjEsImV4cCI6MTQ3MTc1MTYyMX0.S0825N6IliQb65QoJfUXb3IGq-j9OVJpHBh-bcUz_gc"
}
```

使用 `@jwt_required()` 装饰器来保护你的 API

```
@app.route("/protected")
@jwt_required()
def protected():
    return "%s" % current_identity
```

这时候你需要在 HTTP 的 header 中使用 `Authorization: JWT <token>` 才能获取数据

```
% http http://127.0.0.1:5000/protected Authorization:"JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZGVudGl0eSI6MSwiaWF0IjoxNDcxNzUxMzIxLCJuYmYiOjE0NzE3NTEzMjEsImV4cCI6MTQ3MTc1MTYyMX0.S0825N6IliQb65QoJfUXb3IGq-j9OVJpHBh-bcUz_gc"
HTTP/1.0 200 OK
Content-Length: 12
Content-Type: text/html; charset=utf-8
Date: Sun, 21 Aug 2016 03:51:20 GMT
Server: Werkzeug/0.11.10 Python/2.7.10

User(id="1")
```

不带 JWT 的时候会返回如下信息：

```
% http http://127.0.0.1:5000/protected                                                ~
HTTP/1.0 401 UNAUTHORIZED
Content-Length: 125
Content-Type: application/json
Date: Sun, 21 Aug 2016 03:49:51 GMT
Server: Werkzeug/0.11.10 Python/2.7.10
WWW-Authenticate: JWT realm="Login Required"

{
    "description": "Request does not contain an access token",
    "error": "Authorization Required",
    "status_code": 401
}
```

### Express 中使用 JSON Web Token

Auth0 提供了 express-jwt 这个包，在 express 可以很容易的集成。

```
npm install express --save
npm install express-jwt --save
npm install body-parser --save
npm install jsonwebtoken --save
npm install shortid --save
```

本例子中只是最简单的使用方法，更多使用方法参看 [express-jwt](javascript:void())

```
var express = require("express");
var expressJwt = require("express-jwt");
var bodyParser = require("body-parser");
var jwt = require("jsonwebtoken");
var shortid = require("shortid");

var app = express();

app.use(bodyParser.json());
app.use(expressJwt({secret: "secret"}).unless({path: ["/login"]}));
app.use(function (err, req, res, next) {
  if (err.name === "UnauthorizedError") {
    res.status(401).send("invalid token");
  }
});


app.post("/login", function(req, res) {
  var username = req.body.username;
  var password = req.body.password;

  if (!username) {
    return res.status(400).send("username require");
  }
  if (!password) {
    return res.status(400).send("password require");
  }

  if (username != "admin" && password != "password") {
    return res.status(401).send("invaild password");
  }

  var authToken = jwt.sign({username: username}, "secret");
  res.status(200).json({token: authToken});

});

app.post("/user", function(req, res) {
  var username = req.body.username;
  var password = req.body.password;
  var country = req.body.country;
  var age = req.body.age;

  if (!username) {
    return res.status(400).send("username require");
  }
  if (!password) {
    return res.status(400).send("password require");
  }
  if (!country) {
    return res.status(400).send("countryrequire");
  }
  if (!age) {
    return res.status(400).send("age require");
  }

  res.status(200).json({
    id: shortid.generate(),
    username: username,
    country: country,
    age: age
  })
})

app.listen(3000);
```

`express-jwt` 作为 express 的一个中间件，需要设置 `secret` 作为秘钥，unless 可以排除某个接口。

默认的情况下，解析 JWT 失败会抛出异常，可以通过以下设置来处理该异常。

```
app.use(expressJwt({secret: "secret"}).unless({path: ["/login"]}));
app.use(function (err, req, res, next) {
  if (err.name === "UnauthorizedError") {
    res.status(401).send("invalid token");
  }
});
```

`/login` 忽略的 JWT 认证，通过这个接口获取某个用户的 JWT

```
% http POST http://localhost:3000/login username="admin" password="password" country="CN" age=22 
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 143
Content-Type: application/json; charset=utf-8
Date: Sun, 21 Aug 2016 06:57:42 GMT
ETag: W/"8f-iMzAS1K5StDQgtNnVSvqtQ"
X-Powered-By: Express

{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNDcxNzYyNjYyfQ.o5RFJB4GiR28HzXbSptU6MsPwW1tSXSDIjlzn7erG0M"
}
```

不使用 JWT 的时候

```
% http POST http://localhost:3000/user username="hexiangyu" password="password" ~
HTTP/1.1 401 Unauthorized
Connection: keep-alive
Content-Length: 13
Content-Type: text/html; charset=utf-8
Date: Sun, 21 Aug 2016 07:00:02 GMT
ETag: W/"d-j0viHsPPu6FaNJ6cXoiFeQ"
X-Powered-By: Express

invalid token
```

使用 JWT 就可以成功调用

```
% http POST http://localhost:3000/user Authorization:"Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNDcxNzYyNjYyfQ.o5RFJB4GiR28HzXbSptU6MsPwW1tSXSDIjlzn7erG0M" username="hexiangyu" password="password" country="CN" age=22
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 66
Content-Type: application/json; charset=utf-8
Date: Sun, 21 Aug 2016 07:04:34 GMT
ETag: W/"42-YnGYuyDLxpVUexEGEcQj1g"
X-Powered-By: Express

{
    "age": "22",
    "country": "CN",
    "id": "r1sFMCUc",
    "username": "hexiangyu"
}
```

## Reference

- [JSON Web Token Introduction](javascript:void())
- [IANA JSON Web Token](javascript:void())
- [Flask-JWT](javascript:void())
- [express-jwt](javascript:void())