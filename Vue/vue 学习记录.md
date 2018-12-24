# Vue 学习记录



## Vue CLI

Vue CLI 是一个基于 Vue.js 进行快速开发的完整系统，提供：

- 通过 `@vue/cli` 搭建交互式的项目脚手架。

- 通过 `@vue/cli` + `@vue/cli-service-global` 快速开始零配置原型开发。

- 一个运行时依赖 (  `@vue/cli-service` )，该依赖：

  - 可升级；
  - 基于 webpack 构建，并带有合理的默认配置；
  - 可以通过项目内的配置文件进行配置；
  - 可以通过插件进行扩展。

- 一个丰富的官方插件集合，集成了前端生态中最好的工具。

- 一套完全图形化的创建和管理 Vue.js 项目的用户界面。

Vue CLI 致力于将 Vue 生态中的工具基础标准化。它确保了各种构建工具能够基于智能的默认配置即可平稳衔接，这样你可以专注在撰写应用上，而不必花好几天去纠结配置的问题。与此同时，它也为每个工具提供了调整配置的灵活性，无需 eject。

## webpack

*webpack* 是一个现代 JavaScript 应用程序的*静态模块打包器(module bundler)*。当 webpack 处理应用程序时，它会递归地构建一个*依赖关系图(dependency graph)*，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 *bundle*。

## 独立构建和运行构建

它们的区别在于前者包含**模板编译器**而后者不包含。

模板编译器的职责是将模板字符串编译为纯 JavaScript 的渲染函数。如果你想要在组件中使用 `template` 选项，你就需要编译器。

- 独立构建包含模板编译器并支持 `template` 选项。 **它也依赖于浏览器的接口的存在，所以你不能使用它来为服务器端渲染。**
- 运行时构建不包含模板编译器，因此不支持 `template` 选项，只能用 `render` 选项，但即使使用运行时构建，在单文件组件中也依然可以写模板，因为单文件组件的模板会在构建时预编译为 `render` 函数。运行时构建比独立构建要轻量30%，只有 16.39 Kb min+gzip大小。

##  项目实录

1. 安装vue-cli 3.x

   ```
   cnpm install -g @vue/cli
   ```

2. 安装 mint-ui

   ```
   cnpm i mint-ui -S
   ```

3. 安装 babel-plugin-component 按需引用 mint-ui 组件

   ```
   cnpm i babel-plugin-component -D
   ```
   然后在 .babelrc 中配置它：

   ```
   {
     "presets": [
       ["env", {
         "modules": false,
         "targets": {
           "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
         }
       }],
       "stage-2"
     ],
     "plugins": ["transform-runtime",["component",[
             {"libraryName":"mint-ui","style":true}
         ]]],
     "env": {
       "test": {
         "presets": ["env", "stage-2"],
         "plugins": ["istanbul"]
       }
     }
   }
   ```

4. 详解vue移动端项目的适配(以mint-ui为例)

   https://www.jb51.net/article/145803.htm

5. 

