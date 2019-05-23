# [webpack4+vue打包简单入门](https://segmentfault.com/a/1190000016505920)

前言
--

最近在研究使用webpack的使用,在查阅了数篇文章后,学习了webpack的基础打包流程.

本来就可以一删了之了,但是觉得未免有点可惜,所以就有了这篇文章,供大家参考.

webpack打包的教程具有时效性,有不少作者在一直维护一篇文章.超过一定时间参考价值就会下降,希望各位了解这一点.

### 使用的依赖一览

    "devDependencies": {
    "clean-webpack-plugin": "^0.1.19",
    "css-loader": "^1.0.0",
    "html-webpack-plugin": "^3.2.0",
    "mini-css-extract-plugin": "^0.4.3",
    "vue-loader": "^15.4.2",
    "vue-style-loader": "^4.1.2",
    "vue-template-compiler": "^2.5.17",
    "webpack": "^4.19.1",
    "webpack-cli": "^3.1.0",
    "webpack-merge": "^4.1.4"
    },
    "dependencies": {
    "vue": "^2.5.17"
    }

都是目前最新的稳定版本.

#### 讲解到的内容

1.  webpack的安装
2.  webpack简单配置
3.  vue的安装以及使用
4.  使用插件
5.  提取css
6.  使用多个配置文件
7.  简单的代码分离

构建目录
----

我们创建一个新的目录(learnwebpack).在下面创建src文件夹和一个index.html.

然后我们使用`npm init`命令初始化一个`package.json`文件,直接一路回车过去就好了.

目录结构:

*   learnwebpack

    *   src
    *   index.html
    *   package.json

src保存我们未经编译的源码.

index.html内容如下:

    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Webpack4 && Vue</title>
    </head>
    <body>
        <div id="root">
            
        </div>
    
        <script src="./dist/app.bundle.js"></script>
    </body>
    </html>

安装webpack
---------

在webpack4中不仅仅需要安装webpack本身,还需要安装webpack-cli.

    npm install webpack webpack-cli --save-dev

使用`--save-dev`参数会将这两个包添加到package.json的开发依赖中.

### webpack4的特性

在webpack4中可以实现0配置打包,也就是说不需要任何设置就可以完成简单的打包操作.

此外最实用的特性之一就是`mode`配置选项用于告诉webpack输出模式是开发还是产品,提供了已经预先定义好的常用配置.

后面的例子中我们会使用到`mode`.

webpack的简单配置
------------

我们在当前目录下新建一个webpack的配置文件webpack.config.js.

此时的目录结构为:

*   learnwebpack

    *   src
    *   index.html
    *   webpack.config.js
    *   package.json

webpack.config.js的内容:

    const {join:pathJoin} = require('path');
    module.exports = {
        entry:{
            app:'./src/index.js' // 入口文件的位置
        },
        output:{
            filename:'[name].bundle.js', // 输出文件名字的格式
            path:pathJoin(__dirname,'./dist') // 输出的绝对路径
        },
    }

### 为什么会有filename上的格式

对于我们当前的应用来说好像没有任何用途,打包后文件名完全可以和入口一样.

使用不同的文件名可以解决相同文件名称导致的浏览器缓存问题.

对于我们当前这个格式来说,打包后的文件名称为`app.bundle.js`.

此外还有一些额外的模板符号:

*   \[hash\] 模块标识符(module identifier)的 hash
*   \[chunkhash\] chunk 内容的 hash
*   \[name\] 模块名称
*   \[id\] 模块标识符(module identifier)
*   \[query\] 模块的 query，例如，文件名 ? 后面的字符串

现在让我们在src文件夹下新建一个index.js,只写一行简单的代码用于测试:

    console.log('hello world!');

此时的目录结构为:

*   learnwebpack

    *   src

        *   index.js
    *   index.html
    *   webpack.config.js
    *   package.json

### 为什么使用index.js

webpack4中零配置的默认入口位置就是当前配置路径下的`./src/index.js`,也就是说如果不指定入口的情况下也是可以打包的.

### 构建我们的项目

运行:

    ./node_modules/.bin/webpack-cli --config webpack.config.js

上方的命令有点冗长,我们使用package.json中`scripts`字段来定义一个快捷方式.

package.json:

    {
      +++
      "scripts": {
        "build": "webpack --config webpack.config.js"
      }
      +++
    }

\+\+\+ 表示`package.json`的其他字段,这部分目前不用关心.

现在运行这条简化的命令,效果和上方一样.

    npm run build

对于npm版本高于5.2.0的同学可以尝试使用npm的附带模块`npx`.

npx可以直接运行`./node_modules/.bin/`下的内容而不必输入路径.

    npx webpack --config webpack.config.js

如果操作正确你的目录下应该会多出一个dist文件夹,内部有一个app.bundle.js文件.

此时的目录结构:

*   learnwebpack

    *   dist

        *   app.bundle.js
    *   src

        *   index.js
    *   index.html
    *   webpack.config.js
    *   package.json

我们可以运行index.html了,如果一切顺利.那么在控制台中应该会出现`hello world!`.

### 错误

当我们打包的时候webpack4会报错,原因就是我们没有指定上文中提到的`mode`属性,webpack默认将他设置为`production`.

此时如果你查看代码,就会发现代码是经过压缩的,这是webpack4的默认行为之一.

    WARNING in configuration
    The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
    You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/

安装Vue
-----

使用npm安装:

    npm -i vue

vue是运行时依赖,webpack需要合适loader将vue文件解释为webpack可以理解的格式用于构建,所以我们需要`vue-loader`来转换vue单文件.

使用npm安装vue-loader:

    npm install vue-loader --save-dev 

vue单文件中分为三个部分,其中`template`部分需要专用的插件进行转换.

安装`vue-template-compiler`:

    npm install  vue-template-compiler --save-dev

这个使用vue-loader自动调用的插件,也是官方默认的,不需要任何配置.

如果你不使用他,打包的时候会报错.

简单来说他的功能是将`template`部分的模板转为`render`函数.

然后我们需要处理css,vue-loader需要css-loader才可以运行.

安装css-loader:

    npm install css-loader --save-dev

css-loader的作用仅仅是将css转为webpack可以解释的类型,如果我们需要将样式使用起来插入到html中,必须使用额外的插件.

安装`vue-style-loader`:

    npm install vue-style-loader --save-dev

`vue-style-loader`是由vue官方维护的你也可以使用其他的loader来处理css,他除了提供了常见的插入样式到html的功能以外还提供了其他的功能,例如热更新样式等.

### 修改配置

webpack.config.js:

    const {join:pathJoin} = require('path');
    const VueLoaderPlugin = require('vue-loader/lib/plugin'); // +++
    module.exports = {
        mode:'production',
        entry:{
            app:'./src/index.js'
        },
        output:{
            filename:'[name].bundle.js',
            path:pathJoin(__dirname,'./dist')
        },
        module: {
            rules: [
                {
                    test: /\.vue$/,
                    use: ['vue-loader'] // +++
                },
                {
                    test: /\.css$/,
                    use: ['vue-style-loader','css-loader'] // +++
                }
            ]
        },
        plugins:[
            new VueLoaderPlugin() // +++
        ]
    }

需要注意的是`VueLoaderPlugin`在`vue-loader`v15的版本中,这个插件是必须启用的.

### 修改目录结构以及文件

我们在`src`下创建一个`pages`文件夹并且在内部创建一个`app.vue`文件.

此时的目录结构:

*   learnwebpack

    *   dist

        *   app.bundle.js
    *   src

        *   pages

            *   app.vue
        *   index.js
    *   index.html
    *   webpack.config.js
    *   package.json

app.vue文件内容:

    <style>
    .test > p {
      background-color: aqua;
    }
    </style>
    
    <template>
        <article class="test">
            <p>{{vue}}</p>
        </article>
    </template>
    
    <script>
    export default {
      data() {
        return {
          vue: "vue"
        };
      }
    };
    </script>

app.vue是由index.js引用的,回过头来我们修改index.js:

    import Vue from 'vue'
    import App from './pages/app.vue';
    
    new Vue({
        el:'#root',
        render:h=>h(App)
    })

激动人心的时候到了,如果一切顺利你的vue应用应该打包完成了,运行index.html就可以查看这个结果了.

使用插件
----

回顾前面的一个话题,**指定输出文件名字格式**.如果你将我们的内容放到一个服务器上运行,你会很容易发现一个问题.

就是缓存,web上的资源浏览器会对其进行缓存,但是对开发的时候,或者产品中的代码更新十分不友好,新的代码不会得到及时的使用.

我们让资源每次的名字不一样就可以简单的解决这个问题,接下来修改我们的`webpack.config.js`:

    const {join:pathJoin} = require('path');
    const VueLoaderPlugin = require('vue-loader/lib/plugin');
    module.exports = {
        +++
        output:{
            filename:'[name].[hash].js',
            path:pathJoin(__dirname,'./dist')
        },
        +++
    }

此时再次修改代码打包每次的名字都不会一样.

但是出现了另外一个问题由于资源名字每次都不一致,每次我都需要手动指定index.html中引用的资源,这个时候该`html-webpack-plugin`出场了.

这个插件可以自动生成一个html文件,并且将资源文件按照正确顺序插入到html中.

安装`html-webpack-plugin`:

    npm install html-webpack-plugin --save-dev

webpack.config.js添加配置

    const {join:pathJoin} = require('path');
    const VueLoaderPlugin = require('vue-loader/lib/plugin');
    const HtmlWebpackPlugin = require('html-webpack-plugin');
    module.exports = {
        entry:{
            app:'./src/index.js'
        },
        output:{
            filename:'[name].[hash]].js',
            path:pathJoin(__dirname,'./dist')
        },
        module: {
            rules: [
                {
                    test: /\.vue$/,
                    use: ['vue-loader']
                },
                {
                    test: /\.css$/,
                    use: ['vue-style-loader','css-loader']
                }
            ]
        },
        plugins:[
            new HtmlWebpackPlugin({
                template:'template.html',// 指定模板html文件
                filename:'index.html'// 输出的html文件名称
            }),
            new VueLoaderPlugin()
        ]
    }

然后我们把`index.html`重命名为`template.html`,将`template.html`中的`script`标签删除.

指定模板后HtmlWebpackPlugin会保留原来html中的内容,而且会在新生成的html文件中把script插入到合适的位置.

### HtmlWebpackPlugin补充

注意:`HtmlWebpackPlugin`必须位于插件数组的首位.

### 运行项目

你会发现在dist目录下会有index.html文件,而我们打包后的index.js的输出文件名也变成了`app.XXXXXXXXXXXXXXXX.js`的格式,更加神奇的是在index.html中自动插入了script标签来引用打包后的js文件.

### 使用CleanWebpackPlugin插件

当你多次打包的时候,你会发现由于使用hash来命名输出的文件每次的文件名称都不一样,导致文件越来越多.

使用CleanWebpackPlugin可以每次构建前清空输出目录.

安装:

    npm install clean-webpack-plugin --save-dev

webpack.config.js:

    const {join:pathJoin} = require('path');
    const VueLoaderPlugin = require('vue-loader/lib/plugin');
    const HtmlWebpackPlugin = require('html-webpack-plugin');
    const CleanWebpackPlugin = require("clean-webpack-plugin");// +++
    module.exports = {
        entry:{
            app:'./src/index.js'
        },
        output:{
            filename:'[name].[hash]].js',
            path:pathJoin(__dirname,'./dist')
        },
        module: {
            rules: [
                {
                    test: /\.vue$/,
                    use: ['vue-loader']
                },
                {
                    test: /\.css$/,
                    use: ['vue-style-loader','css-loader']
                }
            ]
        },
        plugins:[
            new HtmlWebpackPlugin({
                template:'template.html',
                filename:'index.html'
            }),
            new VueLoaderPlugin(),
            new CleanWebpackPlugin(['dist'])// +++ 运行前删除dist目录
        ]
    }

现在无论运行几次构建dist目录下都是干净的.

提取css
-----

现在我们来将vue单文件中的css提取出来到一个单独的css文件中.

webpack4中完成这个功能的插件是`mini-css-extract-plugin`.

安装:

    npm install mini-css-extract-plugin --save-dev

webpack.config.js:

    const {join:pathJoin} = require('path');
    const VueLoaderPlugin = require('vue-loader/lib/plugin');
    const HtmlWebpackPlugin = require('html-webpack-plugin');
    const CleanWebpackPlugin = require("clean-webpack-plugin");
    const MiniCssExtractPlugin = require('mini-css-extract-plugin');// +++
    module.exports = {
        entry:{
            app:'./src/index.js'
        },
        output:{
            filename:'[name].[hash]].js',
            path:pathJoin(__dirname,'./dist')
        },
        module: {
            rules: [
                {
                    test: /\.vue$/,
                    use: ['vue-loader']
                },
                {
                    test: /\.css$/,
                    use: [
                        {
                            loader:MiniCssExtractPlugin.loader // +++
                        },
                        'css-loader'
                    ]
                }
            ]
        },
        plugins:[
            new HtmlWebpackPlugin({
                template:'template.html',
                filename:'index.html'
            }),
            new MiniCssExtractPlugin({
                filename:'style.css' // 指定输出的css文件名.
            }),
            new VueLoaderPlugin(),
            new CleanWebpackPlugin(['dist'])
        ]
    }

运行我们的项目,现在dist目录下已经有一个style.css的文件了,而且htmlwebpackplugin自动将style.css插入到了index.html中.

使用多个配置
------

考虑以下问题:

*   开发的时候我们希望打包速度快,有源代码提示,有热重载(本篇文章没有涉猎)...
*   发布产品我们希望输出的内容不会被缓存,有代码压缩...

是时候使用多个配置了,一个用于开发一个用于生产.

但是我们还可以做的更好使用`webpack-merge`插件,可以将多个配置合成.

安装`webpack-merge`:

    npm install webpack-merge --save-dev

首先我们定义一个包含基本信息的webpack配置文件,这个文件被其他配置文件依赖.

webpack.common.js:

    const {join:pathJoin} = require('path');
    const CleanWebpackPlugin = require("clean-webpack-plugin");
    const VueLoaderPlugin = require('vue-loader/lib/plugin');
    const HtmlWebpackPlugin = require('html-webpack-plugin');
    
    module.exports = {
        entry:{
            app:'./src/index.js'
        },
        output:{
            filename:'[name].[hash].js',
            path:pathJoin(__dirname,'./dist')
        },
        plugins:[
            new HtmlWebpackPlugin({
                template:'template.html',
                filename:'index.html'
            }),
            new VueLoaderPlugin(),
            new CleanWebpackPlugin(['dist'])
        ]
    }

可以看到这里定义的都是最基本的信息.

然后我们定义开发配置并且在内容使用`webpack-merge`插件和webpack.config.js进行合并.

webpack.dev.js:

    const merge = require('webpack-merge');
    const common = require('./webpack.common.js');
    
    module.exports = merge(common, {
        mode: 'development', // 不压缩代码,加快编译速度
        devtool: 'source-map', // 提供源码映射文件调试使用
        module: {
            rules: [
                {
                    test: /\.vue$/,
                    use: ['vue-loader']
                },
                {
                    test: /\.css$/,
                    use: ['vue-style-loader','css-loader'] // 使用vue-style-loader直接插入到style标签中
                }
            ]
        },
    })

定义生产配置.

webpack.prod.js:

    const merge = require('webpack-merge');
    const common = require('./webpack.common.js');
    const MiniCssExtractPlugin = require('mini-css-extract-plugin')
    
    module.exports = merge(common,{
        mode:'production', // 压缩代码
        module: {
            rules: [
                {
                    test: /\.vue$/,
                    use: ['vue-loader']
                },
                {
                    test: /\.css$/,
                    use: [
                        {
                            loader:MiniCssExtractPlugin.loader // 提取css到外部文件中
                        },
                        'css-loader'
                    ]
                }
            ]
        },
        plugins:[
            new MiniCssExtractPlugin({
                filename:'style.css'
            })
        ]
    })

最后修改一下我们package.json中的快捷方式:

    +++
    "scripts": {
        "dev": "webpack --config webpack.dev.js",
        "build": "webpack --config webpack.prod.js"
    },
    +++

删除之前的webpack.config.js.

完成后的目录结构:

*   learnwebpack

    *   dist
    *   src

        *   pages

            *   app.vue
        *   index.js
    *   package.json
    *   template.html
    *   webpack.common.js
    *   webpack.dev.js
    *   webpack.prod.js

尝试运行`npm run dev`和`npm run build`吧.

简单的代码分离
-------

我们在index.js中引用了Vue,如果你查看打包后的代码你会发现vue本身会被打包到同一个文件中,如果我们有另外一个文件也引用了vue同样的会被打包到该文件中.

显然我们的代码依赖vue,但是vue不应该存在两份,如果可以将vue单独提取出来就好了,这个问题就是要代码分离.

在webpack4中删除了原来的CommonsChunkPlugin插件,内部集成的optimization.splitChunks选项可以直接进行代码分离.

修改webpack.dev.js:

    const merge = require('webpack-merge');
    const common = require('./webpack.common.js');
    
    module.exports = merge(common, {
        mode: 'development',
        devtool: 'source-map',
        optimization:{ // +++
            splitChunks:{ // +++
                chunks:'initial' // +++ initial(初始块)、async(按需加载块)、all(全部块)
    
            }
        },
        module: {
            rules: [
                {
                    test: /\.vue$/,
                    use: ['vue-loader']
                },
                {
                    test: /\.css$/,
                    use: ['vue-style-loader','css-loader']
                }
            ]
        },
    })

我们要做的就是将打包后写入到index.html的文件从目前的一个文件拆分成多个,并且写入到index.html中.

也就是说在入口处分离代码.

运行:

    npm run dev

你会发现在dist目录下多出了一个文件保存着vue,另外一个文件中保存着我们编写的代码.

参考
--

vue部分:

> [https://cn.vuejs.org/v2/guide...](https://cn.vuejs.org/v2/guide/deployment.html#%E6%A8%A1%E6%9D%BF%E9%A2%84%E7%BC%96%E8%AF%91)  
> [https://vue-loader.vuejs.org/zh/](https://vue-loader.vuejs.org/zh/)  
> [https://www.npmjs.com/package...](https://www.npmjs.com/package/vue-style-loader)

webpack部分:

> [https://webpack.docschina.org...](https://webpack.docschina.org/plugins/split-chunks-plugin/)  
> [https://github.com/webpack-co...](https://github.com/webpack-contrib/mini-css-extract-plugin)  
> [https://github.com/jantimon/h...](https://github.com/jantimon/html-webpack-plugin)

参考的其他教程:

> [https://segmentfault.com/a/11...](https://segmentfault.com/a/1190000015326528)  
> [https://segmentfault.com/a/11...](https://segmentfault.com/a/1190000014251654)  
> [http://www.cnblogs.com/anani/...](http://www.cnblogs.com/anani/p/9255816.html)  
> [https://blog.csdn.net/bubblin...](https://blog.csdn.net/bubbling_coding/article/details/8158541)