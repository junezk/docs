# 【实践总结】优雅的处理vue项目异常

### 背景

- 你还在为处理Uncaught (in promise) ReferenceError烦恼吗？
- 你还在为捕获异常反复的写try catch吗？
- 你还在为每一个promise写catch吗？

##### 是时候一站式统一处理异常！！！（针对vue项目）

#### 全局异常捕获

```
Vue.config.errorHandler = function (err, vm, info) {
  // 指定组件的渲染和观察期间未捕获错误的处理函数。这个处理函数被调用时，可获取错误信息和 Vue 实例。
  
  // handle error
  // `info` 是 Vue 特定的错误信息，比如错误所在的生命周期钩子
  // 只在 2.2.0+ 可用  
}
复制代码
```

> 注意：面对异常处理，同步异常和异步异常应该区别对待分别处理。

### vue核心源码剖析

通过阅读源码看一下vue是如何将Vue.config.errorHandler接口暴露给使用者。

#### 同步异常处理方案

```
// 定义异常处理函数，判断用户是否自定义Vue.config.errorHandler，定义则直接调用，未定义执行vue本身异常处理。
function globalHandleError(err, vm, info) {
  if (Vue.config.errorHandler) {
    try {
      return config.errorHandler.call(null, err, vm, info)
    } catch (e) {
      logError(e, null, 'config.errorHandler');
    }
  }
  logError(err, vm, info);
}
try {
  // vue正常执行代码被包裹在try内，有异常会调用globalHandleError
} catch (e) {
  globalHandleError(e, vm, '对应信息');
}
复制代码
```

#### 异步异常处理方案

```
// 定义异步异常处理函数，对于自身没有捕获异常的promise统一执行catch
function invokeWithErrorHandling(
  handler,
  context,
  args,
  vm,
  info
) {
  var res;
  try {
    res = args ? handler.apply(context, args) : handler.call(context);
    if (res && !res._isVue && isPromise(res) && !res._handled) {
      res.catch(function (e) { return handleError(e, vm, info + " (Promise/async)"); });
      // 异步代码例如promise可以统一为其定义Promise.prototype.catch()方法。
      res._handled = true;
    }
  } catch (e) {
    handleError(e, vm, info);
  }
  return res
}

// 所有的钩子函数调用异常处理函数
function callHook(vm, hook) {
  var handlers = vm.$options[hook];
  // 为所有钩子增加异常处理
  var info = hook + " hook";
  if (handlers) {
    for (var i = 0, j = handlers.length; i < j; i++) {
      invokeWithErrorHandling(handlers[i], vm, null, vm, info);
    }
  }
}
复制代码
```

### 知识延伸

```
// vue接口是能处理同步异常以及部分钩子中的异步异常，对于方法中的异常无法有效处理，我们可以仿照源码增加方式中的异步异常处理，避免为每一个promise写catch
Vue.mixin({
  beforeCreate() {
    const methods = this.$options.methods || {}
    Object.keys(methods).forEach(key => {
      let fn = methods[key]
      this.$options.methods[key] = function (...args) {
        let ret = fn.apply(this, args)
        if (ret && typeof ret.then === 'function' && typeof ret.catch === "function") {
          return ret.catch(Vue.config.errorHandler)
        } else { // 默认错误处理
          return ret
        }
      }
    })
  }
})
复制代码
```

### 完整代码

下面是全局处理异常的完整代码，已经封装成一个插件

```
errorPlugin.js

/**
 * 全局异常处理
 * @param {
 * } error 
 * @param {*} vm 
 */
const errorHandler = (error, vm, info) => {
  console.error('抛出全局异常')
  console.error(vm)
  console.error(error)
  console.error(info)
}
let GlobalError = {
  install: (Vue, options) => {
  /**
   * 全局异常处理
   * @param {
   * } error 
   * @param {*} vm 
   */
    Vue.config.errorHandler = errorHandler
    Vue.mixin({
      beforeCreate() {
        const methods = this.$options.methods || {}
        Object.keys(methods).forEach(key => {
          let fn = methods[key]
          this.$options.methods[key] = function (...args) {
            let ret = fn.apply(this, args)
            if (ret && typeof ret.then === 'function' && typeof ret.catch === "function") {
              return ret.catch(errorHandler)
            } else { // 默认错误处理
              return ret
            }
          }
        })
      }
    })
    Vue.prototype.$throw = errorHandler
  }
}
export default GlobalError
复制代码
```

#### 使用

```
// 在入口文件中引入
import ErrorPlugin from './errorPlugin'
import Vue from 'vue'
Vue.use(ErrorPlugin)
复制代码
```

### 写在最后

增加全局异常处理有助于

- 提高代码健壮性
- 减少崩溃
- 快速定位bug

### 资料参考

- [github.com/vuejs/vue/b…](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Fblob%2Fdev%2Fsrc%2Fcore%2Futil%2Ferror.js)
- [cn.vuejs.org/v2/api/#err…](https://link.juejin.im/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fapi%2F%23errorHandler)