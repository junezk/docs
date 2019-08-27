# [vue项目接入mock&& axios 通用配置](https://segmentfault.com/a/1190000019964344)



### 前言

兵马未动,粮草先行; 同理,项目开发过程中经常会出现接口未出, 前端页面已搭建完毕的情况;此时为了提高前端的开发效率,解放生产力,我们 FE 可以按照预定的接口文档做一些接口模拟的工作,等等后端小伙伴开发完接口后我们只需要替换一个接口基地址即可...

### 初始准备

这里已 vue 项目为例, 所需模块:mockjs, express; 直接 npm 进行安装即可

```
1.Mock.js  // 用来快速构造随机数据  http://mockjs.com/examples.html 
2.express // 用来快速搭建本地服务 http://expressjs.jser.us/
```

### 项目中使用

一般会在 vue 项目中建一个 mock 文件夹, 里面放上对应处理文件:

#### 接口分发模块

主要是分发前端过来的请求 URL 以及启动本地服务

```js
let Mock = require('mockjs');
let express = require('express');
let app = express();

let bodyParser = require('body-parser'); // 解析post请求

// mock 数据相关 api
let homeAPI = require('./home');
let specialAPI = require('./special');
let appAPI = require('./app');

app.use(bodyParser.json());

// 设置跨域
app.all('*', function (req, res, next) {
    res.header('Access-Control-Allow-Origin', '*');
    res.header('Access-Control-Allow-Methods', 'PUT, GET, POST, DELETE, OPTIONS');
    // 此处根据前端请求携带的请求头进行配置
    res.header('Access-Control-Allow-Headers', 'X-Requested-With, Content-Type');
    next();
});

// 1.首页接口: post
app.post('/android/home', (req, res) => {
    // 模拟超时
    // setTimeout(() => {
    //     res.json(Mock.mock(homeAPI.getHome(req)));
    // }, 12000);
    res.json(Mock.mock(homeAPI.getHome(req)));
    console.log('请求首页数据成功...');
})

// 2.专题接口: post
app.post('/android/special/list', (req, res) => {
    res.json(Mock.mock(specialAPI.getSpecialList(req)))
    console.log('请求专题列表数据成功...');
})
app.post('/android/special/detail', (req, res) => {
    res.json(Mock.mock(specialAPI.getSpecialDetail(req)))
    console.log('请求专题详情数据成功...');
})

// 3.APP 应用接口: post
app.post('/android/special/app', (req, res) => {
    res.json(Mock.mock(appAPI.getAppList(req)))
    console.log('请求app列表数据成功...');
})

app.listen('3000', () => {
    console.log('mock服务器启动ing中... port: 3000')
})
```

#### 接口数据模块

以上面的 specialApi 为例, 这是用来处理 xxx/special/list接口的模块

```js
let Mock = require('mockjs');
let Random = Mock.Random;
// 专题模块
// 1.专题列表
let specialList = [];
let total = 100;
// 准备 100 条专题列表数据
for (let index = 0; index < total; index++) {
    specialList.push(
        Mock.mock({
            id: '@increment',
            title: '@ctitle',
            desc: '<p>'+Random.cparagraph()+'</p>',
            icon: 'photo/special/1380/special_1380.jpg',
            view_count: '@natural(60, 1000)',
            comment_count: '@natural(60, 100)',
            save_money: '@float(10, 50, 2, 2)',
            app_count: '@natural(10, 100)',
            detail_icon: 'https://images.tutuapp.com/photo/special/000/001/' + '@natural(100, 200)' + '/414x155.jpg',
        })
    );
}

module.exports = {
// 列表数据返回
    getSpecialList: config => {
        // 参数解析
        console.log(config.body);
        let { page = 1, size, lang, order_by } = config.body;
        let tempList = [];
        let pageList;
        // 排序类型
        tempList = order_by === 'view' ? specialList.reverse() : specialList;
        // 分页处理
        pageList = tempList.filter((item, index) => index < page * size && index >= (page-1)*size);
        // 返回处理结果, 这里没做异常状态码处理
        return {
            status: {
                code: 0,
                message: '请求成功',
                time: '2019-07-03 16:45:12',
            },
            data: pageList,
        };
    },
// 列表详情数据返回
    getSpecialDetail: config => {
        let { id, page = 1, size, lang } = config.body;
        return {
            status: {
                code: 0,
                message: '请求成功',
                time: '2019-07-03 16:45:12',
            },
            data: {
                detail: specialList[Math.ceil(1 + Math.random() * 98)],
                total: Math.ceil(Math.random() * 100),
            },
        };
    },
};
```

#### 启动本地服务

这里我们需要在package.json 文件script 字段下新建一行命令去开启服务器

```
    "mock": "node src/mock/index.js"
    在终端下执行 npm run mock 即可开启服务, 接下来我们就可以放心去请求数据了
```

### axios 通用配置

#### 1.创建 axios 实例

```js
// 1.引入所需依赖: axios,Vue,store,router,工具函数 toast 等等...
import axios from 'axios';
import Vue from 'vue';

// 创建 axios 实例
const Axios = axios.create({
    // 添加初始化配置
    baseURL: process.env.BASE_API, // 基地址
    timeout: 10000, // 超时
    // withCredentials: true, // send cookies when cross-domain requests
});

// const TOKEN = window.sessionStorage.getItem('token'); // when set token

// 请求拦截器
Axios.interceptors.request.use(
    config => {
        // TODO: 添加请求头处理逻辑,如添加 token; 还可以设置开启请求 loading 动画
        // config.headers['X-Token'] = TOKEN
        return config;
    },
    error => {
        console.log('request error:', error); // for debug
        return Promise.reject(error);
    }
);

/**
 * 两种方式做统一处理
 * 1.通过 http状态码status:   response.status 和 error.response.status处理
 * 2.通过 response.data中自定义的状态码code:  response.data.code 处理(异常处理直接在第一个回调重处理)
 */

// 响应拦截器
Axios.interceptors.response.use(
    response => {
        const res = response.data;
        // 1.通过自定义 code 做统一处理
        // 2.也可以通过 response.status === 200 处理; 这里采用 1
        if (res.status.code === 0) {
            return res.data; // 成功
        } else {
            // for example: 异常的 code 进行自定义处理
            if (res.status.code === 50008 || res.status.code === 50012) {
                // 做一些提示的 toast 处理或者跳转
                alert('有错误!!!');
            }
            return Promise.reject(new Error(res.status.message || 'Error'));
        }
        
    },
    error => {
        // 响应错误处理: 如超时,断网,权限等...
        console.log('response error:', error); // for debug 本地开发下没有状态码显示
        console.log(error.response); // for debug
        // const res = error.response;
        // if (res) {
        //     // 请求已发出，但是不在2xx的范围
        //     errorHandle(response.status, response.data.message);
        //     return Promise.reject(res);
        // } else {
        //     // 处理断网或超时...
        //     // 网络状态可以在app.vue中控制着一个全局的断网提示组件的显示隐藏
        //     if (!window.navigator.onLine) {
        //         console.log('断网了....');
        //     } else {
        //         return Promise.reject(error);
        //     }
        // }
    }
);

/**
 * 请求失败后的错误统一处理
 * @param {Number} status 请求失败的状态码
 */
const errorHandle = (status, tips) => {
    // 状态码判断
    switch (status) {
        //     // 401: 未登录状态，跳转登录页
        //     case 401:
        //         toLogin();
        //         break;
        //     // 403 token过期
        //     // 清除token并跳转登录页
        //     case 403:
        //         tip('登录过期，请重新登录');
        //         localStorage.removeItem('token');
        //         store.commit('loginSuccess', null);
        //         setTimeout(() => {
        //             toLogin();
        //         }, 1000);
        //         break;
        // 404请求不存在
        case 404:
            console.log('请求的资源不存在');
            break;
        default:
            console.log(tips);
    }
};

// 实现插件, 暴露install方法, 可以给 vue 实例使用
// 通过 this.$http.post 或者 get 去请求
export const http = {
    install(Vue) {
        Vue.prototype.$http = Axios;
    }
}

// 导出 Axios实例
export default Axios
```

#### 2.使用



```js
1.通过插件使用, 在 mian.js 导入
    import { http } from './api/http';
    Vue.use(http);
    在组件实例中通过 this.$http.post | get去请求
    
2.将请求方法抽离, 引入上面导出的 Axios 实例
    import request from './http';

    // 首页请求
    export function getHomeData(data) {
        return request({
            url: '/home',
            method: 'post',
            data
        })
    }
```