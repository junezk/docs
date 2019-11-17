# vue-router结合vuex实现用户权限控制

### 1.服务端准备工作

为了实现前端校验用户，后端需要在用户登录的时候记录下该用户的状态并加密之后返回给前端。之后该用户的所有请求都应该附带这个加密后的状态，后端取到这个状态解密，并与之前保存的状态对比，以此来判断该用户是否登录或合法。

我这里使用了node简单了写了个本地的express服务，来实现上述功能。完整的代码直接贴出来：

```
// server.js
const express = require('express');
const bodyParser = require('body-parser');
const jwt = require('jsonwebtoken');

const app = express();
// secret是后端加密的密钥
const secret = 'rhwl';

app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Methods', 'GET,HEAD,OPTIONS,POST,PUT');
  res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept, Authorization');
  if (req.method.toLowerCase() === 'options') {
    return res.end();
  }
  next();
});
app.use(bodyParser.json());
app.post('/login', (req, res) => {
  const { username } = req.body;
  if (username === 'admin') { // 如果是合法用户，使用jwt进行加密生成token
    res.json({
      code: 0,
      username: 'admin',
      token: jwt.sign({ username: 'admin' }, secret, {
        expiresIn: 20,
      }),
    });
  } else {
    res.json({
      code: 1,
      data: '用户名不存在',
    });
  }
});
app.get('/validate', (req, res) => {
  const token = req.headers.authorization; // 在请求头中附带token信息
  jwt.verify(token, secret, (err, decode) => { // 验证token是否合法
    if (err) {
      return res.json({
        code: 1,
        data: '当前token无效',
      });
    }
    // 如果验证合法，重新生成新的token，并返回信息
    res.json({
      username: decode.username,
      code: 0,
      token: jwt.sign({ username: 'admin' }, secret, {
        expiresIn: 20,
      }),
    });
  });
});

app.listen(3000, ()=>{
    console.log('服务器在3000端口运行');
});
```

### 2.项目中axios封装

然后我们在项目中封装符合自己需求的ajax请求，现在通常都是基于axios库。在自己封装的ajax插件中，要在每次的请求头中添加上token。代码实现：

```
// ajaxResquest.js
import axios from 'axios';

class ajaxResquest {
    constructor(){
        // 根据当前模式自动切换baseURL
        this.baseURL = process.env.NODE_ENV === 'development' ? 'http://localhost:3000' : '/';
        this.timeout = 5000; // 设置请求超时为5s
    }
    request(config){
        const instance = axios.create({
            baseURL: this.baseURL,
            timeout: this.timeout,
        });
        
        instance.interceptor.request.use((config) => {
            // 将保存在本地的token添加到每次请求的请求头中
            // 这样就可以实现在请求时顺带附加用户的校验信息的需求
            config.headers.Authorization = localStorage.getItem('token');
            return config;
        }, (err) => {
            return Promise.reject(err);
        });
        instance.interceptor.response.use((req,res) => {
            return req.data;
        }, (err) => {
            Promise.reject(err);
        });
        
        // 将使用request时候需要的参数也添加到instance中
        return instance(config);
    }
}

export default new ajaxRequest();
```

然后统一管理项目api接口：

```
// api.js
import ajax from 'ajaxResquest';
export const userLogin = (username) => ajax.request({url: '/login', method: 'POST', data: {
    username,
}});
export const userValidate = () => ajax.request({url: '/validate'});
```

接下来我们在项目中具体实现用户登陆和权限校验的需求。

### 3.vuex记录用户登录

先将登陆组件配合vuex使用来触发用户登陆的行为，并且将用户登录之后的信息保存在vuex中，登陆组件的代码：

```
// userLogin component
<template>
    <div>
        <el-input style="width:200px" v-model="username"></el-input>
        <el-button @click="login">登录</el-button>
    </div>
</template>
<script>
export default {
    data(){
        return {
            username: '',
        }
    },
    methods: {
        login(){
            // 这里触发vuex中的actions，在vuex中调用用户登陆接口
            // 从而将用户登陆之后的状态保存至vuex中
            this.$store.dispatch('login', this.username).then((data) => {
                // 登陆成功之后，路由跳转至用户账户页或者进行你需要的操作
                this.$router.push('/profile');
            });
        }
    }
}
</script>
```

接着是vuex的store.js

```
// store.js
import Vue from 'vue';
import Vuex from 'vuex';
import {userLogin, userValidate} from 'api.js';

Vue.use(Vuex);

export default Vuex.store({
    state: {
        username: '',  
    },
    mutations: {
        setUsername(state, username){
            state.username = username;
        }
    },
    actions: {
        async login({commit}, username){
            const res = await userLogin(username);
            if (res.code === 1) { // 登录失败
                return Promise.reject(res);
            }
            // 登录成功后将接口返回的token保存在本地
            localStorage.setItem('token', res.token);
            // 将用户名保存在vuex中
            commit('setUsername', username);
        }
    }
});
```

经过上面的操作，我们将用户登录中调用登录接口的操作通过vuex实现，将成功登录后的用户名保存在vuex中，此时的token保存在浏览器本地。但是vuex中的数据并不是持久数据，刷新之后保存的用户名就会消失，接下来我们实现刷新页面或者路由跳转时进行用户校验，如果验证通过则会生成新的token和username并保存。

### 4.vuex配合vue-router实现登录校验

当用户刷新页面时，或者点击其他页面切换路由router时，需要调用后端的validate接口，该接口通过验证已保存的token校验当前用户是否合法。我们在vuex的store.js中添加以下代码：

```
export default Vuex.store({
    state: {
        username: '',
    },
    mutations: {
        setUsername(state, username){
            state.username = username;
        }  
    },
    actions: {
        async login({commit}, username){
            ...
        },
        async validate({commit}) {
            // 调用userValidate时，会将
            const res = await userValidate();
            if (res.code === 1) { // 此时用户校验失败
                return Promise.reject(res);
            }
            // 如果校验成功，重新保存token和username 
            localStorage.setItem('token', res.token);
            commit('setUsername', res.username);
        }
    }
});
```

基本上我们通过上面的代码就实现了用户权限控制所需要的所有前提操作：

- 用户成功登陆在本地保存token
- 在自己封装的ajax的请求头部添加保存的token信息
- 后端服务提供对前端token的校验能力

那么接下来就就是路由router刷新或改变的时候如何进行权限控制了。

### 5.vue-router钩子实现用户权限控制

使用过vue-router的同学们都知道，路有也是有钩子函数的，在官方文档里面被称为[导航守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)。导航守卫允许我们可以精准的在每个路由变化的时候进行操作，我们就这里判断用户权限。在vue项目的的main.js中修改：

```
import Vue from 'vue';
import App from './App.vue';
import router from './router';
import store from './store';
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

// 在这里使用路由的导航守卫进行权限控制
// 可以自定义不需要校验用户的路由白名单
const whiteList = ['/'];
router.beforeEach(async  (to, from, next) => {
    // 要去的页面是白名单，直接跳转
    if (whiteList.includes(to.path)) {
        next();
    }
    // 不是白名单，调用vuex中的validate行为
    const flag = await store.dispatch('validate');
    if (flag) { // 用户校验通过，直接跳转
        next();
    } else { // 用户校验失败
        next('/login'); // 跳转至用户登陆页
        // 顺带说一下，这里还可以在router中的meta属性中添加isNeeded: true/false
        // 然后配合这个属性更加精细的控制未通过用户校验时的页面是否允许跳转
    }
});

// vuex
Vue.use(ElementUI);
Vue.config.productionTip = false;

new Vue({
    router,
    store,
    render: h => h(App),
}).$mount('#app');
```

好了，以上就是由我整理的vuex配合路由router实现用户权限控制，希望对大家有所帮助。 （这次分享的内容来源于某家的vue高级课程，由本人进行整理和分享~）

*作者简介： 晨光，[人和未来](http://www.genetalks.com/about-us.html)大数据前端工程师。*