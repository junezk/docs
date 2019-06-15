# Vue后台管理系统用户认证封装

> 在使用vue技术栈开发各类管理系统时，经常需要相同的开发目录架设。在日常开发中，整理了一套自己的开发目录。主要包括`vuejs`、`vue-router`、`vuex`、`axios`等常用类库。

## 开发依赖

```
 "vuex": "^3.1.0",
 "babel-polyfill": "^6.26.0",
 "element-ui": "^2.9.1",
 "vue": "^2.5.2",
 "vue-router": "^3.0.1",
 "axios": "^0.18.0",
```

## Git仓库地址

[vue-template](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FHarhao%2Fvue-template)

## 开发目录

`vue-template`主要使用`axios`做请求处理，在`src/api/index.js`中对`axios`做了二次封装，然后把封装函数`gl_ajax`挂载在全局`vue.prototype`上，通过各层嵌套组件通过原型可以获取`gl_ajax`函数。在`router/index.js`主要是路由定义和路由导航守卫功能。详细情况可以参考提供源码地址。

```
|-- App.vue(挂载根组件)
|-- api（封装请求相关模块）
|   |-- api.js（公用api接口定义）
|   |-- config.js（项目请求地址和其他参数）
|   |-- index.js（axios二次封装定义）
|-- components（公用业务组件）
|-- main.js（程序执行入口）
|-- plugins（开发vue插件目录）
|   |-- el_icon(Icon组件)
|   |   |-- Icon.vue
|   |   |-- assets
|   |   |   |-- error.svg
|   |   |   |-- info.svg
|   |   |   |-- success.svg
|   |   |   |-- warning.svg
|   |   |-- index.js
|   |-- el_message（message提示组件）
|       |-- index.js
|       |-- main.js
|       |-- message.vue
|-- router（路由定义和路由守卫）
|   |-- index.js
|-- store（vuex定义和action）
|   |-- index.js
|   |-- types.js
|-- views（业务组件）
    |-- Home.vue
    |-- Login.vue
    |-- index.vue
```

## `main`入口

`main`入口对类库进行挂载，二次封装的函数`gl_ajax`、第三方组件库`elementui`。`vue`官方提议对挂载在`vue`原型链上的自定义函数加前缀`$`区分，这里使用`alias:$gl_ajax`来进行挂载。

```
import Vue from "vue";
import "babel-polyfill";
import App from "./App";
import router from "./router";
import store from "./store/index.js";
import { gl_ajax } from "./api/index.js";
import ElementUI from "element-ui";
import "element-ui/lib/theme-chalk/index.css";
Vue.prototype.$gl_ajax = gl_ajax;
Vue.config.productionTip = false;
Vue.use(ElementUI);
new Vue({
  el: "#app",
  store,
  router,
  components: { App },
  template: "<App/>"
});
```

## 路由定义和路由守卫

开发后台管理系统，某些页面需要对用户或游客进行权限验证。所以在路由文件添加路由守卫，阻止未登录用户对需权限的页面的操作。

在使用`vue`中采取前后端分离的形式，通过`JWT`获取对`api`接口的请求权限`token`,在登录中获取`token`，然后保存在`sessionStorage`中，之所以不把`token`保存在`vuex`中，是因为浏览器刷新会导致`store`数据重置。失去登录`token`。这里采用的是`sessionStorage`加`vuex`的形式，避免了上述的弊端。

```
// 页面刷新时，重新赋值token
if (window.sessionStorage.getItem("token")) {
  store.commit(types.LOGIN, window.sessionStorage.getItem("token"));
}
```

下面是`router/index.js`的完整定义源码

```
import Vue from "vue";
import VueRouter from "vue-router";
import store from "../store/index";
import * as types from "../store/types";
Vue.use(VueRouter);
const routes = [{
    path: '',
    redirect: '/login'
  }, {
    path: "/",
    component: resolve => require(["@/views/Home"], resolve),
    children: [{
      path: '/admin',
      name: 'admin',
      component: resolve => require(["@/views/index"], resolve)
    }]
  },
  {
    path: "/login",
    name: "Login",
    component: resolve => require(["@/views/Login"], resolve)
  }
];

// 页面刷新时，重新赋值token
if (window.sessionStorage.getItem("token")) {
  store.commit(types.LOGIN, window.sessionStorage.getItem("token"));
}

const router = new VueRouter({
  mode: "history",
  routes
});

//路由导航守卫
router.beforeEach((to, from, next) => {
  if (to.path == "/login") {
    store.commit(types.LOGOUT);
    next();
  } else {
    const token = sessionStorage.getItem("token");
    if (!token) {
      next({
        path: "/login",
        query: {
          redirect: to.fullPath
        }
      });
    } else {
      next();
    }
  }
});
export default router;
```

## 全局数据储存`vuex`

在路由导航中，使用`store`加`sessionStorage`形式保存登录`token`,在`store/index.js`中也定义了登录和退出操作`mutations`。

把退出操作定义在`store`中，是为了下面对登录权限过期，返回`401`状态码，进行重定向操作功能。

```
import Vue from "vue";
import Vuex from "vuex";
import * as types from "./types";
Vue.use(Vuex);
export default new Vuex.Store({
  state: {
    token: null
  },
  mutations: {
    [types.LOGOUT]: (state, data) => {
      sessionStorage.removeItem("token");
      state.token = null;
    },
    [types.LOGIN]: (state, data) => {
      sessionStorage.setItem("token", data);
      state.token = data;
    }
  },
  actions: {},
  getters: {}
});
```

## `axios`封装

`api/index.js`文件中对`axios`进行了二次封装。同一根目录下`config.js`定义请求超时时间、开发和线上请求`URL`。

## `axios`请求拦截

对`axios`请求错误拦截是重点部分。主要对权限过期`401`和请求超时操作`store`对保存的`token`清除，然后再对路由进行重定向。

```
axios.interceptors.response.use(
  response => {
    return response;
  },
  error => {
    if (error.response) {
      switch (error.response.status) {
        case 401:
          store.commit(types.LOGOUT);
          router.replace({
            path: "/login",
            query: {
              redirect: router.currentRoute.path
            }
          });
      }
      const message = error.response.data.message ?:'未知错误';
      Message.error(message);
    } else if (error.code == 'ECONNABORTED' && error.message.indexOf('timeout') != -1) {
      Message.error("请求已超时");
    }
    return Promise.reject(error);
  }
);
```

## `gl_ajax`请求封装

`responseType`是`axios`请求文件下载需要的接受类型，文件是通过`application/json`方式进行获取。

```
export const gl_ajax = params => {
  return axios({
      method: params.method.toLowerCase(),
      url: `${axios.defaults.baseURL}${params.url}`,
      data: params.method != "get" ? params.data : "",
      params: params.method == "get" ? params.data : "",
      responseType: params.file ? "blob" : ""
    })
    .then(res => {
      params.success && params.success(res);
    })
    .catch(err => {
      params.error && params.error(err);
    });
};
```

## `gl_ajax`使用

在`vue`的原型上定义了`$gl_ajax`函数，在`login`常见登录中需要请求接口获取`token`权限，下面对`$gl_ajax`介绍使用方式和对`token`保存操作。

`self.$store.commit(types.LOGIN,token)`在`vuex`中保存`token`,会在`sessionStorage`和`store`分别保存一份`token`值。防止浏览器刷新造成`vuex`中`token`丢失。

```html
<template>
    <div>
        <button @click='loginSystem'>login</button>
    </div>
</template>
<script>
import * as types from "../store/types.js";
export default {
  name: "Login",
  data() {
    return {};
  },
  methods: {
    loginSystem() {
      const self = this;
      this.$gl_ajax({
        url: "/login",
        method: "post",
        data: { ...self.loginData },
        success(res) {
          if (res.data.status == "ok") {
            const token = res.data.jwt;
            self.$store.commit(types.LOGIN, token);
            self.message("login success");
            self.$router.push({
              path: "/admin"
            });
          }
        },
        error(err) {}
      });
    }
  }
};
</script>
<style>
</style>
```

如果觉得有帮助，可以给个赞~或star~呀

> github地址：[vue-template](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FHarhao%2Fvue-template)