# goquery selector(选择器) 示例大全

最近研究Go爬虫相关的知识，使用到goquery这个库比较多，尤其是对爬取到的HTML进行选择和查找匹配的内容时，goquery的选择器使用尤其多，而且还有很多不常用但又很有用的选择器，这里总结下，以供参考。

如果大家以前做过前端开发，对jquery不会陌生，goquery类似jquery，它是jquery的go版本实现。使用它，可以很方便的对HTML进行处理。

## 基于HTML Element 元素的选择器

这个比较简单，就是基于`a`,`p`等这些HTML的基本元素进行选择，这种直接使用Element名称作为选择器即可。比如`dom.Find("div")`。

```go
func main() {
	html := `<body>

				<div>DIV1</div>
				<div>DIV2</div>
				<span>SPAN</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
```

以上示例，可以把`div`元素筛选出来，而`body`,`span`并不会被筛选。

## ID 选择器

这个是使用频次最多的，类似于上面的例子，有两个`div`元素，其实我们只需要其中的一个，那么我们只需要给这个标记一个唯一的`id`即可，这样我们就可以使用`id`选择器，精确定位了。

```go
func main() {
	html := `<body>

				<div id="div1">DIV1</div>
				<div>DIV2</div>
				<span>SPAN</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("#div1").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
```

## Element ID 选择器

`id`选择器以`#`开头，紧跟着元素`id`的值，使用语法为`dom.Find(#id)`,后面的例子我会简写为`Find(#id)`,大家知道这是代表goquery选择器的即可。

如果有相同的ID，但是它们又分别属于不同的HTML元素怎么办？有好办法，和Element结合起来。比如我们筛选元素为`div`,并且`id`是`div1`的元素，就可以使用`Find(div#div1)`这样的筛选器进行筛选。

所以这类筛选器的语法为`Find(element#id)`，这是常用的组合方法，比如后面讲的过滤器也可以采用这种方式组合使用。

## Class选择器

`class`也是HTML中常用的属性，我们可以通过`class`选择器来快速的筛选需要的HTML元素，它的用法和`ID`选择器类似，为`Find(".class")`。

```go
func main() {
	html := `<body>

				<div id="div1">DIV1</div>
				<div class="name">DIV2</div>
				<span>SPAN</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find(".name").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
```

以上示例中，就筛选出来`class`为`name`的这个`div`元素。

## Element Class 选择器

`class`选择器和`id`选择器一样，也可以结合着HTML元素使用，他们的语法也类似`Find(element.class)`，这样就可以筛选特定element、并且指定class的元素。

## 属性选择器

一个HTML元素都有自己的属性以及属性值，所以我们也可以通过属性和值筛选元素。

```go
func main() {
	html := `<body>

				<div>DIV1</div>
				<div class="name">DIV2</div>
				<span>SPAN</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div[class]").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
```

示例中我们通过`div[class]`这个选择器，筛选出Element为`div`并且有`class`这个属性的，所以第一个`div`没有被筛选到。

刚刚上面这个示例是采用是否存在某个属性为筛选器，同理，我们可以筛选出属性为某个值的元素。

```go
    dom.Find("div[class=name]").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
```

这样我们就可以筛选出`class`这个属性值为`name`的`div`元素。

当然我们这里以`class`属性为例，还可以用其他属性，比如`href`等很多，自定义属性也是可以的。

除了完全相等，还有其他匹配方式，使用方式类似，这里统一列举下，不再举例

| 选择器                | 说明                                                  |
| :-------------------- | :---------------------------------------------------- |
| Find(“div[lang]")     | 筛选含有lang属性的div元素                             |
| Find(“div[lang=zh]")  | 筛选lang属性为zh的div元素                             |
| Find(“div[lang!=zh]") | 筛选lang属性不等于zh的div元素                         |
| Find(“div[lang¦=zh]") | 筛选lang属性为zh或者zh-开头的div元素                  |
| Find(“div[lang*=zh]") | 筛选lang属性包含zh这个字符串的div元素                 |
| Find(“div[lang~=zh]") | 筛选lang属性包含zh这个单词的div元素，单词以空格分开的 |
| Find(“div[lang$=zh]") | 筛选lang属性以zh结尾的div元素，区分大小写             |
| Find(“div[lang^=zh]") | 筛选lang属性以zh开头的div元素，区分大小写             |

以上是属性筛选器的用法，都是以一个属性筛选器为例，当然你也可以使用多个属性筛选器组合使用，比如： `Find("div[id][lang=zh]")`,用多个中括号连起来即可。当有多个属性筛选器的时候，要同时满足这些筛选器的元素才能被筛选出来。

## parent>child选择器

如果我们想筛选出某个元素下符合条件的子元素，我们就可以使用子元素筛选器，它的语法为`Find("parent>child")`,表示筛选parent这个父元素下，符合child这个条件的最直接（一级）的子元素。

```go
func main() {
	html := `<body>

				<div lang="ZH">DIV1</div>
				<div lang="zh-cn">DIV2</div>
				<div lang="en">DIV3</div>
				<span>
					<div>DIV4</div>
				</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("body>div").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
```

以上示例，筛选出`body`这个父元素下，符合条件的最直接的子元素`div`，结果是`DIV1、DIV2、DIV3`，虽然`DIV4`也是`body`的子元素，但不是一级的，所以不会被筛选到。

那么问题来了，我就是想把`DIV4`也筛选出来怎么办?就是要筛选`body`下所有的`div`元素，不管是一级、二级还是N级。有办法的，goquery考虑到了，只需要把大于号(`>`)改为空格就好了。比如上面的例子，改为如下选择器即可。

```go
    dom.Find("body div").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
```

## prev+next相邻选择器

假设我们要筛选的元素没有规律，但是该元素的上一个元素有规律，我们就可以使用这种下一个相邻选择器来进行选择。

```go
func main() {
	html := `<body>

				<div lang="zh">DIV1</div>
				<p>P1</p>
				<div lang="zh-cn">DIV2</div>
				<div lang="en">DIV3</div>
				<span>
					<div>DIV4</div>
				</span>
				<p>P2</p>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div[lang=zh]+p").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
}
```

这个示例演示了这种用法，我们想选择`<p>P1</p>`这个元素，但是没啥规律，我们发现它前面的`<div lang="zh">DIV1</div>`很有规律，可以选择，所以我们就可以采用`Find("div[lang=zh]+p")`达到选择`P`元素的目的。

这种选择器的语法是`("prev+next")`，中间是一个加号(+)，+号前后也是选择器。

> 本文为原创文章，转载注明出处，欢迎扫码关注公众号`flysnow_org`或者网站http://www.flysnow.org/，第一时间看后续精彩文章。一些比较可耻的网站抓取我的文章会去掉版权信息，这里再写一段，大家见谅。

## prev~next选择器

有相邻就有兄弟，兄弟选择器就不一定要求相邻了，只要他们共有一个父元素就可以。

```go
	dom.Find("div[lang=zh]~p").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Text())
	})
```

刚刚的例子，只需要把`+`号换成`~`号,就可以把`P2`也筛选出来，因为`P2`、`P1`和`DIV1`都是兄弟。

兄弟选择器的语法是`("prev~next")`,也就是相邻选择器的`+`换成了`~`。

## 内容过滤器

有时候我们使用选择器选择出来后后，希望再过滤一下，这时候就用到过滤器了，过滤器有很多，我们先讲内容过滤器这一种。

```go
dom.Find("div:contains(DIV2)").Each(func(i int, selection *goquery.Selection) {
	fmt.Println(selection.Text())
})
```

`Find(":contains(text)")`表示筛选出的元素要包含指定的文本，我们例子中要求选择出的`div`元素要包含`DIV2`文本，那么只有一个`DIV2`元素满足要求。

此外还有`Find(":empty")`表示筛选出的元素都不能有子元素（包括文本元素），只筛选那些不包含任何子元素的元素。

`Find(":has(selector)")`和`contains`差不多，只不过这个是包含的是元素节点。

```go
dom.Find("span:has(div)").Each(func(i int, selection *goquery.Selection) {
	fmt.Println(selection.Text())
})
```

以上示例表示筛选出包含`div`元素的`span`节点。

## :first-child过滤器

`:first-child`过滤器，语法为`Find(":first-child")`，表示筛选出的元素要是他们的父元素的第一个子元素，如果不是，则不会被筛选出来。

```go
func main() {
	html := `<body>

				<div lang="zh">DIV1</div>
				<p>P1</p>
				<div lang="zh-cn">DIV2</div>
				<div lang="en">DIV3</div>
				<span>
					<div style="display:none;">DIV4</div>
					<div>DIV5</div>
				</span>
				<p>P2</p>
				<div></div>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div:first-child").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Html())
	})
}
```

以上例子中，我们使用`Find("div")`会筛选出所有的`div`元素，但是我们加了`:first-child`后，就只有`DIV1`和`DIV4`了，因为只有这两个是他们父元素的第一个子元素，其他的`DIV`都不满足。

## :first-of-type过滤器

`:first-child`选择器限制的比较死，必须得是第一个子元素，如果该元素前有其他在前面，就不能用`:first-child`了，这时候`:first-of-type`就派上用场了，它要求只要是这个类型的第一个就可以，我们把上面的例子微调下。

```go
func main() {
	html := `<body>

				<div lang="zh">DIV1</div>
				<p>P1</p>
				<div lang="zh-cn">DIV2</div>
				<div lang="en">DIV3</div>
				<span>
					<p>P2</p>
					<div>DIV5</div>
				</span>
				<div></div>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div:first-of-type").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Html())
	})
}
```

改动很简单，把原来的`DIV4`换成了`P2`，如果我们还使用`:first-child`,`DIV5`是不能被筛选出来的，因为它不是第一个子元素，它前面还有一个`P2`。这时候我们使用`:first-of-type`就可以达到目的，因为它要求是同类型第一个就可以。`DIV5`就是这个`div`类型的第一个元素，`P2`不是`div`类型，被忽略。

## :last-child 和 :last-of-type过滤器

这两个正好和上面的`:first-child`、`:first-of-type`相反，表示最后一个，这里不再举例，大家可以自己试试。

## :nth-child(n) 过滤器

这个表示筛选出的元素是其父元素的第n个元素，n以1开始。所以我们可以知道`:first-child`和`:nth-child(1)`是相等的。通过指定`n`，我们就很灵活的筛选出我们需要的元素。

```go
func main() {
	html := `<body>

				<div lang="zh">DIV1</div>
				<p>P1</p>
				<div lang="zh-cn">DIV2</div>
				<div lang="en">DIV3</div>
				<span>
					<p>P2</p>
					<div>DIV5</div>
				</span>
				<div></div>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div:nth-child(3)").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Html())
	})
}
```

这个示例会筛选出`DIV2`，因为`DIV2`是其父元素`body`的第三个子元素。

## :nth-of-type(n) 过滤器

`:nth-of-type(n)`和 `:nth-child(n)` 类似，只不过它表示的是同类型元素的第n个,所以`:nth-of-type(1)` 和 `:first-of-type`是相等的，大家可以自己试试，这里不再举例。

## nth-last-child(n) 和:nth-last-of-type(n) 过滤器

这两个和上面的类似，只不过是倒序开始计算的，最后一个元素被当成了第一个。大家自己测试下看看效果，很明显。

## :only-child 过滤器

`Find(":only-child")` 过滤器，从字面上看，可以猜测出来，它表示筛选的元素，在其父元素中，只有它自己，它的父元素没有其他子元素，才会被匹配筛选出来。

```go
func main() {
	html := `<body>
				<div lang="zh">DIV1</div>
				<span>
					<div>DIV5</div>
				</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div:only-child").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Html())
	})
}
```

示例中`DIV5`就可以被筛选出来，因为它是它的父元素`span`达到唯一子元素，但`DIV1`就不是，所以不能呗筛选出来。

## :only-of-type 过滤器

上面的例子，如果想筛选出`DIV1`怎么办？可以使用`Find(":only-of-type")`,因为它是它的父元素中，唯一的`div`元素，这就是`:only-of-type`过滤器所要做的，同类型元素只要只有一个，就可以被筛选出来。大家把上面的例子改成`:only-of-type`试试，看看是否有`DIV1`。

## 选择器或(|)运算

如果我们想同时筛选出`div`,`span`等元素怎么办？这时候可以采用多个选择器进行组合使用，并且以逗号(,)分割，`Find("selector1, selector2, selectorN")`表示，只要满足其中一个选择器就可以被筛选出来，也就是选择器的或(|)运算操作。

```go
func main() {
	html := `<body>
				<div lang="zh">DIV1</div>
				<span>
					<div>DIV5</div>
				</span>

			</body>
			`

	dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
	if err!=nil{
		log.Fatalln(err)
	}

	dom.Find("div,span").Each(func(i int, selection *goquery.Selection) {
		fmt.Println(selection.Html())
	})
}
```



## 小结

goquery 是解析HTML网页必备的利器，在爬虫抓取网页的过程中，灵活的使用goquery不同的选择器，可以让我们的抓取工作事半功倍，大大提升爬虫的效率。