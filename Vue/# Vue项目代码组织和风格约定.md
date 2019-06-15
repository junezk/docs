# Vue项目代码组织和风格约定

[原文地址](https://link.juejin.im/?target=https%3A%2F%2Fbetgar.github.io%2F2019%2F06%2F12%2Ffrontend-little-tangle-vue-project-code-organization-conventional-style-guideline%2F)

> `风格约定`但不限于`代码风格`，还有一些其他的`默认约定`。

## 代码组织和分层

> 代码组织是一个仁者见仁，智者见智的话题，没有**银弹**。不过不管怎么变化，指导思想还是不变的**高内聚，低耦合**。
>
> 强烈推荐两篇文章，能够拓宽你的视野，带你走向新高度。
>
> [用 Feature First 的方式管理前端项目复杂度](https://juejin.im/post/5cb00b1c6fb9a068a256b37c)

***代码组织的优雅，模块化才能够做好。***

### 分层

> 按照**职能的不同**进行不同维度进行层级划分，层级划分之后，进行进一步的**模块划分**（原则上，每一个文件夹都是一个模块）

## 文件夹和文件命名

> 选择适合自己的风格。

1. 文件夹和文件都使用**kebab-case**

   > **kebab-case**重度使用者可以选择这种。

2. 文件夹使用**kebab-case**， 文件使用**Pascal Case**

   > 建议使用这种。

   ```
   文件夹：event-bus
   文件：EventBus.ts
   ```
   
3. 例外

   - index文件不受上述约束
   - 工具自动生成的文件（自己考虑是否受约束）

## 模块化原则

> 模块化代码首先要做到代码的分层、隔离、抽象。
>
> 不同模块完成不同的职能，不同职能之间相互协作。

1. 每个模块保持一个入口和出口

   > 对于外部模块来说，尽量保证一个入口
   >
   > 对于内部子模块来说，尽量保证一个出口
   >
   > 如果按照**文件夹**作为模块界限，每个文件夹下都有一个出口（可以默认为index文件）

2. 模块的入口名称默认index或者文件夹名字的文件

   ```javascript
   // 例如： group文件夹
   group/
   |---index.ts   // A. 默认作为入口
   |---group.ts   // B. 也可以默认作为入口
   
   // 二者任选其一就好，A方案应该是大家默认的方案；B方案，检索代码的时候更方便
   ```
   
3. 模块内部分层

   > 模块内部还可以有`base`, `common`, `components`, `helper`,`utils`,`filter`,`config`等层级（词穷了.....）

## `import`和`export`原则

1. `import`导入，指定到文件

   > 指定到文件能够提高编译打包的速度

   ```
   // 指定到index文件
   import { Logger } from './common/index';
   ```

## `common`模块

> 独立文章说明。

## `router`模块

> `router`模块的风格约定。

### 模块结构

```
router
├── helper
│   ├── ImportRoute.ts
│   └── RouteGenerator.ts
├── modules
│   ├── AboutRoutes.ts
│   └── HomeRoutes.ts
├── router.ts
└── Routes.ts
```

- helper: 帮助工具方法

- modules: 不同的业务模块

- router: vue-router初始化的地方，也是模块入口

- Routes: `RouteConfig`的出口，其它模块都从这里获取`route`配置，从而达到解耦的目的，尤其是不同的`views`里面的路由跳转，使用`Routes`配置达到解耦的目的。

  例子：

  ```
  import { HomeRoute } from 'Routes';
  
  // 跳转,这样没有硬编码任何的route信息，全部都是从Routes配置来，达到解耦的目的。
  this.$router.push({
      name: HomeRoute.name
  })
  ```

### `modules`子模块

> 文件名：`views`下文件夹名（模块名） + Routes结尾
>
> **modules下的文件，最好和views下文件夹一一对应，方便维护**（对模块切分有较高的理解）

```
例如：
views
├── group
│   ├── xxx1.vue
│   └── xxx2.vue
├── report
│   ├── xxx3.vue
│   └── xxx4.vue
├── Home.vue
```

> `modules`对应的就是

```
例如：
router
├── modules
│   ├── GroupRoutes.ts
|   ├── ReportRoutes.ts
│   └── HomeRoutes.ts
```

> 导出模块配置：

```javascript
// HomeRoutes.ts
export const HomeRoute = {
  path: '/',
  name: 'HomeRoute',
  component: 'Home',
};

// 必须导出一个数组，因为是一个模块的配置信息，可能有多个配置，还可以进行配置层级关系
export default [HomeRoute];
```

**注意：** 这只是个思路，具体的操作还要灵活运用。

### `RouteConfig`风格约定

1. `RouteConfig`变量名

   > `vue`文件名+Route结尾

   ```javascript
   // 文件Home.vue
   // 变量名HomeRoute
   export const HomeRoute = {
     path: '/',
     name: 'HomeRoute',
     component: 'Home',
   };
   ```
   
2. `name`属性

   > 和`RouteConfig`变量名保持一致

3. `component`属性

   > 如果**异步加载**，`component`需要使用相对于`views`的`path`格式，因为在`ImportRoute`中统一处理。

   ```javascript
   // ImportRoute.ts统一处理
   export function importRoute(file: string) {
     // @see https://github.com/webpack/webpack/issues/1949
     return () => import(/* webpackChunkName: "chunk-[request][index]" */ '@/views/' + file + '.vue');
   }
   ```
   
4. `path`属性

   > 没有最佳实践，最好使用`restful`风格约束
   >
   > 如果使用`route.query`等之类的参数传递，**面包屑导航**很难处理。

5. `meta`属性

   > 没有最佳实践，多数情况下有这么几个属性

   ```javascript
   export const HomeRoute = {
     path: '/',
     name: 'HomeRoute',
     component: 'Home', // component: () => import('@/views/Home.vue')
     meta: {
         // title的值可以为`i18n`的语言文件key，方便做国际化
         title: '首页', // 作为menu.title和breadcrumb.title备选
         icon: '', // icon的class,作为menu.icon和breadcrumb.icon备选
         menu: {
             title: '首页',
             visible: true,
             icon: '',  // icon的class
         },
         breadcrumb: {
             title: '路径名',
             visible: false, // 有的时候不需要在面包屑上渲染
             icon: '', // icon的class
         },
         auth: {
             roles: [1, 2, 3]
         }
     }
   };
   ```
   
6. `props`属性

   > [路由组件传参](https://link.juejin.im/?target=https%3A%2F%2Frouter.vuejs.org%2Fzh%2Fguide%2Fessentials%2Fpassing-props.html)，更多高级用法，请查看[例子](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue-router%2Fblob%2Fdev%2Fexamples%2Froute-props%2Fapp.js)
   >
   > 使用`props`方式把`components`和`$route`解耦，这样`components`既可以单独使用，也可以当作**子组件使用**，而且方便测试。
   >
   > 特殊场景可以不使用`props`，例如本来就不是通用的组件，是需要组合在一起使用的父子组件，是可以和`route`耦合的。

   ***如果 props 被设置为 true，route.params 将会被设置为组件属性。***

   ```javascript
   // 函数式(动态)
   const router = new VueRouter({
     routes: [{ 
           path: '/search', 
        	component: SearchUser, 
           // route是SearchUser内部的this.$route
        	props: (route) => ({ query: route.query.q }) 
       }]
   })
   ```
   
```javascript
   // 静态
   const router = new VueRouter({
     routes: [{ 
           path: '/promotion/from-newsletter',
           component: Promotion, 
           props: { newsletterPopup: false } 
       }]
   })
   ```

## `store`模块

> `vue`应用的状态模块

### 模块结构

> [参考官方文件结构](https://link.juejin.im/?target=https%3A%2F%2Fvuex.vuejs.org%2Fzh%2Fguide%2Fstructure.html)
>
> [官方购物车例子](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvuex%2Ftree%2Fdev%2Fexamples%2Fshopping-cart)

```
store
├── StoreTypes.ts       # actions mutations getters类型
├── Actions.ts          # 根级别的 action
├── Mutations.ts        # 根级别的 Mutations
├── Getters.ts          # 根级别的 getters
├── modules             # 模块
│   ├── xxxStore.ts     # 子模块
│   ├── SystemStore.ts  # SystemStore子模块
├── index.ts            # 入口
```

- `index`： 作为入口

- `StoreTypes`作为类型的常量文件

  > [参考使用常量替代-mutation-事件类型](https://link.juejin.im/?target=https%3A%2F%2Fvuex.vuejs.org%2Fzh%2Fguide%2Fmutations.html%23%E4%BD%BF%E7%94%A8%E5%B8%B8%E9%87%8F%E6%9B%BF%E4%BB%A3-mutation-%E4%BA%8B%E4%BB%B6%E7%B1%BB%E5%9E%8B)
  >
  > 所以mutation和action的类型全部使用常量，方便其它模块和store模块的解耦。

### modules子模块

> 文件名：模块名 + Store结尾
>
> **modules下的业务store，最好和views下文件夹对应，方便维护**

```
# 例子： 
LocaleStore.ts    # i18n模块
LoginStore.ts     # 登录模块
UserStore.ts      # 用户模块
```

### `Store module`约定

> [参考Store Module](https://link.juejin.im/?target=https%3A%2F%2Fvuex.vuejs.org%2Fzh%2Fguide%2Fmodules.html)
>
> 主要约定`module`内部代码结构大致如下：
>
> `SystemStore`为例：

- 需要使用前缀的地方使用模块名作为前缀

  > 例子中**模块名**为`System`

> **约定声明大致顺序如下**：除state部分外其他都是可选

- state声明（例如：`systemState`）
- getter types声明（可选，灵活运用）
- getters声明（可选, 灵活运用）
- mutation types 声明（可选）
- mutations声明（可选）
- action types声明（可选）
- actions声明（可选）
- store options导出

```javascript
// 例子: SystemStore.ts
// 声明state
const systemState: SystemState = {
  initialized: false,
};

// 声明GetterTypes 
export const SystemGetterTypes = {
  IS_SYSTEM_INITIALIZED: 'IS_SYSTEM_INITIALIZED',
};

// 声明getters
const getters = {
    [SystemGetterTypes.IS_SYSTEM_INITIALIZED](state: SystemState, getters: any, rootState: any){
    	return state.initialized;
    }
};

// 声明MutationTypes
export const SystemMutationTypes = {
  SYSTEM_SET_INITIALIZED: 'SYSTEM_SET_INITIALIZED',
};
// 声明mutations
const mutations: MutationTree<SystemState> = {
  [SystemMutationTypes.SYSTEM_SET_INITIALIZED]: (
    state: SystemState,
    payload: boolean
  ) => {
    state.initialized = payload;
    logger.log('system-store.initialized: ' + payload);
  },
};

// 声明ActionTypes
export const SystemActionTypes = {
  SYSTEM_UPDATE_INITIALIZED: 'SYSTEM_UPDATE_INITIALIZED',
  SYSTEM_RESET: 'SYSTEM_RESET',
  SYSTEM_INIT: 'SYSTEM_INIT',
};

// 声明actions
const actions = {
  [SystemActionTypes.SYSTEM_UPDATE_INITIALIZED]: (
    { commit }: ActionContext<SystemState, any>,
    initialized: boolean
  ) => {
    commit(SystemMutationTypes.SYSTEM_SET_INITIALIZED, initialized);
  },
  [SystemActionTypes.SYSTEM_RESET]: ({
    commit,
    dispatch,
  }: ActionContext<SystemState, any>) => {
    // 清空所有使用store储存的数据.
  },
  [SystemActionTypes.SYSTEM_INIT]: (
    { commit, dispatch }: ActionContext<SystemState, any>,
    payload: {
      user: UserModel;
      userCookie: UserCookie;
    }
  ) => {
      // 初始化数据
  },
};

// 导出storeOptions
const storeOptions = {
  state: systemState,
  getters,
  mutations,
  actions,
};

export default storeOptions;
```

**注意**：其中的action types, mutation types和 getter types 不强制要求，在`es6`环境中使用官方的`mapState`，`mapGetters`, `mapActions`, `mapMutations`工具函数更方便。

## `api`模块

> `api`模块是接口服务层，主要做一些对参数的转换处理，同时解耦其它业务层。

### 模块结构

> 模块的大致参考结构

```javascript
api
├── api.ts    # 入口
└── modules   # 子模块
    ├── DictionaryService.ts   # 具体的业务模块
    ├── GroupService.ts
    ├── HistoryService.ts
```

### `service module`约定

> 文件名：模块名 + Service结尾
>
> **modules下的业务service，和views下文件夹对应，方便维护**
>
> 如果单个service文件对应的业务模块接口太多，可以使用文件夹来进一步分割。

```javascript
//例子：LoginService
// 声明url，导出url方便mock模块使用
export const GET_SIGN_IN = '/api/login';
// 声明service函数
export function signIn(data: {account: string; pass: string}) {
  return ajax({
      url: GET_SIGN_IN,
      data, 
      method: 'post'
  });
}
```

**URL前缀约定：**

> 查询：使用`GET`作为前缀（特殊情况例外）
>
> 更新：使用`UPDATE`
>
> 新增：使用`ADD`
>
> 删除：使用`DELETE`
>
> 其它：EXPORT，IMPORT，UPLOAD，DOWNLOAD等

**service 函数**名前缀约定：

- 对于单个**实体**可以考虑使用get, add, update, delete作为前缀

## `i18n`模块

> 主要是建议下代码的分割的约定。

### 模块结构

```
i18n
├── i18n.ts
├── index.d.ts
└── locales
    ├── en_US.js
    ├── modules
    │   └── actions
    │       ├── en_US.js
    │       └── zh_CN.js
    └── zh_CN.js
```

### 语言文件组织

> 语言模块的组织，主要按照模块（文件夹层次）来组织，能够最小的减少冲突可能性。

**语言文件约定：**

- 方言文件放置在`locals`文件夹下
- 文件以**方言编码**命名

层级组织的例子：

```
views
├── account
│   ├── Account.vue
│   ├── locales
│   │   ├── en_US.js
│   │   └── zh_CN.js
│   └── XXX.vue
├── feedback
│   ├── locales
│   │   ├── en_US.js
│   │   └── zh_CN.js
│   └── Suggestion.vue
├── locales
│   │   ├── en_US.js
│   │   └── zh_CN.js
```

```
// views/feedback/locales/zh_CN.js
export const feedbackModule = {
  // 语言模块属性：模块名+module结尾
  // 可选的方式就是：严格按照文件夹层次来构造属性层级(缺点取属性值时太长，后期可以使用__dirname自动生成)
  label: '您的意见:',
    textarea: {
        placeholder: '请写下您的意见与建议, 500字符以内'
  }  
};
```
```
// views/locales/zh_CN.js
import feedbackModule from '../feedback/locales/zh_CN.js';

export default {
  // 如果模块众多建议加上views  
  // views: {
  //    ...feedbackModule,
  // }
  
  // 如果模块少，直接
  feedbackModule,
};

```
```
// i18n/locales/zh_CN.js
import viewsLocales from '../../views/locales/zh_CN';

export default {
    ...viewsLocales
}
```

## `mock`模块

> mock模块主要做些数据模拟的工作

### 模块结构

> 子模块的大致参考结构

```
mock
├── mock.js   # 入口
└── modules   # 子模块
    ├── LoginMock.js   # 具体的业务模块
```

### `mock module`约定

> 文件名：模块名 + mock结尾
>
> **modules下的mock文件，和api模块下文件对应，方便维护**

**方法约定：**

- `mock`模块方法和`service`请求方法对应

  > 例如：`LoginService`的`signIn` mock模块对应也叫`signIn`

- `mock module`需要提供`setup`方法

  > setup方法供外部统一调用，作为mock module的开关

> 例子：

```
// LoginMock.js
import Mock from 'mockjs';
import { genSuccessResult, genFailResult } from './mock-utils';
import { GET_SIGN_IN } from '../api/api';
import Cookies  from 'js-cookie';

const signIn = (data) => {
    Cookies.set('SESSIONID', 'mock SESSIONID');
    return genSuccessResult({
      msg: '登陆成功',
    });
}
```
```
export default {
  setup() {
      Mock.mock(GET_SIGN_IN, 'post', signIn);
  }
}
```
```
// mock.js
import loginMocker from './modules/login';

export const start = function() {
  loginMocker.setup();
};
```

```
// main.js
import mocker from './mock/mock'
if (process.env.NODE_ENV === 'development') {
  mocker.start();
} 
```

## 其它模块

> 其它模块如`filters`，`directives`,`test`目前没有总结，以后补充。