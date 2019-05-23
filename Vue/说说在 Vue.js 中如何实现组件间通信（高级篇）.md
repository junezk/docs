# 说说在 Vue.js 中如何实现组件间通信（高级篇）

之前说过，可以使用 props 将数据从父组件传递给子组件。其实还有其它种的通信方式，下面我们一一娓娓道来。

1 自定义事件
=======

通过自定义事件，我们可以把数据从子组件传输回父组件。子组件通过 `$emit()` 来触发事件，而父组件通过 `$on()` 来监听事件，这是典型的观察者模式。

html：

    <div id="app">
        <p>总数：{{total}}</p>
        <deniro-component @increase="setTotal"
                          @reduce="setTotal"
        ></deniro-component>
    </div>


js：

    Vue.component('deniro-component', {
        template: '\
        <div>\
        <button @click="increase">+1</button>\
        <button @click="reduce">-1</button>\
        </div>',
        data: function () {
            return {
                counter: 0
            }
        },
        methods: {
            increase: function () {
                this.counter++;
                this.$emit('increase', this.counter);
            },
            reduce: function () {
                this.counter--;
                this.$emit('reduce', this.counter);
            }
        }
    });
    
    var app = new Vue({
        el: '#app',
        data: {
            total: 0
        },
        methods: {
            setTotal: function (total) {
                this.total = total;
            }
        }
    });


效果：

  

![](//upload-images.jianshu.io/upload_images/3386108-55a0e5ae9aa36791.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/266/format/webp)

示例中有两个按钮，分别实现加 1 与减 1 操作。点击按钮后，执行组件中定义的 increase 或 reduce 方法，在方法内部，使用 `$emit` 把值传递回父组件。 `$emit` 方法的第一个参数是使用组件时定义的事件名，示例中是 `@increase` 与 `@reduce`：

    <deniro-component @increase="setTotal"
                      @reduce="setTotal"
    ></deniro-component>


这两个事件又绑定了 `setTotal` 方法，该方法修改了 total 值。 `$emit` 方法的其它参数是需要回传给父组件的参数。

也可以使用 `v-on` 加 `.native` 来监听原生事件，比如这里监听组件的点击事件：

html：

    <div id="app">
        ...
        <deniro-component ...
                          @click.native="click"
        ></deniro-component>
    </div>


js：

    ...
    var app = new Vue({
        el: '#app',
        data: {
            total: 0
        },
        methods: {
            ...
            click: function () {
                console.log("原生点击事件");
            }
        }
    });


这样，点击按钮后，就可以捕获原生的点击事件啦O(∩_∩)O~

**注意：**这里监听的是这个组件根元素的原生点击事件。

2 v-model 方式
============

也可以使用 v-model 方式来直接绑定父组件变量，把数据从子组件传回父组件。

html：

    <div id="app2">
        <p>总数：{{total}}</p>
        <deniro-component2 v-model="total"
        ></deniro-component2>
    </div>


js：

    Vue.component('deniro-component2', {
        template: '\
        <div>\
        <button @click="click">+1</button>\
        </div>',
        data: function () {
            return {
                counter: 0
            }
        },
        methods: {
            click: function () {
                this.counter++;
                this.$emit('input', this.counter);
            }
        }
    });
    
    var app2 = new Vue({
        el: '#app2',
        data: {
            total: 0
        }
    });


效果：

![](//upload-images.jianshu.io/upload_images/3386108-dce268b156a18838.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/266/format/webp)

我们使用 `v-model="total"` 直接绑定变量 total。接着在子组件中，在 `$emit` 方法传入事件名 `input`，这样 Vue.js 就会自动找到 `v-model 绑定的变量啦O(∩_∩)O~

我们也可以使用自定义事件来实现上述示例——

html：

    <div id="app3">
        <p>总数：{{total}}</p>
        <deniro-component3 @input="setTotal"
        ></deniro-component3>
    </div>


​    

js：

    Vue.component('deniro-component3', {
        template: '\
        <div>\
        <button @click="click">+1</button>\
        </div>',
        data: function () {
            return {
                counter: 0
            }
        },
        methods: {
            click: function () {
                this.counter++;
                this.$emit('input', this.counter);
            }
        }
    });
    
    var app3 = new Vue({
        el: '#app3',
        data: {
            total: 0
        },
        methods: {
            setTotal: function (total) {
                this.total = total;
            }
        }
    });


效果与上例相同。

我们还可以在自定义的表单输入组件中利用 v-model，实现数据双向绑定：

html：

    <div id="app4">
        <p>总数：{{total}}</p>
        <deniro-component4 v-model="total"
        ></deniro-component4>
        <button @click="increase">+1</button>
    </div>


js：

    Vue.component('deniro-component4', {
        props: ['value'],
        template: '<input :value="value" @input="update">+1</input>',
        data: function () {
            return {
                counter: 0
            }
        },
        methods: {
            update: function (event) {
                this.$emit('input', event.target.value);
            }
        }
    });
    
    var app4 = new Vue({
        el: '#app4',
        data: {
            total: 0
        },
        methods: {
            increase: function () {
                this.total++;
            }
        }
    });


效果：

![](//upload-images.jianshu.io/upload_images/3386108-f77fc03650378442.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/266/format/webp)

这里我们首先利用 v-model，在自定义组件中绑定了 total 变量。然后在组件内部，定义了 props 为 `['value']`，**注意这里必须为 value**，才能接收绑定的 total 变量。接着在组件模板中把接收到的 value 值（即 total 变量值），作为 `<input>` 元素的初始值，并绑定 input 事件。下一步，在 input 事件中，通过 `this.$emit('input', event.target.value)` 把 total 值传回父组件的 `<button @click="increase">+1</button>`。最后在 increase 方法中，递增 total 值。

这个示例，我们综合使用了 props 、v-model和自定义事件，实现了数据的双向绑定。

总的来说，一个具有双向绑定的 v-model 组件具有以下特征：

1.  使用 props 接收父组件的 value。
2.  子组件中拥有可以更新 value 的 HTML 元素，当更新 value 时，触发 input 事件。事件内部使用 `$emit` 将新的 value 值回传给父组件。

3 非父子组件
=======

非父子组件指的是兄弟组件或者跨多级组件。

3.1 中央事件总线
----------

我们可以创建一个空的 Vue 实例作为中央事件总线，实现非父子组件之间的通信。

html：

    <div id="app5">
        <p>监听子组件消息：{{message}}</p>
        <deniro-component5></deniro-component5>
    </div>


js：

    var bus = new Vue();
    Vue.component('deniro-component5', {
        template: '<button @click="sendMessage">发送消息</button>',
        methods: {
            sendMessage: function () {
                bus.$emit('on-message', '来自于 deniro-component5 的消息');
            }
        }
    });
    var app5 = new Vue({
        el: "#app5",
        data: {
            message: ''
        },
        mounted: function () {
            var that = this;
    
            bus.$on('on-message', function (message) {
                that.message = message;
            })
        }
    });


**注意：** 因为 `bus.$on()` 中的函数，this 指向的是本身，所以我们必须在外层定义一个 that，让它引用 mounted 对象。

效果：

![](//upload-images.jianshu.io/upload_images/3386108-ee54b902fbb5226c.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/539/format/webp)

首先创建了一个空的 Vue 实例作为中央事件总线。然后在定义的子组件绑定的 click 事件中，通过 `bus.$emit()` 发送消息。接着在初始化 app 实例的 `mounted` 函数时，使用 `bus.$on()` 方法监听消息。

这种方式可以实现组件间任意通信。我们还可以扩展 bus 实例，为它添加 data、methods、computed 等属性，这些都是公共属性，可以共用。所以在此可以放置需要共享的信息，比如用户登陆昵称等。使用时只需要初始化一次 bus 即可，所以在单页面富客户端中应用广泛。

如果项目较大，那么可以使用具有状态管理的 vuex 哦O(∩_∩)O~

3.2 父子链
-------

子组件可以使用 `this.$parent` 来访问父组件实例；而父组件可以使用 `this.$children` 来访问它的所有子组件实例。这些方法可以递归向上或向下，直到根实例或者叶子实例。

html：

    <div id="app6">
        <p>消息：{{message}}</p>
        <deniro-component6></deniro-component6>
    </div>


js：

    Vue.component('deniro-component6', {
        template: '<button @click="sendMessage">发送消息</button>',
        methods: {
            sendMessage: function () {
                //通过父链找到父组件，修改相应的变量
                this.$parent.message='来自于 deniro-component6 的消息';
            }
        }
    });
    var app6 = new Vue({
        el: "#app6",
        data: {
            message: ''
        }
    });


效果：

![](//upload-images.jianshu.io/upload_images/3386108-85ee60b5ec1145e7.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/539/format/webp)

**注意：**只有在万不得已的情况下，才使用父子链，实现组件间任意通信。因为这样做，会让两个组件之间紧耦合，代码变得难理解与维护。如果只是父子组件之间的通信，尽量采用 `props` 与自定义事件 `$emit` 来实现。

3.3 子组件索引
---------

如果一个组件的子组件较多且是动态渲染的场景，使用 `this.$children` 来遍历这些子组件较麻烦。这时就可以使用 `ref` 来为子组件指定索引名称，方便后续查找。

html：

    <div id="app7">
        <button @click="getChild">获取子组件实例</button>
        <deniro-component7 ref="child"></deniro-component7>
    </div>


js：

    Vue.component('deniro-component7', {
        template: '<div>deniro-component7</div>',
        data: function () {
            return {
                message: '登陆不到两周，InSight探测器意外捕捉到火星的风声'
            }
        }
    });
    var app7 = new Vue({
        el: "#app7",
        methods: {
            getChild: function () {
                //使用 $refs 来访问组件实例
                console.log(this.$refs.child.message);
            }
        }
    });


输出结果：

> 登陆不到两周，InSight探测器意外捕捉到火星的风声

**注意：**`$refs` 只在组件渲染完成后才会被赋值，而且它是非响应式的。所以只有在万不得已的情况下才使用它。

[本文示例代码](https://jsfiddle.net/deniro/5r0nezh2/)

* * *

总结如下：

| 通信方式                   | 通信方向       |
| -------------------------- | -------------- |
| `props`【推荐】            | 父组件到子组件 |
| 自定义事件 `$emit`【推荐】 | 子组件到父组件 |
| 中央事件总线【推荐】       | 组件间任意通信 |
| 父子链                     | 组件间任意通信 |
| 子组件索引                 | 父组件到子组件 |