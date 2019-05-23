

#  在 Vue.js 中如何实现组件间通信

1 用法
====

假设父组件的模板包含子组件，我们可以通过 props 来正向地把数据从父组件传递给子组件。props 可以是字符串数组，也可以是对象。

html：

    <div id="app">
        <deniro-component message="＂嫦娥四号＂成功发射升空 飞向月球背面要登陆"></deniro-component>
    </div>


js：

    Vue.component('deniro-component', {
        props: ['message'],
        template:
            '<div>{{message}}</div>'
    });


​    
    var app = new Vue({
        el: '#app',
        data: {}
    });


渲染结果：

    <div id="app"><div>＂嫦娥四号＂成功发射升空 飞向月球背面要登陆</div></div>


组件中的 props 与 data 函数的区别是：props 中的数组来自于父级，而 data 函数是组件内定义的数据，所以它的作用域是组件本身。它们都可以在 template、computed 以及 methods 中使用。

如果需要传递多个数据，直接在 props 数组中定义即可。

**因为 HTML 不区分大小写，所以当使用 DOM 模板时，建议直接使用短横分隔（kebab-case）的命名方式来定义 props 参数名称。**

html：

    <div id="app2">
        <deniro-component2 special-message="微软向谷歌投降重构Edge 但IE浏览器为何将长存"></deniro-component2>
    </div>


js：

    Vue.component('deniro-component2', {
        props: ['specialMessage'],
        template:
            '<div>{{specialMessage}}</div>'
    });


​    
    var app2 = new Vue({
        el: '#app2',
        data: {}
    });


**注意：**如果使用的是字符串模板，那么可以忽略这个限制。

**有时候，传递过来的数据来自于父级的动态数据，这是我们可以使用 `v-bind` 来动态绑定 props 中的数据。**

html：

    <div id="app3">
        <input type="text" v-model="parentMessage">
        <deniro-component3 :message="parentMessage"></deniro-component3>
    </div>


js：

    Vue.component('deniro-component3', {
        props: ['message'],
        template:
            '<div>{{message}}</div>'
    });


​    
    var app3 = new Vue({
        el: '#app3',
        data: {
            parentMessage:''
        }
    });


效果：

这里我们使用 v-model 绑定了父级数据 parentMessage。所以当在输入框中键入的内容，也会同步传递给子组件。

**注意：如果不使用 `v-bind` 来传递数字、布尔对象、数组或者对象，那么传递的仅仅是字符串！**

html：

    <div id="app4">
        <deniro-component4 :message="['a','b','c']"></deniro-component4>
        <deniro-component4 message="['a','b','c']"></deniro-component4>
    </div>


js：

    Vue.component('deniro-component4', {
        props: ['message'],
        template:
            '<div>参数类型：{{typeof message}}；参数长度：{{message.length}}</div>'
    });


​    
    var app4 = new Vue({
        el: '#app4',
        data: {}
    });


输出结果：

> 参数类型：object；参数长度：3  
> 参数类型：string；参数长度：13

2 改变单向数据流
=========

Vue2.x 中，通过 props 传递的数据是单向的，也就是说，如果父组件中的数据发生变化，那么也会影响到子组件中的数据。

有时候，因为业务场景的要求，我们需要改变这种单向数据流的设计。

2.1 作为初始值
---------

子组件把父组件传递过来的值作为初始值，然后在自己的作用域内随意修改与使用它。这可以通过在子组件的 data 内定义一个新数据，用于引用父组件中的参数。

html：

    <div id="app5">
        <deniro-component5 :init-value="0"></deniro-component5>
    </div>


js：

    Vue.component('deniro-component5', {
        props: ['initValue'],
        template:
            '<div>初始值：{{initValue}}；<button @click="counter++">{{counter}}</button></div>',
        data:function () {
            return {
                counter:this.initValue
            }
        }
    });
    
    var app5 = new Vue({
        el: '#app5',
        data: {}
    });


效果：

组件中定义了 counter，它被初始化为传入的 initValue，初始化之后，counter 就与 initValue 无关啦O(∩_∩)O~

2.2 转换后传入
---------

有时候，我们必须把 props 中的值转换后传入子组件，这可以通过计算属性来实现。

html：

    <div id="app6">
        <deniro-component6 :border="5" :width="200"></deniro-component6>
    </div>


js：

    Vue.component('deniro-component6', {
        props: ['border', 'width'],
        template:
            '<div :style="style">一个算法通吃三大棋类</div>',
        computed: {
            style: function () {
                return {
                    border: this.border + 'px solid red',
                    width: this.width + 'px'
                }
            }
        }
    });
    
    var app6 = new Vue({
        el: '#app6',
        data: {}
    });


​    

效果：

**注意：**在 JS 中，对象与数组是引用类型，即子组件中定义的变量如果赋值的是这些类型，那么改变这些变量的值，是会影响到父组件的！

3 验证
====

我们自定义的组件，如果需要提供给第三方使用，那么最好进行参数验证，这就需要用到对象写法。如果验证失败，那么会在浏览器的控制台输出警告日志。

html：

    <div id="app7">
        <deniro-component7 :a="'str'"></deniro-component7>
    </div>


js：

    <script src="https://cdn.bootcss.com/vue/2.2.2/vue.js"></script>
    <script>
        Vue.component('deniro-component7',{
           props:{
               a:Number
           },
            template:
                '<div >{{a}}</div>',
        });
    
        var app7 = new Vue({
            el: '#app7',
            data: {}
        });
    </script>


**注意：**这里引入了开发版的 vue.js（生产版一般是 vue.min.js）。

控制台输出：

其它示例：

    Vue.component('deniro-component8', {
        props: {
            //必须是数字类型
            a: Number,
            //数字类型或者字符串类型
            b: [Number, String],
            //布尔类型，默认值为 true
            c: {
                type: Boolean,
                default: true
            },
            //字符串类型，必须传值
            d: {
                type: String,
                required: true
            },
            //自定义
            e: {
                validator: function (v) {
                    return v > 1;
                }
            }
        }
    });


type 可以为这些类型：String、Number、Boolean、Object、Array、Function。

* * *

以上示例 [DEMO](https://jsfiddle.net/deniro/Lqcxgy9j/)