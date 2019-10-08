# Vue路由之JWT身份认证

## 一、JWT身份认证简介

> **JSON Web Token（JWT）**是目前最流行的**跨域**身份验证解决方案，相较于session机制，**服务器就不需要保存任何 session 数据了**，也就是说，**服务器变成无状态了**，从而比较容易实现扩展。JWT 实际上是一个令牌（Token），服务器会将一些**元数据**、**指定的secret**进行**签名并生成token**，并返回给客户端，客户端得到这个服务器返回的令牌后，需要**将其存储到 Cookie 或 localStorage 中**，此后，**每次与服务器通信都要带上这个令牌**，可以把它放到 Cookie 中自动发送，但这样做不能跨域，所以更好的做法是将其放到 HTTP 请求头 **Authorization** 字段里面。

## 二、JWT的使用

① 安装并引入**jsonwebtoken**模块；
② 对**元数据**、**secret**密钥进行签名，并生成对应的token；
③ 对token进行校验**是否过期**；

```js
const jwt = require("jsonwebtoken"); // 引入jwt
const secret = "this is a private key"; // 指定一个用于生成token的密钥字符串
const token = jwt.sign({ foo: 'bar' }, secret, { // 传入元数据和secret密钥，并指定过期时间生成token
    expiresIn: 5, // 单独一个数字表示多少秒
    // expiresIn: "10h", // 表示10小时后过期
    // expiresIn: "2d" // 表示2天后过期
});
console.log(`token is ${token}`);
setTimeout(() => { // 5秒后对该token进行校验
    jwt.verify(token, secret, (err, decoded) => {
        console.log(err);
        if (err) {
            console.log('token 已经失效了.');
        } else {
            console.log(`token data is ${JSON.stringify(decoded)}`);
        }
    });
}, 5000);
```

> 生成的token为一个很长的字符串，分为三部分，每部分由.号隔开，即 **头部.载荷.签名**，5秒后token校验结果为error，即token已经过期，校验的时候，会得到token的解码数据，主要包括生成token时候的**元数据**、**token的签发时间(iat)**、**token的过期时间(exp)**

```
// 生成的token字符串为
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJpYXQiOjE1NjY3MzE4MzEsImV4cCI6MTU2NjczMTgzNn0.cZZkExNnVqBtnfQN2vtU2Z7JB0PBo1CFyC5NiOywg54
// token decoded后的数据
token data is {"foo":"bar","iat":1566731831,"exp":1566731836}
```

## 三、封装axios

> 由于在使用jwt认证的时候，客户端向服务器发起请求的时候，**都要带上token**，即要获取到token并将其**放到请求头的Authorization字段**中，服务器才能从authorization中取出token并进行校验，所以我们必须通过**拦截器**去实现，在每次请求之前将请求进行拦截，然后添加上token，再继续向服务器发起请求。

```
import axios from "axios";
class Request {
    constructor() {
        this.baseURL = process.env.NODE_ENV === 'development' ? 'http://localhost:3000' : '/'; // 设置请求baseURL
        this.timeout = 2000; // 设置请求超时时间
    }
    request(config){// 这里的config是请求的时候传递的参数配置对象，比如url、method、data等
        const instance = axios.create({ // 创建axios实例
            baseURL: this.baseURL,
            timeout: this.timeout,
        });
        // 设置拦截器
        instance.interceptors.request.use((config) => { // 请求拦截之后就是要使用这个config, config表示整个请求对象
            config.headers.Authorization = localStorage.getItem('token'); // 将token从localStorage中取出并添加到请求头的Authorization字段上
            return config; // 返回请求对象，继续向服务器发起请求
        }, err => Promise.reject(err));
        // 设置响应拦截器
        instance.interceptors.response.use(res => res.data, err => Promise.reject(err));
    
        return instance(config);
    }
}
export default new Request();
```

## 四、通过路由钩子进行登录校验

> 我们需要在路由跳转之前，进行登录校验，即**校验登录的token是否已经过期**，如果token没有失效，则可以继续访问页面；如果token已经失效，那么检查一下**所访问的页面是否需要登录才能访问**，如果是需要登录后才能访问，那么跳转到登录页面；如果是不需要登录也能访问的页面则继续访问；

```
const whiteList = ["/"]; // 定义一个白名单列表
router.beforeEach(async (to, from, next) => {
  if (whiteList.includes(to.path)) { // 如果是访问的白名单中的页面
    return next(); // 不需要校验，直接返回继续访问该页面
  }
  const isTokenAvailable = await store.dispatch('validate'); // 校验token是否失效
  if (isTokenAvailable) { // 如果token未失效
    if(to.path === "/login") { // 如果访问的是login页面，则回到首页
      next("/");
    } else { // 如果访问的不是login页面，则继续访问当前要访问的页面
      next();
    }
  } else { // 如果token失效了
    const needLogin = to.matched.some(item => item.meta.needLogin); // 检测要访问的页面是否需要登录才能访问
    if(needLogin) { // 如果访问的页面是需要登录的
      next("/login"); // 跳转到登录页面
    } else { // 如果访问的页面是不需要登录的，则直接继续访问
      next();
    }
  }
});
```

> 上面item.meta.needLogin，这个needLogin是在router中进行自定义配置的，在配置路由的时候，**允许通过meta属性配置一些自定义的元数据**，如下所示:

```
export default new Router({
  routes: [
    {
      path: '/profile',
      name: 'profile',
      component: Profile,
      meta: {needLogin: true}
    }
  ]
})
```

## 五、总结

> jwt认证，主要就是Vue路由钩子**beforeEach()**的应用，以及请求拦截器的封装，在每次**路由跳转前**进行token认证(校验)，检测token是否失效，其校验过程就是**向服务器发起一个请求**，比如"/validate"，由于客户端请求**拦截器的作用**，会在发起"/validate"请求之前，在请求头的**Authorization**字段加上token，服务器收到token后就能对token是否有效进行校验了，然后返回token校验结果，**客户端再根据token的校验结果进行路由的具体跳转**。