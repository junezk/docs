# Vue 教程

很高兴你能来学习这套Vue2.0视频课程，现在越来越多的小公司开始使用Vue来作为前端开发框架了，而且Vue的生态也越来越好，无论你是前端的老手还是新手，都有必要认真并系统的学习，有可能的话尽量在工作中使用它了。

## [#](https://jspang.com/posts/2017/02/23/vue2-1.html#第1节：走起我的vue2-0)第1节：走起我的Vue2.0

很高兴你能来学习这套Vue2.0视频课程，现在越来越多的小公司开始使用Vue来作为前端开发框架了，而且Vue的生态也越来越好，无论你是前端的老手还是新手，都有必要认真并系统的学习，有可能的话尽量在工作中使用它了。



<iframe frameborder="0" width="100%" src="https://v.qq.com/iframe/player.html?vid=v0378yw6ger&amp;tiny=0&amp;auto=0" allowfullscreen="allowfullscreen" style="box-sizing: border-box; height: 35rem;"></iframe>
### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#学习这套课程你需要的前置知识：)学习这套课程你需要的前置知识：

1. HTML的基础知识，你需要达到中级水平，写前端页面的结构代码完全没有问题。
2. CSS的基础知识，最好做过半年以上的切图和布局，最好了解CSS3的知识。
3. Javascript的基础知识，对基本语法掌握，这个要求不高，因为遇到难的我会在视频中讲解。
4. node.js初级知识，只需要会npm的使用和项目初始化就可以了

如果你对上面三个知识还不了解，建议不要冒进学习，会影响你的学习热情。但你已经是科班出身或者在培训班毕业后想继续学习，是完全没有难度的。

学习预期：这套视频会有很多季，每一季会有几集构成，像美国的电视剧一样，从最基础一直讲到项目实战，你学完这套视频就可以直接上手项目，也希望你能找到一份不错的工作。

学习方法：学完视频之后，一定要动手写一遍，如果不写你真的到项目中一定写错或者写不出来，要在学习时就不断采坑。

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#下载vue2-0的两个版本：)下载Vue2.0的两个版本：

官方网站：

- 开发版本：包含完整的警告和调试模式
- 生产版本：删除了警告，进行了压缩

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#项目结构搭建)项目结构搭建

这个部分要视频中有详细讲解。

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#live-server使用)live-server使用

用npm进行全局安装

```text
npm install live-server -g
```

在项目目录中打开

```text
live-server
```

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#编写第一个helloworld代码：)编写第一个HelloWorld代码：

```xml
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>Helloworld</title>
</head>
<body>
    <h1>Hello World</h1>
    <hr>
    <div id="app">
        {{message}}
    </div>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            data:{
                message:'hello Vue!'
            }
        })
    </script>
</body>
</html>
```

## [#](https://jspang.com/posts/2017/02/23/vue2-1.html#第2节：v-if-v-else-v-show-指令)第2节：v-if v-else v-show 指令

通过上节课的学习，我们已经搭建好了开发环境，并且写出了一个简单的HelloWorld程序，我们今天主要学习v-if v-else 和 v-show。其实我们这一季学习的都是Vue2.0的内部指令。

<iframe frameborder="0" width="100%" src="https://v.qq.com/iframe/player.html?vid=b03780rmmvi&amp;tiny=0&amp;auto=0" allowfullscreen="allowfullscreen" style="box-sizing: border-box; height: 35rem;"></iframe>
### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#v-if的使用：)v-if的使用：

v-if:是vue 的一个内部指令，指令用在我们的html中。

v-if用来判断是否加载html的DOM，比如我们模拟一个用户登录状态，在用户登录后现实用户名称。

关键代码：

```html
 <div v-if="isLogin">你好，JSPang！</div>
```

完整html代码：

```text
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>v-if & v-show & v-else</title>
</head>
<body>
    <h1>v-if 判断是否加载</h1>
    <hr>
    <div id="app">
        <div v-if="isLogin">你好：JSPang</div>
        <div v-else>请登录后操作</div>
 
    </div>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            data:{
               isLogin:false
            }
        })
    </script>
</body>
</html>
```

这里我们在vue的data里定义了isLogin的值，当它为true时，网页就会显示：你好：JSPang，如果为false时，就显示请登录后操作。

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#_2-v-show的使用：)2.v-show的使用：

调整css中display属性，DOM已经加载，只是CSS控制没有显示出来。

```html
<div v-show="isLogin">你好：JSPang</div>
```

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#v-if-和v-show的区别：)v-if 和v-show的区别：

- v-if： 判断是否加载，可以减轻服务器的压力，在需要时加载。
- v-show：调整css dispaly属性，可以使客户端操作更加流畅。

## [#](https://jspang.com/posts/2017/02/23/vue2-1.html#第3节：v-for指令-：解决模板循环问题)第3节：v-for指令 ：解决模板循环问题

v-for指令是循环渲染一组data中的数组，v-for 指令需要以 item in items 形式的特殊语法，items 是源数据数组并且item是数组元素迭代的别名。

<iframe frameborder="0" width="100%" src="https://v.qq.com/iframe/player.html?vid=o03798eqe6j&amp;tiny=0&amp;auto=0" allowfullscreen="allowfullscreen" style="box-sizing: border-box; height: 35rem;"></iframe>
### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#一、基本用法：)一、基本用法：

模板写法

```html
 <li v-for="item in items">
        {{item}}
</li>
```

js写法

```javascript
var app=new Vue({
     el:'#app',
     data:{
         items:[20,23,18,65,32,19,54,56,41]
     }
})
```

完整代码：

```text
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>V-for 案例</title>
</head>
<body>
    <h1>v-for指令用法</h1>
    <hr>
    <div id="app">
       <ul>
           <li v-for="item in items">
                {{item}}
           </li>
       </ul>
    </div>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            data:{
                items:[20,23,18,65,32,19,54,56,41]
            }
        })
    </script>
</body>
</html>
```

这是一个最基础的循环，先在js里定义了items数组，然后在模板中用v-for循环出来，需要注意的是，你需要那个html标签循环，v-for就写在那个上边。

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#二、排序)二、排序

我们已经顺利的输出了我们定义的数组，但是我需要在输出之前给数组排个序，那我们就用到了Vue的computed:属性。

```javascript
computed:{
    sortItems:function(){
          return this.items.sort();
    }
}
```

我们在computed里新声明了一个对象sortItems，如果不重新声明会污染原来的数据源，这是Vue不允许的，所以你要重新声明一个对象。

如果一切顺利的话，你已经看到了结果，但是这个小程序还是有个小Bug的，现在我把数组修改成这样。

```text
items:[20,23,18,65,32,19,5,56,41]
```

我们把其中的54修改成了5，我们再看一下结果，发现排序结果并不是我们想要的。

我们可以自己编写一个方法sortNumber，然后传给我们的sort函数解决这个Bug。

```text
function sortNumber(a,b){
            return a-b
  }
```

用法

```text
computed:{
    sortItems:function(){
    return this.items.sort(sortNumber);
    }
 }
```

经过一番折腾，我们终于实现了真正的数字排序，这是在工作中非常常用的，一定要学好，记住。

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#三、对象循环输出)三、对象循环输出

我们上边循环的都是数组，那我们来看一个对象类型的循环是如何输出的。

我们先定义个数组，数组里边是对象数据

```javascript
students:[
  {name:'jspang',age:32},
  {name:'Panda',age:30},
  {name:'PanPaN',age:21},
  {name:'King',age:45}
]
```

在模板中输出

```javascript
<ul>
   <li v-for="student in students">
       {{student.name}} - {{student.age}}
   </li>
</ul>
```

加入索引序号：

```javascript
//数组对象方法排序:
function sortByKey(array,key){
    return array.sort(function(a,b){
      var x=a[key];
      var y=b[key];
      return ((x<y)?-1:((x>y)?1:0));
   });
}
```

有了数组的排序方法，在computed中进行调用排序

```javascript
sortStudent:function(){
     return sortByKey(this.students,'age');
}
```

注意：vue低版本中 data里面的items和computed里面可以一样，但是高版本，是不允许相同名称。有很多小伙伴踩到了这个坑，这里提醒学习的小伙伴，根据自己版本的不同，请修改代码。（感谢网友：tannnb的指正）。

## [#](https://jspang.com/posts/2017/02/23/vue2-1.html#第4节：v-text-v-html)第4节：v-text & v-html

我们已经会在html中输出data中的值了，我们已经用的是,这种情况是有弊端的，就是当我们网速很慢或者javascript出错时，会暴露我们的。Vue给我们提供的v-text,就是解决这个问题的。我们来看代码： 

```html
<span>{{ message }}</span>=<span v-text="message"></span><br/>
```

如果在javascript中写有html标签，用v-text是输出不出来的，这时候我们就需要用`v-html`标签了。

```html
<span v-html="msgHtml"></span>
```

双大括号会将数据解释为纯文本，而非HTML。为了输出真正的HTML，你就需要使用v-html 指令。 需要注意的是：在生产环境中动态渲染HTML是非常危险的，因为容易导致`XSS`攻击。所以只能在可信的内容上使用v-html，永远不要在用户提交和可操作的网页上使用。 完整代码：

```text
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>v-text & v-html 案例</title>
</head>
<body>
    <h1>v-text & v-html 案例</h1>
    <hr>
    <div id="app">
        <span>{{ message }}</span>=<span v-text="message"></span><br/>
        <span v-html="msgHtml"></span>
    </div>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            data:{
                message:'hello Vue!',
                msgHtml:'<h2>hello Vue!</h2>'
            }
        })
    </script>
</body>
</html>
```

## [#](https://jspang.com/posts/2017/02/23/vue2-1.html#第5节：v-on：绑定事件监听器)第5节：v-on：绑定事件监听器

v-on 就是监听事件，可以用v-on指令监听DOM事件来触发一些javascript代码。

<iframe frameborder="0" width="100%" src="https://v.qq.com/iframe/player.html?vid=k0380uowbyd&amp;tiny=0&amp;auto=0" allowfullscreen="allowfullscreen" style="box-sizing: border-box; height: 35rem; color: rgb(44, 62, 80); font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Roboto, Oxygen, Cantarell, &quot;Fira Sans&quot;, &quot;Droid Sans&quot;, &quot;Helvetica Neue&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, Dengxian, Simsun, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial;"></iframe>
一、使用绑定事件监听器，编写一个加分减分的程序。

![img](http://7xjyw1.com1.z0.glb.clouddn.com/20170227091542.png)

程序代码

```text
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>v-on事件监听器</title>
</head>
<body>
    <h1>v-on 事件监听器</h1>
    <hr>
    <div id="app">
       本场比赛得分： {{count}}<br/>
       <button v-on:click="jiafen">加分</button>
       <button v-on:click="jianfen">减分</button>
 
    </div>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            data:{
                count:1
            },
            methods:{
                jiafen:function(){
                    this.count++;
                },
                jianfen:function(){
                    this.count--;
                }
            }
        })
    </script>
</body>
</html>
```

我们的v-on 还有一种简单的写法，就是用@代替。

```text
<button @click="jianfen">减分</button>
```

我们除了绑定click之外，我们还可以绑定其它事件，比如键盘回车事件v-on:keyup.enter,现在我们增加一个输入框，然后绑定回车事件，回车后把文本框里的值加到我们的count上。 绑定事件写法：

```text
<input type="text" v-on:keyup.enter="onEnter" v-model="secondCount">
```

javascript代码：

```javascript
onEnter:function(){
     this.count=this.count+parseInt(this.secondCount);
}
```

因为文本框的数字会默认转变成字符串，所以我们需要用`parseInt()函数`进行整数转换。

你也可以根据键值表来定义键盘事件： ![img](http://7xjyw1.com1.z0.glb.clouddn.com/20170227001137.jpg)

## [#](https://jspang.com/posts/2017/02/23/vue2-1.html#第6节：v-model指令)第6节：v-model指令

这节课强烈建议你看视频，文字版你可能会看蒙！ 

v-model指令，我理解为绑定数据源。就是把数据绑定在特定的表单元素上，可以很容易的实现双向数据绑定。

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#一、我们来看一个最简单的双向数据绑定代码：)一、我们来看一个最简单的双向数据绑定代码：

html文件

```html
<div id="app">
    <p>原始文本信息：{{message}}</p>
    <h3>文本框</h3>
    <p>v-model:<input type="text" v-model="message"></p>
</div>
```

javascript代码：

```text
var app=new Vue({
  el:'#app',
  data:{
       message:'hello Vue!'
  }
 })
```

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#二、修饰符)二、修饰符

- .lazy：取代 imput 监听 change 事件。
- .number：输入字符串转为数字。
- .trim：输入去掉首尾空格。

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#三、文本区域加入数据绑定)三、文本区域加入数据绑定

```text
<textarea  < cols="30" rows="10" v-model="message"></textarea>
```

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#四、多选按钮绑定一个值)四、多选按钮绑定一个值

```html
<h3>多选按钮绑定一个值</h3>
<input type="checkbox" id="isTrue" v-model="isTrue">
<label for='isTrue'>{{isTrue}}</label>
```

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#五、多选绑定一个数组)五、多选绑定一个数组

```html
<h3>多选绑定一个数组</h3>
       <p>
            <input type="checkbox" id="JSPang" value="JSPang" v-model="web_Names">
            <label for="JSPang">JSPang</label><br/>
            <input type="checkbox" id="Panda" value="Panda" v-model="web_Names">
            <label for="JSPang">Panda</label><br/>
            <input type="checkbox" id="PanPan" value="PanPan" v-model="web_Names">
            <label for="JSPang">PanPan</label>
            <p>{{web_Names}}</p>
       </p>
```

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#六、单选按钮绑定数据)六、单选按钮绑定数据

```html
<h3>单选按钮绑定</h3>
<input type="radio" id="one" value="男" v-model="sex">
<label for="one">男</label>
<input type="radio" id="two" value="女" v-model="sex">
<label for="one">女</label>
<p>{{sex}}</p>
```

## [#](https://jspang.com/posts/2017/02/23/vue2-1.html#第7节：v-bind-指令)第7节：v-bind 指令

v-bind是处理HTML中的标签属性的，例如



就是一个标签，![img]()![img]()

<iframe frameborder="0" width="100%" src="https://v.qq.com/iframe/player.html?vid=j0380r6d4pe&amp;tiny=0&amp;auto=0" allowfullscreen="allowfullscreen" style="box-sizing: border-box; height: 35rem; color: rgb(44, 62, 80); font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Roboto, Oxygen, Cantarell, &quot;Fira Sans&quot;, &quot;Droid Sans&quot;, &quot;Helvetica Neue&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, Dengxian, Simsun, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial;"></iframe>
html文件：

```text
<div id="app">
    <img v-bind:src="imgSrc"  width="200px">
</div>
```

在html中我们用v-bind:src=”imgSrc”的动态绑定了src的值，这个值是在vue构造器里的data属性中找到的。

js文件：

```text
var app=new Vue({
    el:'#app',
    data:{
          imgSrc:'http://baidu.com/wp-content/uploads/2017/02/vue01-2.jpg'
     }
})
```

我们在data对象在中增加了imgSrc属性来供html调用。

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#v-bind-缩写)v-bind 缩写

```text
<!-- 完整语法 -->
<a v-bind:href="url"></a>
<!-- 缩写 -->
<a :href="url"></a>
```

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#绑定css样式)绑定CSS样式

在工作中我们经常使用v-bind来绑定css样式：

在绑定CSS样式是，绑定的值必须在vue中的data属性中进行声明。 1、直接绑定class样式

```html
<div :class="className">1、绑定classA</div>
```

2、绑定classA并进行判断，在isOK为true时显示样式，在isOk为false时不显示样式。

```html
<div :class="{classA:isOk}">2、绑定class中的判断</div>
```

3、绑定class中的数组

```text
<div :class="[classA,classB]">3、绑定class中的数组</div>
```

4、绑定class中使用三元表达式判断

```text
<div :class="isOk?classA:classB">4、绑定class中的三元表达式判断</div>
```

5、绑定style

```text
<div :style="{color:red,fontSize:font}">5、绑定style</div>
```

6、用对象绑定style样式

```text
<div :style="styleObject">6、用对象绑定style样式</div>
var app=new Vue({
   el:'#app',
   data:{
       styleObject:{
           fontSize:'24px',
           color:'green'
            }
        }
})
```

## [#](https://jspang.com/posts/2017/02/23/vue2-1.html#第8节：其他内部指令-v-pre-v-cloak-v-once)第8节：其他内部指令(v-pre & v-cloak & v-once)

<iframe frameborder="0" width="100%" src="https://v.qq.com/iframe/player.html?vid=y0380ywmwuf&amp;tiny=0&amp;auto=0" allowfullscreen="allowfullscreen" style="box-sizing: border-box; height: 35rem; color: rgb(44, 62, 80); font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Roboto, Oxygen, Cantarell, &quot;Fira Sans&quot;, &quot;Droid Sans&quot;, &quot;Helvetica Neue&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, Dengxian, Simsun, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial;"></iframe>
\>

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#v-pre指令)v-pre指令

在模板中跳过vue的编译，直接输出原始值。就是在标签中加入v-pre就不会输出vue中的data值了。

```text
<div v-pre>{{message}}</div>
```

这时并不会输出我们的message值，而是直接在网页中显示

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#v-cloak指令)v-cloak指令

在vue渲染完指定的整个DOM后才进行显示。它必须和CSS样式一起使用，

```text
[v-cloak] {
  display: none;
}
<div v-cloak>
  {{ message }}
</div>
```

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#v-once指令)v-once指令

在第一次DOM时进行渲染，渲染完成后视为静态内容，跳出以后的渲染过程。

```text
<div v-once>第一次绑定的值：{{message}}</div>
<div><input type="text" v-model="message"></div>
```

### [#](https://jspang.com/posts/2017/02/23/vue2-1.html#第一季总结：)第一季总结：

第一季我们学习了Vue中的指令，这也是Vue中最容易理解的部分，以前我也只是使用Vue，通过这个课程的制作，我也对Vue指令有了全面和更深刻的了解。对于小伙伴们的学习，我想说的是，你看完视频后一定要动手去敲写代码，这样你才能理解并记住。我们第二季的课程也已经在准备中了，第二季我们将讲解Vue的全局API。

测试一下

GitHub源码地址： https://github.com/shenghy/VueDemo

我们在第一季中学习了Vue的内部指令，在你掌握了第一季内容后，你可以学习本教程，第二季的教程会比第一季要难一些，组件的编写。

# 第二季 全局API(共9集)

## 第1节：Vue.directive 自定义指令


### 一、什么是全局API？

全局API并不在构造器里，而是先声明全局变量或者直接在Vue上定义一些新功能，Vue内置了一些全局API，比如我们今天要学习的指令Vue.directive。说的简单些就是，在构造器外部用Vue提供给我们的API函数来定义新的功能。

### 二、Vue.directive自定义指令

我们在第一季就学习了内部指令，我们也可以定义一些属于自己的指令，比如我们要定义一个v-jspang的指令，作用就是让文字变成绿色。

在自定义指令前我们写一个小功能，在页面上有一个数字为10，数字的下面有一个按钮，我们每点击一次按钮后，数字加1.

你不妨模仿下面的功能，在自己本地先写出这个效果。我用JSRun提供了预览和代码展示功能，你也可以在线调试。 

写好了这个功能，我们现在就自己定义一个全局的指令。我们这里使用Vue.directive( );

```text
Vue.directive('jspang',function(el,binding,vnode){
        el.style='color:'+binding.value;
});
```

可以看到数字已经变成了绿色，说明自定义指令起到了作用。可能您看这个代码还是有些不明白的，比如传入的三个参数到底是什么。

### 三、自定义指令中传递的三个参数

- el: 指令所绑定的元素，可以用来直接操作DOM。
- binding: 一个对象，包含指令的很多信息。
- vnode: Vue编译生成的虚拟节点。

### 四、自定义指令的生命周期

自定义指令有五个生命周期（也叫钩子函数），分别是 bind,inserted,update,componentUpdated,unbind

1. bind:只调用一次，指令第一次绑定到元素时调用，用这个钩子函数可以定义一个绑定时执行一次的初始化动作。
2. inserted:被绑定元素插入父节点时调用（父节点存在即可调用，不必存在于document中）。
3. update:被绑定于元素所在的模板更新时调用，而无论绑定值是否变化。通过比较更新前后的绑定值，可以忽略不必要的模板更新。
4. componentUpdated:被绑定元素所在模板完成一次更新周期时调用。
5. unbind:只调用一次，指令与元素解绑时调用。

```text
bind:function(){//被绑定
     console.log('1 - bind');
},
inserted:function(){//绑定到节点
      console.log('2 - inserted');
},
update:function(){//组件更新
      console.log('3 - update');
},
componentUpdated:function(){//组件更新完成
      console.log('4 - componentUpdated');
},
unbind:function(){//解绑
      console.log('1 - bind');
}
```

## 第2节：Vue.extend构造器的延伸

### 一、什么是Vue.extend

Vue.extend 返回的是一个“扩展实例构造器”,也就是预设了部分选项的Vue实例构造器。经常服务于Vue.component用来生成组件，可以简单理解为当在模板中遇到该组件名称作为标签的自定义元素时，会自动调用“扩展实例构造器”来生产组件实例，并挂载到自定义元素上。

由于我们还没有学习Vue的自定义组件，所以我们先看跟组件无关的用途。

### 二、自定义无参数标签

我们想象一个需求，需求是这样的，要在博客页面多处显示作者的网名，并在网名上直接有链接地址。我们希望在html中只需要写`<author></author>`，这和自定义组件很像，但是他没有传递任何参数，只是个静态标签。

我们的Vue.extend该登场了，我们先用它来编写一个扩展实例构造器。代码如下：

```text
var authorExtend = Vue.extend({
    template:"<p><a :href='authorUrl'>{{authorName}}</a></p>",
    data:function(){
    return{
          authorName:'JSPang',
          authorUrl:'http://www.jspang.com'
          }
    }
});
```

这时html中的标签还是不起作用的，因为扩展实例构造器是需要挂载的，我们再进行一次挂载。

```text
new authorExtend().$mount('author');
```

这时我们在html写`<author><author>`就是管用的。我们来看一下全部代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>vue.extend-扩展实例构造器</title>
</head>
<body>
    <h1>vue.extend-扩展实例构造器</h1>
    <hr>
    <author></author>
 
    <script type="text/javascript">
       var authorExtend = Vue.extend({
           template:"<p><a :href='authorUrl'>{{authorName}}</a></p>",
           data:function(){
               return{
                   authorName:'JSPang',
                   authorUrl:'http://www.jspang.com'
               }
           }
       });
       new authorExtend().$mount('author');
    </script>
</body>
</html>
```

最终实现结果： 

### 三、挂载到普通标签上

还可以通过HTML标签上的id或者class来生成扩展实例构造器，Vue.extend里的代码是一样的，只是在挂载的时候，我们用类似jquery的选择器的方法，来进行挂载就可以了。

```text
new authorExtend().$mount('#author');
```

##  第3节：Vue.set全局操作

Vue.set 的作用就是在构造器外部操作构造器内部的数据、属性或者方法。比如在vue构造器内部定义了一个count为1的数据，我们在构造器外部定义了一个方法，要每次点击按钮给值加1.就需要用到Vue.set。 

### 一、引用构造器外部数据：

什么是外部数据，就是不在Vue构造器里里的data处声明，而是在构造器外部声明，然后在data处引用就可以了。外部数据的加入让程序更加灵活，我们可以在外部获取任何想要的数据形式，然后让data引用。 看一个简单的代码：

```text
//在构造器外部声明数据
 var outData={
    count:1,
    goodName:'car'
};
var app=new Vue({
    el:'#app',
    //引用外部数据
    data:outData
})
```

二、在外部改变数据的三种方法：

1、用Vue.set改变

```text
function add(){
       Vue.set(outData,'count',4);
 }
```

2、用Vue对象的方法添加

```text
app.count++;
```

3、直接操作外部数据

```text
outData.count++;
```

其实这三种方式都可以操作外部的数据，Vue也给我们增加了一种操作外部数据的方法。

### 三、为什么要有Vue.set的存在?

由于Javascript的限制，Vue不能自动检测以下变动的数组。

- 当你利用索引直接设置一个项时，vue不会为我们自动更新。
- 当你修改数组的长度时，vue不会为我们自动更新。

看一段代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>Vue.set 全局操作</title>
</head>
<body>
    <h1>Vue.set 全局操作</h1>
    <hr>
    <div id="app">
        <ul>
            <li v-for=" aa in arr">{{aa}}</li>
        </ul>
       
    </div>
    <button onclick="add()">外部添加</button>
 
    <script type="text/javascript">
     
        function add(){
            console.log("我已经执行了");
           app.arr[1]='ddd';
           //Vue.set(app.arr,1,'ddd');
        }
        var outData={
            arr:['aaa','bbb','ccc']
        };
        var app=new Vue({
            el:'#app',
            data:outData
        })
    </script>
</body>
</html>
```

这时我们的界面是不会自动跟新数组的，我们需要用Vue.set(app.arr,1,’ddd’)来设置改变，vue才会给我们自动更新，这就是Vue.set存在的意义。

## 第4节：Vue的生命周期（钩子函数）

Vue一共有10个生命周期函数，我们可以利用这些函数在vue的每个阶段都进行操作数据或者改变内容。 

其实在Vue的官网有一张图已经很好的诠释了生命周期，我在这里就不再多讲了，直接贴图，然后上程序代码。 ![img](http://7xjyw1.com1.z0.glb.clouddn.com/vue-lifecycle.png)

我们直接来看一段代码：

```text
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>构造器的声明周期</title>
</head>
<body>
    <h1>构造器的声明周期</h1>
    <hr>
    <div id="app">
        {{message}}
        <p><button @click="jia">加分</button></p>
    </div>
        <button onclick="app.$destroy()">销毁</button>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            data:{
                message:1
            },
            methods:{
                jia:function(){
                    this.message ++;
                }
            },
            beforeCreate:function(){
                console.log('1-beforeCreate 初始化之后');
            },
            created:function(){
                console.log('2-created 创建完成');
            },
            beforeMount:function(){
                console.log('3-beforeMount 挂载之前');
            },
            mounted:function(){
                console.log('4-mounted 被创建');
            },
            beforeUpdate:function(){
                console.log('5-beforeUpdate 数据更新前');
            },
            updated:function(){
                console.log('6-updated 被更新后');
            },
            activated:function(){
                console.log('7-activated');
            },
            deactivated:function(){
                console.log('8-deactivated');
            },
            beforeDestroy:function(){
                console.log('9-beforeDestroy 销毁之前');
            },
            destroyed:function(){
                console.log('10-destroyed 销毁之后')
            }
 
        })
    </script>
</body>
</html>
```

## 第5节：Template 制作模版

### 一、直接写在选项里的模板

直接在构造器里的template选项后边编写。这种写法比较直观，但是如果模板html代码太多，不建议这么写。

javascript代码：

```text
var app=new Vue({
     el:'#app',
     data:{
         message:'hello Vue!'
      },
     template:`
        <h1 style="color:red">我是选项模板</h1>
     `
})
```

这里需要注意的是模板的标识不是单引号和双引号，而是，就是Tab上面的键。

### 二、写在template标签里的模板

这种写法更像是在写HTML代码，就算不会写Vue的人，也可以制作页面。

```text
   <template id="demo2">
             <h2 style="color:red">我是template标签模板</h2>
    </template>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            data:{
                message:'hello Vue!'
            },
            template:'#demo2'
        })
    </script>
```

### 三、写在script标签里的模板

这种写模板的方法，可以让模板文件从外部引入。

```text
<script type="x-template" id="demo3">
        <h2 style="color:red">我是script标签模板</h2>
    </script>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            data:{
                message:'hello Vue!'
            },
            template:'#demo3'
        })
    </script>
```

这节课我们学习了Template的三种写法，以后学习到vue-cli的时候还会学到一种xxx.vue的写法。

## [#](https://jspang.com/posts/2017/02/24/vue2-2.html#第6节：component-初识组件)第6节：Component 初识组件

前言（废话）：component组件是Vue学习的重点、重点、重点，重要的事情说三遍。所以你必须学好Vue component。其实组件就是制作自定义的标签，这些标签在HTML中是没有的。比如：，那我们就开始学习这种技巧吧。 

### [#](https://jspang.com/posts/2017/02/24/vue2-2.html#一、全局化注册组件)一、全局化注册组件

全局化就是在构造器的外部用Vue.component来注册，我们注册现在就注册一个的组件来体验一下。

```text
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>component-1</title>
</head>
<body>
    <h1>component-1</h1>
    <hr>
    <div id="app">
        <jspang></jspang>
    </div>
 
    <script type="text/javascript">
        //注册全局组件
        Vue.component('jspang',{
            template:`<div style="color:red;">全局化注册的jspang标签</div>`
        })
        var app=new Vue({
            el:'#app',
            data:{
            }
        })
    </script>
</body>
</html>
```

我们在javascript里注册了一个组件，在HTML中调用了他。这就是最简单的一个组件的编写方法，并且它可以放到多个构造器的作用域里。

### [#](https://jspang.com/posts/2017/02/24/vue2-2.html#二、局部注册组件局部)二、局部注册组件局部

局部注册组件局部注册组件和全局注册组件是向对应的，局部注册的组件只能在组件注册的作用域里进行使用，其他作用域使用无效。

```text
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>component-1</title>
</head>
<body>
    <h1>component-1</h1>
    <hr>
    <div id="app">
      <panda></panda>
    </div>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            components:{
                "panda":{
                    template:`<div style="color:red;">局部注册的panda标签</div>`
                }
            }
        })
    </script>
</body>
</html>
```

从代码中你可以看出局部注册其实就是写在构造器里，但是你需要注意的是，构造器里的components 是加s的，而全局注册是不加s的。

### [#](https://jspang.com/posts/2017/02/24/vue2-2.html#三、组件和指令的区别)三、组件和指令的区别

组件注册的是一个标签，而指令注册的是已有标签里的一个属性。在实际开发中我们还是用组件比较多，指令用的比较少。因为指令看起来封装的没那么好，这只是个人观点。

## [#](https://jspang.com/posts/2017/02/24/vue2-2.html#第7节：component-组件props-属性设置)第7节：Component 组件props 属性设置

props选项就是设置和获取标签上的属性值的，例如我们有一个自定义的组件,这时我们想给他加个标签属性写成 意思就是熊猫来自中国，当然这里的China可以换成任何值。定义属性的选项是props。 

### [#](https://jspang.com/posts/2017/02/24/vue2-2.html#一、定义属性并获取属性值)一、定义属性并获取属性值

定义属性我们需要用props选项，加上数组形式的属性名称，例如：props:[‘here’]。在组件的模板里读出属性值只需要用插值的形式，例如.

```text
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>component-2</title>
</head>
<body>
    <h1>component-2</h1>
    <hr>
    <div id="app">
      <panda here="China"></panda>
    </div>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            components:{
                "panda":{
                    template:`<div style="color:red;">Panda from {{ here }}.</div>`,
                    props:['here']
                }
            }
        })
    </script>
</body>
</html>
```

上面的代码定义了panda的组件，并用props设置了here的属性值，在here属性值里传递了China给组件。 最后输出的结果是红色字体的Panda from China.

### [#](https://jspang.com/posts/2017/02/24/vue2-2.html#二、属性中带’-‘的处理方式)二、属性中带’-‘的处理方式

我们在写属性时经常会加入’-‘来进行分词，比如：，那这时我们在props里如果写成props:[‘form-here’]是错误的，我们必须用小驼峰式写法props:[‘formHere’]。

html文件：

```text
<panda from-here="China"></panda>
```

javascript文件：

```text
 var app=new Vue({
            el:'#app',
            components:{
                "panda":{
                    template:`<div style="color:red;">Panda from {{ here }}.</div>`,
                    props:['fromHere']
                }
            }
        })
```

PS：因为这里有坑，所以还是少用-为好

### [#](https://jspang.com/posts/2017/02/24/vue2-2.html#三、在构造器里向组件中传值)三、在构造器里向组件中传值

把构造器中data的值传递给组件，我们只要进行绑定就可以了。就是我们第一季学的v-bind:xxx.

我们直接看代码:

Html文件：

```text
<panda v-bind:here="message"></panda>
```

javascript文件：

```text
 var app=new Vue({
            el:'#app',
            data:{
               message:'SiChuan' 
            },
            components:{
                "panda":{
                    template:`<div style="color:red;">Panda from {{ here }}.</div>`,
                    props:['here']
                }
            }
        })
```

## 第8节：Component 父子组件关系

在实际开发中我们经常会遇到在一个自定义组件中要使用其他自定义组件，这就需要一个父子组件关系。 

### [#](https://jspang.com/posts/2017/02/24/vue2-2.html#一、构造器外部写局部注册组件)一、构造器外部写局部注册组件

上面上课我们都把局部组件的编写放到了构造器内部，如果组件代码量很大，会影响构造器的可读性，造成拖拉和错误。

我们把组件编写的代码放到构造器外部或者说单独文件。

我们需要先声明一个对象,对象里就是组件的内容。

```text
var jspang = {
   template:`<div>Panda from China!</div>`
}
```

声明好对象后在构造器里引用就可以了。

```javascript
components:{
    "jspang":jspang
}
```

html中引用

```text
 <jspang></jspang>
```

二、父子组件的嵌套 我们先声明一个父组件，比如叫jspang，然后里边我们加入一个city组件，我们来看这样的代码如何写。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>component-3</title>
</head>
<body>
    <h1>component-3</h1>
    <hr>
    <div id="app">
      <jspang></jspang>  
    </div>
    <script type="text/javascript">
       var city={
           template:`<div>Sichuan of China</div>`
       }
        var jspang = {
            template:`<div>
                    <p> Panda from China!</p>
                    <city></city>
            </div>`,
            components:{
                "city":city
            }
        }
        var app=new Vue({
            el:'#app',
            components:{
                "jspang":jspang
            }
           
        })
    </script>
</body>
</html>
```

## 第9节：Component 标签

标签是Vue框架自定义的标签，它的用途就是可以动态绑定我们的组件，根据数据的不同更换不同的组件。 

1.我们先在构造器外部定义三个不同的组件，分别是componentA,componentB和componentC.

```text
 var componentA={
     template:`<div>I'm componentA</div>`
}
 var componentB={
      template:`<div>I'm componentB</div>`
}
var componentC={
    template:`<div>I'm componentC</div>`
}
```

2.我们在构造器的components选项里加入这三个组件。

```text
components:{
    "componentA":componentA,
    "componentB":componentB,
    "componentC":componentC,
}
```

3.我们在html里插入component标签，并绑定who数据，根据who的值不同，调用不同的组件。

```text
<component v-bind:is="who"></component>
```

这就是我们的组件标签的基本用法。我们提高以下，给页面加个按钮，每点以下更换一个组件。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>component-4</title>
</head>
<body>
    <h1>component-4</h1>
    <hr>
    <div id="app">
       <component v-bind:is="who"></component>
       <button @click="changeComponent">changeComponent</button>
    </div>
 
    <script type="text/javascript">
        var componentA={
            template:`<div style="color:red;">I'm componentA</div>`
        }
        var componentB={
            template:`<div style="color:green;">I'm componentB</div>`
        }
        var componentC={
            template:`<div style="color:pink;">I'm componentC</div>`
        }
       
        var app=new Vue({
            el:'#app',
            data:{
                who:'componentA'
            },
            components:{
                "componentA":componentA,
                "componentB":componentB,
                "componentC":componentC,
            },
            methods:{
                changeComponent:function(){
                    if(this.who=='componentA'){
                        this.who='componentB';
                    }else if(this.who=='componentB'){
                        this.who='componentC';
                    }else{
                        this.who='componentA';
                    }
                }
            }
        })
    </script>
</body>
</html>
```

# 第三季 选项（共6集）

前言：Vue2.0视频教程已经出了两季，这些都是基础，为的就是能让新手快速进入。这一季讲的是基础中的选项，选项就是在Vue构造器里的配置功能的前缀（Vue已经给我们定义好了），Vue有很多选项，我们将在这一级教程中一一介绍。

## [#](https://jspang.com/posts/2017/03/26/vue2-3.html#第1节：propsdata-option-全局扩展的数据传递)第1节：propsData Option 全局扩展的数据传递

propsData 不是和属性有关，他用在全局扩展时进行传递数据。先回顾一下全局扩展的知识，作一个



的扩展标签出来。实际我们并比推荐用全局扩展的方式作自定义标签，我们学了组件，完全可以使用组件来做，这里只是为了演示propsData的用法。 

<iframe frameborder="0" width="100%" src="https://v.qq.com/iframe/player.html?vid=c0387wtdk8e&amp;tiny=0&amp;auto=0" allowfullscreen="allowfullscreen" style="box-sizing: border-box; height: 35rem; color: rgb(44, 62, 80); font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Roboto, Oxygen, Cantarell, &quot;Fira Sans&quot;, &quot;Droid Sans&quot;, &quot;Helvetica Neue&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, Dengxian, Simsun, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial;"></iframe>


代码如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>PropsData Option Demo</title>
</head>
<body>
    <h1>PropsData Option Demo</h1>
    <hr>
    <header></header>
 
    <script type="text/javascript">
       var  header_a = Vue.extend({
           template:`<p>{{message}}</p>`,
           data:function(){
               return {
                   message:'Hello,I am Header'
               }
           }
       }); 
       new header_a().$mount('header');
    </script>
</body>
</html>
```

扩展标签已经做好了，这时我们要在挂载时传递一个数字过去，我们就用到了propsData。 我们用propsData三步解决传值： 1.在全局扩展里加入props进行接收。propsData:{a:1} 2.传递时用propsData进行传递。props:[‘a’] 3.用插值的形式写入模板。 完整代码：

```javascript
var  header_a = Vue.extend({
    template:`<p>{{message}}-{{a}}</p>`,
    data:function(){
        return {
            message:'Hello,I am Header'
        }
    },
    props:['a']
}); 
new header_a({propsData:{a:1}}).$mount('header');
```

总结：propsData在实际开发中我们使用的并不多，我们在后边会学到Vuex的应用，他的作用就是在单页应用中保持状态和数据的。

## [#](https://jspang.com/posts/2017/03/26/vue2-3.html#第2节：computed-option-计算选项)第2节：computed Option 计算选项

computed 的作用主要是对原数据进行改造输出。改造输出：包括格式的编辑，大小写转换，顺序重排，添加符号……。 

### [#](https://jspang.com/posts/2017/03/26/vue2-3.html#一、格式化输出结果：)一、格式化输出结果：

我们先来做个读出价格的例子：我们读书的原始数据是price:100 但是我们输出给用户的样子是（￥100元）。 主要的javascript代码：

```javascript
computed:{
    newPrice:function(){
        return this.price='￥' + this.price + '元';
    }
}
```

全部代码:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>Computed Option 计算选项</title>
</head>
<body>
    <h1>Computed Option 计算选项</h1>
    <hr>
    <div id="app">
        {{newPrice}}
    </div>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            data:{
                price:100
            },
            computed:{
                newPrice:function(){
                    return this.price='￥' + this.price + '元';
                }
            }
        })
    </script>
</body>
</html>
```

现在输出的结果就是：￥100元。

### [二、用计算属性反转数组](https://jspang.com/posts/2017/03/26/vue2-3.html#二、用计算属性反转数组)

例如：我们得到了一个新闻列表的数组，它的顺序是安装新闻时间的顺序正序排列的，也就是早反生的新闻排在前面。这是反人类的，我们需要给他反转。这时我们就可以用到我们的计算属性了。

没有排序的新闻列表，是安装日期正序排列的。

```javascript
var newsList=[
    {title:'香港或就“装甲车被扣”事件追责 起诉涉事运输公司',date:'2017/3/10'},
    {title:'日本第二大准航母服役 外媒：针对中国潜艇',date:'2017/3/12'},
    {title:'中国北方将有明显雨雪降温天气 南方阴雨持续',date:'2017/3/13'},
    {title:'起底“最短命副市长”：不到40天落马，全家被查',date:'2017/3/23'},
];
```

我们希望输出的结果：

- 起底“最短命副市长”：不到40天落马，全家被查-2017/3/23
- 中国北方将有明显雨雪降温天气 南方阴雨持续-2017/3/13
- 日本第二大准航母服役 外媒：针对中国潜艇-2017/3/12
- 香港或就“装甲车被扣”事件追责 起诉涉事运输公司-2017/3/10 我们的在computed里的javascript代码：我们用js原生方法给数组作了反转。

```javascript
computed:{
    reverseNews:function(){
        return this.newsList.reverse();
    }
}
```

总结：computed 属性是非常有用，在输出数据前可以轻松的改变数据。所以说这节课的代码必须要多敲几遍，加深印象。

## [#](https://jspang.com/posts/2017/03/26/vue2-3.html#第3节：methods-option-方法选项)第3节：Methods Option 方法选项

在以前的学习中，已经大量的使用了构造器里的methods选项，但是并没有仔细和系统的讲解过，这节课我们用点时间把methods这个选项涉及的东西都讲一讲。

<iframe frameborder="0" width="100%" src="https://v.qq.com/iframe/player.html?vid=w0388d4eq9l&amp;tiny=0&amp;auto=0" allowfullscreen="allowfullscreen" style="box-sizing: border-box; height: 35rem; color: rgb(44, 62, 80); font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Roboto, Oxygen, Cantarell, &quot;Fira Sans&quot;, &quot;Droid Sans&quot;, &quot;Helvetica Neue&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, Dengxian, Simsun, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial;"></iframe>


我们还是复习一下最简单的使用方法，一个数字，每点击一下按钮加1.（做过很多次了，你们可以先不看代码和视频自己试着写一下）

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>methods Option</title>
</head>
<body>
    <h1>methods Option</h1>
    <hr>
    <div id="app">
        {{ a }}
        <p><button @click="add">add</button></p>
    </div>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            data:{
                a:1
            },
            methods:{
                add:function(){
                    this.a++
                }
            }
        })
    </script>
</body>
</html>
```

### [#](https://jspang.com/posts/2017/03/26/vue2-3.html#一、methods中参数的传递)一、methods中参数的传递

使用方法和正常的javascript传递参数的方法一样，分为两部：

- 在methods的方法中进行声明，比如我们给add方法加上一个num参数，就要写出`add:function(num){}`.
- 调用方法时直接传递，比如我们要传递2这个参数，我们在button上就直接可以写。`<button @click=”add(2)”></button>`.

现在知道了加参数的方法，看一段完整的代码，代码中给add添加了num参数，并在按钮上调用传递了。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>methods Option</title>
</head>
<body>
    <h1>methods Option</h1>
    <hr>
    <div id="app">
        {{ a }}
        <p><button @click="add(2)">add</button></p>
    </div>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            data:{
                a:1
            },
            methods:{
                add:function(num){
                    if(num!=''){this.a+=num}
                    else{this.a++}
                }
            }
        })
    </script>
</body>
</html>
```

这时，再点击按钮是每次加2个数字。

### [#](https://jspang.com/posts/2017/03/26/vue2-3.html#二、methods中的-event参数)二、methods中的$event参数

传递的$event参数都是关于你点击鼠标的一些事件和属性。我们先看看传递的方法。

传递：

```text
<button @click=”add(2,$event)”>add</button> 。
```

我们这时候可以打印一下，看看event到底是个怎样的对象。你会发现，它包含了大部分鼠标事件的属性。 ![img](http://7xjyw1.com1.z0.glb.clouddn.com/web20170326113703.png)

### [#](https://jspang.com/posts/2017/03/26/vue2-3.html#native-给组件绑定构造器里的原生事件。)native 给组件绑定构造器里的原生事件。

在实际开发中经常需要把某个按钮封装成组件，然后反复使用，如何让组件调用构造器里的方法，而不是组件里的方法。就需要用到我们的.native修饰器了。

现在我们把我们的add按钮封装成组件：

声明btn对象：

```javascript
var btn={
    template:`<button>组件Add</button>`     
}
```

在构造器里声明：

```javascript
components:{
    "btn":btn
 }
```

用.native修饰器来调用构造器里的add方法

```html
<p><btn @click.native="add(3)"></btn></p>
```

### [#](https://jspang.com/posts/2017/03/26/vue2-3.html#四、作用域外部调用构造器里的方法)四、作用域外部调用构造器里的方法

这种不经常使用，如果你出现了这种情况，说明你的代码组织不够好。

```html
  <button onclick="app.add(4)" >外部调用构造器里的方法</button>
```

## [#](https://jspang.com/posts/2017/03/26/vue2-3.html#第4节：watch-选项-监控数据)第4节：Watch 选项 监控数据

数据变化的监控经常使用，我们可以先来看一个简单的数据变化监控的例子。例如天气预报的穿衣指数，它主要是根据温度来进行提示的，当然还有其它的，咱们就不考虑了。 

### [#](https://jspang.com/posts/2017/03/26/vue2-3.html#一、看一个监控变化的案例)一、看一个监控变化的案例

温度大于26度时，我们建议穿T恤短袖，温度小于26度大于0度时，我们建议穿夹克长裙，温度小于0度时我们建议穿棉衣羽绒服。

先来模拟一个温度变化的情况：我们使用按钮来加减温度。

二、用实例属性写watch监控

有些时候我们会用实例属性的形式来写watch监控。也就是把我们watch卸载构造器的外部，这样的好处就是降低我们程序的耦合度，使程序变的灵活。

```text
app.$watch('xxx',function(){})
```

还是上边的案例我们改成实例方法的模式。

```javascript
app.$watch('temperature',function(newVal,oldVal){
    if(newVal>=26){
        this.suggestion=suggestion[0];
    }else if(newVal<26 && newVal >=0)
    {
        this.suggestion=suggestion[1];
    }else{
        this.suggestion=suggestion[2];
    }
 
})
```

效果和上面是一样的。

## [#](https://jspang.com/posts/2017/03/26/vue2-3.html#第5节：mixins-混入选项操作)第5节：Mixins 混入选项操作

<iframe frameborder="0" width="100%" src="https://v.qq.com/iframe/player.html?vid=a038878er31&amp;tiny=0&amp;auto=0" allowfullscreen="allowfullscreen" style="box-sizing: border-box; height: 35rem; color: rgb(44, 62, 80); font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Roboto, Oxygen, Cantarell, &quot;Fira Sans&quot;, &quot;Droid Sans&quot;, &quot;Helvetica Neue&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, Dengxian, Simsun, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial;"></iframe>


Mixins一般有两种用途：

1. 在你已经写好了构造器后，需要增加方法或者临时的活动时使用的方法，这时用混入会减少源代码的污染。
2. 很多地方都会用到的公用方法，用混入的方法可以减少代码量，实现代码重用。

### [#](https://jspang.com/posts/2017/03/26/vue2-3.html#一、mixins的基本用法)一、Mixins的基本用法

我们现在有个数字点击递增的程序，假设已经完成了，这时我们希望每次数据变化时都能够在控制台打印出提示：“数据发生变化”.

代码实现过程:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>Mixins Option Demo</title>
</head>
<body>
    <h1>Mixins Option Demo</h1>
    <hr>
    <div id="app">
        <p>num:{{ num }}</p>
        <P><button @click="add">增加数量</button></P>
    </div>
 
    <script type="text/javascript">
        //额外临时加入时，用于显示日志
        var addLog={
            updated:function(){
                console.log("数据放生变化,变化成"+this.num+".");
            }
        }
        var app=new Vue({
            el:'#app',
            data:{
                num:1
            },
            methods:{
                add:function(){
                    this.num++;
                }
            },
            mixins:[addLog]//混入
        })
    </script>
</body>
</html>
```

### [#](https://jspang.com/posts/2017/03/26/vue2-3.html#mixins的调用顺序)mixins的调用顺序

从执行的先后顺序来说，都是混入的先执行，然后构造器里的再执行，需要注意的是，这并不是方法的覆盖，而是被执行了两边。

在上边的代码的构造器里我们也加入了updated的钩子函数：

```text
updated:function(){
      console.log("构造器里的updated方法。")
},
```

这时控制台输出的顺序是：

```text
mixins数据放生变化,变化成2.
构造器里的updated方法。
```

PS：当混入方法和构造器的方法重名时，混入的方法无法展现，也就是不起作用。

### [#](https://jspang.com/posts/2017/03/26/vue2-3.html#三、全局api混入方式)三、全局API混入方式

我们也可以定义全局的混入，这样在需要这段代码的地方直接引入js，就可以拥有这个功能了。我们来看一下全局混入的方法：

```text
Vue.mixin({
    updated:function(){
        console.log('我是全局被混入的');
    }
})
```

PS：全局混入的执行顺序要前于混入和构造器里的方法。

## [#](https://jspang.com/posts/2017/03/26/vue2-3.html#第6节：extends-option-扩展选项)第6节：Extends Option 扩展选项

通过外部增加对象的形式，对构造器进行扩展。它和我们上节课讲的混入非常的类似。

<iframe frameborder="0" width="100%" src="https://v.qq.com/iframe/player.html?vid=l03884omoq0&amp;tiny=0&amp;auto=0" allowfullscreen="allowfullscreen" style="box-sizing: border-box; height: 35rem; color: rgb(44, 62, 80); font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Roboto, Oxygen, Cantarell, &quot;Fira Sans&quot;, &quot;Droid Sans&quot;, &quot;Helvetica Neue&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, Dengxian, Simsun, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial;"></iframe>


### [#](https://jspang.com/posts/2017/03/26/vue2-3.html#一、extends我们来看一个扩展的实例。)一、extends我们来看一个扩展的实例。

```text
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>Extends Optin Demo</title>
</head>
<body>
    <h1>Extends Optin Demo</h1>
    <hr>
    <div id="app">
        {{message}}
        <p><button @click="add">add</button></p>
    </div>
 
    <script type="text/javascript">
        var bbb={
            created:function(){
                console.log("我是被扩展出来的");
            },
            methods:{
                add:function(){
                    console.log('我是被扩展出来的方法！');
                }
            }
        };
        var app=new Vue({
            el:'#app',
            data:{
                message:'hello Vue!'
            },
            methods:{
                add:function(){
                    console.log('我是原生方法');
                }
            },
            extends:bbb
        })
    </script>
</body>
</html>
```

### 二、delimiters 选项

因为这节课内容比较少，所以我们把要讲的最后一个选项一起讲了。delimiters的作用是改变我们插值的符号。Vue默认的插值是双大括号{{}}。但有时我们会有需求更改这个插值的形式。

```text
delimiters:['${','}']
```

现在我们的插值形式就变成了${}。这季的内容就这些了，我们下季见吧。

# 第四季 实例和内置组件（共4集）

概述：实例就是在构造器外部操作构造器内部的属性选项或者方法，就叫做实例？实例的作用就是给原生的或者其他javascript框架一个融合的接口或者说是机会，让Vue和其他框架一起使用。

## [#](https://jspang.com/posts/2017/04/09/vue2-4.html#第1节：实例入门-实例属性)第1节：实例入门-实例属性

<iframe frameborder="0" width="100%" src="https://v.qq.com/iframe/player.html?vid=e03918t5wy2&amp;tiny=0&amp;auto=0" allowfullscreen="allowfullscreen" style="box-sizing: border-box; height: 35rem; color: rgb(44, 62, 80); font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Roboto, Oxygen, Cantarell, &quot;Fira Sans&quot;, &quot;Droid Sans&quot;, &quot;Helvetica Neue&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, Dengxian, Simsun, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial;"></iframe>


### [#](https://jspang.com/posts/2017/04/09/vue2-4.html#vue和jquery-js一起使用)Vue和Jquery.js一起使用

1、下载并引入jquery框架

下载可以去官网进行下载，我这里使用的版本是3.1.1，下载好后在需要的页面引入就可以了。当然你还有很多其它的方法引入jquery，只要可以顺利引入就可以了。

```html
<script < type="text/javascript" src="../assets/js/jquery-3.1.1.min.js"></script>
```

试着作一个案例，在DOM被挂载后修改里边的内容。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <script type="text/javascript" src="../assets/js/jquery-3.1.1.min.js"></script>
    <title>Early Examples Demo</title>
</head>
<body>
    <h1>Early Examples Demo</h1>
    <hr>
    <div id="app">
        {{message}}
    </div>
 
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            data:{
                message:'hello Vue!'
            },
            //在Vue中使用jQuery
            mounted:function(){
                $('#app').html('我是jQuery!');
            }
        })
    </script>
</body>
</html>
```

现在页面显示是：我是jQuery，而不是hello Vue了。

### [#](https://jspang.com/posts/2017/04/09/vue2-4.html#二、实例调用自定义方法)二、实例调用自定义方法

在Vue的构造器里我们写一个add方法，然后我们用实例的方法调用它。

构造器里的add方法：

```javascript
methods:{
    add:function(){
        console.log("调用了Add方法");
    }
}
```

实例调用：

```javascript
app.add();
```

PS：我们有可能把app.add()的括号忘记或省略，这时候我们得到的就是方法的字符串，但是并没有执行，所以必须要加上括号。

作业：利用console.log把app打印出来，看看里边到底包含了哪些属性和方法。因为里边的属性很多，这里只是简单的介绍。

## [#](https://jspang.com/posts/2017/04/09/vue2-4.html#第2节：实例方法)第2节：实例方法

<iframe frameborder="0" width="100%" src="https://v.qq.com/iframe/player.html?vid=b0391n1b72x&amp;tiny=0&amp;auto=0" allowfullscreen="allowfullscreen" style="box-sizing: border-box; height: 35rem; color: rgb(44, 62, 80); font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, Roboto, Oxygen, Cantarell, &quot;Fira Sans&quot;, &quot;Droid Sans&quot;, &quot;Helvetica Neue&quot;, &quot;Hiragino Sans GB&quot;, &quot;Microsoft YaHei&quot;, Dengxian, Simsun, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial;"></iframe>


### [#](https://jspang.com/posts/2017/04/09/vue2-4.html#一、-mount方法)一、$mount方法

$mount方法是用来挂载我们的扩展的，我们先来复习一下扩展的写法。

这里我们作了jspang的扩展，然后用$mount的方法把jspang挂载到DOM上，我们也生成了一个Vue的实例，直接看代码。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>Examples Method Demo</title>
</head>
<body>
    <h1>Examples Method Demo</h1>
    <hr>
    <div id="app">
        {{message}}
    </div>
    
    <script type="text/javascript">
      var jspang = Vue.extend({
          template:`<p>{{message}}</p>`,
          data:function(){
              return {
                  message:'Hello ,I am JSPang'
              }
          }
      })
      var vm = new jspang().$mount("#app")
    </script>
</body>
</html>
```

这段代码我们在学习extends的时候已经写过一次，这里就不作过多解释了。

### [#](https://jspang.com/posts/2017/04/09/vue2-4.html#二、-destroy-卸载方法)二、$destroy() 卸载方法

用$destroy()进行卸载。

我写了一个button按钮，点击后卸载整个挂载。

html：

```html
<p><button onclick="destroy()">卸载</button></p>
```

javascript

```javascript
function destroy(){
   vm.$destroy();
}
```

PS:$destroy()后边必须要有括号，没有括号是无用的。

### [#](https://jspang.com/posts/2017/04/09/vue2-4.html#三、-forceupdate-更新方法)三、$forceUpdate() 更新方法

```javascript
vm.$forceUpdate()
```

### [#](https://jspang.com/posts/2017/04/09/vue2-4.html#四、-nexttick-数据修改方法)四、$nextTick() 数据修改方法

当Vue构造器里的data值被修改完成后会调用这个方法，也相当于一个钩子函数吧，和构造器里的updated生命周期很像。

```javascript
function tick(){
    vm.message="update message info ";
    vm.$nextTick(function(){
        console.log('message更新完后我被调用了');
    })
}
```

### [#](https://jspang.com/posts/2017/04/09/vue2-4.html#安装vue的控制台调试工具。)安装Vue的控制台调试工具。

每个人的安装方法不同，不作太多的介绍，可能需要你科学上网。

## [#](https://jspang.com/posts/2017/04/09/vue2-4.html#第3节：实例事件)第3节：实例事件

实例事件就是在构造器外部写一个调用构造器内部的方法。这样写的好处是可以通过这种写法在构造器外部调用构造器内部的数据。 

我们还是写一个点击按钮，持续加1的例子。

一、$on 在构造器外部添加事件。

```javascript
app.$on('reduce',function(){
    console.log('执行了reduce()');
    this.num--;
});
```

$on接收两个参数，第一个参数是调用时的事件名称，第二个参数是一个匿名方法。

如果按钮在作用域外部，可以利用$emit来执行。

```javascript
//外部调用内部事件
function reduce(){
    app.$emit('reduce');
}
```

二、$once执行一次的事件

```javascript
app.$once('reduceOnce',function(){
    console.log('只执行一次的方法');
    this.num--;
 
});
```

三、$off关闭事件

```javascript
//关闭事件
function off(){
   app.$off('reduce');
}
```

## [#](https://jspang.com/posts/2017/04/09/vue2-4.html#第4节：内置组件-slot讲解)第4节：内置组件 -slot讲解

slot是标签的内容扩展，也就是说你用slot就可以在自定义组件时传递给组件内容，组件接收内容并输出。 

先来定义一个的组件，这个组件用来显示博主的一些信息。

我们在Vue 构造器里的data中给出了信息，信息如下：（博客地址，网名，使用技能）

```javascript
data:{
    jspangData:{
        bolgUrl:'http://jspang.com',
        netName:'技术胖',
        skill:'Web前端'
    }
},
```

我们用`<template></template>`标签的方式定义了组件：

```html
<template id="tmp">
    <div>
        <p>博客地址：</p>
        <p>网名：</p>
        <p>技术类型：</p>
        
    </div>
</template>
```

我们现在就可以用slot功能让组件接收传递过来的值，并在模板中接收显示。

slot的使用需要两步： 1、在HTML的组件中用slot属性传递值。

```html
<jspang>
    <span slot="bolgUrl">{{jspangData.bolgUrl}}</span>    
    <span slot="netName">{{jspangData.netName}}</span>    
    <span slot="skill">{{jspangData.skill}}</span>    
</jspang>
```

2、在组件模板中用标签接收值。

```html
<template id="tmp">
    <div>
        <p>博客地址：<slot name="bolgUrl"></slot></p>
        <p>网名：<slot name="netName"></slot></p>
        <p>技术类型：<slot name="skill"></slot></p>
        
    </div>
</template>
```

我们贴出这个案例的全部代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="../assets/js/vue.js"></script>
    <title>Slot content extend Demo</title>
</head>
<body>
    <h1>Slot content extend Demo</h1>
    <hr>
    <div id="app">
    <jspang>
        <span slot="bolgUrl">{{jspangData.bolgUrl}}</span>    
        <span slot="netName">{{jspangData.netName}}</span>    
        <span slot="skill">{{jspangData.skill}}</span>    
    </jspang>
    </div>
 
 
<template id="tmp">
    <div>
        <p>博客地址：<slot name="bolgUrl"></slot></p>
        <p>网名：<slot name="netName"></slot></p>
        <p>技术类型：<slot name="skill"></slot></p>
        
    </div>
</template>
 
    <script type="text/javascript">
        var jspang={
            template:'#tmp'
        }
 
        var app=new Vue({
            el:'#app',
            data:{
               jspangData:{
                   bolgUrl:'http://jspang.com',
                   netName:'技术胖',
                   skill:'Web前端'
               }
            },
            components:{
                "jspang":jspang
            }
        })
    </script>
</body>
</html>
```