# Vue知识点

## Prop

1. Prop如果为数组或对象类型，default值必须从函数中返回。这是为了保证不同component实例中propB都是独立的实例。

```
props: {
    propA: {
        type: Number,
        default: 100
    },
    propB: {
        type: Object,
        default: function () {
            return { a: 1 };
        }
    }
}
```
2. 非prop特性使用$attrs传递。

传递给组件的属性，如果没有相应的prop定义，则该属性会放在$attrs中，常见的是基于html元素构建的基础组件，元素的原生属性可通过$attrs传递。

```
// 实际问题是对el-input再封装时，要透传原有属性，但发现placeholder，rows等属性并未生效
// 是因为这些原生属性在$attrs里，透传要兼顾$props和$attrs

<el-input v-bind="finalProps"></el-input>

computed: {
    finalProps () {
        const {$props, $attrs} = this;
        return {...$props, ...$attrs};
    }
}
```

## 代码规范

1.  组件编码规范

导出一个清晰、组织有序的组件，使得代码易于阅读和理解。同时也便于标准化。
能避免操作 dom 就尽量避免，实在要用的话最好使用 ref 来代替 querySelector 等选择器方法
一个 .vue 的文件行数最好控制在 200 行左右
善用 v-if 和 v-show。比如，涉及到权限的必须用 v-if 而非 v-show。例如，用户必须登录后才能查看的，请用 v-if

请尽量保证数据流的可追踪性。尽量不要使用 $parent，而是通过 props 属性接收父组件的传入

参考：[Vue组件设计规范](https://pablohpsilva.github.io/vuejs-component-style-guide/#/chinese)

2. 按需加载组件

一般配合 Vue-Router 使用，适用于大型应用，将应用分割成小的代码块，只在需要的时候才从服务器加载。

实现方式:

    - 异步组件实现
    - es6 import
好处：

- 按需加载，节省首次加载实践，提高速度，也算是一个性能优化；
- 组件只会加载一次，加载完成后会缓存下来，使用一个组件多次使用的场景。

```
// 异步组件实现
export default new Router({
    routes: [
        {
            path: '/test',
            name: 'test',
            component: resolve => require(['../components/Test'], resolve)
        },
    ]
})

// ES6 import
const Test1 = () => import('../components/Test1')
const Test2 = () => import('../components/Test2')

export default new Router({
    routes: [
        {
            path: '/test1',
            name: 'test1',
            component: Test1
        },
        {
            path: '/test2',
            name: 'test2',
            component: Test2
        }
    ]
})
```

3. 过滤器

用于一些常见的文本格式化，如展示发布时间。

```
<template>
    <!-- 在双花括号中 -->
    <span>{{ message | capitalize }}</span>
    <!-- 在 `v-bind` 中 -->
    <div :message="message | capitalize"></div>
</template>

<script>
    export default {
        data() {
            return {
                message:1
            }
        },
        filters: {
            capitalize: function (value) {
                if (!value) return ''
                value = value.toString()
                return value.charAt(0).toUpperCase() + value.slice(1)
            }
        }
    }
</script>
```

4. v-for 循环加 key

当 Vue.js 用 v-for 正在更新已渲染过的元素列表时，它默认用 就地复用 策略。如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序，而是简单复用此处每个元素，并且确保它在特定索引下显示已被渲染过的每个元素。

稍微深入一点，加 key 的原因是为了给 virtualDom中 的 diff 做优化，结果就是提高 virtualDom 更新效率。

5. 方法抽取
在开始我们项目之前，我们应该知道，有哪些方法是必要的、共有的，类似于我们的工具方法一样。这里我罗列一些供参考：

    1. 网络请求
    
    项目可能用到几种请求？（POST,GET,PUT,DELETE…）对于不同的请求，我们的请求头也需要改变，是抽取成四个方法，还是在一个方法上变异？

    哪种更适合现在的业务场景？

    请求出错如何处理（组件内捕获？全部捕获？其他？…）

    为什么提出这个问题呢？因为这是我前不久我的leader给我提的优化方案。之前的项目都没在意过这个，后来是把axios包了一层，使用了类似jQuery的请求方式，实现了组件内捕获业务场景错误（校验失败之类的），全部捕获请求错误（404，500，400等）。

    2. 数据转换

    - 日期格式转换

    - 日期与字符串转换

    关于日期都所有操作，建议采用momentJs。功能强大而且兼容性好，我深深的记得之前一个钉钉项目中日期操作不兼容iphone手机的时候，那个bug让我找了半天才找到根源。官网传送门：http://momentjs.cn/

    - 数字与字符串转换

    3. 交互方法

    你的应用应该包含一套用于提示用户的组件：

    - Toast

    用于提示用户一些信息

    - Confirm

    用户提示用户是否确认接下来的操作。对于一些重要的操作，比如提交表单，删除信息等操作，务必使用。

    - Loading

    当用户在上传或者下载图片等其他资源的时候，用于提示进度信息。当然这个请求方法要能够获得到相应的进度信息才行，伪造须谨慎（手动斜眼笑）。

    - Spinner

    加载数据时的提示，俗称“菊花图“

    这里要说的是，你可以对这些方法进行二次封装，以便更方便的使用。包装后，可能就可以像下面这样使用了：

```
Dialog.toast('cool')
Dialog.confirm('Are you sure?',()=>{Dialog.toast('confirm')})
Dialog.spinner('loading...')
```

## 常见的坑

1. 对象和数组的更新检测

由于 JavaScript 的限制，Vue 不能检测以下变动的数组：

- 当你利用索引直接设置一个项时，例如：vm.items[indexOfItem] = newValue
- 当你修改数组的长度时，例如：vm.items.length = newLength

还是由于 JavaScript 的限制，Vue 不能检测对象属性的添加或删除

解决方法： 对于数组来说可以使用 vm.$set或改用可观察数组的变异方法，对于对象来说可以使用 vm.$set 或 Object.assign

e.g. [https://jsfiddle.net/hysunny/eywraw8t/228152/](https://jsfiddle.net/hysunny/eywraw8t/228152/)

2. mixins同名选项混合问题

当我们想覆盖一个组件的一些东西或想扩展某个组件时，可以用 Vue 的 mixins。

不过要注意:
    当组件和混合对象含有同名选项时，同名钩子函数将混合为一个数组，都会被调用；混合对象的钩子将在组件自身钩子之前调用。
    值为对象的选项，例如 methods, components 和 directives，将被混合为同一个对象。两个对象键名冲突时，取组件对象的键值对。

3. v-if 与 v-for 的优先级问题

v-if 与 v-for 一起使用时，v-for 具有比 v-if 更高的优先级，因此如果想有条件的跳过循环的执行，则需将 v-if 置于外层元素（或 template）上。如：

```
<ul v-if="todos.length">
    <li 
        v-for="todo in todos" 
        :key="todo.id"
    >
       {{ todo.name }}
    </li>
</ul>
<p v-else>No todos left!</p>
```

4. 遍历对象顺序不一致

使用 v-for 遍历对象时，是按 Object.keys() 的结果遍历，但是不能保证它的结果在不同的 JavaScript 引擎 下是一致的。

比如： 在 ios 下当对象的key为字母时，排序为降序，其他机型为升序。

解决方法： 如果要保证顺序，可以加个排序的 filter 或者 改用数组。

参考： https://github.com/vuejs/vue/issues/1827

5. event bus 多次触发

$on 的事件可在 created 或 mounted 注册，需在 beforeDestroy 或 destoryed 的时候使用用 $off 销毁，否则在某些情况下会被被多次触发。

e.g. https://jsfiddle.net/hysunny/eywraw8t/232163/

6. 变量命名

变量名不要以_、$开头，因为名字以 _ 或 $ 开始的属性不会被 Vue 实例代理，因为它们可能与 Vue 的内置属性与 API 方法冲突。

需要用 vm.$data._property 访问它们。

e.g: https://jsfiddle.net/hysunny/eywraw8t/224835/

7. vue 2.0 给组件绑定事件无效

对于一般的 html 元素，绑定自定义事件使用 v-on即可，但是在某个组件的 根元素 上监听一个 原生事件 ，比如：

```
<my-component v-on:click="handleClick" />
```

我们会发现这样是不起作用的，可以使用 .native 修饰符(某些情况) 或是 $listeners

```
<my-component v-on:click.native="handleClick" />
```

e.g. 用.native给自定义组件绑定事件



