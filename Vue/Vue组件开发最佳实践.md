# [Vue组件开发最佳实践](https://www.cnblogs.com/ytu2010dt/p/6523586.html)



看了老外的一篇关于[组件开发的建议](https://pablohpsilva.github.io/vuejs-component-style-guide/#/)(强烈建议阅读英文原版)，感觉不错翻译一下加深理解。

这篇文章制定一个统一的规则来开发你的vue程序，以至于达到一下目的。 

1. 让开发者和开发团队更容易发现一些事情。

2. 让你更好的利用你的IDE。

3. 让你更加容易的使用打包工具

4. 让你的代码更容易碎片化以达到复用的目的。

基于模块开发
------

用一些功能单一的小模块来组织你的应用

### Why?

对于你自己和你团队的人来说较小的模块更容易看懂 维护 复用和调试。

### How?

每个组件应该保持单一、独立、可复用、可测试。把你很大的组件拆分成功能单一的小组件，尽量不让一个组件的代码超过100行，保持组件独立。最好是写个组件应用的小demo。

组件命名
----

组件命名应该遵从以下几点原则

*   有意义: 名字不要太详细，也不要太抽象。
*   短: 名字最好是2-3个单词。
*   可读的:容易让人能读出来以便我们可以更容易的讨论它。

vue组件也应该遵循以下原则

*   遵从元素命名规范 包括连字符，不要使用保留字
*   为了在其他项目中复用，应该以某个模块名字作为命名空间

### Why?

*   为了让我们更好地通过名字来交流这个组件，这个组件必须 短 有意义 可读

### How?

    <!-- 建议这样 -->
    <app-header></app-header>
    <user-list></user-list>
    <range-slider></range-slider>
    
    <!-- 避免这样 -->
    <btn-group></btn-group> <!-- 足够短但是不容易发音，使用`button-group`代替 -->
    <ui-slider></ui-slider> <!-- 所有的组件都是ui元素，所以这样命名无意义 -->
    <slider></slider> <!--不是我们适应的风格 -->


保证组件模板中的表达式简短
-------------

vue的行内式表达式都是js。当着这些js很有效，但是也很复杂。因此你应该保持行内表达式简洁

### Why?

*   复杂的行内表达式可读性差
*   这些行内表达式不能任意地方复用，这样会导致代码冗余
*   IDE不支持行内式js的语法校验

### How?

把复杂的语法移动到methods或者计算属性中

    <!-- recommended -->
    <template>
        <h1>
            {{ `${year}-${month}` }}
        </h1>
    </template>
    <script type="text/javascript">
      export default {
        computed: {
          month() {
            return this.twoDigits((new Date()).getUTCMonth() + 1);
          },
          year() {
            return (new Date()).getUTCFullYear();
          }
        },
        methods: {
          twoDigits(num) {
            return ('0' + num).slice(-2);
          }
        },
      };
    </script>
    
    <!-- avoid -->
    <template>
        <h1>
            {{ `${(new Date()).getUTCFullYear()}-${('0' + ((new Date()).getUTCMonth()+1)).slice(-2)}` }}
        </h1>
    </template>


保证组件的props简单
------------

尽管vue支持通过props传递复杂的object，但是你要尽量保持props传递的数据简单,尽量只传递基本数据类型(strings, numbers, booleans)

### Why?

*   简洁的props让你的接口api比较简单
*   props只传递简单类型数据和函数，让我们组件的api看起来更像原生html的属性。
*   props只传递简单类型数据，让其他开发者容易明白传什么参数。
*   props传递复杂数据类型，让你的组件很难重构，也会造成代码冗余。

### How?

vue component 只传递简单数据类型或者函数如下

```html
<!-- recommended -->
<range-slider
  :values="[10, 20]"
  min="0"
  max="100"
  step="5"
  :on-slide="updateInputs"
  :on-end="updateResults">
</range-slider>

<!-- avoid -->
<range-slider :config="complexConfigObject"></range-slider>
```

对你组件的props做一些限制
---------------

vue 组件中props就是api,健壮且可预测的api让别人更容易使用你的组件  
组件的props通过html属性来编写，这些值可以是vue的简答字符串（:attr="value" or v-bind:attr="value"）也可以不写。你应该对props做一些限制

### Why?

对props做一些限制保证你的组件正常工作，即使别人没有按照你预想的方式调用你的组件。

### How?

* 属性设置默认值
* 属性设置数据类型校验
* 使用组件之前检查props是否存在

    <template>
      <input type="range" v-model="value" :max="max" :min="min">
    </template>
    <script type="text/javascript">
      export default {
        props: {
          max: {
            type: Number, // [1*] This will validate the 'max' prop to be a Number.
            default() { return 10; },
          },
          min: {
            type: Number,
            default() { return 0; },
          },
          value: {
            type: Number,
            default() { return 4; },
          },
        },
      };
    </script>

将组件设定为this
----------

在组件内部上下文中,this指的是vue组件实例，因此在其他上下文中使用它的时候保证'this'在组件中可以使用  
换句话说,不要这样写 `const self = this;`

### Why?

*   通过把`this`分配给会改变名字的组件，告诉开发者this是一个组件实例。

### How?

组件结构
----

让你的组件代码按照一定的顺序编写

### Why?

*   用export导出一个清晰的对象，提高代码可读性，同时让开发着统一代码结构
*   按一下顺序排列,让代码容易被找到 (name; extends; props, data and computed; components; watch and methods; lifecycle methods, etc.);
*   增加`name`属性，这样再使用vue devtools时便于开发测试。
*   按照一定的规则写css
*   按照如下顺序组织代码template-script-style

### How?

```html
<template lang="html">
    <div class="Ranger__Wrapper">
        <!-- ... -->
    </div>
</template>

<script type="text/javascript">
  export default {
        // Do not forget this little guy
    name: 'RangeSlider',
    // compose new components
    extends: {},
    // component properties/variables
    props: {
            bar: {}, // Alphabetized
            foo: {},
            fooBar: {},
        },
    // variables
    data() {},
    computed: {},
    // when component uses other components
    components: {},
    // methods
    watch: {},
    methods: {},
    // component Lifecycle hooks
    beforeCreate() {},
    mounted() {},
};
</script>

<style scoped>
  .Ranger__Wrapper { /* ... */ }
</style>
```

组件事件命名
------

vue提供的vue处理函数和表达式是严格绑定在vm上的。每个组件事件应该遵循一个良好的命名规范从而避免开发中出现的问题。

### Why?

*   开发者任意使用事件名称会导致混乱,例如用了原生的事件名。
*   随意命名事件会导致dom模板不协调。

### How?

*   事件命名按照kebab-cased(不用驼峰法)规范(例如download-success)
*   一个事件名称对应唯一的事件
*   事件名应该已动词(例如client-api-load)或名词(例如 drive-upload-success)结尾

避免使用this.$patent
----------------

vue支持组件嵌套，子组件获得父组件上下文，但是获得外部上下文违反了组件独立的规定，所有不要使用this.$patent

### Why?

*   就像其他组件一样vue组件也应该独立工作
*   如果组件依赖他的父组件那么他将难以复用。

### How？

*   通过attribute/properties将数据从父组件传递给子组件
*   在属性表达式中把在父组件中定义的会掉函数传递到子组件
*   从子组件emit事件到父组件

谨慎使用`this.$refs`
----------------

vue 支持组件通过 `this.$refs`来获得组件或者dom元素的上下文，大部分情况下这中用法应该被禁止。当你用他的时候也应该谨慎防止错误的组件api。

### Why?

*   就像其他组件一样,vue的组件应该是独立的,不能适应所有应用场景的组件是一个不好的组件
*   大部分情况下属性和事件已经足够用了

### How？

* 设计好的组件api
* 多考虑一些组件在其余业务场景下的重用
* 不要写一些特殊的代码,如果你需要些说明你需要设计一个新的组件
* 检查是否有props缺失,如果是的话补全这些缺陷。
* 检查所有的事件(event)大多数时候开发者只记得通过props实现父子组件通信吗，而忘记通过自定义事件。
* 以设计好的api和组件独立性为目的来更新你的组件
* 当props和自定义事件实在达不到目的再用`this.$refs`
* 当元素不能用数据绑定或者指令操作时，用`this.$refs`是比jquery和`document.getElement*`好一些的选择

    <!-- good, no need for ref -->
    <range :max="max"
      :min="min"
      @current-value="currentValue"
      :step="1"></range>

    <!-- good example of when to use this.$refs -->
    <modal ref="basicModal">

      <h4>Basic Modal</h4>
      <button class="primary" @click="$refs.basicModal.close()">Close</button>
    </modal>
    <button @click="​$refs.basicModal.open()">Open modal</button>

    <!-- Modal component -->
    <template>
      <div v-show="active">
        <!-- ... -->
      </div>
    </template>

    <script>
      export default {
        // ...
        data() {
            return {
                active: false,
            };
        },
        methods: {
          open() {
              this.active = true;
          },
          hide() {
              this.active = false;
          },
        },
        // ...
      };
    </script>

    <!-- avoid accessing something that could be emitted -->
    <template>
      <range :max="max"
        :min="min"
        ref="range"
        :step="1"></range>
    </template>

    <script>
      export default {
        // ...
        methods: {
          getRangeCurrentValue() {
              return this.$refs.range.currentValue;
          },
        },
        // ...
      };
    </script>

使用组件名称作为css作用域
==============

vue 组件的名字作为css根作用域类名是极好的。

### Why?

*   css在style标签加上scoped能有效的防止组件内css污染外部组件的css
*   css根作用域类名和组件名相同,让开发者容易理解他们是一个组件中的。

### How？

把组件名作为css命名空间依赖BEM和OOCSS(面向对象css)。在style标签上加scoped。加了scoped告诉vue在编译时给每个类名都加一个后缀，从而避免污染其余组件或者全局样式。

    <style scoped>
        /* recommended */
        .MyExample { }
        .MyExample li { }
        .MyExample__item { }
    
        /* avoid */
        .My-Example { } /* not scoped to component or module name, not BEM compliant */
    </style>

为你的组件写api文档
-----------

一个vue实例通过实例化应用中的组件而来。这个实例通过组件属性配置而来。如果组件要提供给其他开发者使用，这些定制的属性也就是组件的api应该写在readme.md中。

### Why?

*   文档提供给开发者一个关于组件的概要，使开发者不需要看组件的源码。这样组件比较容易让人接受和使用。
*   组件的api是使用组件需要配置项的指导。特别是对于那些只使用这个组件的开发者。
*   组件正式的文档告诉开发者当组件代码变化了怎么去做兼容
*   `README.md` 是一个文档应该先被阅读的。github等代码仓库通过README.md 来展示代码内容

### How？

给一个组件增加`README.md`

    range-slider/
    ├── range-slider.vue
    ├── range-slider.less
    └── README.md

其余还包括给你的组件写小demo，对组件做eslint代码审查。。