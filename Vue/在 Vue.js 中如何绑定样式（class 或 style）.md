# 在 Vue.js 中如何绑定样式（class 或 style）

转自：https://www.jianshu.com/p/db4c7e7b7f2c

在数据绑定中，最常见就是动态绑定元素的 class 或内联样式 style 咯，它们也是 HTML 的属性，所以可以使用 v-bind 指令 。

1 绑定 class
==========

1.1 对象语法
--------

使用 `v-bind:class` 属性，实现动态切换 class。

html：

    <div id="app">
        <div :class="{'hidden':isHidden}">看得见我嘛</div>
    </div>


js：

    <script>
        var app = new Vue({
            el: '#app',
            data: {
                isHidden:true
            }
        });
    </script>


输出结果：

> <div class="hidden">看得见我嘛</div>

**注意：** `:class` 是 `v-bind:class` 的简写形式。

示例中的类名 hidden 依赖于数据 isHidden ，当 isHidden 为 true 时， div 就会拥有类名 hidden，为 false 时就没有该类名 。

**我们也可以一次传入多个属性，而且 `:class` 可以和常规的 class 同时使用。**

html：

    <div id="app">
        <div class="strong" :class="{'hidden':isHidden,'bigger':isBigger}">看得见我嘛</div>
    </div>


js：

    <script>
        var app = new Vue({
            el: '#app',
            data: {
                isHidden: false,
                isBigger: true
            }
        });
    </script>


输出结果：

> <div class="strong bigger">看得见我嘛</div>

当 `:class` 中的某个属性值为 true 时，就会加载对应的类名。

**当 `:class` 的表达式过长或逻辑复杂时，我们可以通过计算属性来绑定。**

html：

    <div id="app2">
        <div :class="customClasses"></div>
    </div>


js：

    var app2 = new Vue({
        el: '#app2',
        data: {
            isHidden: false,
            isBigger: true
        },
        computed: {
            customClasses: function () {
                return {
                    display: !this.isHidden,
                    'bigger-text': !this.isHidden && this.isBigger
                }
            }
        }
    });


输出结果：

> <div class="display bigger-text"></div>

**注意：**如果样式名称带有 `-`，那么必须加上单引号（示例中的 `bigger-text`）才能被正确识别。

1.2 数组语法
--------

也可以使用数组语法，给 `:class` 绑定一个 class 数组。

html：

    <div id="app">
        <div :class="[strongerClass,biggerClass]">号外！号外</div>
    </div>


js：

    <script>
        var app = new Vue({
            el: '#app',
            data: {
                strongerClass:'strong',
                biggerClass:'bigger'
            }
        });
    </script>


输出结果：

> <div class="strong bigger">号外！号外</div>

**数组中的元素也可以使用三元表达式来计算**

html：

    <div id="app2">
        <div :class="[isStrong?strongerClass:'']">号外！号外</div>
    </div>


js：

    var app2 = new Vue({
        el: '#app2',
        data: {
            isStrong:true,
            strongerClass:'strong'
        }
    });


输出结果：

> <div class="strong">号外！号外</div>

**当需要设置的 class 很多时，我们可以在数组语法的基础上使用对象语法，从而简化表达式，让代码变得更易维护。**

html：

    <div id="app3">
        <div :class="[{'strong':isStrong}]">号外！号外</div>
    </div>


js：

    var app3 = new Vue({
        el: '#app3',
        data: {
            isStrong:true
        }
    });


输出结果：

> <div class="strong">号外！号外</div>

**也可以在 computed 或 methods 中返回需要设定的数组。**

html：

    <div id="app4">
        <div :class="btnClass">号外！号外</div>
    </div>


js：

    var app4 = new Vue({
        el: '#app4',
        data: {
            size: 'small',
            isGreen: true
        },
        computed: {
            btnClass: function () {
                return [
                    'btn',
                    {
                        ['btn-' + this.size]: this.size !== '',
                        ['btn-green']: this.isGreen
                    }
                ];
            }
        }
    });


输出结果：

> <div class="btn btn-small btn-green">号外！号外</div>

**注意：** 上述示例使用了 ECMAScript 6 语法，所以在 webStorm 中需要进行设置（Setting → JavaScript → 选择 ECMAScript 6 ）否则会出现红色波浪线哦：

![](//upload-images.jianshu.io/upload_images/3386108-11ff5d0d030f439e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/879/format/webp)

* * *

在业务中会经常会利用计算属性为元素动态设置类名，尤其是在编写可复用的组件时。所以在开发过程中，如果表达式较长或者逻辑较为复杂，建议优先使用计算属性哦O(∩_∩)O~

1.3 应用于组件
---------

如果直接在自定义组件中使用 `class` 或 `:class`，那么样式规则就会直接应在这个组件的根元素上。

html：

    <div id="app">
        <text-component :class="{'isStrong':isStrong}"></text-component>
    </div>


js：

    <script>
        Vue.component('text-component', {
            template: '<p class="content">不懂基因测序的学霸不是好的人工智能公司 CEO</p>'
        });
        var app = new Vue({
            el: '#app',
            data: {
                isStrong: true
            }
        });
    </script>


css：

    <style type="text/css">
        .isStrong{font-weight: bold}
    </style>


渲染后代码：

> <p class="content isStrong">不懂基因测序的学霸不是好的人工智能公司 CEO</p>

[效果](https://jsfiddle.net/deniro/zdrw82jc/)：  

![](//upload-images.jianshu.io/upload_images/3386108-7b8ba9f5366e0e59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/362/format/webp)

* * *

这种方式仅适用于把样式应用于自定义组件的根元素。如果需要给自定义组件中的子元素设置样式，我们可以使用组件的 props 来实现。

2 绑定内联样式
========

也可以使用 `v-bind:style` 或 `:style` 直接给 HTML 元素绑定样式，它也有对应的对象语法与数组语法。

html：

    <div id="app">
        <div :style="border">马斯克太空网计划扩大 FCC已允许1.2万颗卫星入轨</div>
    </div>


js：

    <script>
        var app = new Vue({
            el: '#app',
            data: {
                border:{
                    border:'1px solid #00F',
                    textShadow:'0 0 .3em gray'
                }
            }
        });
    </script>


因为 JS 属性不支持短横分隔命名，所以我们这里使用 CSS 也支持的驼峰命名法。

渲染后代码：

> <div style="border: 1px solid rgb(0, 0, 255); text-shadow: gray 0px 0px 0.3em;">马斯克太空网计划扩大 FCC已允许1.2万颗卫星入轨</div>

[效果](https://jsfiddle.net/deniro/kg2Lph0e/)：

![](//upload-images.jianshu.io/upload_images/3386108-8223040d334fec0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/395/format/webp)