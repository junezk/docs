# Vue 3.x学习笔记

## 1、安装

前安装过vue的2.0版本，你需要把2.0相关的删除

```shell
npm uni -g vue-cli
```

### 1.1  通过Vue-cli脚手架安装

```shell
yarn global add @vue/cli
npm install -g @vue/cli
cnpm install -g @vue/cli
```

安装后通过`vue -V`检查版本，应该是4.5.*。使用Vue-cli创建项目

```shell
vue create prj-vue1
yarn serve  # 运行项目
```

### 1.2 通过Vite脚手架创建项目

```shell
npm init vite-app prj-vue2   # 使用npm 安装
cd prj-vue2
npm install
npm run dev
# 或用yarn
yarn create vite-app prj-vue2
cd prj-vue2
yarn
yarn dev
```

1.3 使用scss

使用scss的话，需要安装`sass-loader`和`node-sass`

## 2、目录结构

src：源目录

assets：静态文件目录

components：自定义组件目录

App.vue：根组件，被main.js引用

## 3、绑定数据

### 3.1 绑定普通数据

### 3.2 `v-bind` 绑定属性，动态参数和属性

### 3.3 `v-html` 绑定html 

### 3.4 `v-for`  循环绑定，遍历对象的各个属性

```vue
v-for="value, index in array"
v-for="value, attr, index in object"
```

## 4、自定义方法

在`methonds`中定义自定义方法函数

```vue
methods:{
	setMsg(){
	
	},...
}
```

点击事件触发方法

```vue
<button v-on:click="setMsg()"></button>   //简写为@click
<button @click="setMsg()"></button>
```

## 5、绑定数据到style、class

```
:class="{}"    // 可以绑定到变量、数组甚至对象,常用的是结合三目运算符
:style="{}"      
```

## 6、事件

事件参数、事件对象

1次事件可以执行多个方法

事件修饰符

```
.stop
.prevent   阻止默认事件行为
.capture
.self
.once
.passiv
```

按键修饰符

```
.tab
.enter
.delete   //同时捕捉“删除”和“退格”键
.esc
.space
.up
.down
.left
.right
```

## 7、双向数据绑定

获取表单数据

```vue
<input type="text" id="username" />
<input type="text" ref="age" />
可以使用原生js的语法：（原生dom操作非常耗费性能，尽量不用）
var usernameObj = document.querySelector("#username")
alert(usernameObj.value)
或者使用vue ref
alert(this.$refs.age.value)
```

双向数据绑定

```
data中定义数据变量，标签中使用  v-model="var"
```

## 8、vue模板中使用Javascript表达式

表达式、条件判断、计算属性和watch监听

v-if  v-else   v-else-if，如果需要一组标签需要控制的话，可以使用Template

v-show 通过css控制显示，内容会产生，但不显示

## 9、计算属性

```
computed:{
	// 计算属性使用的变量如果修改了，则属性值会跟着改变
}
```

## 10、监听数据变化

```
watch:{
	// 监听变量的变化，执行某些代码
}
```

11、Vue3中集成Sass/scsss

11.1 安装sass-loader node-sass

```bash
npm install -D sass-loader  node-sass
```

11.2 style中配置sass/scss

```vue
<style lang="scss" scoped>  <!--scoped 表示样式作用域为当前组件-->
	scss语法
</style>
```

## 11、Vue中的组件

组件通常放到components中，命名为 `xx.vue`，组件通常由template、script和style组成。

引用组件的话，需要引入和挂载。

```vue
import Home from './components/Home'

components: {
	"v-home": Home    //v-home为给组件标签起的别名，如不冲突，也可直接使用组件名称
}
```

组件可以嵌套引用。

## 12、组件间传值

### 12.1 父组件给子组件传值

通过给子组件设置属性，子组件内部设置`props`来实现。

```vue
<my-header :title="值或者变量"></my-header>  <!--父组件中引用子组件-->

<!--子组件中定义-->
<h2>{{title}}</h2>
<script>
export default{
	data() {
		return {
		
		}
	},
	props: ["title"]
}
</script>
```

甚至可以把父组件本身（this）作为变量传递给子组件。

父组件向子组件传递数据是一个**单向数据流**，子组件不能修改父组件的值。

### 12.2 父组件主动获取子组件数据，执行子组件方法

1. 调用子组件时定义一个ref

   ```vue
   <v-header ref="header"></v-header>
   ```

2. 父组件获取子组件数据

   ```vue
   this.$refs.header.属性
   ```

3. 父组件执行子组件方法

   ```vue
   this.$refs.header.方法
   ```

### 12.3 子组件获取父组件数据，执行父组件方法

```
this.$parent.属性
this.$parent.方法()
```

这种方式的数据是**双向**的，可以相互修改数据。

## 13、自定义组件事件实现子组件给父组件传值

```vue
<!--父组件中自定义事件-->
@事件名="方法"
<!--子组件中调用自定义事件-->
<script>
export default{
    emits:["事件名称"],
	data() {
		return {		
		}
	},
    methods:{
        run(){
            this.$emit("事件名", "传递的值，可选")
        }
    }
	props: ["title"]
}
</script>
```

## 14、非父子组件传值

没有引用关系的组件是非父子组件。

通过调用第三方模块`mitt`来实现。

```
yarn install mitt
```

## 15、自定义组件的双向数据绑定

### 15.1 使用`v-model`实现双向数据绑定

### 15.2 自定义组件中使用`slots`

## 16、Vue3.x中，非Prop的Attribute继承

一个非`prop`的`attribute`是指传向一个组件，但该组件并没有相应的`props`或`emits`定义的`attribute`。常见的包括`class`、`style`和`id`属性。

可以在组件中设置禁用属性的继承，然后自定义继承。

```vue
<template>
<div>
    <input v-bind="$attrs" />  <!--在这里设置继承属性的标签-->
</div>
</template>

<script>
export default{
    inheritAttrs: false,   //禁用默认集成
	data() {
        ...
	},
}
</script>
```

当有多个根节点时，也需要指定继承属性的节点标签。

## 17、Vue的生命周期函数

beforeCreate  、created、beforeMounted、mounted（最常用）、beforeUnmount（销毁，常用）、unmounted

## 18、使用第三方模块

Axios、FetchJsonp

## 19、Mixin实现组件功能的复用

定义mixin，然后在组件中通过`mixins`引用mixin来实现复用

```vue
<script>
import baseMixin from '...'
export default{
    mixins: [baseMixin],
    inheritAttrs: false,   //禁用默认集成
	data() {
        ...
	},
}
</script>
```

### 19.1 mixin选项的合并

组件和mixin中都定义同一名称的属性时，当前组件的属性会覆盖mixin中的相同属性，方法同样。

### 19.2 全局配置使用Mixin

在入口文件中通过 `app.mixin(mixin模块)`来全局配置，然后在组件中就可以直接使用mixin中定义的属性和方法了。

## 20、Vue3中的Teleport

Vue3.x中的组件模板属于该组件，但有时我们想把模板中的内容移动到当前组件之外的DOM中，这时可以使用Teleport。

```vue
<teleport to="body">   <!--将在body中输出内容-->
	...
</teleport>
```

to的目标可以时标签，也可以是id（#id）

### 20.1 使用Teleport实现模态对话框组件

## 21、Composition API介绍

解决大型项目中的代码组织与逻辑复用问题。

目前，我们使用的是“options” API 构建组件。 为了将逻辑添加到Vue组件中，我们填充（options）属性，如data、methods、computed等。 这种方法最大的缺点是，它本身不是一个工作的JavaScript代码。 您需要确切地知道模板中可以访问哪些属性以及this关键字的行为。在底层，Vue编译器需要将此属性转换为工作代码。正因为如此，我们无法从自动建议或类型检查中获益。

Composition API希望将通过当前组件属性、可用的机制公开为JavaScript函数来解决这个问题。 Vue核心团队将组件Composition API描述为“一套附加的、基于函数的api，允许灵活地组合组件逻辑”。 使用Composition API编写的代码更易读，并且场景不复杂，这使得阅读和学习变得更容易。

`Composition API` 提供了以下函数：

- setup
- ref
- reactive
- watchEffect
- watch
- computed
- toRefs
- 生命周期的hooks

### 21.1 setup方法

vue3.0将组件的逻辑都写在了函数内部，setup()会取代vue2.x的data()函数，返回一个对象，暴露给模板。而且只在初始化的时候调用一次，因为值可以被跟踪，所以我们通过vue3来改变编程习惯。

在`setup`方法中，通过`ref、reactive`定义相应式数据，使用数据需要return。

```vue
<script>
import {ref, reactive, toRefs} from "vue"
export default {
	setup(){
        var name = ref("名称")
        var userinfo = reactive({
            username:"xxx",
            age:20
        })
        var getData = () => {
            alert(userinfo.username)
            alert(name.value)  //获取ref的数据有些不同
            userinfo.username = "新名字"
            name.value = "新值"  
        }
        return {name, userinfo, getData}
    }
}
</script>
```

`reactive` 接口，它的作用是处理你的对象让它经过 `Proxy` 的加工变为一个响应式的对象。

### 21.2 toRefs 解构响应式对象的数据

把一个响应式对象转换成普通对象，该普通对象的每一个property都是一个ref。

```vue
<script>
import {ref, reactive, toRefs} from "vue"
return {
    ...toRefs(userinfo)
}
</script>
```

### 21.3 computed 计算属性

```vue
<script>
import {ref, reactive, toRefs} from "vue"
setup() {
    let fullName = computed(() => {
        return userinfo.firstname + userinfo.lastname
    })
    return {
        ...toRefs(userinfo),
        fullName
    }
}
</script>
```

### 21.4 readonly “深层”只读代理

传入一个对象（响应式或者普通）或ref，返回一个原始对象的只读代理。该对象内部的任何嵌套的属性也是只读的。

```vue
<script>
import {ref, reactive, toRefs, readonly} from "vue"
setup() {
    let obj = {
        username = "非响应式数据"
        age: 20
    }
    var userinfo = reactive({
            username:"相应式数据",
            age:20
        })
    userinfo = readonly(userinfo)   // userinfo变成了只读非响应式数据
    let fullName = computed(() => {
        return userinfo.firstname + userinfo.lastname
    })
    return {
        ...toRefs(userinfo),
        fullName
    }
}
</script>
```

obj由于没有使用`reactive`定义，所以是个非响应式数据，也叫原始对象。

### 21.5 watchEffect 监听响应式对象数据

### 21.6 watch监听数据，功能更强大一些

- 监听到reactive对象返回的对象
- 可以指定监听对象
- 可以得到数据变化前后的值

```vue
<script>
import {ref, reactive, toRefs, watch} from "vue"
setup() {
    var userinfo = reactive({
            username:"相应式数据",
            age:20
        })
    return {
        ...toRefs(userinfo),
        fullName
    }
}
</script>
```

### 21.7 生命周期函数

生命周期中除了beforeCreate、Created函数。

### 21.8 props 接收父组件的传值

在 `Vue2.0` 中我们可以使用 `props` 属性值完成父子通信，在 3.x 中则需要定义 `props` 属性去指定接受值的类型，然后利用 `setup` 的第一个参数获取 `props` 使用。

```vue
<!--父组件，调用content组件-->
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <div>{{msg}}的年龄为{{age}}</div>
    <div>{{double}}</div>
    <Content :msg='msg' />
    <button @click="add">+</button>
  </div>
</template>
<script>
export default {
  name: 'App',
  components:{
    Content
  },
  setup() {
    const state = reactive({
        msg:'王大合', age:18,
        double : computed(() =>{
      return state.age * 2
    })
    })
    function add() {
      state.age += 1
    }
    return {...toRefs(state),add}
  }
}
</script>
```

```vue
<!--子组件定义-->
<template><div>{{data}}</div></template>
<script>
import {ref} from 'vue'
export default {
 name:'content',
 props:{
     msg:String
 },
 setup(props) {
     const data = ref(props.msg)
     return {data}
 }
}
</script>
```

`setup` 函数的第二个参数是一个上下文对象 `context`，这个上下文对象中包含了一些有用的属性，这些属性在 `Vue2.0` 中需要通过 `this` 才能访问到，在 `vue3.0` 中，访问他们变成以下形式：

```js
setup(props, ctx) {
  console.log(ctx) // 在 setup() 函数中无法访问到 this
  console.log(this) // undefined
}
```

具体能访问到以下有用的属性：

- slot
- attrs
- emit

## 22、 Provider Inject组件跨多层传值

```vue
<!--在父组件中定义-->
<script> 
provide：{
	title: "要传递的值"
}//或者用方法返回对象也可以
provide(){
    return {
        title: "要传递的值"
    }
}  // 两种方法都可以
</script>

<!--在子组件中引用-->
<script>
export default{
    inject: ['title']	// 这里title要加引号
}
</script>
```

传统方式的父组件变量修改后不会传递到子组件。

Composition  Api方式下，变量的修改可以进行同步传递，而且是双向传递。

## 23、引入typescript

### 23.1 安装

输入`vue add typescript`命令安装Typescript。或者使用`yarn add typescript -D`。

1. 是否使用class-style 组件语法？选择n（n产生的代码和antd 给出的示例代码一致）
2. 是否使用Babel，选`Y`
3. 是否切换所有.js文件为.ts文件？选Y
4. 是否允许.js文件被编译？选Y
5. 是否检查所有的依赖？选n

### 23.2 初始化`tsconfig.json`

```shell
npx tsc --init
```

### 23.3 使用typescript 

在`script`标签中声明使用`typescript`语言。

```vue
<script lang="ts">
import { defineComponent } from "vue"

import Home from './components/Home.vue'  //这个地方需要给出扩展名
export default defineComponent({
	...
})
```

如果`import App from App.vue`报错：Cannot find module './App.vue'，这时因为`ts`还没识别vue文件，需要进行下面的配置。

在项目根目录添加`shim.d.ts`文件

```typescript
declare module "*.vue" {
  import { Component } from "vue";
  const component: Component;
  export default component;
}
```

### 23.4 原始数据类型

JavaScript 的类型分为两种：原始数据类型（[Primitive data types](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)）和对象类型（Object types）。

原始数据类型包括：布尔值、数值、字符串、`null`、`undefined` 以及 [ES6 中的新类型 `Symbol`](http://es6.ruanyifeng.com/#docs/symbol)。

下面主要介绍**前五种**原始数据类型在 TypeScript 中的应用。

- **布尔值**  在 TypeScript 中，使用 `boolean` 定义布尔值类型

  ```typescript
  let isDone: boolean = false
  ```

  注意，使用构造函数 `Boolean` 创造的对象**不是**布尔值：

  ```typescript
  let createdByNewBoolean: boolean = new Boolean(1)
  // new Boolean() 返回的是一个 Boolean 对象
  let createdByBoolean: boolean = Boolean(1)
  // 直接调用 Boolean 也可以返回一个 boolean 类型
  ```

  在 TypeScript 中，`boolean` 是 JavaScript 中的基本类型，而 `Boolean` 是 JavaScript 中的构造函数。

- **数值** 使用 `number` 定义数值类型

  ```typescript
  let decLiteral: number = 6;
  let hexLiteral: number = 0xf00d;
  // ES6 中的二进制表示法
  let binaryLiteral: number = 0b1010;
  // ES6 中的八进制表示法
  let octalLiteral: number = 0o744;
  let notANumber: number = NaN;
  let infinityNumber: number = Infinity;
  ```

  编译结果：

  ```js
  var decLiteral = 6;
  var hexLiteral = 0xf00d;
  // ES6 中的二进制表示法
  var binaryLiteral = 10;
  // ES6 中的八进制表示法
  var octalLiteral = 484;
  var notANumber = NaN;
  var infinityNumber = Infinity;
  ```

  其中 `0b1010` 和 `0o744` 是 [ES6 中的二进制和八进制表示法](http://es6.ruanyifeng.com/#docs/number#二进制和八进制表示法)，它们会被编译为十进制数字。

- **字符串** 使用 `string` 定义字符串类型

  ```typescript
  let myName: string = 'Tom';
  let myAge: number = 25;
  // 模板字符串
  let sentence: string = `Hello, my name is ${myName}.
  I'll be ${myAge + 1} years old next month.`;
  ```

  其中 ` ` ` 用来定义 ES6 中的模板字符串，`${expr}` 用来在模板字符串中嵌入表达式。
  
- **空值** JavaScript 没有空值（Void）的概念，在 TypeScript 中，可以用 `void` 表示没有任何返回值的函数

  ```typescript
  function alertName(): void {
      alert('My name is Tom');
  }
  ```

   声明一个 `void` 类型的变量没有什么用，因为你只能将它赋值为 `undefined` 和 `null`

  ```typescript
  let unusable: void = undefined
  ```

  与 `void` 的区别是，`undefined` 和 `null` 是所有类型的子类型。也就是说 `undefined` 类型的变量，可以赋值给 `number` 类型的变量，而 `void` 类型的变量不能赋值给 `number` 类型的变量

  ```typescript
  let u: undefined = undefined;
  let n: null = null
  // 这样不会报错
  let num: number = undefined;
  let num: number = u;
  ```

## 24、 搭建第一个 Vite 项目

### 24.1 初始化项目

```shell
npm init @vitejs/app my-vue-app --template vue
yarn create @vitejs/app my-vue-app --template vue
# 然后
cd my-vue-app
yarn
yarn dev
```

vite支持的预定义模板有：

- `vanilla`
- `vue`
- `vue-ts`
- `react`
- `react-ts`
- `preact`
- `preact-ts`
- `lit-element`
- `lit-element-ts`

### 24.2 添加sass支持

```shell
yarn add sass
```

### 24. 3 安装 router、vuex组件

```
yarn add vue-router@4.0
yarn add vuex@4.0
```

## 25、Pinia的使用

### 25.1 安装

```bash
yarn add pinia@next
```

创建根`pinia` 对象(the root store)，并传递给 `app`。

```js
import { createPinia } from 'pinia'
app.use(createPinia())
```

### 25.2 什么是存储对象（Store）

存储对象是存储应用程序**状态**和**业务逻辑**的实体对象，换句话说，它存储着全局状态。它有点儿像一个后台组件，所有任都可以读取和写入。存储对象包含三个概念：`state`、`getters` 和 `actions`，等效于组件的`data`、`computed`和`methods`。

### 25.3 什么时候用存储对象？

存储对象应该包含贯穿整个应用的状态数据，这些数据会在多个地方被使用。另一方面，存储对象应该避免保存某一个页面某个组件的本地状态数据。

### 25.4 核心概念

#### 25.4.1 定义存储对象（Store）

存储对象使用 `defineStore()`函数定义，它需要定义一个唯一的 `id` 属性。

```js
import { defineStore } from 'pinia'

// useStore could be anything like useUser, useCart
export const useStore = defineStore({
  // unique id of the store across your application
  id: 'storeId',
})
```

##### 使用存储对象

只有当 `useStore()` 函数被调用时，`Store`对象才被创建和使用。你可以定义很多`Store`对象，甚至在不同的文件中分别定义`Store`对象。

不能在`setup()`函数外调用`useStore()`函数，下面是错误的例子：

```js
import { createRouter } from 'vue-router'
const router = createRouter({
  // ...
})

// ❌ Depending on where you do this it will fail
const store = useStore()
router.beforeEach((to, from, next) => {
  if (store.isLoggedIn) next()
  else next('/login')
})
```

一定要确保应用加载（`app.mount()` 或者 `new Vue()`）后再调用 `useStore()`函数：

```js
router.beforeEach((to) => {
  // ✅ This will work because the router starts its navigation after pinia is installed
  const store = useStore()

  if (to.meta.requiresAuth && !store.isLoggedIn) return '/login'
})
```

一旦存储对象被实例化后，就可以访问定义在`state`、`getters` 和 `actions`中的任意属性。

`Store`对象是`reactive`对象，getters不用使用`.value`来访问，页不能解构它：

```js
    const store = useStore()
    // ❌ This won't work because it breaks reactivity
    // it's the same as destructuring from `props`
    const { name, doubleCount } = store
```

#### 25.4.2 状态（State）

状态是存储对象的核心部分。通常state被定义为一个函数，返回值为状态的初始值，这样Pinia即可以在服务端运行也可以在客户端执行。

```js
import { defineStore } from 'pinia'

const useStore = defineStore({
  id: 'storeId',
  // can also be defined with an arrow function if you prefer that syntax
  state() {
    return {
      // all these properties will have their type inferred automatically
      counter: 0,
      name: 'Eduardo',
      isAdmin: true,
    }
  },
})
```

##### 访问 `state`

可以直接读写`state`的值：

```js
const store = useStore()
store.counter++
```

##### 重置  `state`

调用`$reset()`方法可以重置`state`为初始值。

```js
const store = useStore()
store.$reset()
```

##### 使用`option API`

如果不使用composition API，可以使用 `computed`, `methods`, ...，也可以使用`mapState()`将状态属性映射为只读的计算属性。

```js
import { mapState } from 'pinia'

export default {
  computed: {
    // gives access to this.counter inside the component
    // same as reading from store.counter
    ...mapState(useStore, ['counter'])
    // same as above but registers it as this.myOwnName
    ...mapState(useStore, {
      myOwnName: 'counter',
      // you can also write a function that gets access to the store
      double: store => store.counter * 2,
      // it can have access to `this` but it won't be typed correctly...
      magicValue(store) {
        return store.someGetter + this.counter + this.double
      },
    }),
  },
}
```

##### 修改`state`

如果要修改状态的属性值，例如通过`form`表单，可以使用`mapWritableState()`。

```js
import { mapWritableState } from 'pinia'

export default {
  computed: {
    // gives access to this.counter inside the component and allows setting it
    // this.counter++
    // same as reading from store.counter
    ...mapWritableState(useStore, ['counter'])
    // same as above but registers it as this.myOwnName
    ...mapWritableState(useStore, {
      myOwnName: 'counter',
    }),
  },
}
```

##### 转换状态 `Mutating state`

不同于直接调用 `store.counter++` 修改 `store`，可以调用 `$patch`方法一次修改多个属性值。

```js
store.$patch({
  4counter: store.counter + 1,
  name: 'Abalam',
})
```

一些转换（mutations）很难用这样的语法实现：例如集合的修改（pushing、removing、splicing）需要创建新的集合。正因为这样， `$patch`方法也可以接受一个实现这类逻辑的函数：

```js
cartStore.$patch((state) => {
  state.items.push({ name: 'shoes', quantity: 1 })
  state.hasChanged = true
})
```

##### 替换状态（Replacing the `state`）

可以直接将`$state`属性替换成新的数据对象。

```js
store.$state = { counter: 666, name: 'Paimon' }
// or replace the whole state of your application by changing the state of the pinia instance
pinia.state.value = {}
```

##### 监视 （Watching the `state`）

```js
watch(
  pinia.state,
  (state) => {
    // persist the whole state to the local storage whenever it changes
    localStorage.setItem('piniaState', JSON.stringify(state))
  },
  { deep: true }
)
```

或者监视某一具体值：

```js
watch(
  () => pinia.state.value.cart,
  (cartState) => {
    // persist the whole state to the local storage whenever it changes
    localStorage.setItem('cart', JSON.stringify(cartState))
  },
  { deep: true }
)
```

```js
import { defineStore } from 'pinia'

const useCartStore = defineStore({
  // ...
})

const cartStore = useCartStore()

// watch the whole state of the cart
watch(
  () => cartStore.$state,
  () => {
    // do something
  },
  { deep: true }
)

// you can also watch a getter
watch(
  () => cartStore.totalAmount,
  () => {
    // do something
  }
)
```

#### 25.4.3 获取器 （`Getters`）

获取器等效于状态的计算值，定义在`defineStore()`函数的`getters`属性中。Getters接受`state`作为第一个参数，这里鼓励使用箭头函数定义。

```js
export const useStore = defineStore({
  id: 'main',
  state: () => ({
    counter: 0,
  }),
  getters: {
    doubleCount: (state) => state.counter * 2,
  },
})
```

大部分情况下，获取器依赖于state的值，也可以使用其它getters来定义，可以使用`this`来获取`state`对象。在Typescript中，getters需要定义返回类型。由于Typescript的一些限制，箭头函数和this调用不受影响。

```typescript
export const useStore = defineStore({
  id: 'main',
  state: () => ({
    counter: 0,
  }),
  getters: {
    // automatically infers the return type as a number
    doubleCount(state) {
      return state.counter * 2
    },
    // the return type **must** be explicitly set
    doublePlusOne(): number {
      // autocompletion and typings for the whole store
      return this.counter * 2 + 1
    },
  },
})
```

可以通过`store`实例直接访问获取器：

```vue
<template>
  <p>Double count is {{ store.doubleCount }}</p>
</template>

<script>
export default {
  setup() {
    const store = useStore()

    return { store }
  },
}
</script>
```

访问其它存储对象的getters

```js
import { useOtherStore } from './other-store'

export const useStore = defineStore({
  id: 'main',
  state: () => ({
    // ...
  }),
  getters: {
    otherGetter(state) {
      const otherStore = useOtherStore()
      return state.localData + otherStore.data
    },
  },
})
```

##### 在`setup()`中使用

```js
export default {
  setup() {
    const store = useStore()

    store.counter = 3
    store.doubleCount // 6
  },
}
```

##### 在options API中使用

```js
import { mapState } from 'pinia'

export default {
  computed: {
    // gives access to this.doubleCounter inside the component
    // same as reading from store.doubleCounter
    ...mapState(useStore, ['doubleCount'])
    // same as above but registers it as this.myOwnName
    ...mapState(useStore, {
      myOwnName: 'doubleCounter',
      // you can also write a function that gets access to the store
      double: store => store.doubleCount,
    }),
  },
}
```

#### 25.4.4 行为（`Actions`）

行为相当于组件的`methods`，通常用来实现业务逻辑。

```js
export const useStore = defineStore({
  id: 'main',
  state: () => ({
    counter: 0,
  }),
  actions: {
    increment() {
      this.counter++
    },
    randomizeCounter() {
      this.counter = Math.round(100 * Math.random())
    },
  },
})
```

`actions`可以是异步的，可以在定义中使用`await`访问API请求，甚至其它的`actions`。

```js
import { mande } from 'mande'
const api = mande('/api/users')
export const useUsers = defineStore({
  id: 'users',
  state: () => ({
    data: userData,
    // ...
  }),

  actions: {
    async registerUser(login, password) {
      try {
        this.userData = await api.post({ login, password })
        showTooltip(`Welcome back ${this.userData.name}!`)
      } catch (error) {
        showTooltip(error)
        // let the form component display the error
        return error
      }
    },
  },
})
```

Actions调用和methods一样：

```js
export default defineComponent({
  setup() {
    const main = useMainStore()
    // call the action as a method of the store
    main.randomizeCounter()

    return {}
  },
})
```

##### 在`setup()`中使用

```js
export default {
  setup() {
    const store = useStore()

    store.randomizeCounter()
  },
}
```

##### 在options API中使用

```js
import { mapActions } from 'pinia'

export default {
  methods: {
    // gives access to this.increment() inside the component
    // same as calling from store.increment()
    ...mapActions(useStore, ['increment'])
    // same as above but registers it as this.myOwnName()
    ...mapActions(useStore, { myOwnName: 'doubleCounter' }),
  },
}
```

##### 监视 `actions`

使用`store.$onAction()`监视 `actions`及其返回值。

```js
const unsubscribe = someStore.$onAction(
  ({
    name, // name of the action
    store, // store instance, same as `someStore`
    args, // array of parameters passed to the action
    after, // hook after the action returns or resolves
    onError, // hook if the action throws or rejects
  }) => {
    // a shared variable for this specific action call
    const startTime = Date.now()
    // this will trigger before an action on `store` is executed
    console.log(`Start "${name}" with params [${args.join(', ')}].`)

    // this will trigger if the action succeeds and after it has fully run.
    // it waits for any returned promised
    after((result) => {
      console.log(
        `Finished "${name}" after ${
          Date.now() - startTime
        }ms.\nResult: ${result}.`
      )
    })

    // this will trigger if the action throws or returns a promise that rejects
    onError((error) => {
      console.warn(
        `Failed "${name}" after ${Date.now() - startTime}ms.\nError: ${error}.`
      )
    })
  }
)
// manually remove the listener
unsubscribe()
```

默认情况下，`actions`监听者属于在`setup()`中初始化 `store`的组件，也就是说，当该组件卸载（unmounted）的时候，监听也被移除。

#### 25.4.5 插件（`Plugins`）

Pinia 存储对象可以借助于底层API而进行扩展：

- 增加新的属性  Add new properties to stores
- 增加新的选项  Add new options when defining stores
- 增加新的方法  Add new methods to stores
- 包装已存在的方法  Wrap existing methods
- 修改甚至取消行为  Change or even cancel actions
- 实现外部请求  Implement side effects like local storage
- Apply **only** to specific stores

用 `pinia.use()`来添加Plugins：

```js
import { createPinia } from 'pinia'
// add a property named `secret` to every store that is created after this plugin is installed
// this could be in a different file
function SecretPiniaPlugin() {
  return { secret: 'the cake is a lie' }
}
const pinia = createPinia()
// give the plugin to pinia
pinia.use(SecretPiniaPlugin)

// in another file
const store = useStore()
store.secret // 'the cake is a lie'
```

##### 介绍

Pinia插件是一个函数，返回添加到store对象的属性。它有一个选填参数`context`：

```js
export function myPiniaPlugin(context) {
  context.pinia // the pinia created with `createPinia()`
  context.app // the current app created with `createApp()` (Vue 3 only)
  context.store // the store the plugin is augmenting
  context.options // the options object defining the store passed to `defineStore()`
  // ...
}
```

```js
const sharedRef = ref('shared')
pinia.use(({ store }) => {
  // each store has its ‎个人‎`hello` property
  store.hello = ref('secret')
  // it gets automatically unwrapped
  store.hello // 'secret'

  // all stores are sharing the value `shared` property
  store.shared = sharedRef
  store.shared // 'shared'
})
```

#####  增加新的`state`

```js
const globalSecret = ref('secret')
pinia.use(({ store }) => {
  // `secret` is shared among all stores
  store.$state.secret = globalSecret
  store.secret = globalSecret
  // it gets automatically unwrapped
  store.secret // 'secret'

  // we need to check if the state has been added yet because of
  // the limitation mentioned during the introduction
  if (!store.$state.hasOwnProperty('hasError')) {
    // Each store has its own `hasError`
    store.$state.hasError = ref(false)
  }
  // this one must always be set
  store.hasError = toRef(store.$state, 'hasError')
})
```

##### 增加新的扩展属性

当从第三方的库，或者简单对象（非reactive）中获取数据添加到Pinia存储时，需要先使用`markRaw()`函数：

```js
import { markRaw } from 'vue'
// adapt this based on where your router isj
import { router } from './router'

pinia.use(({ store }) => {
  store.router = markRaw(router)
})
```

##### 调用`$subscribe`避免重复订阅

```js
let isRegistered
pinia.use(({ store }) => {
  if (!isRegistered) {
    store.$subscribe(() => {
      // react to store changes
    })
    isRegistered = true
  }
})
```

##### TypeScript

######  Typing plugins

```typescript
import { PiniaPluginContext } from 'pinia'

export function myPiniaPlugin(context: PiniaPluginContext) {
  // ...
}
```

###### Typing new store properties

```typescript
import 'pinia'

declare module 'pinia' {
  export interface PiniaCustomProperties {
    hello: string
  }
}
```

```typescript
pinia.use(({ store }) => {
  store.hello = 'Hola'
  // @ts-expect-error: this will still add a string because refs get unwrapped
  store.hello = ref('Hola')
})
```

###### Typing new state

```typescript
import 'pinia'

declare module 'pinia' {
  export interface PiniaCustomStateProperties<State> {
    hello: string
  }
}
```



## 99、待学习解决问题汇总

### 99.1 组件的受控模式和非受控模式

什么叫组件的受控模式和非受控模式？

如果你也想写一个组件库，记得每个输入类的组件都应该有受控和非受控模式。[1]

### 99.2 组件的API设计

[1]: https://www.zhihu.com/question/463736268 "问题来源地址"