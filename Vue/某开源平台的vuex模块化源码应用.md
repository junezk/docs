# 某开源平台的vuex模块化源码应用

> 先告诉阅读本篇文章的同学本人水平有限，菜鸟一只:smile:。
>
> - 继上次分享了在vue官方文档的一些小细节，很多大佬说基本操作，不够深，多看看源码
> - 其实本人也只是个小菜鸟，来分享的也是自己学习过程，源码解说不了，不过最近刚好在看
> - 然后今天分享的内容是某开源框架里的vuex应用，会上代码，而不是教什么是mutation什么是getters，怎么用action等等。

> 提示：**本篇文章不适合没用过vuex的新人或已经在大型项目中熟练使用vuex的大佬阅读**

首先我这篇分享基于某开快速开发平台的源码，大概花了2天专门整理了一下它前端项目中的vuex应用模式，但是我要先给你们讲个我觉得蛮搞笑的事情，你们会很感兴趣。

首先，这个框架不是我选的……，我只是被选用来填坑，当然本人不介意，该框架开源4个月，但是官方文档从开始到现在几乎是无法吐槽的“简洁”，所以开发过程基本上都是自己查源码。

在上个月的某一天该框架发布了一个小版本，但是在版本日志中出现了明显的链接放错了，找不到修复该问题后的正确使用链接，介于我对于该框架文档简陋的体验已久且我很关注这个问题，我在他们论坛里发了贴，吐槽版本日志链接都放错了。结果第二天：

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/8/14/16c8f92682f8e646?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

nice！我去看了看之前错的版本日志，更nice！:smile:把我帖子删了的同时，默默还把自己的错误改了回来。（帖子除了这个标题，我还发了一句“更新的版本日志的第几个错误的链接放错了，进去打开找不到演示说明”大概这意思，没有任何恶语相向）

好了，故事结束了，还是开心的说代码吧！

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/8/19/16ca793cb58c7c61?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

首先各位可以看一看封面图（封面图展示不全已替换在文章中插入），是我总结的大概该项目vuex应用结构，我们一点一点讲解。

## index模块

main引入store就不用说了，不过需要注意这个地方引入的是路径的方式

```js
//main.js

import store from './store/'
```



![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/8/14/16c8f92682ab61e7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

因为引入方式是使用了路径，所以会默认加载路径中的index.js(这个地方可能很多人就不理解，当引入方式为路径时，将其视为一个CommonJs包处理，查找规则依次为目录下为index.js,index.json,index.node)，index的内容是：



```js
import app from './modules/app'
import user from './modules/user'
import permission from './modules/permission'
import getters from './getters'

Vue.use(Vuex)

export default new Vuex.Store({
  modules: {
    app,
    user,
    permission
  },
  getters
})
```

这部分内容就很简单，将store分成了几个模块，在index中统一export出去供全局调用。

## module（store模块）

> 建议对照我的封面图看这一部分

这一部分，其实是就是我们常看的入门教学中的state.js文件，将其根据业务拆分成了app.js（全局状态配置），permission.js（存放路由，包含对路由做操作的actions），user.js（存放用户信息，包含登录接口和登录处理个人信息的actions)，结合代码分别来介绍一下里面的内容应用：

### app.js

```js
import Vue from 'vue'
import {
  SIDEBAR_TYPE,
  DEFAULT_THEME,
  DEFAULT_LAYOUT_MODE
} from "@/store/mutation-types"

const app = {
  state: {
    sidebar: {
      opened: true,
      withoutAnimation: false
    },
    device: 'desktop',
    theme: '',
    layout: '',
    contentWidth: '',
    fixedHeader: false,
    fixSiderbar: false,
    autoHideHeader: false,
    color: null,
    weak: false,
    multipage: false //默认多页签模式
  },
  mutations: {
    SET_SIDEBAR_TYPE: (state, type) => {
      //将系统sidebar为展开或是折叠状态
      state.sidebar.opened = type
      //修改state状态之后，使用localstorage存储state相应的变量，
      //解决vuex状态刷新丢失问题
      Vue.ls.set(SIDEBAR_TYPE, type)//ls为vue-ls的使用方式，相当于setLocalstorage
    },
    CLOSE_SIDEBAR: (state, withoutAnimation) => {
      Vue.ls.set(SIDEBAR_TYPE, true)
      state.sidebar.opened = false
      state.sidebar.withoutAnimation = withoutAnimation
    },
    TOGGLE_DEVICE: (state, device) => {
      state.device = device
    },
    TOGGLE_THEME: (state, theme) => {
      // setStore('_DEFAULT_THEME', theme)
      Vue.ls.set(DEFAULT_THEME, theme)
      state.theme = theme
    }
    
    ·······
    
  },
  actions: {
  	//action中存放的都是对mutations的异步操作，很简单
    setSidebar: ({ commit }, type) => {
      //异步触发mutations的SET_SIDEBAR_TYPE
      commit('SET_SIDEBAR_TYPE', type)
    },
    CloseSidebar({ commit }, { withoutAnimation }) {
      commit('CLOSE_SIDEBAR', withoutAnimation)
    },
    ToggleDevice({ commit }, device) {
      commit('TOGGLE_DEVICE', device)
    },
    ToggleTheme({ commit }, theme) {
      commit('TOGGLE_THEME', theme)
    }
    
    ·······
    
  }
}

export default app
```

该文件中就是常见的state，mutations，actions三部分，而其中开始处引用了mutation-types文件，里面存放的只是一些统一的字符串变量名，用于在localstorage中存贮时的键名，几乎都是大写，可以先贴出来，后面不单独讲解：

```js
export const ACCESS_TOKEN = 'Access-Token'
export const SIDEBAR_TYPE = 'SIDEBAR_TYPE'
export const DEFAULT_THEME = 'DEFAULT_THEME'
export const DEFAULT_LAYOUT_MODE = 'DEFAULT_LAYOUT_MODE'
export const DEFAULT_COLOR = 'DEFAULT_COLOR'
export const DEFAULT_COLOR_WEAK = 'DEFAULT_COLOR_WEAK'
export const DEFAULT_FIXED_HEADER = 'DEFAULT_FIXED_HEADER'
export const DEFAULT_FIXED_SIDEMENU= 'DEFAULT_FIXED_SIDEMENU'
export const DEFAULT_FIXED_HEADER_HIDDEN = 'DEFAULT_FIXED_HEADER_HIDDEN'
export const DEFAULT_CONTENT_WIDTH_TYPE = 'DEFAULT_CONTENT_WIDTH_TYPE'
export const DEFAULT_MULTI_PAGE = 'DEFAULT_MULTI_PAGE'
export const USER_NAME = 'Login_Username'
export const USER_INFO = 'Login_Userinfo'
export const USER_AUTH = 'LOGIN_USER_BUTTON_AUTH'
export const SYS_BUTTON_AUTH = 'SYS_BUTTON_AUTH'
```

每个state几乎都是一样的，后面只会在每个对象的第一个变量或者方法中做简单注释，应该有vuex应用基础的就可以看懂，还是很简单的。 可以看一下对SET_SIDEBAR_TYPE的调用：

```
mounted () {
	//系统初始化，读取localstorage的SIDEBAR_TYPE值调用SET_SIDEBAR_TYPE
	store.commit('SET_SIDEBAR_TYPE', Vue.ls.get(SIDEBAR_TYPE, true))
}
复制代码
```

思路比较简单，将所有要存储的state在localstorage中存储时，使用系统统一对应的字符串名字创建键值对存储，对相应的state进行操作的mutations方法中也进行localstorage更新。

### permission.js

permission中存放的是路由信息，以及根据角色对路由进行操作的方法，该框架使用的是前端只存放基础路由。

```js
//router.config里面存放基础路由
import { asyncRouterMap, constantRouterMap } from "@/config/router.config"

/**
 * 过滤账户是否拥有某一个权限，并将菜单从加载列表移除
 */
function hasPermission(permission, route) {
  if (route.meta && route.meta.permission) {
    let flag = -1
    for (let i = 0, len = permission.length; i < len; i++) {
      flag = route.meta.permission.indexOf(permission[i])
      if (flag >= 0) {
        return true
      }
    }
    return false
  }
  return true
}

/**
 * 单账户多角色时，使用该方法可过滤角色不存在的菜单
 */
// eslint-disable-next-line
function hasRole(roles, route) {
  if (route.meta && route.meta.roles) {
    return route.meta.roles.indexOf(roles.id)
  } else {
    return true
  }
}

function filterAsyncRouter(routerMap, roles) {
  const accessedRouters = routerMap.filter(route => {
    if (hasPermission(roles.permissionList, route)) {
      if (route.children && route.children.length) {
        route.children = filterAsyncRouter(route.children, roles)
      }
      return true
    }
    return false
  })
  return accessedRouters
}


const permission = {
  state: {
    routers: constantRouterMap,//基础路由
    addRouters: []//根据用户角色判断后要添加的剩余路由
  },
  mutations: {
    SET_ROUTERS: (state, data) => {
    //添加路由，并合并完整路由到routers变量，实际应用加载的路由为routers 
      state.addRouters = data
      state.routers = constantRouterMap.concat(data)
    }
  },
  actions: {
    GenerateRoutes({ commit }, data) {
      return new Promise(resolve => {
        const { roles } = data
        let accessedRouters = filterAsyncRouter(asyncRouterMap, roles)
        resolve()
      })
    },
    // 动态添加主界面路由，需要缓存
    UpdateAppRouter({ commit }, routes) {
      return new Promise(resolve => {
        let routelist = routes.constRoutes;
        resolve()
      })
    }
  }
}

export default permission
```

然后看一下对路由state的操作应用，对于路由的操作没有直接调用mutations中的SET_ROUTERS，而是采用了异步的UpdateAppRouter去更新：

```
let constRoutes = [];
  constRoutes = generateIndexRouter(menuData);
  // 添加主界面路由
  store.dispatch('UpdateAppRouter',  { constRoutes }).then(() => {
    // 根据roles权限生成可访问的路由表
    // 动态添加可访问路由表
    router.addRoutes(store.getters.addRouters)
    const redirect = decodeURIComponent(from.query.redirect || to.path)
    if (to.path === redirect) {
      // hack方法 确保addRoutes已完成 
      next({ ...to, replace: true })
    } else {
      // 跳转到目的路由
      next({ path: redirect })
    }
  })
复制代码
```

希望你可以仔细阅读一下这段代码，其中在router（此处router指的是vue的router而不是router变量）的addRoutes时，所添加的路由是通过store的getters方法，然后方便理解也放一下getters的内容，后面不会单独讲解：

```js
import Vue from 'vue'
import { USER_INFO} from "@/store/mutation-types"
const getters = {
  device: state => state.app.device,
  theme: state => state.app.theme,
  color: state => state.app.color,
  token: state => state.user.token,
  avatar: state => {state.user.avatar = Vue.ls.get(USER_INFO).avatar; return state.user.avatar},
  username: state => state.user.username,
  nickname: state => {state.user.realname = Vue.ls.get(USER_INFO).realname; return state.user.realname},
  welcome: state => state.user.welcome,
  permissionList: state => state.user.permissionList,
  userInfo: state => {state.user.info = Vue.ls.get(USER_INFO); return state.user.info},
  addRouters: state => state.permission.addRouters//获取用户权限所拥有的route
}

export default getters
```

getters内容更简单，就是从各个module中的state中获取变量。

整个对路由的操作就很清晰了：

1. 整个路由操作过程就比较清晰了，通过调用routestate的异步方法UpdateAppRouter去操作SET_ROUTERS设置state中的addRouters和routers变量（即用户所拥有的权限路由），
2. 然后使用vueRouter的router.addRoutes（'要添加的路由'）方法去补全完整路由，要添加的路由则通过store的getters方法获取route中的的addRouters变量。

### user.js

user.js里面存放主要是用户的登录以及登陆后对token和用户信息在vuex中的处理，我觉得这部分可以给很多人的项目中借鉴，放上完整代码：

```js
import Vue from 'vue'
import { login, logout } from "@/api/login" //登录登出api调用方法
import { ACCESS_TOKEN, USER_NAME,USER_INFO,USER_AUTH,SYS_BUTTON_AUTH } from "@/store/mutation-types"
import { welcome } from "@/utils/util"

const user = {
  state: {
    token: '', //用户token
    username: '',
    realname: '',
    welcome: '',
    avatar: '',
    permissionList: [],
    info: {}
  },

  mutations: {
    SET_TOKEN: (state, token) => {
      //setToken在state中
      state.token = token
    },
    SET_NAME: (state, { username, realname, welcome }) => {
      state.username = username
      state.realname = realname
      state.welcome = welcome
    },
    SET_AVATAR: (state, avatar) => {
      state.avatar = avatar
    },
    SET_PERMISSIONLIST: (state, permissionList) => {
      state.permissionList = permissionList
    },
    SET_INFO: (state, info) => {
      state.info = info
    },
  },

  actions: {
    // 登录
    Login({ commit }, userInfo) {
      return new Promise((resolve, reject) => {
        //调用登录接口
        login(userInfo).then(response => {
          if(response.code =='200'){
            const result = response.result
            const userInfo = result.userInfo
            //在localstorage和state中存储token等信息，当然localstorage设置了有效期
            Vue.ls.set(ACCESS_TOKEN, result.token, 7 * 24 * 60 * 60 * 1000)
            Vue.ls.set(USER_NAME, userInfo.username, 7 * 24 * 60 * 60 * 1000)
            Vue.ls.set(USER_INFO, userInfo, 7 * 24 * 60 * 60 * 1000)
            commit('SET_TOKEN', result.token)
            commit('SET_INFO', userInfo)
            commit('SET_NAME', { username: userInfo.username,realname: userInfo.realname, welcome: welcome() })
            commit('SET_AVATAR', userInfo.avatar)
            resolve(response)
          }else{
            reject(response)
          }
        }).catch(error => {
          reject(error)
        })
      })
    },
    // 获取用户信息
    GetPermissionList({ commit }) {
      return new Promise((resolve, reject) => {
        let v_token = Vue.ls.get(ACCESS_TOKEN);
        let params = {token:v_token};
        queryPermissionsByUser(params).then(response => {
          const menuData = response.result.menu;
          const authData = response.result.auth;
          const allAuthData = response.result.allAuth;
          //Vue.ls.set(USER_AUTH,authData);
          sessionStorage.setItem(USER_AUTH,JSON.stringify(authData));
          sessionStorage.setItem(SYS_BUTTON_AUTH,JSON.stringify(allAuthData));
          if (menuData && menuData.length > 0) {
            commit('SET_PERMISSIONLIST', menuData)
          } else {
            reject('getPermissionList: permissions must be a non-null array !')
          }
          resolve(response)
        }).catch(error => {
          reject(error)
        })
      })
    },

    // 登出
    Logout({ commit, state }) {
      return new Promise((resolve) => {
        let logoutToken = state.token;
        //清除state和localstorage中的token等信息
        commit('SET_TOKEN', '')
        commit('SET_PERMISSIONLIST', [])
        Vue.ls.remove(ACCESS_TOKEN)
        //console.log('logoutToken: '+ logoutToken)
        logout(logoutToken).then(() => {
          resolve()
        }).catch(() => {
          resolve()
        })
      })
    },

  }
}

export default user
```

然后我们看一下对它的代码调用：

```js
 import {mapActions} from "vuex"
 
 ······
 
 methods: {
	...mapActions(["Login", "Logout"]),
	
	······
	
	handleSubmit() {
        let that = this
        let loginParams = {
          remember_me: true
        };
        // 使用账户密码登陆
        that.form.validateFields(['username', 'password', 'inputCode'], {force: true}, (err, values) => {
           if (!err) {
             loginParams.username = values.username
             loginParams.password = md5(values.password)
             loginParams.password = values.password
             //此处的Login方法即调用的user.js中的异步actions的Login()
             that.Login(loginParams).then((res) => {
               that.loginSuccess()
             }).catch((err) => {
               console.log('err',err)
               that.requestFailed(err);
             })
           }
         })
      },
 }
```

在用户登录这块，不太一样的处理是，它没有在我们的登录页面methods中写登录逻辑，而是将登录接口和登陆后的操作统一写在了actions方法中，登录页面表单验证通过之后，直接调用vuex的action去调用登录接口，在登录接口中做user的state的相关处理。

这样，整个框架里对vuex的各个部分应用和结构基本清晰了，如果还有串联不太起来的地方建议结合我的封面图和文件路径截图查看一下，如果对vuex有过一定应用，但是没有进行模块化的开发者，我觉得你看完会对之前自己的一些思考有一些灵光。

当然在最后还是要说一下本篇文章不太适用于：

- 没有使用过vuex，不知道state ，mutation ，getters，action 等等怎么用的开发者（建议可以结合官方文档去看看别人的解读，我就是这么入门的）
- 对在项目中已经总结出自己一套vuex策略的大佬，也不要笑话我啦:satisfied:，可以提些建议给我更多启发

另外本人写文章一般都是分享自己学习的探索路程，可能对更多刚入门前端或者正从入门到进阶前端开发者更有用些，**欢迎点赞鼓励和评论指正**:v::v:

如果反馈不错，下次分享一篇**自己使用Egg.js+Mysql+Vue自己博客的全路程**,因为自己第一次用node框架写完整的项目，MySql也是第一次用，所以到时候会附上很多第一次用Egg和MySql遇到的问题，服务器部署问题和新手直接上手的代码.