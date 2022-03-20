# 如何使用SASS编写可重用的CSS

> 本文 **GitHub：**[github.com/qq449245884…](https://github.com/qq449245884/xiaozhi) 上已经收录，更多往期高赞文章的分类，也整理了很多我的文档，和教程资料。欢迎Star和完善，大家面试可以参照考点复习，希望我们一起有点东西。

[Sass](https://sass-lang.com/) 是一个CSS预处理程序，至今使用广泛，它之所以流行，是因为它修复了几个CSS缺陷：

Sass 也是 [Bootstrap 4](https://getbootstrap.com/) 运行的基础。这意味着为了理解如何操作引导代码而学习Sass是非常有帮助的，而不是覆盖代码(这是大多数开发人员的定制方法)。理解Sass可以更好地理解源代码级别的工具。

当使用CSS时，我们经常在一个全局环境设置，这样可能会错误地设置元素的样式。

自定义CSS(即使有CSS变量)仍然是非常冗余的。CSS不是为我们今天所拥有的那种复杂的架构设计的，我们遇到了在另一个样式表中导入一个样式表的问题，这可能会导致一个非常大的样式库，没有适当的文档可能无法理解它。

在本文中，我们将重点讨论为什么预处理程序很重要，并特别强调SASS及其将规则组合在一起的能力。使用Sass为设计现代web组件提供了一种更合理的方法。

我们还将通过演示示例来了解为什么要使用这些预处理程序，演示如何将样式划分为更小的特定组件，而不必强迫用户下载大量不需要的CSS文件。

## 安装 sass

sass 的安装方式有很多种，可以查看[这个地址](https://sass-lang.com/install)，这里我们使用 `npm` 安装：

```
npm install -g sass
复制代码
```

## CSS 存在哪些问题？

在则开始学习前端基础知识时，我们会接触到传统的 CSS，涉及到使用类或`id`之类的标识符来处理和操作HTML元素。

在使用CSS的时候，我们经常需要修改样式来实现预期的要求。 组织大型样式表确实很压力。 保持类的作用域以避免意外地设置样式有时候有的很累。

即使引入了 CSS 变量以减少声明的重复，但使用预处理器仍可以解决变量的一些问题。 例如：较长的变量名。

即使有了CSS3，我们仍然需要依赖一些技巧来设计用户界面的样式，在编写 HTML 时，嵌套和可视化层次结构样式有助于理解，但这是常规 CSS 做不到。

## CSS 预处理器是什么？

可以把预处理器看作是一个加工的机器，一端允许我们编写独特的语法，另一端，预处理器帮我们生成对应的常规的 CSS。CSS 预处理程序通常会增加一些纯CSS中不存在的特性，比如 [mixin](https://sass-lang.com/documentation/at-rules/mixin)、[嵌套选择器](http://thesassway.com/beginner/the-inception-rule)、[继承选择器](https://github.com/webplatform/webplatform.github.io/blob/master/docs/tutorials/inheritance_and_cascade/index.html)。同时也给了我们以结构化的方式来编写样式。CSS 预处理程序有 [LESS](http://lesscss.org/), [stylus](http://stylus-lang.com/), [Sass](https://sass-lang.com/), [PostCSS](https://postcss.org/)。如前所述，本文主要关注作为预处理器的 Sass。

![img](如何使用SASS编写可重用的CSS.assets/16f7d52dbf035e74)

## SASS 还是 SCSS ？

SCSS 是 Sass 3 引入新的语法，其语法完全兼容 CSS3，并且继承了 Sass 的强大功能。Sass 和 SCSS 其实是同一种东西，我们平时都称之为 Sass，两者之间不同之处有以下两点：

1.文件扩展名不同，Sass 是以“.sass”后缀为扩展名，而 SCSS 是以“.scss”后缀为扩展名

2.语法书写方式不同，Sass 是以严格的缩进式语法规则来书写，不带大括号({})和分号(;)，而 SCSS 的语法书写和我们的 CSS 语法书写方式非常类似。

先来看一个示例：

**Sass语法**：

```
$font-stack:    Helvetica, sans-serif
$primary-color: #333

body
  font: 100% $font-stack
  color: $primary-color
复制代码
```

**Scss语法**：

```
$font-stack:    Helvetica, sans-serif;
$primary-color: #333;

body {
  font: 100% $font-stack;
  color: $primary-color;
}
复制代码
```

在上面的代码示例中，我们注意到 Sass 和 SCSS 编写样式的不同。 请注意，它们都使用`$`来声明变量。

## SCSS 中的概念

#### 嵌套和作用域

当设计 HTML文件的样式时，SCSS 使我们能够在样式表中拥有相同的 HTML 视觉层次结构，这样我们就可以以一种更容易理解的方式来设计样式。例如，设计这个`index.html`的样式

```
<nav class = 'sidebar'>
  <ul>
    <li> <a> </a></li>
  </ul>
</nav>
复制代码
```

**CSS:**

```
nav ul {
  margin: 0;
  padding: 0;
  list-style: none;
}
nav li {
  display: inline-block;
}
nav a {
  display: block;
  padding: 6px 12px;
  text-decoration: none;
}
复制代码
```

**SCSS:**

```
nav {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  li { display: inline-block; }

  a {
    display: block;
    padding: 6px 12px;
    text-decoration: none;
  }
}
复制代码
```

从上面的 CSS 代码示例可以看出，我们能够推断 HTML 文件的结构，同时保持实现的简短。这样做的另一个好处是，它有助于避免拼写错误，而且还可以看到，我们已经确定了一些规则的作用域，因此这些样式只用于`nav`。

后代样式规则适用于 SCSS，例如：

```
.container{
  .left-area{
    ...
  }
}
复制代码
```

这意味着，具有`class = "left-area"` 的`container`类的所有后代都将受到规则的影响。基本的CSS 选择器仍然适用于 SCSS，比如：

** `>` 选择器**

```
.container{
  > .left-area{
   ...
 }
}
复制代码
```

现在，只有类为`container`的直接子类才会获得样式。

> css 中`“>”`是 css3 中特有的选择器**，A > B** 表示选择 A 元素的所有子B元素，与`A B`不同的是，`A B` 选择所有后代元素，为**A > B**只选择`一代`。

**父选择器(&)**

如果我们想通过添加一个类来修改一个类，我们可以使用父选择器，它主要用于添加辅助样式改变元素样式的情况，这也会起到修饰符的作用，`&` 在 scss中表示自身的意思。

```
.container{
  &.right-area{
   background-color : #0000
 }
}
复制代码
```

上述代码等同于

```
.container.right-area {
  background-color : #0000
}
复制代码
```

我们还可以使用父选择器将角色限定在另一个类中，就像这样

在代码示例中，由于父选择器的原因，`color:#fff`只适用于`.theme-dark`类。

#### 变量

通常，在 CSS 中，我们通过使用`@import`将不同的样式表链接到主 CSS 中，这意味着必须下载额外的CSS文件。

如果有一种方法可以使用 SCSS 将所有这些输入解析为一个 CSS 文件，该怎么办。 CSS 中变量的概念来自 JS 方法。

请注意，SCSS中的`@import`用于将部分内容获取到其他`SCSS`文件中，但它们不会成为 CSS 文件，名称前用`_`表示。

#### 使用 scss 变量

SCSS中的变量以美元符号`$`开头。

```
$color: #f002
.color{
  $text_color: #ddd;
  background-color: $color;
  color: $text_color;
  text-shadow:0 0 2px darken($text_color, 40%);
}
复制代码
```

从上面的代码中，我们注意到`$text_color`只能在代码块内访问。

## Mixins

SCSS 的另一个了不起的特性是它能够将可重用的样式打包在一起，并允许根据需要将样式导入到另一个样式块中，从而减少代码中的冗余。

#### 声明

创建`mixin`非常简单，只需在样式块之前添加`@mixin`和`mixin`名称，如下所示

```
@mixin {insert name} {
     //write CSS code here
}
复制代码
```

示例：

```
@mixin button {  
    font-size: 1em;  
    padding: 0.5em 1.0em;  
    text-decoration: none;  
    color: #fff;  
}
复制代码
```

由于历史原因，连字符和下划线被认为是相同的，也就是说`@mixin mixin-name { }` 和 `@mixin mixin_name { }` 是一样的。

#### 用法

要在代码块中使用`mixin`，我们必须使用`@include`，然后接`mixin`的名称。

```
.button-green {  
    @include button;  
    background-color: green;  
}
复制代码
```

这里通过`@include`调用我之前创建名为`button`的mixin模块，那么解析后的CSS就如下所示：

```
.button-green {  
    font-size: 1em;  
    padding: 0.5em 1.0em;  
    text-decoration: none;  
    color: #fff;  
    background-color: green;  
}
复制代码
```

使用`mixin`的另一种方法是使用参数，就像 JS 中的函数一样，我们可以声明一个全局变量并将其设置为`mixin`的一个参数，这里更新了上面的`button`模块，增加了名为`background`的参数并将其传递给模块。

```
@mixin button($background) {  
    font-size: 1em;  
    padding: 0.5em 1.0em;  
    text-decoration: none;  
    color: #fff;  
    background: $background;  
}
复制代码
```

注意到参数被设置为一个变量并成为`backround`属性的值。如果我们想创建一个绿色的按钮，那么就可以使用以下代码：

```
.button-green {  
    @include button(green);  
}
复制代码
```

你可能会好奇如果在定义mixin时定义了参数，但是在`@include`调用时没有传递参数会发生什么。这种情况下你会收到一个编译错误的提示。同时我相信这种情况一定不是你想看到的。你可以通过在`mixin`中定义参数的时候给它设置一个默认值，从而来避免这种错误。

```
@mixin button($background: green) {  
    font-size: 1em;  
    padding: 0.5em 1.0em;  
    text-decoration: none;  
    color: #fff;  
    background: $background;  
}
复制代码
```

现在如果你在调用的时候忘记传递参数值:

```
.button-green {  
  @include button;  
}
复制代码
```

你的代码将会使用你设置的参数默认值来解析，在这个例子中也就是`green`这个值。

```
.button-green {  
    font-size: 1em;  
    padding: 0.5em 1.0em;  
    text-decoration: none;  
    color: #fff;  
    background: green;  
}
复制代码
```

## 函数

SCSS 中的函数是 SASS 功能的重要组成部分，它们允许我们定义可在整个样式表中重用的复杂操作。有很多内置的 Sass 函数可供我们使用，查看[文档](https://sass-lang.com/documentation/at-rules/function)了解更多信息。

这里列出了一些常用的函数：

- [lighten(![color,](https://juejin.im/equation?tex=color%2C)amount)](https://sass-lang.com/documentation/modules/color#lighten)：使颜色更浅。
- [darken(![color,](https://juejin.im/equation?tex=color%2C)amount)](https://sass-lang.com/documentation/modules/color#darken)：使颜色变深。
- [adjust-hue(![color,](https://juejin.im/equation?tex=color%2C)degrees)](https://sass-lang.com/documentation/modules/color#adjust-hue)：改变颜色的色调。
- [mix(![color1,](https://juejin.im/equation?tex=color1%2C)color2, $weight)](https://sass-lang.com/documentation/modules/color#mix) ：将两种颜色根据一定的比例混合在一起，生成另一种颜色
- [hue($color)](https://sass-lang.com/documentation/modules/color#hue) ：获取颜色的色调组件。
- [saturation($color)](https://sass-lang.com/documentation/modules/color#saturation) ：获取颜色的饱和度组件。
- [lightness($color)](https://sass-lang.com/documentation/modules/color#lightness) :获取颜色的亮度组件。

我们也可以定义自己的 Sass 函数，要实现函数的声明和返回内容我们需要使用`function`和`return`两个指令，类似于其他语言中的关键字。

```
@function 函数名(形参) {
  @return;
}
复制代码
```

使用时时直接使用 函数名即可:

```
@function getWidth($w) {
  @return $w * 2;
}

.main{
  max-width: #{getWidth(20)}px;
}
复制代码
```

编译后:

```
.main {
  max-width: 40px;
}
复制代码
```

## 具有控制流的条件样式

#### if()

`if()` 是 Sass 的一个内建函数，与之相似的 `@if` 则是一个内建指令。`if()` 用来做条件判断并返回特定值，示例如下：

```
@mixin test($condition) { 
  $color: if($condition, blue, red); 
  color:$color 
}
  
.firstClass { 
  @include test(true); 
} 
 
.secondClass { 
  @include test(false); 
}
复制代码
```

编译结果：

```
.firstClass {
  color: blue; 
} 
.secondClass { 
  color: red; 
}
复制代码
```

在上例中，`if()` 函数内的三个参数分别代表：测试条件，测试成功返回值，测试失败返回值（除了 false 和 null 之外的所有测试条件都被视为测试成功）。如果使用数字作为上述示例的条件，同样会返回测试成功的值：

```
.firstClass { 
  @include test(1); 
}
复制代码
```

#### @if

`@if` 后跟一个表达式，如果表达式的结果为 `true`，则返回特定的样式，示例如下：

```
@mixin txt($weight) { 
  color: white; 
  @if $weight == bold { font-weight: bold;} 
} 

.txt1 { 
  @include txt(none); 
} 

.txt2 { 
  @include txt(bold); 
}
复制代码
```

编译结果：

```
.txt1 { 
  color: white; 
} 
.txt2 { 
  color: white; 
  font-weight: bold; 
}
复制代码
```

#### @for

`@for` 指令常用于循环输出。`@for` 有两种使用方式：`from start through end` 和 `from start to end`，两者的区别在于，前者遍历的范围是 `[start, end]`, 而后者的遍历范围是 `[start, end-1]`。在 `@for` 循环中使用一个固定变量来替代遍历到的元素。如果你想实现从大到小的遍历，只需让 `start` 大于 `end` 即可。下面是 `@for` 的一个简单示例：

```
@for $i from 1 through 4 { 
  .col-#{$i} { 
    width: 100/4 * $i + %;
  } 
}
```

使用上面的这个例子，我们可以创建一套简单的栅格系统，编译结果如下:

```
.col-1 { 
  width: 25%; 
} 
.col-2 { 
  width: 50%; 
} 
.col-3 { 
  width: 75%; 
} 
.col-4 { 
  width: 100%; 
}
```

## 总结

在本文中，我们试图了解使用SCSS编写功能CSS的基本知识，并且还大致了解了一些Sass/SCSS原理。 我希望我们将这些实践用于为我们的应用程序编写更轻松，更优化的样式。

------

**代码部署后可能存在的BUG没法实时知道，事后为了解决这些BUG，花了大量的时间进行log 调试，这边顺便给大家推荐一个好用的BUG监控工具 [Fundebug](https://www.fundebug.com/?utm_source=xiaozhi)。**

原文：[www.proqsolutions.com/creating-re…](https://www.proqsolutions.com/creating-reusable-css-components-sass-mini-framework/)

------

## 交流

干货系列文章汇总如下，觉得不错点个Star，欢迎 加群 互相学习。

> [github.com/qq449245884…](https://github.com/qq449245884/xiaozhi)

我是小智，公众号「大迁世界」作者，**对前端技术保持学习爱好者。我会经常分享自己所学所看的干货**，在进阶的路上，共勉！