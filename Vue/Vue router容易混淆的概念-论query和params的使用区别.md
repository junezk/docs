# Vue router容易混淆的概念-论query和params的使用区别

## **前言**

路由传参的时候，有俩兄弟，一个叫`query`，一个叫`parmas`

你说他们俩长得也不像吧，可这用法实在是太类似了
下面就让我们分别从**vue路由**和**Node接收**两个角度讲他们的区别

## **vue路由中的传参**

假设我们现在需要实现一个路由切换，点击之切换到`W组件`
并传递一个`id`值和一个`age`值

我们运用`router-link`来写
然后一连串的疑惑就产生了

```
<router-link :to="{ A: 'xxx', query: { xx:'xxx' }}" />
<router-link :to="{ A: 'xxx', parmas: { xx:'xxx' }}" />
routes:{ ??? }
复制代码
```

**对于query和parmas来说**

1. A用name还是path？
2. routes要怎么写？
3. url长什么样？
4. 会有什么隐藏的坑么

## **query**：

```
<router-link :to="{ name: 'W', query: { id:'1234'，age:'12' }}"/>
<router-link :to="{ path: '/W', query: { id:'1234',age:'12' }}"/>
复制代码
```

`name`和`path`都可以用

前者的`routes`基于`name`设置

```
{
  path: '/hhhhhhh', //这里可以任意
  name: 'W',  //这里必须是W
  component: W
}
复制代码
```

然后就把`path`匹配添加到`url`上去

```
http://localhost:8080/#/hhhhhhh?id=1234&age=12
复制代码
```

后者基于`path`来设置`routes`

```
{
  path: '/W', //这里必须是W
  name: 'hhhhhhhh',  //这里任意
  component: W
}
复制代码
url:http://localhost:8080/#/W?id=1234&age=12
```

这两种方法，都可以自定义`path`的样式， 不过一个是在`router-link to`里面定义，一个则是在`routes`里面定义

在接收参数的时候都是使用`this.$route.query.id`

## **parmas**：

```
<router-link :to="{ name: 'W', params: { id:'1234',age:'12' }}"/>
复制代码
```

这里只能用`name`不能用`path`，不然会直接无视掉`params`中的内容

然后在`routes`中添加

```
{
      path:'/W/:id/:age',
      name:'W',
      component:W
}
复制代码
```

这里的`name`与上面`router-link`中的`name`保持一致

`url`就取决于这个`path`的写法 `http://localhost:8080/#/W/1234/12`
注意，`path`里面的`/w`可以任意写，写成`/hhhhh`也可以 但是！
**/:id和/:age不能省略，且不能改名字**

**不写的话，第一次点击可以实现组件跳转** **且可以通过this.$route.parmas.id获取到传过来的id值，但如果** **刷新页面，传过来的id值和age值就会丢失**

从这也能看出params比query严格

## **Node中的req.query和req.params**

在后端中，要接受前端的`axios`请求 于是我们又碰到了这哥俩

什么样的`axios`请求对应什么样的接受方式？
还有不止是`req.query`,`req.params`,又混进来一个`req.body`
好家伙，乱成一锅粥

假设前端现在用axios向后端发送一个请求，发送id值请求后端的数据

## **req.query**

```
axios.get(`/api/?id=1234`)
复制代码
```

或者

```
axios.get(`/api`，{ params:{id:'1234' })
复制代码
```

在前端里面，`router`怎么发送的就怎么收
`query`发送的就用`this.$route.query`接收
`params`发送的就用`this.$route.params`接收

但是在这里，虽然第二种方式里面有`params`
但这两种我们都要用`req.query.id`来获取里面的`id`值

```
router.get('/api',function(req,res){
console.log(req.query.id)
.......
})
复制代码
```

## **req.params**

那如果直接把`id`值写进发送的`url`里面呢

```
axios.get(`/api/1234`)
复制代码
```

看这个形式有没有觉得很眼熟
它跟上面`params`的`url`非常像, 我们就反向操作一下

```
router.get('/api/:id',function(req,res){
    console.log(req.params.id)
    .......
    })
复制代码
```

如果它是这么请求的

```
axios.get(`/api/1234-12`)
复制代码
```

中间用`-`或者`&`隔开
那我们也可以在获取时的路径上这么写

```
router.get('/api/:id-:age',function(req,res){
    console.log(req.params.id)
    console.log(req.params.age)
    .......
    })
复制代码
```

## **req.body**

上面两个都是处理`get`请求的
而这位小兄弟就是用来处理`post`请求的 (需要安装`body-parser`中间件）

```
axios.post(`/api`,{ id:'1234' })
复制代码
```

我们就用`req.body`来接收

```
router.get('/api',function(req,res){
console.log(req.body.id)
.......
})
复制代码
```

## 总结

我们归纳了`query`和`params`在前端路由以及后端接收中的区别
容易混淆的东西还是得多写，多总结