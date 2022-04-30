# Go语言中使用正则提取匹配的字符串

我们在做爬虫的过程中，需要对爬取到的内容处理，比如说提取出我们需要的内容和文本，比如城市信息、人员信息等等，除了字符串查找外，使用正则匹配是比较优雅和方便的方案。

这篇文章，主要以提取URL中的日期和文章名为例，来举例说明如何使用正则提取字符串。

比如这个URL `http://www.flysnow.org/2018/01/20/golang-goquery-examples-selector.html`，这是前段时间写的一篇关于goquery的使用教程。从这个URL我们可以看到有年月日的日期信息，还有最后面的文章的名称信息，这样一个URL，我们如何从中得到这些信息呢？这就要用到正则表达式的分组了。

正则表达式的分组，以括号`()`表示，每一对括号就是我们匹配到的一个文本，可以把他们提取出来。

通过上面URl的分析，我们定义正则表达式如下：

```
^http://www.flysnow.org/([\d]{4})/([\d]{2})/([\d]{2})/([\w-]+).html$
```

`^$`分别表示匹配的开始和结束，界定我们正则表达式的范围。

`[\d]{4}`表示我们要正好匹配4位数字，因为年份是4位，所以我们定义为匹配4位。后面的月份和天是2位，所以定义为2位。

`[\w-]`匹配字符串和中杠，加号(+)表示匹配1个或者多个。

然后他们都加了括号`()`，意味着我们要提取这些字符串。

下面看下完整的源代码。

```go
flysnowRegexp := regexp.MustCompile(`^http://www.flysnow.org/([\d]{4})/([\d{2})/([\d]{2})/([\w-]+).html$`)
params :=flysnowRegexp.FindStringSubmatch("http://www.flysnow.org/2018/01/20/golang-goquery-examples-selector.html")

for _,param :=range params {
    fmt.Println(param)
}
```

运行打印输出：

```
http://www.flysnow.org/2018/01/20/golang-goquery-examples-selector.html
2018
01
20
golang-goquery-examples-selector
```

`FindStringSubmatch`方法是提取出匹配的字符串，然后通过`[]string`返回。我们可以看到，第1个匹配到的是这个字符串本身，从第2个开始，才是我们想要的字符串。

```go
fmt.Println("年份为："+params[1])
fmt.Println("月份为："+params[2])
fmt.Println("天数为："+params[3])
fmt.Println("文章名为："+params[4])
```

这样我们需要的这篇文章的信息就被提取出来了。

正则对于处理文章很好用，关于更多Golang正则的使用，可以参考官方的这篇正则表达式的介绍。https://github.com/google/re2/wiki/Syntax