# LESS即学即用

前言
--

我们大家都知道HTML和CSS不属于编程语言而是属于标记语言，所以很难像JS一样定义变量、编写方法、实现模块化开发等。而目前的CSS编写模式中，都是定义一些公共的样式类名，哪一块的HTML需要这个样式，就去增加对应的样式类名，所以我们经常看到一个标签上存在很多样式类名，在这种模式中我们要时常关注CSS的优先级，避免样式的重叠覆盖...

为了解决CSS的这一困境，CSS预处理预编译的思想脱颖而出，比较具有代表性的有LESS、SASS、Stylus。它们在传统的CSS基础上增加了大量的新的语法，编写方式，常用的函数等，可以让我们的CSS像JS一样成为一门编程语言。本文主要介绍LESS的语法和使用。

一、LESS的编译
---------

编写完成的LESS代码是不能直接在浏览器中运行的，需要编译成正常的CSS代码。那么我们首先就来学习一下常用的LESS编译方式。

### 1.在浏览器中调用LESS.JS

LESS只支持在现代浏览器中运行（最新版本的Chrome, Firefox, Safari 和 IE）。我们不建议在生产环境中使用LESS客户端，因为在将LESS编译成CSS的时候，用户会看到加载延迟的现象，即便在浏览器中有不足一秒的加载延迟，但也会降低性能。

首先引入我们设置样式的LESS文件，**注意：这里的rel='stylesheet/less'**

    <link type="text/css" rel="stylesheet/less" href="1.less"/>

然后引入less.js文件

    <script src="//cdnjs.cloudflare.com/ajax/libs/less.js/3.8.1/less.min.js" ></script>

**浏览器中调用有个好处:可以开启监视模式,只要我们的LESS改变，在一定时间内，浏览器就会重新的编译，我们可以看到想要的效果**。具体操作如下:

        <script charset="utf-8" type="text/javascript">
        //->在引入LESS之前设置一个全局的变量less,配置一些参数值(根据情况自行选择需要配置的项)
          var less = {
            //->env:设置运行的环境(生产模式还是开发模式)
            //production:编译后的CSS缓存到本地localStorage中
            //development:没有把编译后的CSS缓存到本地,在URL不是标准的格式下(例如:file://...),自动设置为development
            env: "development",
            //->poll:在监视模式下，每两次请求之间的时间间隔(ms)
            poll:500
            }
        </script>     
        <script src="//cdnjs.cloudflare.com/ajax/libs/less.js/3.8.1/less.min.js" ></script>
        <script charset="utf-8" type="text/javascript">
          //->启用监视模式(env必须要设置成development)
          less.watch();
        </script>   

**需要特别注意的是，由于浏览器端使用时是使用ajax来拉取.less文件，因此直接在本机文件系统打开（即地址是file://开头）或者是有跨域的情况下会拉取不到.less文件，导致样式无法生效**。所以一般less做css预处理，很少采用引用less的方式，一般是把less编译成css之后，引用生成的css文件。

    // index.html文件
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>less</title>
        <link type="text/css" rel="stylesheet/less" href="css/styles.less">
    </head>
    <body>
        <header>
            <p>Hello World!</p>
        </header>
    <script src="./less.min.js" type="text/javascript"></script>   
    </body>
    </html>
    
    // styles.less文件
    @blue:#5B83AD;
    @size:50px;
    header {
        color: @blue;
        font-size: @size;
    }

如果直接引入less文件，便会出现如下错误：

可以尝试用一个简单的web server启动网站就能解决，譬如lite-server

    npm install -g lite-server
    lite-server //to run

### 2、使用NODE命令编译LESS(单个文件)

这种方式是目前项目中最常用的方式，适用于项目的生产环境，它是把我们的LESS文件编译成CSS文件，我们项目中直接的引入CSS文件即可，**基本步骤：安装->编译/压缩编译->或者使用NODE代码实现批量编译等**

把LESS模块安装到全局NODE环境中

    npm install less -g

使用命令进行编译

        //->把styles.less文件编译成styles.css文件(如果没有这个CSS文件自己会创建)
        lessc styles.less styles.css
    
        //->编译完成的CSS文件是经过压缩的
        lessc styles.less styles.min.css -x或者--compress

经过node命令编译代码前后对比:

    // styles.less文件(编译前)
    @blue:#5B83AD;
    @size:50px;
    header {
        color: @blue;
        font-size: @size;
    }
    
    // styles.css文件(编译后)
    header {
      color: #5B83AD;
      font-size: 50px;
    }

### 3、在NODE环境中编写批量编译的代码

我们在上述用NODE命令编译的时候，一次只能编译一个文件，这样，如果页面中有多个LESS，每一次编译都是比较耗费时间的，所以我们结合NODE的FS文件读写操作，可以写一套批量编译的代码。

        //->定义编译文件目录和目标导出目录
        var dirPath = "./less/", tarPath = "./css/";
        //->导入NODE中需要使用的模块
        var fs = require("fs"),
            less = require("less");
        //->读取dirPath中的所有文件,检查文件的类型,只有LESS文件我们才进行存储
        var ary = [],
            files = fs.readdirSync(dirPath);
        files.forEach(function (file, index) {
            /\.(LESS)/i.test(file) ? ary.push(file) : null;
        });
        //->把目录下的所有文件进行编译,把编译完成的结果保存在指定的目录中
        ary.forEach(function (file) {
            var newFile = file.replace(".less", ".css"),
                conFile = fs.readFileSync(dirPath + file, "utf-8");
            less.render(conFile, {compress: true}, function (error, output) {
                fs.writeFileSync(tarPath + newFile, output.css, "utf-8");
            });
        });

### 4、使用工具编译LESS

目前常用的编译工具有：Koala(据说目前最流行的)、在线编译([tool.oschina.net/less](https://link.juejin.im?target=http%3A%2F%2Ftool.oschina.net%2Fless))、SimpLESS等。本篇文章不做过多的说明。

二、LESS的语法
---------

LESS的基础语法基本上分为以下几个方面：变量、混合(Mixins)、嵌套规则、运算、函数、作用域等。

### 1.变量

**和JS中的变量一样，只是LESS的变量定义不是使用VAR而是使用@**。

      //->LESS代码
        @link-color: #428bca;
        @link-color-hover: darken(@link-color, 10%);
        a {
            color: @link-color;
            &:hover {
                color: @link-color-hover;
            }
        }
      //->编译为CSS的结果
        a {
            color: #428bca;
        }
        a:hover {
            color: #3071a9;
        }

除了上述用变量存储公用的属性值，我们还可以用变量存储公用的URL、选择器等等

        //->LESS代码
        .@{selector} {
            width: 100px;
            height: 100px;
            @{property}: #000;
            background: url("@{bgImg}/test.png");
        }
        @selector: box;
        @bgImg: "../img";
        @property: color;

### 2.混合(Mixins)

#### ① 基本使用

混合可以将一个定义好的class A轻松的引入到另一个class B中，从而简单实现class B继承class A中的所有属性。我们还可以带参数地调用，就像使用函数一样。

        //->LESS代码
        .public {
            width: 100px;
            height: 100px;
        }
        nav ul {
            .public;
            list-style: none;
        }
        //->编译为CSS的结果
        .public {
            width: 100px;
            height: 100px;
        }
        nav ul {
            width: 100px;
            height: 100px;
            list-style: none;
        }

上述的代码，nav ul是把public中设定的样式属性值copy了一份到自己的样式中。如果你想在编译完成的结果中不输出public这个样式的结果，只需要按照下述的代码编写即可：

        //->LESS代码
        .public() {//->在选择器后面加上()就可以不编译这个样式了
            width: 100px;
            height: 100px;
        }
        nav ul {
            .public;//如果public有子孙元素的样式，同样也会被复制过来
            list-style: none;
        }
        //->编译为CSS的结果
        nav ul {
            width: 100px;
            height: 100px;
            list-style: none;
        }

#### ② extend

虽然在上述的案例中，nav ul把public中的样式继承了过来，但是原理却是把代码copy一份过来，这样编译后的CSS中依然会存留大量的冗余CSS代码，为了避免这一点，我们可以使用extend伪类来实现样式的继承使用。**和原来的选择器共用一套样式，但要保证原来的选择器不加括号。**

        //->LESS代码
        .public {
            width: 100px;
            height: 100px;
        }
        nav ul {
            &:extend(.public);
            list-style: none;
        }
        //->编译为CSS的结果
        .public, nav ul {
            width: 100px;
            height: 100px;
        }
        nav ul {
            list-style: none;
        }

### 3.嵌套规则

我们可以在一个选择器中嵌套另一个选择器来实现继承，这样很大程度减少了代码量，并且代码看起来更加的清晰。

    //->LESS代码
    #header {
      color: black;
      .navigation {
        font-size: 12px;
      }
      .logo {
        width: 300px;
        &:hover { text-decoration: none }
      }
    }
    
    //->编译为CSS的结果
    #header { color: black; }
    #header .navigation {
      font-size: 12px;
    }
    #header .logo { 
      width: 300px; 
    }
    #header .logo:hover {
      text-decoration: none;
    }

### 4.函数 & 运算

运算提供了加，减，乘，除操作；我们可以做属性值和颜色的运算，这样就可以实现属性值之间的复杂关系。LESS中的函数一一映射了JavaScript代码，如果你愿意的话可以操作属性值。

任何数字、颜色或者变量都可以参与运算. 来看一组例子:

    @base: 5%;
    @filler: @base * 2;
    @other: @base + @filler;
    color: #888 / 4;
    background-color: @base-color + #111;
    height: 100% / 2 + @filler;

LESS 提供了一系列的颜色运算函数. 颜色会先被转化成 HSL 色彩空间, 然后在通道级别操作:

    lighten(@color, 10%);     // return a color which is 10% *lighter* than @color
    darken(@color, 10%);      // return a color which is 10% *darker* than @color
    复制代码

LESS提供了一组方便的数学函数，你可以使用它们处理一些数字类型的值:

    round(1.67); // returns `2`
    ceil(2.4);   // returns `3`
    floor(2.6);  // returns `2`

### 5.命名空间和作用域

LESS 中的作用域跟其他编程语言非常类似，首先会从本地查找变量或者混合模块，如果没找到的话会去父级作用域中查找，直到找到为止。

    @var: red;
    #page {
      @var: white;
      #header {
        color: @var; // white
      }
    }
    #footer {
      color: @var; // red  
    }

**如果觉得文章对你有些许帮助，欢迎在[我的GitHub博客](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fljianshu%2FBlog)点赞和关注，感激不尽！**

参考文章
----

- [LESS学习:了解LESS和编译LESS](https://link.juejin.im?target=http%3A%2F%2Fold.zhufengpeixun.cn%2Fqianduanjishuziliao%2FmobileDevelopment%2F2016-07-22%2F527.html)

- [LESS学习:LESS的基础语法](https://link.juejin.im?target=http%3A%2F%2Fold.zhufengpeixun.cn%2Fqianduanjishuziliao%2FmobileDevelopment%2F2016-07-22%2F528.html)

- [LESS一种动态样式语言](https://link.juejin.im?target=http%3A%2F%2Fwww.bootcss.com%2Fp%2Flesscss%2F)