# vue中如何实现后台管理系统的权限控制

## 一、前言

在广告机项目中，角色的权限管理是卡了挺久的一个难点。首先我们确定的权限控制分为两大部分，其中根据粒的大小分的更细：

- 接口访问的权限控制
- 页面的权限控制
  - 菜单中的页面是否能被访问
  - 页面中的按钮（增、删、改）的权限控制是否显示

下面我们就看一看是如何实现这些个权限控制的。

## 二、接口访问的权限控制

接口权限就是对用户的校验。正常来说，在用户登录时服务器需要给前台返回一个Token,然后在以后前台每次调用接口时都需要带上这个Token,

然后服务端获取到这个Token后进行比对，如果通过则可以访问。

现有的做法是在登录成功的回调中将后台返回的Token直接存储到`sessionStorage`，然在请求时将Token取出放入headers中传给后台，代码如下：

```
this.$http({
          method: 'get',
          url: 'test/query?id=20',
          withCredentials: true,
          headers: {
            token: sessionStorage.getItem('token'),
            name: sessionStorage.getItem('name')    //应后台需求传的用户名
          }
        }).then(response => {
          //请求成功后的操作
        })
```

后来在一些文章中发现axios可以在拦截器中直接将Token塞入`config.headers.Authorization`中，作为全局传入。下面是代码部分：

```
//main.js
import axios from 'axios'

// 实例化Axios，并进行超时设置
const service = axios.create({
    timeout: 5000
})
// baseURL
// axios.defaults.baseURL = 'https://api.github.com';

// http request 拦截器
// 每次请求都为http头增加Authorization字段，其内容为token
service.interceptors.request.use(
    config => {
        if (store.state.user.token) {
            config.headers.Authorization = `token ${store.state.user.token}`;
        }
        return config
    },
    err => {
        return Promise.reject(err)
    }
);
export default service
```

## 三、页面权限控制

在前面已经说到，页面权限控制又分为两种：

- 菜单中的页面是否能被访问
- 页面中的按钮（增、删、改）的权限控制是否显示

这些权限一般是在固定页面进行配置，保存后记录到数据库中。

------

按钮权限暂且不提，页面访问权限在实现中又可以分为两种方式：

- 显示所有菜单，当用户访问不在自己权限内的菜单时，提示权限不足
- 只显示当前用户能访问的权限内菜单，如果用户通过URL进行强制访问，则会直接进入404

既然展现出来后不能点，那算几个意思，逗我玩儿呢？所谓眼不见为净，综合考虑后，肯定是方案二比较符合良好的用户体验。

好，我们现在梳理一下大致的页面访问权限的流程：

![权限的流程](vue中如何实现后台管理系统的权限控制.assets/16cfa3fd181e548d)

在对流程梳理完成后我们开始进行详细的编写。

#### 1、创建路由表

创建路由表实际上没有什么难度，照着[vue-router官方文档](https://link.juejin.im/?target=https%3A%2F%2Frouter.vuejs.org%2Fzh-cn%2Fessentials%2Fgetting-started.html)给的示例直接写就行了。但是因为有部分页面是不需要访问权限的，

所以需要将登录、404、维护等页面写到默认的路由中，而将其它的需要权限的页面写到一个变量或者一个文件中，这样可

以有效的减轻后续的维护压力。

下面将index.js的代码贴上，异步路由将适量减少，以免占过多篇幅。

```
// router/index.js
import Vue from 'vue'
import Router from 'vue-router'
import App from '@/App'
import store from '../store/index'

Vue.use(Router);

//手动跳转的页面白名单
const whiteList = [
  '/'
];
//默认不需要权限的页面
const constantRouterMap = [
  {
    path: '/',
    name: '登录',
    component: (resolve) => require(['@/components/login'], resolve)
  },
  {
    path: '/index',
    name: 'nav.Home',
    component: (resolve) => require(['@/components/index'], resolve)
  },
  {
    path: '/templateMake',
    name: '模板制作',
    component: (resolve) => require(['@/components/Template/templateMake'], resolve)
  },
  {
    path: '/programMack',
    name: '节目制作',
    component: (resolve) => require(['@/components/Template/programMack'], resolve)
  },
  {
    path: '/release',
    name: '节目发布',
    component: (resolve) => require(['@/components/Program/release'], resolve)
  }
]

//注册路由
export const router = new Router({
  routes: constantRouterMap
});

//异步路由（需要权限的页面）
export const asyncRouterMap = [

  {
    path: '/resource',
    name: 'nav.Resource',
    meta: {
      permission: []
    },
    component: (resolve) => require(['@/components/Resource/resource'], resolve)
  },
  {
    path: '/template',
    name: 'nav.Template',
    meta: {
      permission: []
    },
    component: (resolve) => require(['@/components/Template/template'], resolve)
  },
  {
    path: '/generalSet',
    name: 'nav.System',
    meta: {
      permission: []
    },
    component: (resolve) => require(['@/components/SystemSet/generalSet'], resolve)
  },
  {
    path: '',
    name: 'nav.Log',
    component: App,
    children: [
      {
        path: '/userLog',
        name: 'nav.UserLog',
        meta: {
          permission: []
        },
        component: (resolve) => require(['@/components/Log/userLog'], resolve),
      },
      {
        path: '/operatingLog',
        name: 'nav.SystemLog',
        meta: {
          permission: []
        },
        component: (resolve) => require(['@/components/Log/operatingLog'], resolve),
      },
    ]
  }
  ]
];
```

注意事项：这里有一个需要非常注意的地方就是 404 页面一定要最后加载，如果放在constantRouterMap一同声明了404，后面的所以页面都会被拦截到404，详细的问题见[addRoutes when you've got a wildcard route for 404s does not work](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-router%2Fissues%2F1176)

#### 2、页面访问权限

在开始时我们梳理了一个大致的页面访问权限流程。下面我们先实现最核心的部分：

![访问权限流程](vue中如何实现后台管理系统的权限控制.assets/16cfa3fd18417224)



我们首先获取用户权限列表，在这里我们将接触到vuex状态管理，[官方文档](https://link.juejin.im/?target=https%3A%2F%2Fvuex.vuejs.org%2Fzh-cn%2Fstate.html)有详细介绍，这里就不过多描述了，下面请看代码：

```
// store/index.js
import Axios from 'axios'
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex);
const axios = Axios.create();

const state = {
  mode: 'login',
  list: []
};

const getters = {};

const mutations = {
  setMode: (state, data) => {
    state.mode = data
  },
  setList: (state, data) => {
    state.list = data
  }
};

const actions = {
  // 获取权限列表
  getPermission({commit}) {
    return new Promise((resolve, reject) => {
      axios({
        url: '/privilege/queryPrivilege?id=' + sessionStorage.getItem('privId'),
        methods: 'get',
        headers: {
          token: sessionStorage.getItem('token'),
          name: sessionStorage.getItem('name')
        }
      }).then((res) => {
        // 存储权限列表
        commit('setList', res.data.cust.privileges[0].children);
        resolve(res.data.cust.privileges[0].children)
      }).catch(() => {
        reject()
      })
    })
  }
};

export default new Vuex.Store({
  state,
  mutations,
  actions,
  getters
})
```

好了，我们现在请求后台拿到了权限数据，并将数据存放到了vuex中，下面我们需要利用返回数据匹配之前写的异步路由表，将匹配结果和静态路由表结合，开成最终的实际路由表。

其中最关键的是利用vue-router2.2.0版本新添加的一个addRoutes方法，我们看看官方文档如何解释此方法的：

> router.addRoutes(routes) 2.2.0+  动态添加更多的路由规则。参数必须是一个符合 routes 选项要求的数组。

那我们现在就可以开始使用addRoutes进行路由匹配了。下面看代码：

```
// router/index.js
/**
 * 根据权限匹配路由
 * @param {array} permission 权限列表（菜单列表）
 * @param {array} asyncRouter 异步路由对象
 */
function routerMatch(permission, asyncRouter) {
  return new Promise((resolve) => {
    const routers = [];
    // 创建路由
    function createRouter(permission) {
         // 根据路径匹配到的router对象添加到routers中即可
      permission.forEach((item) => {
        if (item.children && item.children.length) {
          createRouter(item.children)
        }
        let path = item.path;
        // 循环异步路由，将符合权限列表的路由加入到routers中
        asyncRouter.find((s) => {
          if (s.path === '') {
            s.children.find((y) => {
              if (y.path === path) {
                y.meta.permission = item.permission;
                routers.push(s);
              }
            })
          }
          if (s.path === path) {
            s.meta.permission = item.permission;
            routers.push(s);
          }
        })
      })
    }

    createRouter(permission)
    resolve([routers])
  })
}
```

然后我们编写导航钩子

```
// router/index.js
router.beforeEach((to, form, next) => {
  if (sessionStorage.getItem('token')) {
    if (to.path === '/') {
      router.replace('/index')
    } else {
      console.log(store.state.list.length);
      if (store.state.list.length === 0) {
          //如果没有权限列表，将重新向后台请求一次
        store.dispatch('getPermission').then(res => {
            //调用权限匹配的方法
          routerMatch(res, asyncRouterMap).then(res => {
              //将匹配出来的权限列表进行addRoutes
            router.addRoutes(res[0]);
            next({ ...to, replace: true })
          })
        }).catch(() => {
          router.replace('/')
        })
      } else {
        if (to.matched.length) {
          next()
        } else {
          router.replace('/')
        }
      }
    }
  } else {
    if (whiteList.indexOf(to.path) >= 0) {
      next()
    } else {
      router.replace('/')
    }
  }
});
```

到这里我们已经完成了对页面访问的权限控制，接下来我们来讲解一下操作按扭的权限部分。

## 四、数据操作权限

是否还记得前面的路由配置中我们多出来的一个代码，下面我们拿出来看看：

```
//异步路由（需要权限的页面）
export const asyncRouterMap = [

  {
    path: '/resource',
    name: 'nav.Resource',
    meta: {
      permission: []
    },
    component: (resolve) => require(['@/components/Resource/resource'], resolve)
  },
  {
    path: '/template',
    name: 'nav.Template',
    meta: {
      permission: []
    },
    component: (resolve) => require(['@/components/Template/template'], resolve)
  },
  {
    path: '/generalSet',
    name: 'nav.System',
    meta: {
      permission: []
    },
    component: (resolve) => require(['@/components/SystemSet/generalSet'], resolve)
  },
  {
    path: '',
    name: 'nav.Log',
    component: App,
    children: [
      {
        path: '/userLog',
        name: 'nav.UserLog',
        meta: {
          permission: []
        },
        component: (resolve) => require(['@/components/Log/userLog'], resolve),
      },
      {
        path: '/operatingLog',
        name: 'nav.SystemLog',
        meta: {
          permission: []
        },
        component: (resolve) => require(['@/components/Log/operatingLog'], resolve),
      },
    ]
  }
  ]
];
```

为每个路由页面增加meta字段。在routerMatch函数中将匹配到的详细权限字段赋值到这里。这样在每个页面的route对象中就会得到这个字段。

```
asyncRouter.find((s) => {
          if (s.path === '') {
            s.children.find((y) => {
              if (y.path === path) {
                  //赋值
                y.meta.permission = item.permission;
                routers.push(s);
              }
            })
          }
          if (s.path === path) {
            s.meta.permission = item.permission;
            routers.push(s);
          }
        })
```

接下来我们编写一个vue自定义指令对页面中需要进行鉴权的元素进行判断，比如类似这样的：

```
<a @click="upload" v-allow="'3'"></a> /* 3代表一个上传权限的ID，权限中有3则显示按钮 */
```

我们直接注册一个全局指令，利用vnode来访问vue的方法。代码如下：

```
//main.js
//按扭权限指令
Vue.directive('allow', {
  inserted: (el, binding, vnode) => {
    let permissionList = vnode.context.$route.meta.permission;
    if (!permissionList.includes(binding.value)) {
      el.parentNode.removeChild(el)
    }
  }
})
```

至此为止，权限控制流程就已经完全结束了，在最后我们再看一下完整的权限控制流程图吧.

## 五、路由控制完整流程图

![流程图](vue中如何实现后台管理系统的权限控制.assets/16cfa3fd18b05c8a)



六、参考文献

> 1. [Vue + ElementUI 手撸后台管理网站之权限控制](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fharsima%2Farticle%2Fdetails%2F77949448)
> 2. [手摸手，带你用vue撸后台之权限控制](https://juejin.im/post/591aa14f570c35006961acac)