# axios 中文文档【译】

基于Promise的http库，适用于浏览器和node.js。

## 原文

[点击](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fmzabriskie%2Faxios)查看原文

## 特色

- 浏览器端发起XMLHttpRequests请求
- node端发起http请求
- 支持Promise API
- 拦截请求和返回
- 转化请求和返回（数据）
- 取消请求
- 自动转化json数据
- 客户端支持抵御[XSRF（跨站请求伪造）](https://link.juejin.im/?target=http%3A%2F%2Fbaike.baidu.com%2Flink%3Furl%3DeFrTMGA9IsLxlOnyqKky-t6vTs3g6YoAfFc1sYmv2fVVS1FrfIoI3q3jxUV_o1AgIIoLdk9N0Ni_TxLItoRU3K)

## 安装

使用npm：

```
$ npm i axios
```

使用 bower

```
$ bower instal axios
```

使用cdn

```html
<!--国内bootCDN-->
<script src="https://cdn.bootcss.com/axios/0.18.0/axios.min.js"></script>
```

## 示例

发起一个`GET`请求

```js
const axios = require('axios')

//发起一个user请求，参数为给定的ID
axios.get('/user?ID=12345')
.then(function(respone){
    console.log(response);
})
.catch(function(error){
    console.log(error);
});

//可配置的方式实现上面的代码如下
axios.get('/user',{
    params:{
        ID:12345
    }
}).then(function(response){
        console.log(response);
  }).catch(function(error){
        console.log(error)
    });
```

**注意：async/await是ECMAScript2017的语法，ie和一些老旧的浏览器不支持，请小心使用。**
**（译者话：使用ts的话可以很容易的编译和使用，也不需要任何垫片，目标编译成es5即可）**

发起一个`POST`请求

```js
axios.post('/user',{
    firstName:'friend',
    lastName:'Flintstone'
})
.then(function(response){
    console.log(response);
})
.catch(function(error){
    console.log(error);
});
```

发起一个多重并发请求

```js
function getUserAccount(){
    return axios.get('/user/12345');
}

function getUserPermissions(){
    return axios.get('/user/12345/permissions');
}

axios.all([getUerAccount(),getUserPermissions()])
    .then(axios.spread(function(acc,pers){
        //两个请求现在都完成
    }));
```

**译者话：因为axios.all使用的是类似Primise.all的功能，所以如果其中有一个请求出现了错误那么就会停止请求，所以建议对于单个请求最好附加上处理的catch。**

## axios API

------

根据自己需求可以对axios传递一些的设置来生成请求（Request）。

```js
axios(config)
//发起 POST请求

axios({
    method:'post',//方法
    url:'/user/12345',//地址
    data:{//参数
        firstName:'Fred',
        lastName:'Flintstone'
    }
});

//通过请求获取远程图片
axios({
    method:'get',
    url:'http://bit.ly/2mTM3Ny',
    responseType:'stream'
})
    .then(function(response){
        response.data.pipe(fs.createWriteStream('ada_lovelace.jpg'))
    })
复制代码
axios(url[,config])
//发起一个GET请求
axios('/user/12345/);
```

## 请求方法的别名。

为了方便，axios提供了所有请求方法的别名支持

**axios.request(config)**
**axios.get(url[,config])**
**axios.delete(url[,config])**
**axios.head(url[,config])**
**axios.options(url[,config])**
**axios.post(url[,data[,config]])**
**axios.put(url[,data[,config]])**
**axios.patch(url[,data[,config]])**

### 注意

当时用别名方法时`url`,`method`,以及`data`属性不必在config中明确指定（如上面示例的post方法请求）。

## 并发 Concurrency

有用的方法

**axios.all(iterable)**
**axios.spread(callback)**

### 创建一个实例（instance）

你可以使用自定义设置创建一个新的实例

**axios.create([config])**

```js
var instance = axios.create({
    baseURL:'http://some-domain.com/api/',
    timeout:1000,
    headers:{'X-Custom-Header':'foobar'}
});
```

### 实例方法

下面列出了一些实例可用方法，具体的设置将在实例中应用。

**axios#request(config)**
**axios#get(url[,config])**
**axios#delete(url[,config])**
**axios#head(url[,config])**
**axios#options(url[,config])**
**axios#post(url[,data[,config]])**
**axios#put(url[,data[,config]])**
**axios#patch(url[,data[,config]])**

## Requeset Config请求设置

以下列出了一些请求时的设置选项。只有`url`是必须的，如果没有指明`method`的话，默认的请求方法是`GET`.

```js
{
    //`url`是服务器链接，用来请求
    url:'/user',

    //`method`是发起请求时的方法
    method:`get`,

    //`baseURL`如果`url`不是绝对地址，那么将会加在其前面。
    //可以很方便的对相对地址的axios实例设置`baseUrl`。
    baseURL:'http://some-domain.com/api/',

    //`transformRequest`允许请求的数据在发送至服务器之前进行转化。
    //这个只适用于`PUT`,`POST`,`PATCH`方法。
    //数组中的最后一个函数必须返回一个字符串或者一个`ArrayBuffer`,或者`Stream`,`Buffer`实例,`ArrayBuffer`,`FormData`
    //此外你也可能需要设置下headers对象
    transformRequest:[function(data,headers){
        //依自己的需求对请求数据进行处理
        return data;
    }],

    //`transformResponse`允许对返回的数据传入then/catch之前进行处理
    transformResponse:[function(data){
        //依需要对数据进行处理
        return data;
    }],

    //`headers`是自定义的要被发送的信息头
    headers:{'X-Requested-with':'XMLHttpRequest'},

    //`params`是请求连接中的请求参数，必须是一个纯对象，或者URLSearchParams对象
    params:{
        ID:12345
    },
    
    //`paramsSerializer`是一个可选的函数，用来控制和序列化参数
    //例如：（https://ww.npmjs.com/package/qs,http://api.jquery.com/jquery.param/)
    paramsSerializer: function(params){
        return Qs.stringify(params,{arrayFormat:'brackets'})
    },

    //`data`是请求时作为请求体的数据
    //只适用于应用的'PUT','POST','PATCH'，请求方法
    //当没有设置`transformRequest`时，必须是以下其中之一的类型（不可重复？）：
    //-string(字符串),plain object(纯对象),ArrayBuffer,ArrayBufferView,URLSearchParams
    //-限浏览器：FormData(表格数据),File(文件数据),Blob
    //-限Node：Stream
    data:{
        firstName:'fred'
    },
    //`timeout`定义请求的时间，单位是毫秒。
    //如果请求的时间超过这个设定时间，请求将会停止。
    timeout:1000,
    
    //`withCredentials`表明跨跨域请求书否需要证明。
    withCredentials:false //默认值

    //`adapter`适配器，允许自定义处理请求，这会使测试更简单。
    //返回一个promise，并且提供一个有效的相应。（查看[response docs](#response-api)）
    adapter:function(config){
        /*...*/
    },

    //`auth`表明HTTP基础的认证应该被使用，并且提供证书。
    //这个会设置一个`authorization` 头（header），并且覆盖你在header设置的Authorization头信息。
    auth:{
        username:'janedoe',
        password:'s00pers3cret'
    },

    //`responsetype`表明服务器返回的数据类型，这些类型的设置应该是
    //'arraybuffer','blob','document','json','text',stream'
    responsetype:'json',

    //`responseEncoding`表明解析相应的编码方式。
    //**注意**会忽视responseType为stream或者客户端的请求。
    responseEncoding:'utf8'//默认值

    //`xsrfCookieName`时cookie做xsrf会话时的名字。
    xsrfCookieName:'XSRF-TOKEN',//默认值

    //`xsrfHeaderName` 是http头（header）的名字，并且该头携带xsrf的值
    xrsfHeadername:'X-XSRF-TOKEN'，//默认值

    //`onUploadProgress`允许处理上传过程的进程事件
    onUploadProgress: function(progressEvent){
        //本地过程事件发生时想做的事
    },

    //`onDownloadProgress`允许处理下载过程的进程事件
    onDownloadProgress: function(progressEvent){
        //下载过程中想做的事
    },

    //`maxContentLength` 定义http返回内容的最大字节容量
    maxContentLength: 2000,

    //`validateStatus` 定义promise的resolve和reject。
    //http返回状态码，如果`validateStatus`返回true（或者设置成null/undefined），promise将会resolve；其他的promise将reject。
    validateStatus: function(status){
        return status >= 200 && stauts < 300;//默认
    },

    //`maxRedirect`定义重导向到node.js中的最大数量。
    //如果值为0，那么没有redirect。
    maxREdirects:5,//默认值

    //`socketPath`定义一个在node.js中使用的 `UNIX Socket`。
    //例如 `/var/run/docker.sock`发送请求到docker daemon。
    //`socketPath`和`proxy`只可定义其一。
    //如果都定义则只会使用`socketPath`。
    socketPath:null,//默认值

    //`httpAgent` 和 `httpsAgent`当产生一个http或者https请求时分别定义一个自定义的代理，在nodejs中。
    //这个允许设置一些选选个，像是`keepAlive`--这个在默认中是没有开启的。
    httpAgent: new http.Agent({keepAlive:treu}),
    httpsAgent: new https.Agent({keepAlive:true}),

    //`proxy`定义服务器的主机名字和端口号。
    //`auth`表明HTTP基本认证应该跟`proxy`相连接，并且提供证书。
    //这个将设置一个'Proxy-Authorization'头(header)，覆盖原先自定义的。
    proxy:{
        host:127.0.0.1,
        port:9000,
        auth:{
            username:'cdd',
            password:'123456'
        }
    },

    //`cancelTaken` 定义一个取消，能够用来取消请求
    //（查看 下面的Cancellation 的详细部分）
    cancelToken: new CancelToken(function(cancel){
    })
}
```

## 返回响应概要 Response Schema

一个请求的返回包含以下信息

```js
{
    //`data`是服务器的提供的回复（相对于请求）
    data{},

    //`status`是服务器返回的http状态码
    status:200,


    //`statusText`是服务器返回的http状态信息
    statusText: 'ok',

    //`headers`是服务器返回中携带的headers
    headers:{},

    //`config`是对axios进行的设置，目的是为了请求（request）
    config:{}

    //`request`是获取当前相应的请求
    //它是node.js中最后一次的 ClientRequest的实例（在redirect中）
    //或者是在浏览器中的XMLHttpREquest实例
}
```

使用`then`时，你会接受到下面的信息：

```js
axios.get('/user/12345')
    .then(function(response){
        console.log(response.data);
        console.log(response.status);
        console.log(response.statusText);
        console.log(response.headers);
        console.log(response.config);
    });
```

使用`catch`时，或者传入一个`reject callback`作为`then`的第二个参数，那么返回的错误信息将能够被处理。

------

## 默认设置（Config Default)

你可以设置一个默认的设置，这设置将在之后的每次请求中生效。

### 全局默认设置 Global axios defaults

```js
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type']='application/x-www-form-urlencoded';
```

### 实例中自定义默认值 Custom instance default

```js
//当创建一个实例时进行默认设置
var instance = axios.create({
    baseURL:'https://api.example.com'
});

//或者在实例创建之后改变默认值
instance.defaults.headers.common['Authorization'] = AUTH_TOKEN;
```

### 设置优先级 Config order of precedence

设置(config)将按照优先顺序合并起来。首先的是在`lib/defaults.js`中定义的默认设置，其次是`defaults`实例属性的设置，最后是请求中`config`参数的设置。越往后面的等级越高，会覆盖前面的设置。
看下面这个例子：

```js
//使用默认库的设置创建一个实例，
//这个实例中，使用的是默认库的timeout设置，默认值是0。
var instance = axios.create();

//覆盖默认库中timeout的默认值
//此时，所有的请求的timeout时间是2.5秒
instance.defaults.timeout = 2500;

//覆盖该次请求中timeout的值，这个值设置的时间更长一些
instance.get('/longRequest',{
    timeout:5000
});
```

## 拦截器 interceptors

你可以在`请求`或者`返回`被`then`或者`catch`处理之前对他们进行拦截。

```js
//添加一个请求拦截器
axios.interceptors.request.use(function(config){
    //在请求发送之前做一些事
    return config;
},function(error){
    //当出现请求错误是做一些事
    return Promise.reject(error);
});

//添加一个返回拦截器
axios.interceptors.response.use(function(response){
    //对返回的数据进行一些处理
    return response;
},function(error){
    //对返回的错误进行一些处理
    return Promise.reject(error);
});
```

如果你需要在稍后移除拦截器,你可以

```js
var myInterceptor = axios.interceptors.request.use(function(){/*...*/});
axios.interceptors.request.eject(myInterceptor);
```

你可以在一个axios实例中使用拦截器

```js
var instance = axios.create();
instance.interceptors.request.use(function(){/*...*/});
```

## 错误处理 Handling Errors

```js
axios.get('user/12345')
    .catch(function(error){
        if(error.response){
            //存在请求，但是服务器的返回一个状态码
            //他们是在2xx之外
            console.log(error.response.data);
            console.log(error.response.status);
            console.log(error.response.headers);
        }else if(error.request){
          //如果是请求时的错误，且没有收到相应
          //`error.request`是一个浏览器的XMLHttpRequest实例，或者node.js的 ClientRequest实例。
          console.log(error.request)
        } 
        else{
            //一些错误是在设置请求时触发的
            console.log('Error',error.message);
        }
        console.log(error.config);
    });
```

你可以使用`validateStatus`设置选项自定义HTTP状态码的错误范围。

```js
axios.get('user/12345',{
    validateStatus:function(status){
        return status < 500;//当返回码小于等于500时视为错误
    }
});
```

## 取消 Cancellation

你可以使用**cancel token**取消一个请求

> axios的cancel token API是基于已经撤销的**cnacelable promises proposal**提议。

你可以使用`CancelToken.source`工厂函数创建一个cancel token，如下：

```js
var CancelToken = axios.CancelToken;
var source = CancelToken.source();

axios.get('/user/12345', {
    cancelToken:source.toke
}).catch(function(thrown){
    if(axiso.isCancel(thrown)){
        console.log('Rquest canceled', thrown.message);
    }else{
        //handle error
    }
});

//取消请求(信息参数设可设置的)
source.cancel("操作被用户取消");
```

你可以给`CancelToken`构造函数传递一个executor function来创建一个cancel token:

```js
var CancelToken = axios.CancelToken;
var source = CancelToken.source()

axios.get('/user/12345',{
  cancelToke:source.token
}).catch(function(thrown){
  if(axios.isCancel(throw)){
    console.log('请求已取消',throw.message)
  }else{
    //处理错误
  }
})

axios.post('/user/2345',{
  name:'new name'
},{
  cancelToken:source.toke
})

source.cancel('错做已被用户取消！')
```

你也可以给`CancelToke`的构造函数传递一个处理函数来生成一个cancel token。

```js
const CancelToken = axios.CancelToken
let cancel;

acios.get('/user/12345',{
  cancelToken:new CancelToken(function excutor(c){
    // 一个处理函数接受一个cancel函数作为参数
    cancel = c
  })
})

// 取消请求
cancel()

// 注意：你可以使用同一个cancel token取消多个请求。
```

------

## 使用 application/x-www-form-urlencoded 格式

默认情况下，axios串联js对象为`JSON`格式。为了发送`application/x-wwww-form-urlencoded`格式数据，
你可以使用一下的设置。

### 浏览器 Browser

在浏览器中你可以如下使用`URLSearchParams` API:

```js
var params = new URLSearchParams();
params.append('param1','value1');
params.append('param2','value2');
axios.post('/foo',params);
```

注意：`URLSearchParams`不支持所有的浏览器，但是这里有个[垫片](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FWebReflection%2Furl-search-params) （poly fill）可用（确保垫片在浏览器全局环境中）

其他方法：你可以使用[`qs`](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fljharb%2Fqs)库来对数据编码。

```js
var qs = require('qs');
axios.post('/foo', qs.stringify({'bar':123}));
```

或者其他方法(es6)

```js
import qs from 'qs';
const data = {'bar':123};
const options = {
  method:'POST',
  headers:{
    'content-type':'application/x-www-from-urlencoded'
  },
  data:qs.stringify(data),
  url
}
axios(options)
```

## 兼容Semver

当axios放出`1.0`版本时，一些不兼容的更改将会放在新的版本中。例如`0.5.1`和`0.5.4`有相同的api，但是`0.6.0`会有不兼容的变化。

**译者：小知识**
**在npn的版本习惯中，版本使用的习惯是a.b.c,其中c的变化是一些bug的修复，b是一些功能的添加，a是大的版本的变化，会出现不兼容的情况。**

## Node.js

在nodejs中，你可以如下使用`querystring`:

```js
var querystring = require('querystring');
axios.post('http://something.com/', querystring.stringify({foo:'bar'}));
```

你同样可以使用`qs`库。

------

## promises

axios 基于原生的ES6 Promise 实现。如果环境不支持请使用[垫片](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fstefanpenner%2Fes6-promise).

## TypeScript

axios 包含`TypeScript`定义

```js
import axios from 'axios'
axios.get('/user?ID=12345')
```


