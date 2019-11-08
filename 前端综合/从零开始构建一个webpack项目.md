# 从零开始构建一个webpack项目

### 1、新建项目

新建一个空文件夹，用于创建项目，使用 npm init 命令创建一个 package.json 文件。
输入这个命令后，终端会问你一系列诸如项目名称，项目描述，作者等信息，也可以使用 npm init -y 这个命令来一次生成 package.json 文件，这样终端不会询问你问题。

### 2、安装 webpack

安装 webapck 时把 webpack-cli 也装上是因为在 webpack4.x 版本后 webpack 模块把一些功能分到了 webpack-cli 模块，所以两者都需要安装，安装方法如下：

```
npm install webpack webpack-cli --global    //这是安装全局webpack及webpack-cli模块
npm install webpack webpack-cli --save-dev  //这是安装本地项目模块
复制代码
```

### 3、新建文件目录

在根目录件夹中新建两个文件夹，分别为 src 文件夹和 dist 文件夹，接下来再创建三个文件:此时，项目结构如下

- index.html --放在 dist 文件夹中；
- hello.js --放在 src 文件夹中；
- index.js --放在 src 文件夹中；

#### 3.1、 index.html 中写下 html 代码，它的作用是为了引入我们打包后的 js 文件：

```
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Webpack Project</title>
  </head>
  <body>
    <div id="root"></div>
    <script src="bundle.js"></script>
    <!--这是打包之后的js文件，我们暂时命名为bundle.js-->
  </body>
</html>
复制代码
```

#### 3.2、在 hello.js 中导出一个模块：

```js
// hello.js
module.exports = function() {
  let hello = document.createElement('div')
  hello.innerHTML = 'welcome to China!'
  return hello
}
```

#### 3.3、在 index.js 中引入这个模块（hello.js）:

```js
//index.js
const hello = require('./hello.js')
document.querySelector('#root').appendChild(hello())
```

上述操作就相当于我们把 hello.js 模块合并到了 index.js 模块，之后我们打包时就只需把 index.js 模块打包成 bundle.js 即可。

#### 3.4、进行最简单的 webpack 打包

```
// 在终端中使用如下命令进行打包：
webpack src/index.js --output dist/bundle.js
```

上述就相当于把 src 文件夹下的 index.js 文件打包到 dist 文件下的 bundle.js，这时就生成了 bundle.js 供 index.html 文件引用。现在打开 index.html 就可以看到我们的页面了。

### 4、配置 webpack.config.js

上述打包方式太 low 了，我们可以在当前项目的根目录下新建一个配置文件 webpack.config.js 用来配置打包方式。 webpack.config.js 配置如下

```js
const path = require('path') // 处理绝对路径
module.exports = {
  entry: path.join(__dirname, '/src/index.js'), // 入口文件
  output: {
    path: path.join(__dirname, '/dist'), //打包后的文件存放的地方
    filename: 'bundle.js' //打包后输出文件的文件名
  }
}
```

有了这个配置文件，我们只需在终端中运行 webpack 命令就可进行打包，这条命令会自动引用 webpack.config.js 文件中的配置选项。

### 5、构建本地服务器

现在我们是通过打开本地文件来查看页面的，感觉还是有点 low。例如 vue, react 等脚手架都是在本地服务器运行的。所以我们再做进一步优化。

#### 5.1 webpack-dev-server 配置本地服务器

Webpack 提供了一个可选的本地开发服务器，这个本地服务器基于 node.js 构建，它是一个单独的组件，在 webpack 中进行配置之前需要单独安装它作为项目依赖：npm i webpack-dev-server -D

以下是devServer 的一些配置选项:

- contentBase ：设置服务器所读取文件的目录，当前我们设置为"./dist"
- port ：设置端口号，如果省略，默认为 8080
- inline ：设置为 true，当源文件改变时会自动刷新页面
- historyApiFallback ：设置为 true，所有的跳转将指向 index.html

现在我们把这些配置加到 webpack.config.js 文件上，如下：

```js
// webpack.config.js
const path = require('path')
module.exports = {
  entry: path.join(__dirname, '/src/index.js'), // 入口文件
  output: {
    path: path.join(__dirname, '/dist'), //打包后的文件存放的地方
    filename: 'bundle.js' //打包后输出文件的文件名
  },
  devServer: {
    contentBase: './dist', // 本地服务器所加载文件的目录
    port: '8088', // 设置端口号为8088
    inline: true, // 文件修改后实时刷新
    historyApiFallback: true //不跳转
  }
}
```

#### 5.2、package.json 文件中添加启动和打包命令

package.json 文件修改如下

```js
{
  "name": "webpack-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "webpack",
    "dev": "webpack-dev-server --open"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.23.1",
    "webpack-cli": "^3.1.2",
    "webpack-dev-server": "^3.1.10"
  }
}
```

这样我们就可以用以下命令进行本地运行或者打包文件了

- npm run dev 启动本地服务器，webpack-dev-server 就是启动服务器的命令，--open 是用于启动完服务器后自动打开浏览器。
- npm run build 执行打包命令

此时，我们只要输入 npm run dev 就可以在http://localhost:8088/中查看页面了。

### 6、配置常用 loader

loader 可以让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）。loader 可以将所有类型的文件转换为 webpack 能够处理的有效模块，然后你就可以利用 webpack 的打包能力，对它们进行处理。

Loaders 的配置包括以下几方面：

- test：一个用以匹配 loaders 所处理文件的拓展名的正则表达式（必须）
- loader：loader 的名称（必须）
- include/exclude：手动添加必须处理的文件（文件夹）或屏蔽不需要处理的文件（文件夹）（可选）；
- options：为 loaders 提供额外的设置选项（可选）

#### 配置 css-loader 和 sass-loader

如果我们要加载一个 css 文件，需要安装配置 style-loader 和 css-loader。
如果我们要使用 sass，就要配置 sass-loader 和 node-sass。

- css-loader：加载.css 文件
- style-loader：使用 style 标签将 css-loader 内部样式注入到我们的 HTML 页面

```js
const path = require('path')
module.exports = {
  entry: path.join(__dirname, '/src/index.js'), // 入口文件
  output: {
    path: path.join(__dirname, '/dist'), //打包后的文件存放的地方
    filename: 'bundle.js' //打包后输出文件的文件名
  },
  devServer: {
    contentBase: './dist', // 本地服务器所加载文件的目录
    port: '8088', // 设置端口号为8088
    inline: true, // 文件修改后实时刷新
    historyApiFallback: true //不跳转
  },
  module: {
    rules: [
      {
        test: /\.css$/, // 正则匹配以.css结尾的文件
        use: ['style-loader', 'css-loader']
      {
        test: /\.(scss|sass)$/, // 正则匹配以.scss和.sass结尾的文件
        use: ['style-loader', 'css-loader', 'sass-loader']
      }
    ]
  }
}
```

#### 配置 Babel-loader

Babel 其实是一个编译 JavaScript 的平台，它可以编译代码帮你达到以下目的：

- 让你能使用最新的 JavaScript 代码（ES6，ES7...）；
- 让你能使用基于 JavaScript 进行了拓展的语言，比如 React 的 JSX；

```js
module: {
  ...
  rules: [
    {
      test: /\.js$/,
      loader: 'babel-loader',
      include: [resolve('src')]
    }
  ]
}
```

#### 处理图片

处理图片资源时，我们常用的两种 loader 是 file-loader 或者 url-loader。 当使用 url-loader 加载图片，图片小于上限值，则将图片转 base64 字符串，否则使用 file-loader 加载图片。

```js
module: {
  ...
  rules: [
    {
      test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
      loader: 'url-loader',
      options: {
        limit: 10000,
        name: utils.assetsPath('img/[name].[hash:7].[ext]')
      }
    }
  ]
}
```

### 7、配置常用插件

loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。插件接口功能极其强大，可以用来处理各种各样的任务。

#### 7.1、自动生成 html 文件(HtmlWebpackPlugin)

现在我们都是使用一开始建好的 index.html 文件，然后手动引入 bundle.js，如果以后我们引入不止一个 js 文件，那就得更改 index.html 中的 js 文件名，所以能不能自动生成 index.html 且自动引用打包后的 js 呢？
HtmlWebpackPlugin 插件就是用来解决这个问题的：

1. 安装插件 npm i html-webpack-plugin -D
2. 把 dist 文件夹清空
3. 在根目录新建 index.html,内容和原来的 html 一致，只是不引入 js 文件。
4. webpack.config.js 中我们引入了 HtmlWebpackPlugin 插件

```js
plugins: [
  new HtmlWebpackPlugin({
    filename: 'index.html',
    template: 'index.html',
    inject: true,
    minify: {
      removeComments: true,
      collapseWhitespace: true,
      removeAttributeQuotes: true
    }
  })
]
```

此时我们使用 npm run build 进行打包，你会发现，dist 文件夹和 html 文件都会自动生成。

#### 7.2、清理/dist 文件夹(CleanWebpackPlugin)

在每次构建前清理/dist 文件夹，生产最新的打包文件，这时候就用到 CleanWebpackPlugin 插件了。

1. 安装 npm i clean-webpack-plugin -D
2. 配置 webpack.config.js

```js
plugins: [
  new HtmlWebpackPlugin({
    filename: 'index.html',
    template: 'index.html',
    inject: true,
    minify: {
      removeComments: true,
      collapseWhitespace: true,
      removeAttributeQuotes: true
    }
  }),
  new CleanWebpackPlugin(['dist'])
]
```

#### 7.3、热更新(HotModuleReplacementPlugin)

我们要在修改代码后自动更新页面，这就需要 HotModuleReplacementPlugin（HMR）插件

1. devServer 配置项中添加 hot: true 参数。
2. 因为 HotModuleReplacementPlugin 是 webpack 模块自带的，所以引入 webpack 后，在 plugins 配置项中直接使用即可。

```js
plugins: [
  new HtmlWebpackPlugin({
    filename: 'index.html',
    template: 'index.html',
    inject: true,
    minify: {
      removeComments: true,
      collapseWhitespace: true,
      removeAttributeQuotes: true
    }
  }),
  new CleanWebpackPlugin(['dist'])
  new webpack.HotModuleReplacementPlugin()
]
```

#### 7.4、增加 css 前缀

平时我们写 css 时，一些属性需要手动加上前缀，比如-webkit-border-radius: 10px;，在 webpack 中我们可以让他自动加上

1. 安装 npm i postcss-loader autoprefixer -D
2. 在项目根目录下新建 postcss.config.js 文件

```js
module.exports = {
  plugins: [
    require('autoprefixer') // 引用autoprefixer模块
  ]
}
module.exports = {
   ...
  module: {
    rules: [
      {
        test: /\.css$/, // 正则匹配以.css结尾的文件
        use: [
          { loader: 'style-loader' }, // 这里采用的是对象配置loader的写法
          { loader: 'css-loader' },
          { loader: 'postcss-loader' } // 使用postcss-loader
        ]
      }
       ...
    ]
  }
   ...
}
```

#### 7.5、css 分离 ExtractTextPlugin

将 css 成生文件，而非内联。该插件的主要是为了抽离 css 样式,防止将样式打包在 js 中引起页面样式加载错乱的现象。

1. 安装 npm i extract-text-webpack-plugin@next -D
2. 在 webpack.common.js 文件中引入并使用该插件：

```js
const ExtractTextPlugin = require('extract-text-webpack-plugin') //引入分离插件
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/, // 正则匹配以.css结尾的文件
        use: ExtractTextPlugin.extract({
          // 相当于回滚，经postcss-loader和css-loader处理过的css最终再经过style-loader处理
          fallback: 'style-loader',
          use: ['css-loader', 'postcss-loader']
        })
      }
    ]
  },
  plugins: [
    new ExtractTextPlugin('css/index.css') // 将css分离到/dist文件夹下的css文件夹中的index.css
  ]
}
```

此时运行 npm run build 后会发现/dist 文件夹内多出了/css 文件夹及 index.css 文件。

#### 7.6、消除冗余 css

有时候我们 css 写得多了，可能会不自觉的写重复了一些样式，这就造成了多余的代码，以下方法可以优化

1. 安装 npm i purifycss-webpack purify-css glob -D
2. 引入 clean-webpack-plugin 及 glob 插件并使用

```js
const PurifyCssWebpack = require('purifycss-webpack') // 引入PurifyCssWebpack插件
const glob = require('glob') // 引入glob模块,用于扫描全部html文件中所引用的css

plugins: [
  new PurifyCssWebpack({
    paths: glob.sync(path.join(__dirname, 'src/*.html')) // 同步扫描所有html文件中所引用的css
  })
]
```

**至此，一些常用的配置以及弄好了，现在就开始愉快地写代码了**