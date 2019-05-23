# Vue开发中每个人都会遇到的问题

整理自：https://www.jianshu.com/p/0d261f0e1ad0

公司年初开始从jquery转型到vue开发，思想上从jquery的操作DOM到vue的操作数据，刚开始还不太习惯，但用了一段时间发现确实比较方便。在刚开始用vue的时候，也踩了一些坑，现在分享出来，供大家参考。

#### 1\. props单向绑定

vue中的props是单向绑定的，父组件的属性变化时会传递给子组件，子组件内部不应改变props的值，否则控制台会给出警告。  
但如果props的类型为数组或者对象时，在子组件内部改变props的值控制台不会警告。因为数组或对象是地址引用，vue不会检测到props发生改变。所以有的情况需要在子组件内部改变父组件的值，可以将属性定义为数组或者对象类型传入。  
但官方不建议在子组件内改变父组件的值，因为这违反了vue中props单向绑定的思想。

#### 2\. 给对象赋值

由1可以引申出，地址引用类型的数据，例如对象obj ={a:1},如果想要修改obj中的a属性，通过obj.a = 2这样赋值，页面不会更新，需使用vue.set方法更改才会起作用， `Vue.set(this,obj,a,2)`;  
同样，如果要给obj增加一个新属性，如果该属性未在data中声明，页面也不会刷新。也就是vue文档中声明的“Vue 不能检测到对象属性的添加或删除”，同样需要使用vue.set方法进行赋值才好使。

#### 3\. 深拷贝数组或对象

对象或数组的简单赋值，修改新值也会改变原值。这时我们需要获取原值的深拷贝对象。  
对于对象，可以通过`newObj = JSON.parse(JSON.stringfy(obj))`实现。  
对于数组，可以通过 `newArr = […arr]`或者`newArr = arr.slice(0)`来实现。

#### 4\. 给组件增加独有样式

vue中每一个组件都可以自定各自的css样式，如果希望组件内的样式只对当前组件起作用，可以在style标签中增加scoped即可。  
该写法会让vue在渲染组件的时候给每个元素都增加一个data-v-/版本号/的属性，可以保证只针对有同样data-v-data-v-/版本号/的元素应用该样式。

#### 5\. v-for循环key属性

vue中的v-for循环最好加上key属性，否则在高版本(2.2.0+)的vue中控制台会报错。  
key属性需要唯一，理想的 key 值是每项都有唯一 id，全局不需唯一，但在一个循环中需要唯一。

#### 6\. 引用图片

图片引用问题。直接把本地图片地址放在src里没问题。但如果把地址提取出来写在data里或者通过method动态给src赋值则引用不到。  
因为放在template模板里会被webpack打包所以可以，而放在data或者动态赋值，图片路径只是一个字符串webpack不会处理所以引用不到。  
解决办法：通过import或者required引入。`import src from ‘../../img.png’`或者`data:{img:require(‘../../img.png’)}`

#### 7\. 父组件传值

在子组件使用父组件传入的值时，最好复制出一份`props`的值，通过`data`或者`computed`进行赋值。  
`data`赋值与`computed`赋值的区别：  
`data`赋值：`data:{return {aaa: this.aaa}`如果是在`data`中进行赋值，当父组件的`aaa`值发生改变时，不会在重新赋给子组件中的`aaa`。  
`computed`赋值：如果想让子组件跟着父组件修改，需要将赋值操作写在`computed`中。`computed:{aaa(){return this.aaa}`

#### 8\. 对象数组深度监听

后端传过来的数组是一个数组对象，页面中绑定对象中某一具体的属性，当该值变化时调用某个函数，自然想到就是`watch`方法。但如何`watch`数组对象中某一个具体的属性，显然不可能一个个属性写`watch`。  
解决办法：  
1.`watch`整个对象，设置`deep`为`true`，当该对象发生改变时，调用处理函数。  
2.将页面中绑定的属性写在`computed`函数中，`watch`这个`computed`中的函数，当对象值改变时会进入`computed`函数中，进而进入`watch`函数中，再调用处理函数。

#### 9\. 动态增加class

给元素动态增加`class`时，可以在模板中通过`:class={‘hasClass’: ifHasClass}`来实现，当`ifHasClass`为`true`时，该元素会自动加上`hasClass`的样式。  
动态绑定的`class`可以与正常写的一起使用`<a class=‘aaa’ :class={‘bbb’:isBbb}></a>`，但如果在一个元素中使用了两个`class`则会报错`<a class="aaa" class="bbb"></a>`。

#### 10\. 组件中使用定时器及销毁问题

如果我们在页面A中使用了一个定时器，当从页面A跳转到页面B时，如果不手动清除这个定时器，那么它仍旧会执行，这不是我们所期望的。  
通常能想到的常规解决办法就是，在`data`属性中定义一个`timer`，在代码中启动定时器，然后在组件销毁的时候清除定时器。具体代码如下：

        data(){
            return{
                timer:null
            }
        },
        methods:{
            onStartTimer(){
                this.timer = setInterval( () => {
                    // 执行一些操作
                }, 1000)
            }
        },
        beforeDestroy() {
            clearInterval(this.timer);        
            this.timer = null;
        }


但是这里有两个潜在的问题：

1.  它需要在这个组件实例中保存这个`timer`，如果可以的话最好只有生命周期钩子可以访问到它。这并不算严重的问题，但是它可以被视为杂物。
2.  我们的建立代码独立于我们的清理代码，这使得我们比较难于程序化地清理我们建立的所有东西。

`Vue`官方文档给出的解决方案是，在定义timer的时候使用`$once`指令监听`beforeDestroy`这个钩子函数。具体代码为：

        methods:{
            onStartTimer(){
                const timer = setInterval( () => {
                    // 执行一些操作
                }, 1000)
                this.$once('hook:beforeDestroy', () => {            
                    clearInterval(timer);                                    
                })
            }
        },


这样就解决了上面所列的两个问题。类似的这种在离开页面时需要销毁的组件都可以采用此方法。

#### 11\. 本地开发跨域问题

在本地开发请求后端服务器接口的时候，都不可避免的会遇到跨域的问题。解决方法可以通过加一个`node`中间层或者`nginx`做反向代理。但是如果是用`vue-cli`搭建的项目，`vue-cli`在`config`中自带了一个`proxyTable`属性，可以配置这个属性解决跨域的问题。

    // config/index.js
    module.exports = {
      // ...
      dev: {
        proxyTable: {
          '/api': { // 遇到/api的接口都会走此代理,因此在调用接口时，需要在url中增加/api标识
            target: 'http://jsonplaceholder.typicode.com', // 真实的地址
            changeOrigin: true, // 是否启用跨域
            pathRewrite: {
              '^/api': '' // 将接口中的/api替换成''
            }
          }
        }
      }
    }


以上代码会将 `/api/posts/1`请求代理到 `http://jsonplaceholder.typicode.com/posts/1`。

如果不想在每一个接口中都手动增加`/api`标识，可以更改`axios`的默认配置`axios.defaults.baseURL = '/api'`，这样，`axios`会自动帮我们在`url`上加上`/api`的前缀。

**需要特别注意的是，更改完配置后需要重启server才会生效。**