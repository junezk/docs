# Vue 2.0 + Axios + Vue Router 实现CNode社区

这篇文章主要参考组件的设计。

这是我学习Vue之后的上手的第一个作品，还望大家多多指点。

预览[地址](https://link.juejin.im?target=https%3A%2F%2Forangesam.github.io%2FCNode%2Fdist%2Findex.html)          源码[地址](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FOrangeSAM%2FCNode) **

项目技术栈  

*   **Vue2.0**：前端页面展示

*   **Vue-Router**：页面路由切换

*   **Axios**：一个基于 `Promise` 的 HTTP 库，向后端发起请求

*   **ES6**：箭头函数、Promise等等语法很好用

*   **Webpack**：vue init webpck

*   **CSS3**：CSS3过渡动画及样式


##### TODO

功能上：发帖，收藏，回复，点赞，

实现上：使用Vuex共享各组件间的状态

##### 项目截图在文章末尾。

以下为对CNode社区提供的部分API、实现的各个组件介绍以及项目编码中遇到的坑而又是如何解决的的总结。

#### API介绍

![](https://user-gold-cdn.xitu.io/2019/3/3/169425f358f39ff0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

这里由于API内容过多，是以缩略形式展示，需要完整的图，请到项目根目录static文件夹中获取。

#### 组件介绍

##### pagination 分页组件

这个组件HTML和CSS部分的代码相对简单，最重要的是理清各按钮的排布逻辑。比如在点击页码5时，页码1 就应该去除且同时页码6显示在页码5之后；还有比如在目前页码为1的情况下，需要禁止上一页的执行。以下是组件template中的代码。

      <template>
        <div class="pagination">
          <div class="wrap">
            <span @click="changeBtn">首页</span>
            <span @click="changeBtn">上一页</span>
            <span v-if="judge">...</span>
            <span
              v-for="(btn,index) in pagebtns"
              :key="index"
              :class="[{curtPage:btn==curtPage},'pagebtn']"
              @click="changeBtn(btn)"
            >{{btn}}</span>
            <span @click="changeBtn">下一页</span>
          </div>
        </div>
      </template>

第三个span是用来判断当前页码是否大于4，如果大于4就应该用显示`...`表明之前仍有页码。

添加了`v-for`指令的span是用来循环展示页码的。`:class`指令使用数组语法动态绑定类名。如果该页码即为当前页，那么value值为true，类名`curtPage`得以添加到class上。需要注意的是，这里的`pagebtn`是可直接使用的类名，所以是字符串类型；如果是使用了组件data中的属性，那就不应该使用字符串类型。

所有span的click事件是用触发页码变动事件的。这里具体实现上，按钮为文字的和数字的逻辑是不同的。对于文字按钮，我这里使用switch进行判断。因为在点击非页码按钮时是获取不到页面的，这时就需要对所点击按钮的`innerText`做出判断以响应，这里为了方便使用jQuery获取当前页码的上一个DOM节点模拟点击事件；或许要说明的是首页按钮的case，因为有可能页面显示的页码已经不是最开始的时候，所以要把页码初始化。也就是现在写总结时才想起，完全可以在模板代码中直接传递确定的页码作为参数，这样就省了很多事。

      //方法一
      switch (page.target.innerText) {
                case "上一页":
                  $("span.curtPage").prev().click(); break;
                case "下一页":
                  $("span.curtPage").next().click(); break;
                case "首页":
                  this.pagebtns = [1, 2, 3, 4, 5, "..."];
                  this.changeBtn(1); break;
                default:
                  break;
              }
      
      //方法2
      <span @click="changeBtn(1)">首页</span>
      <span @click="changeBtn(curtPage-1)">上一页</span>

实现样式上效果后，更重要是点击了页码后属实可以获取到新一页的内容。用Vue的话来说就是父子组件中的消息传递，那在这里就是子组件（pagination）把消息（新的页码）传递到父组件（pagelist）。显然是使用`$emit()`方法，在子组件中执行完改变页码的最后，使用`$emit()`向父组件发送消息，父组件中用自定义事件接受，代码如下。

      //子组件
      this.$emit("handle", this.curtPage);
      
      //父组件
      <pagination @handle="renderList"></pagination>
      
      renderList(value) {
            this.page = value;
            this.getData();
      }


##### postlist组件

此组件的目的是渲染从API请求到的主题数据。布局也倒是简单，顶部一个主题分类tab，下面则是渲染的主题列表，CSS代码此处不细讲(到这个阶段了，简单的不应该成为问题)。

对接口的请求，那必然涉及到Axios。考虑到这个库的使用相对简单，就不展开介绍使用直接上项目代码好了。get方法的第一个参数是请求的接口地址，第二个参数是接口请求所需的参数。下面是使用了对象的形式传递参数，当然也可以模板字符串拼接在地址中。请求成功后即将加载动画取消，并且将回传的数据赋给`this.posts`；`this.posts`是一个数组，而每一条主题数据又是一个对象，在`v-for`指令中渲染的就是每一个对象。

      methods: {
          getData() {
            this.$http
              .get("https://cnodejs.org/api/v1/topics", {
                params: {
                  page: this.page,
                  limit: 20}})
              .then(response => {
                this.isLoading = false;
                this.posts = response.data.data;
              })
              .catch(err => {
                console.log(err);});
          },
          renderList(value) {
            this.page = value;this.getData();}
      },
          
      beforeMount() {
          this.isLoading = true; //加载成功之前显示加载动画
          this.getData(); //页面加载前获取数据
      }

模板中的代码稍微值得一提的也就动态绑定类名和router的使用了。详见下方代码。

          <span
              :class="[{putgood:(post.good==true),puttop:(post.top==true),
              'putnormal':(post.good!=true&& post.top!=true)}]"
              >
                  {{post|formatTopicType}}
          </span>
      
          <router-link :to="{name:'post_content',params:{id:post.id,name:post.author.loginname}}">
              <span class="title">{{post.title}}</span>
          </router-link>

这里的动态绑定类名是针对主题分类使用的，`putgood`和`puttop`都是在`post.good`和`post.top`为真时才会添加到类中，而`putnormal`则是在主题类别不为置顶和精华时添加到类中。router的使用，上面代码中路由的功能为在点击主题标题后跳转到主题的详情页中，动态绑定的to属性中各key含义为：name 路由名称，params 路由路径中需要的参数（也即API中需要的参数）。

      //路由配置文件
      {
          name: 'post_content',
          path: '/topic/:id&name=:name',
          components: {
            main: Article,
            side: Sidebar
            //router-view中的name
            //点击首页的文章标题后就会用article组件替换掉之前在main
            //中的postlist 组件
          }
        }

现在两部分代码连起来看，就可以清楚地看出路由的动向。代码中另外还提到的Sidebar是下文会提到的另一个组件，写在意味当走向这个路由时，页面不止有name为main的Article组件还有name为Sidebar的组件。

##### Sidebar组件

这个组件是侧边栏组件，用于在主题详情页中渲染主题作者的相关信息。

组件模板分为三个部分，作者概要信息、作者最近主题、作者参与主题。在这里再次用到router在点击用户头像时跳转到用户详情页，具体和上面上面类似，这里就不赘述。

      getSideInfo() {
            this.$http
              .get(`https://cnodejs.org/api/v1/user/${this.$route.params.name}`)
              .then(result => {
                if (result.data.success) {
                  this.userinfo = result.data.data;
                }
              })
              .catch(err => {
                console.log(err);
              });
          }

在这里，我们继续来看到对用户详情API的请求，和之前提到的有些不同的是，这里用到了`this.$route.param`，这个所指向的就是先前提到的路由配置文件中我们设置的路由参数。通过这个我们可以轻易获取到路由中的参数以在组件中使用。

##### Article组件

这个组件是主题详情组件，用于展示主题内容回复等。没有很多相较之前组件的新内容。说下用到的过滤器，由于API返回的创建时间是包含年月日等的具体时间，但希望页面中显示的是”几小时钱“这样，所以需要有过滤器对返回创建时间进行处理，这个过滤器逻辑比较简单就不贴代码了。另外，考虑到这种是通用的过滤器，建议放在全局中，即main.js文件。

在这个页面中，同时存在的还有sidebar，当点击sidebar组件中主题时，由于Vue组件实例复用的原因，生命周期的钩子不会再被调用，这也意味着尽管点击后地址栏URL发生变化，但并不会触发`getArticleData` methods，此时就需要用到侦听器`watch`，以监听$route对象的变化，当路由变化时，则再次请求发起请求。

        watch: {
          $route(to, from) {
            this.getArticleData();
          }
        }

再有就是，除了简单的竖线键值之外，我们还可以在模板中编写JavaScript表达式，这一个feature在需要的时候会很有用的。比如这个组件中每条回复是有点赞数的，但我们并不希望在点赞数为0的时候也显示，这个时候就可以用到三目运算符进行判断（`{{comment.ups.length>0?comment.ups.length:''}}`）。此外，我们希望显示回复的楼层，但返回的回复为从0开始的数组类型，那此时我们在模板这样写就好了（`{{index+1}}楼`）。

##### headerbar组件

即网站顶部导航栏，这部分别的没多少代码，主要编码就是在CSS了。社区用的是float，我用了下flex布局实现。

最后还有个Userinfo组件，但考虑在实现和sidebar组件无差，这里就不具体分享了。

#### 印象很深的坑

路由配置中的`components`少写了`s`,当时beforeMount里的代码都没执行，控制台也没报错，完全懵逼。只好请教了眼尖的扫地僧。（`所有的编程初学者都满级近视`）

#### 项目截图

以下为部分项目截图，由于在技术实现上的重复性以及接口的问题，我并没有实现社区中有的全部页面。

![](https://user-gold-cdn.xitu.io/2019/3/3/169425f358c4df4c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![](https://user-gold-cdn.xitu.io/2019/3/3/169425f358b37e1d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)