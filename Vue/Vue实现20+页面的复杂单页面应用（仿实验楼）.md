# Vue实现20+页面的复杂单页面应用（仿实验楼）

# 前言

初学Vue，搜`Vue项目`时出现的几乎都是TODO应用，音乐播放器之类复杂度并不如想象中高的应用，当然慢慢摸索实现出来也会知道Vue的一些功能，`v-on`,`v-bind`之类的绑定，`v-for`,`v-if` 等逻辑控制。但写完总感觉意犹未尽... 于是想实现一个稍微复杂些的应用。

项目主要实现的是[前端页面部分](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FHuberTRoy%2Fvue-shiyanlou)，数据来源于实验楼原站，原站API不支持跨域调用，需要使用一个[转发](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FHuberTRoy%2Fvue-shiyanlou-backend)。

# 效果展示

![路径](Vue实现20+页面的复杂单页面应用（仿实验楼）.assets/16d75de99ffd2560)

![课程](Vue实现20+页面的复杂单页面应用（仿实验楼）.assets/16d75dbb20555251)

![课程](Vue实现20+页面的复杂单页面应用（仿实验楼）.assets/16d75dcbf334f2ea)

# 技术栈

```
vue2` + `vuex` + `vue-router` + `axios` + `flex
```

# 知识点总结

## vue-router

### 懒加载

路由懒加载由[官方文档](https://link.juejin.im/?target=https%3A%2F%2Frouter.vuejs.org%2Fzh%2Fguide%2Fadvanced%2Flazy-loading.html%23%E6%8A%8A%E7%BB%84%E4%BB%B6%E6%8C%89%E7%BB%84%E5%88%86%E5%9D%97)中说明的作用是打包应用时的按需加载：最终的目的总是要打包发布，所以懒加载一定要知道。

```
const __import__ = file => () => import(`@/pages/${file}.vue`)
...
{
    name: 'home',
    component: __import__('home') 
}
```

### 路由切换过渡动画

单页面应用可以很方便的制作切换不同页面时使用的动画效果，像是手机APP中的左滑右滑之类的。这里实现的过渡动画效果是透明度由0到1的缓慢渐变浮现效果。使用Vue的过渡动画组件即可

![过渡](Vue实现20+页面的复杂单页面应用（仿实验楼）.assets/16d76901a0039464)



```
<template>
  <div id="app">
    <transition name="tab_router_view">
      <router-view></router-view>
    </transition>
  </div>
</template>
<style>
.tab_router_view-enter-active, .tab_router_view-leave-active {
    transition: opacity .8s;
}

.tab_router_view-enter, .tab_router_view-leave-to {
    opacity: 0;
}

.tab_router_view-enter-to, .tab_router_view-leave {
    opacity: 1;
}
</style>
```

这样每次切换路由时都会触发一个透明度从0到1的0.8秒渐变效果：

![渐变](Vue实现20+页面的复杂单页面应用（仿实验楼）.assets/16d7696cb8ed38e0)



### 路由导航守卫

[路由导航守卫](https://link.juejin.im/?target=https%3A%2F%2Frouter.vuejs.org%2Fzh%2Fguide%2Fadvanced%2Fnavigation-guards.html%23%E7%BB%84%E4%BB%B6%E5%86%85%E7%9A%84%E5%AE%88%E5%8D%AB)可以在每次路由变化时让我们做一些功能性的调整，如切换的这个页面如果需要登录那么我们可以这样：

```
...
{
    path: '/profile',
    name: 'profile',
    component: Profile,
    meta: {
        login: true
    }
}
router.beforeEach((to, from, next)=>{
    if (to.meta.login) {
        if (!store.state.loginState.isLogin) {
            next({name: 'login'})
        } else {
            next()
        }
    }
})
```

上面的守卫实现了如果当前页面要求登录但没有登录的话会跳转到登录页面，这样就不需要每个页面都设置一遍检查了， 另外**前置导航守卫**一定要记得调用`next`，否则不会有组件加载出来。

每次切换路由页面后我们很可能还需要一个**后置导航守卫**来替我们将页面滚到最上部

```
router.afterEach((to, from) => {
    window.scrollTo(0, 0)
})
```

区别于前置守卫，后置守卫没有`next`需要调用。

## vuex

[`vuex`](https://link.juejin.im/?target=https%3A%2F%2Fvuex.vuejs.org%2Fzh%2Fguide%2Factions.html)被用来进行状态管理，这是一个全局的数据仓库一类的东西，数据的下载，转换，保存都放在里面进行。Vue的组件里当然也可以进行数据的下载保存，也可以进行不同组件件的传递，但随着业务增多往往会变得相当麻烦。

不用`vuex`状态管理的话一般会顺着思路这么写：

数据由父组件下载并保存下来传递到子组件：

```
<SubComponent :data="dataOne">
</SubComponent>
...
data: function () {
    return {
        dataOne: {}
    }
},
created: async function () {
    let res = await get('http://www.ceshi.com')
    this.dataOne = res.data
}
```

子组件内有一个触发填好后发起页面修改，但子组件是不能直接修改父组件传递过来的数据的，所以又需要发送一个信号让父组件接受后由父组件修改：

子组件

```
props: {
    data: {
        type: Object,
        require: true
    }
},
methods: {
    change: fucntion (data) {
        this.$emit('change', data)
    }
}
```

父组件

```
<SubComponent :data="dataOne"
              @change="change"
>
</SubComponent>
...
methods: {
    change: async function (data) {
        await get(...)
    }
}
```

多写了一个`change`方法不说，还需要不忘记注册子组件的事件，想想这只是两个组件间的通信，随着组件的增多，本来只需要一个方法的事情会需要两个三个，事件的注册也越来越多，取决于嵌套了多少子组件，稍微不注意忘了注册哪个事件就要debug很长时间。

如果用了`vuex`就会简单很多，因为它是一个**全局**的状态管理。

父组件只需要负责把触发数据下载。

```
<SubComponent></SubComponent>
...
created: async function () {
    await this.$store.dispatch('dataStore/getData')
}
```

子组件内的数据就会跟随更改，修改数据也不用再通知父组件

```
computed: {
    dataOne: function () {
        return this.$store.state.dataStore.dataOne
    }
},
methods: {
    change: async function (data) {
        await this.$store.dispatch('dataStore/getData', data)
    }
}
```

这样的全局状态管理不会因为嵌套的子组件而增加复杂度，而且具有良好的可读性，如何操作数据在`store`里进行修改，组件里只需要触发即可。

## API跨域配置

实验楼原站也是使用Vue编写，前后端分离，使用API通讯，我们可以把API抓取下来使用，当然我已经抓好了，[实验楼API后端](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FHuberTRoy%2Fvue-shiyanlou-backend)，实验楼的API与展示在同一个域名下，所以不存在跨域问题，但我们开发的话就不是在同一个域名下，要使用的话绕不开的一环就是跨域了。由于服务器端不支持跨域，要使用只能做一个[转发](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FHuberTRoy%2Fvue-shiyanlou-backend)，先来了解一下跨域吧。

### 同源策略

同源策略限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的重要安全机制。也就是说 在`http://www.baidu.com`的Js，如果没有跨域配置是无法与`http://juejin.im`进行通讯的。

### 服务器端跨域配置

不管后端使用何种语言，要解决跨域就是要让服务器的回复中添加以下头：

```
Access-Control-Allow-Origin:*
Access-Control-Allow-Methods:POST
Access-Control-Allow-Headers:x-requested-with,content-type
```

`Access-Control-Allow-Origin` 表示可以请求的源，`*`表示任意一个都可以。

`Access-COntrol-Allow-Methods` 表示请求时的方法，`POST`表示只有`POST`请求才可以进行跨域请求，`GEt,PUT,DELETE`之类的如果没有标明统统不可以。

`Access-Control-Allow-Headers`表示跨域允许的首部。

### Vue中的配置

本地开发的话可以在Vue的配置文件中进行配置`config/index.js`，下面是配置的开发环境的跨域请求：

```
dev: {
    proxyTable: {
          '/api': {  
            target: 'http://localhost:8000/api',// 设置你调用的接口域名
            changeOrigin: true,  
            pathRewrite: {  
              '^/api/': '' // 这里替换的是 target 中的内容，使用的时候 '/api/demo' 就相当于'http://localhost:8000/api/demo'。
            }  
          }
    }
}
```

生产环境下的跨域：

生产环境下如果跨域不需要携带cookies认证那服务器配置了上面说的几个响应头即可，否则的话还需要

```
Access-Control-Allow-Credentials:true
```

以及

```
Access-Control-Allow-Origin
```

不可以为`*`，`*`与credentials是冲突的。

最后使用 axios 请求时需要配置`withCredentials = true`。

## CSS布局

项目中使用了`flex`布局，`flex`布局使用起来比较流畅，写起来也很简单，任意元素加上

```
.box {
    display: flex
}
```

这样这个元素的内容就会按照默认的`flex`进行布局

![布局](Vue实现20+页面的复杂单页面应用（仿实验楼）.assets/16d77d80bb2068a1)

默认水平走向，不会换行，从容器的最左端开始排列。 容器里元素的如何排列我们可以通过这些属性来设置：

```
flex-direction  排列方向 row | column
flex-wrap wrap | nowrap
flex-flow 上面两个的简写 row nowrap
justify-content 指定元素主轴（main axis）对其方式 flex-start | flex-end | center | space-between | space-around
align-items 指定元素交叉轴（corss axis）对其方式 flex-start | flex-end | center | baseline | stretch
```

了解上面这一点就可以把常规的布局方式熟悉一下，只用上面这些就可以搭建出首页。

这里是关于`flex`[更详细的资料](https://link.juejin.im/?target=https%3A%2F%2Fwww.runoob.com%2Fw3cnote%2Fflex-grammar.html)。

# 最后

完整的项目请看[GitHub](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FHuberTRoy%2Fvue-shiyanlou)。

克隆后直接启动即可：

```
git clone git@github.com:HuberTRoy/vue-shiyanlou.git
cd vue-shiyanlou
npm install
npm run dev
```