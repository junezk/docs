# 操作URL的黑科技

### 前言

现在有这么个URL：www.baidu.com/s?wd=蔡徐坤&skill=篮球&year=2019 ，怎么才能获取query上的字段呢？这时候正则表达式就派上用场了，效果如图：



![img](https://user-gold-cdn.xitu.io/2019/6/14/16b55dd6cdc004e7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



杀鸡焉用牛刀呢，今天我们来学习下专门用来处理URL的query的接口：URLSearchParams 。

### 简单使用

只需要new一个URLSearchParams的实例即可，代码：

```
let url = '?wd=蔡徐坤&skill=篮球&year=2019';
let searchParams = new URLSearchParams(url);

for (let p of searchParams) {
  console.log(p);
}
// ["wd", "蔡徐坤"]
// ["skill", "篮球"]
// ["year", "2019"]
```

### 获取单个字段

假如现在我只想获取单个字段的值，该怎么办呢？只需要调用这个实例的get方法即可， 代码：

```
searchParams.get('wd') // "蔡徐坤"
searchParams.get('skill') // "篮球"
searchParams.get('year') // "2019"
```

有时候不知道一个字段是否存在，所以想事先校验下。使用实例的has方法进行判断，代码：

```
searchParams.has('wd') // true
searchParams.has('age') // false
```

### 添加字段

实例提供了append方法来添加字段，这个方法接收两个参数，前者是key，后者是value，代码：

```
searchParams.append('age', 26);
searchParams.has('age'); // true
searchParams.get('age'); // 26
```

### 删除字段

现在不想要year字段了，直接使用delete即可，代码：

```
searchParams.delete('year');
searchParams.has('year'); // false
```

### 设置字段

有时候想重写一个字段，而不是添加(append)一个字段，这时候需要使用set方法，比如，我们觉得坤哥不仅会篮球，还会唱，跳，rap。代码：

```
searchParams.set('skill', '篮球 唱 跳 rap');
```

### 转为字符串

修改实例后，有时候需要再转为字符串，进行路由跳转等，使用toString方法

```
searchParams.toString(); // "wd=蔡徐坤&skill=篮球+唱+跳+rap&year=2019&age=26"
```

### 兼容性

现代浏览器基本没有啥大问题，但是IE的支持不是很理想。

![img](https://user-gold-cdn.xitu.io/2019/6/14/16b55dd6cf0d08be?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 外链

- [MDN参考](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FAPI%2FURLSearchParams)
- [博客地址查看 jser.tech](https://link.juejin.im/?target=https%3A%2F%2Fjser.tech%2F2019%2F06%2F14%2F%E6%93%8D%E4%BD%9CURL%E7%9A%84%E9%BB%91%E7%A7%91%E6%8A%80%2F)