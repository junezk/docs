# Vue响应式原理 - 关于Array的特别处理

之前写过一篇[响应式原理-如何监听Array的变化](https://juejin.im/post/5cf606d6f265da1b8e708ba6)，最近准备给团队同事分享，发现之前看的太粗糙了，因此决定再写一篇详细版~

## 一、如何监听数组索引的变化？

### （1）案例分析

相信初学`Vue`的同学一定踩过这个坑，改变数组的索引，没有触发视图更新。 比如下面这个案例：

```
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})
vm.items[1] = 'x' // 不是响应性的
```

以上案例摘抄[Vue官方文档 - 数组更新检测](https://link.juejin.im/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Flist.html%23%E6%95%B0%E7%BB%84%E6%9B%B4%E6%96%B0%E6%A3%80%E6%B5%8B)。

### （2）解决方式

`Vue`官方文档也有给出，使用`Vue.set`即可达到触发视图更新的效果。

```
// Vue.set
Vue.set(vm.items, indexOfItem, newValue);
```

### （3）Vue为何不能监听索引的变化？

`Vue`官方给出了解释，不能检测。

> 由于 JavaScript 的限制，Vue 不能检测以下数组的变动： 当你利用索引直接设置一个数组项时，例如：`vm.items[indexOfItem] = newValue`。

那原因是什么？我在学习的过程中发现很多文章都在断章取义，`Vue`官方给出了解释是【`Vue`不能检测】，而很多文章写出的是【`Object.defineProperty`不能检测】。

但实际上`Object.defineProperty`是可以检测到数组索引的变化的。如下案例：

```
let data = [1, 2];
function defineReactive (obj, key, val) {
    Object.defineProperty(obj, key, {
        enumerable: true,
        configurable: true,
        get: () => {
            console.log('我被读了，我要不要做点什么好?');
            return val;
        },
        set: newVal => {
            if (val === newVal) {
                return;
            }
            val = newVal;
            console.log("数据被改变了，我要渲染到页面上去!");
        }
    })
}

defineReactive(data, 0, 1);
console.log(data[0]);
data[0] = 5;
```

大家可以自己在控制台中尝试一下，答案非常明显了。

![`Object.defineProperty`检测数组索引的变化](https://user-gold-cdn.xitu.io/2019/8/17/16c9e5041acb1e47?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

`Vue`只是没有使用这个方式去监听数组索引的变化，因为尤大认为性能消耗太大，于是在性能和用户体验之间做了取舍。 详细可见这边文章[Vue为什么不能检测数组变动](https://link.juejin.im/?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000015783546)。

![img](https://user-gold-cdn.xitu.io/2019/8/17/16c9e51e9dadb9e6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

好了，终于揭开了谜底，为什么`Vue`为什么不能检测数组变动，因为不做哈哈。

但是我们开发者肯定是有这个需求的，解决方式就是如下，使用`Vue.set`。

```
// Vue.set
Vue.set(vm.items, indexOfItem, newValue);
```

原理？非常明显，在初始的过程中没有循环对所有数组索引监听，但是开发者需要监听哪个索引。`Vue.set`就帮你监听哪个，核心还是`Object.defineProperty`。只是尽可能的避免了无用的数组索引监听。

## 二、如何监听数组内容的增加或减少？

### （1）技能限制

`Object.defineProperty`虽然能检测索引的变化，但的确是监听不到数组的增加或删除的。可以阅读 [Vue官方文档 - 对象变更检测注意事项](https://link.juejin.im/?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Flist.html%23%E5%AF%B9%E8%B1%A1%E5%8F%98%E6%9B%B4%E6%A3%80%E6%B5%8B%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9) 进行了解。

这个时候`Vue`是怎么做的呢？

### （2）巧妙解决

![数组拦截](https://user-gold-cdn.xitu.io/2019/8/17/16c9e5f2a87b44fb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

`Vue`的解决方案，就是重写了数组的原型，更准确的表达是**拦截**了数组的原型。

首先选择了7个能够改变数组自身的几个方法。其次看下案例吧：

```
// 获得原型上的方法
const arrayProto = Array.prototype;

// 创建一个新对象，使用现有的对象来提供新创建的对象的__proto__
const arrayMethods = Object.create(arrayProto); 

// 做一些拦截的操作
Object.defineProperty(arrayMethods, 'push', {
    value(...args) {
        console.log('用户传进来的参数', args);

        // 真正的push 保证数据如用户期望
        arrayProto.push.apply(this, args);
    },
    enumerable: true,
    writable: true,
    configurable: true,
});

let list = [1];

list.__proto__ = arrayMethods; // 重置原型

list.push(2, 3);

console.log('用户得到的list:', list);
```

为什么叫**拦截**，我们在重写案例中的`push`方法时，还需要使用真正的`push`，这样才能保证数组如用户所期望的`push`进去。

可以看到以下效果，我们既能监听到用户传进来的参数，也就是监听到这个数组变化了，还能保证数组如用户所期望的`push`进去。

![结果](https://user-gold-cdn.xitu.io/2019/8/17/16c9e66835650c29?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

为什么使用`arrayMethods`继承真正的原型，因为这样才不会污染全局的`Array.prototype`，因为我们要监听的数组只有`vm.data`中的。

### （3）源码分析

```
export class Observer {
    constructor (value: any) {
        // 如果是数组
        if (Array.isArray(value)) {
            // 如果原型上有__proto__属性， 主要是浏览器判断兼容
            if (hasProto) {
                // 直接覆盖响应式对象的原型
                protoAugment(value, arrayMethods)
            } else {
                // 直接拷贝到对象的属性上，因为访问一个对象的方法时，先找他自身是否有，然后才去原型上找
                copyAugment(value, arrayMethods, arrayKeys)
            }
        } else {
          // 如果是对象
          this.walk(value);
        }
    }
}
```

以上可以看到`Observer`对数组的特别处理。

### （4）数组是如何收集依赖、派发更新的？

我们知道对象是在`getter中收集依赖，setter中派发更新`。 那简单回忆下：

```
function defineReactive (obj, key, val) {
    // 生成一个Dep实例
    let dep = new Dep();
    Object.defineProperty(obj, key, {
        get: () => {
            // 依赖收集
            dep.depend();
        },
        set: () => {
            // 派发更新
            dep.notify();
        },
    })
}
```

为了保证`data`中每个数据有着一对一的`dep`，这里应用了闭包，保证每个`dep`实例不会被销毁。那么问题来了，`dep`是一个局部变量呀~ 而监听数组变化，需要在数组拦截器中进行派发更新。那就访问不到这个`dep`了，就无法知道具体要通知哪些`Watcher`了！

那Vue是怎么做的呢？既然这个访问不到，那就再来一个`dep`吧。

```
export class Observer {
    constructor (value: any) {
        this.value = value // data属性
        this.dep = new Dep() // 挂载dep实例
        // 为数据定义了一个 __ob__ 属性，这个属性的值就是当前 Observer 实例对象
        def(value, '__ob__', this) // 把当前Observer实例挂在到data的__ob__上
    }
}
```

在`Vue`初始化的过程中，给`data`中的每个数据都挂载了当前的`Observer`实例，又在这个实例上挂载了`dep`。这样就能保证我们在数组拦截器中访问到`dep`了。如下：

```
Object.defineProperty(arrayMethods, 'push', {
    value(...args) {
        console.log('用户传进来的参数', args);

        // 真正的push 保证数据如用户期望
        arrayProto.push.apply(this, args);
        
        // this指向当前这个数组，在初始化的时候被赋值__ob__
        console.log(this.__ob__.dep)
    },
    enumerable: true,
    writable: true,
    configurable: true,
});
```

现在我们便可以在拦截器中执行`dep.notify()`啦。

那如何收集依赖呢？

```
// 获取当前data上的 observe实例，也就是__ob__
let childOb = !shallow && observe(val);

function defineReactive (obj, key, val) {
    // 生成一个Dep实例
    let dep = new Dep();
    Object.defineProperty(obj, key, {
        get: () => {
            if (Dep.target) {
                // 依赖收集
                dep.depend();
                
                // 二次收集
                if (childOb.dep) {
                    // 再收集一次依赖
                    childOb.dep.depend();
                }
            }
            return val;
        },
    })
}
```

现在要存放2个`dep`，那自然是要在`getter`中收集2次的，`childOb`其实就是`observe`中返回的`__ob__`。不用在意细节，自行查看源码就知道啦~

### （5）总结

总结一下，针对数组`在getter中收集依赖，在拦截器中触发更新`。

![数组](https://user-gold-cdn.xitu.io/2019/8/17/16c9e82279259f53?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 三、其他思考

### （1）思考：还有哪里可以用到__ob__?

1. 判断某个数组是否已Observer过，避免重复执行。
2. `Vue.set`和`Vue.del`，都是需要访问`dep`的。

### （2）数组赋值算改变长度吗？

因为`Object.defineProperty`不能检测数组的长度变化，例如：`vm.items.length = newLength`。

```
var vm = new Vue({
  data: {
    items: ['a']
  }
})
// 重新赋值，改变长度
vm.items = ['a, 'b', 'c']
```

那`vm.items = ['a, 'b', 'c']`这种情况，`Vue`是如何监听的？这种情况其实监听的是对象`vm`的`items`属性，和数组其实是没关系的。因为之前发现有人误解，这里简单的提示一下~

## 四、总结

本文主要还是讲原理及思路，并不会涉及到很多代码，毕竟源码总会变。同时还要保证自己的js基础扎实，阅读源码才不会吃力哦~ 我就是很吃力的那种😭

如果你觉得对你有帮助，就点个赞吧~

已完成：

Vue源码解读系列篇

- [1. Vue响应式原理-理解Observer、Dep、Watcher](https://juejin.im/post/5cf3cccee51d454fa33b1860)
- [2. 响应式原理-如何监听Array的变化](https://juejin.im/post/5cf606d6f265da1b8e708ba6)
- [3. 响应式原理-如何监听Array的变化？详细版](https://juejin.im/post/5d579cd36fb9a06aea6190db)
- [4. Vue异步更新 - nextTick为什么要microtask优先？](https://juejin.im/post/5d57994ef265da03bd051969)

[Github博客 ](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fyukiyang0729%2Fblog)欢迎交流~

## 五、参考文献

- [记一次思否问答的问题思考：Vue为什么不能检测数组变动](https://link.juejin.im/?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000015783546)
- 书籍《深入浅出Vue.js》