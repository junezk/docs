# 后端鉴权系列： 基于Cookie、Session认证

说起鉴权大家应该都很熟悉, 不过作为前端开发来讲, 鉴权的流程大头都在后端小哥那边, 但是作为一个有志气的开发者肯定要好好学习整个鉴权流程以及方案, 不然怎么跟后端合作😄。

# 常见的鉴权方案

- [基于Cookie, Session认证](https://juejin.im/editor/posts/5d98272be51d4578176b4b6e)
- [JWT认证](https://juejin.im/post/5d9aadbf51882509334fb48b), `Token`认证
- `OAuth2`认证
- `SSO`单点登陆
- `LDAP`认证登陆
- 扫码登陆

## 基于Cookie, Session认证

关于`Cookie`使用推荐阅读,[HTTP cookies](https://link.juejin.im/?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FHTTP%2FCookies)。

先上大家常见的一张`Cookie`, `Session`流程图。

![img](后端鉴权系列 基于Cookie, Session认证.assets/16d9a5b8067a40fd)

### demo展示

下面通过`node` + `koa2` + `redis` + `mongodb`来展示上述的流程。

实现思路:

- 创建用户

```
1. 密码首先md5, 生成随机盐, 再次加盐md5保存数据库
2. 记得salt盐也要保存
```

- 登录时候

```
1. 验证密码是否正确(取出salt,对用户传过来的密码+salt再次签名去批评数据库保存的密码是否一致)
2. 正确后创建session对象(userID)存在redis,并设置过期时间
```

- 业务api

```
1. 获取客户端传过来的cookie
2. 用cookie+签名去redis读取是否有session对象,存在的话取出该用户id去数据库查询用户信息
```

#### 开发前准备

- 安装`node`
- 安装`redis`并且本地启动
- 安装`mongodb`并且本地启动

`note: 下面代码只是供demo展示, 具体代码结构设计在生产环境可不能这么写, 后面我会总结一篇关于koa最佳实践文章`。

### 启动mongodb

这里就不截图了,关于GUI推荐使用Robo 3T。

### 启动redis

![img](后端鉴权系列 基于Cookie, Session认证.assets/16d9a5d5c6913fa5)

然后通过终端查看你的redis有么有存储数据。

![img](后端鉴权系列 基于Cookie, Session认证.assets/16d9a5d9e48b2ea2)

app.js

```js
// app.js

const Koa = require("koa");
const Router = require("koa-router");
const bodyParser = require("koa-bodyparser");
const session = require("koa-session2");
const md5 = require("crypto-js/md5");
const mongoose = require("mongoose");

const config = require("./config.js");
const Store = require("./Store.js");
const User = require("./models/user.js");

const app = new Koa();
const router = new Router();

app.keys = ["this is my secret key"];
mongoose.connect(config.db, { useUnifiedTopology: true });
app.use(bodyParser());

app.use(
  session({
    key: "jssessionId"
  })
);

/**
 * @description 创建用户
 */
router.post("/user", async (ctx, next) => {
  const { username = "", password = "", age, isAdmin } = ctx.request.body || {};
  if (username === "" || password === "") {
    ctx.status = 401;
    return (ctx.body = {
      success: false,
      code: 10000,
      msg: "用户名或者密码不能为空"
    });
  }
  // 先对密码md5
  const md5PassWord = md5(String(password)).toString();
  // 生成随机salt
  const salt = String(Math.random()).substring(2, 10);
  // 加盐再md5
  const saltMD5PassWord = md5(`${md5PassWord}:${salt}`).toString();
  try {
    // 类似用户查找,保存的操作一般我们都会封装到一个实体里面,本demo只是演示为主, 生产环境不要这么写
    const searchUser = await User.findOne({ name: username });
    if (!searchUser) {
      const user = new User({
        name: username,
        password: saltMD5PassWord,
        salt,
        isAdmin,
        age
      });
      const result = await user.save();
      ctx.body = {
        success: true,
        msg: "创建成功"
      };
    } else {
      ctx.body = {
        success: false,
        msg: "已存在同名用户"
      };
    }
  } catch (error) {
    // 一般这样的我们在生成环境处理异常都是直接抛出 异常类, 再有全局错误处理去处理
    ctx.body = {
      success: false,
      msg: "serve is mistakes"
    };
  }
});

// 模拟登陆
router.post("/login", async (ctx, next) => {
  const { username = "", password = "" } = ctx.request.body || {};
  if (username === "" || password === "") {
    ctx.status = 401;
    return (ctx.body = {
      success: false,
      code: 10000,
      msg: "用户名或者密码不能为空"
    });
  }

  // 一般客户端对密码需要md5加密传输过来, 这里我就自己加密处理,假设客户端不加密。
  // 类似用户查找,保存的操作一般我们都会封装到一个实体里面,本demo只是演示为主, 生产环境不要这么写
  try {
    // username在注册时候就不会允许重复
    const searchUser = await User.findOne({ name: username });
    if (!searchUser) {
      ctx.body = {
        success: false,
        msg: "用户不存在"
      };
    } else {
      // 需要去数据库验证用户密码
      const md5PassWord = md5(String(password)).toString();
      const saltMD5PassWord = md5(
        `${md5PassWord}:${searchUser.salt}`
      ).toString();
      if (saltMD5PassWord === searchUser.password) {
        const store = new Store();
        const sid = await store.set(
          {
            id: searchUser._id
          },
          {
            maxAge: 1000 * 60 * 2 // 设定只有120s的有效时间
          }
        );
        ctx.cookies.set("jssessionId", sid);
        ctx.body = {
          success: true,
          msg: "登陆成功"
        };
      } else {
        ctx.body = {
          success: false,
          msg: "密码错误"
        };
      }
    }
  } catch (error) {
    ctx.body = {
      success: false,
      msg: "serve is mistakes"
    };
  }
});

// 获取用户信息
router.get(
  "/user",
  async (ctx, next) => {
    const store = new Store();
    const jssessionId = ctx.cookies.get("jssessionId");
    const userSession = await store.get(jssessionId);
    console.log("获取到请求的cookie", jssessionId, "session", userSession);
    if (!userSession) {
      ctx.status = 401;
      ctx.body = {
        success: false,
        msg: "oAuth Faill"
      };
    } else {
      ctx.userSession = userSession;
      await next();
    }
  },
  async (ctx, next) => {
    try {
      const { id } = ctx.userSession;
      const { name, age, isAdmin } = await User.findOne({ _id: id });
      ctx.body = {
        success: true,
        data: { name, age, isAdmin }
      };
    } catch (error) {
      ctx.body = {
        success: false,
        msg: "serve is mistakes"
      };
    }
  }
);

app.use(router.routes()).use(router.allowedMethods());
app.on("error", (err, ctx) => {
  console.error("server error", err, ctx);
});
app.listen(3000, () => {
  console.log("Server listening on port 3000");
});
```

config.js

```js
module.exports = {
	'db': 'mongodb://localhost:27017/test'
}
```

user.js

```js
const mongoose = require("mongoose");
const { Schema } = mongoose;

const userSchema = new Schema({
  name: String,
  password: String,
  salt: String,
  isAdmin: Boolean,
  age: Number
});

module.exports = mongoose.model("User", userSchema);
```

Store.js

```js
const Redis = require("ioredis");
const { Store } = require("koa-session2");

class RedisStore extends Store {
  constructor() {
    super();
    this.redis = new Redis(); // Connect to 127.0.0.1:6379
  }
  async get(sid, ctx) {
    try {
      const data = await this.redis.get(`jssessionId:${sid}`);
      return JSON.parse(data);
    } catch (err) {
      throw new Error(err);
    }
  }

  async set(session, { sid = this.getID(24), maxAge = 1000000 } = {}, ctx) {
    try {
      // EX: redis支持过了有效期自动删除
      await this.redis.set(
        `jssessionId:${sid}`,
        JSON.stringify(session),
        "EX",
        maxAge / 1000
      );
    } catch (err) {
      throw new Error(err);
    }
    return sid;
  }
}

module.exports = RedisStore;
```

### postman测试接口

![img](后端鉴权系列 基于Cookie, Session认证.assets/16d9a5ec357add27)

注意看返回的Set-Cookie, 接着我们看下redis

![img](后端鉴权系列 基于Cookie, Session认证.assets/16d9a5f057874c52)

已经存在一条数据, 另外它的有效时间是120S,过了120S该数据会自动清除。

### 通过另外接口去访问用户信息。

![img](后端鉴权系列 基于Cookie, Session认证.assets/16d9a5f54699fbe7)

是可以获取到用户信息的,说明一切正常。

### 120S之后再次调用该接口测试是否已经失效。

![img](后端鉴权系列 基于Cookie, Session认证.assets/16d9a5fbbbb8dbbd)

redis里面也确实自动清除了该条数据。

![img](后端鉴权系列 基于Cookie, Session认证.assets/16d9a5fea9892eae)

## 基于Session的方案存在的问题

- 服务端需要存储Session
- 由于Session需要经常进行快速查找,因此我们一般存储在内存中或者内- 存服务器中,当用户数量大的时候,需要占用大量的服务器资源
- 当你需要扩展时候,创建Session服务器不一定是验证Session的服务器,因此你需要把所有Session单独存储并共享
- 由于客户端使用 Cookie 存储 SessionID，在跨域场景下需要进行兼容性处理，同时这种方式也难以防范 CSRF 攻击。

## 备注

有错误的地方欢迎大家斧正, [源码地址](https://link.juejin.im/?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fgithub.com%2FNuoHui%2Fnode-auth)。