# Vue实战038：api接口模块化统一管理

一开始使用Vue时，我们用axios发送请求时一般会将api接口直接写在对应的请求方法中，这样当我们项目需要修改请求时我们就需要先去找到当前的组件，然后再找到对应的事件方法，最后才能将api修改，这样维护起来剧会变的很费时费力（特别是接口比较多的时候）。

```js
//举个栗子
this.$axios.post('http://127.0.0.1:8000/login/',{
  username:this.uname,
  password:this.password,
}).then(res=>{
  if(res.status==200){
    Message.success('登陸成功')
    this.$router.push('/')
  }
}).catch(error=>{
  Message.error(error)
})
```

随着项目的进展，我们会把api提取出来，用一个api.js文件来统一存放api接口，通过expot导出对应的接口，然后在组件中引用该接口。这样当我们需要修改接口时，只要到api文件中统一处理即可，同时也方便我们了解项目中实现了哪些接口，目前这个项目是定义了一个baseURL，然后与接口进行拼接生成一个常量api，再将其导出给其他组件调用。

![Vue实战038：api接口模块化统一管理](Vue实战038：api接口模块化统一管理.assets/92006aa1-f21a-45ec-8e1d-b514b4c62c2b)



如果你的项目对axios进行了封装，那么我们这里还可以对api进一步优化，我们可以在封装的axios设置好基本路由，这样当我们拿到api后系统会为我们自动拼接路由，同时还可以统一的异常处理方案简化了每个请求书写，提高代码的可读性。如何封装axios可以参考文章 [Vue实战037：axios二次封装和使用](https://www.toutiao.com/i6717068795506065927/?group_id=6717068795506065927)，

![Vue实战038：api接口模块化统一管理](Vue实战038：api接口模块化统一管理.assets/d97f64881fbc485db029600d3a159565)



接下来我们继续优化api管理，继续拿我们的登陆接口来解析：
http://127.0.0.1:8000/login/ ，这里可以分为4部分，分别是支持的协议（http协议），ip（127.0.0.1），端口（8000），路由（login/），假设每个api都是从同一台服务发出的，那么每个接口的协议、ip和端口都是一样的，这时我们就可以通过axios.defaults.baseURL = 'http://127.0.0.1:8000/'统一指定。

![Vue实战038：api接口模块化统一管理](Vue实战038：api接口模块化统一管理.assets/41afdd3f-9623-4a81-8823-b62bc87b36b7)



# 如何定义api模块

这里我们准备将axios.js引入api.js中，然后调用axios.js中的接口，接口需要提供3个参数，分别是method、url、params，这里可以用箭头函数，写起来简单。这样我们就定义了一个login方法，只需要传入method和params参数即可。

```
import api from '@/api/axios.js'
export const login = (method, params) => api(method, 'login/', params)
```

# 如何使用

在组件中导入api中的方法，用到哪个导入哪个即可，然后再事件中调用我们的login方法，传入对应的method和params参数接口，是不是比之前简单又简洁了很多，封装还有 个好处就是可以很好的避开操作token，这样就不用在每个接口都手动的提交token操作了。

```
//引入组件中的方法
import {login} from '@/api/api.js'
//调用login方法，传入method和params即可
login('post',{
 username:this.uname,
 password:this.password,
}).then(res=>{
 if(res.status==200){
 this.$message.success('登陸成功')
 this.$router.push('/')
 }
}).catch(error=>{
 this.$message.error(error)
})
```