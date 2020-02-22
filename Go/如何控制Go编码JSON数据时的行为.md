# [如何控制Go编码JSON数据时的行为](https://segmentfault.com/a/1190000021674335)

[golang](https://segmentfault.com/t/golang)

 阅读约 8 分钟

今天来聊一下我在Go中对数据进行 JSON 编码时遇到次数最多的三个问题以及解决方法，大家来看看是不是也为这些问题挠掉了不少头发。

### 自定义JSON键名

这个问题加到文章里我是有所犹豫的，因为基本上大家都会，不过属于同类问题我还是放进来了，对新接触` Go` 的同学更友好些。

我们先从最常见的一个问题说，首先在Go 程序中要将数据编码成JSON 格式时通常我们会先定义结构体类型，将数据存放到结构体变量中。

```
type Address struct {
    Type    string
    City    string  
    Country string
}

type CreditCard struct {
    FirstName string
    LastName  string
    Addresses []*Address
    Remark    string
}

home := &Address{"private", "Aartselaar", "Belgium"}
office := &Address{"work", "Boom", "Belgium"}
card := VCard{"Jan", "Kersschot", []*Address{home, office}, "none"}

js, err := json.Marshal(card)
fmt.Printf("JSON format: %s", js)
```

只有导出的结构体成员才会被编码，这也就是我们为什么选择用大写字母开头的字段名称。在编码时，默认使用结构体字段的名字作为JSON对象中的`key`，但是一般JSON 是给`HTTP接口`返回数据使用的，在接口的规范里针对数据我们一般都要求返回`snake case`风格的字段名。解决这个问题的方法是在结构体声明时在结构体字段标签里可以自定义对应的`JSON key`

所以我们把结构体声明改为如下即可：

```
type Address struct {
    Type    string  `json:"type"`
    City    string  `json:"city"`
    Country string  `json:"country"`
}
```

### 编码JSON时忽略掉指定字段

并不是所有数据我们都期望编码到`JSON`中暴露给外部接口的，所以针对一些敏感的字段我们往往希望将其从编码后的`JSON`数据中忽略掉。那么上面也说了只有导出的结构体成员才会被编码，有的同学会问我直接用小写的字段名不行吗？可是未导出字段只能在包内访问，像这种携带内部敏感数据的往往都是应用的基础数据，由项目的公共包来提供的。那么怎么既能维持字段的导出性又能让其在`JSON`数据中被忽略掉呢? 还是使用结构体的标签进行注解，比如下面定义的结构体，可以把身份证`IdCard`字段在`JSON`数据中去掉：

```
type User struct {
    Name    string  `json:"name"`
    Age     Int     `json:"int"`
    IdCard  string  `json:"-"`
}
```

`encoding/json`的源码中和文档中都列举了通过结构体字段标签控制数据`JSON`编码行为的说明：

```
// Field is ignored by this package.
Field int `json:"-"`

// Field appears in JSON as key "myName".
Field int `json:"myName"`

// Field appears in JSON as key "myName" and
// the field is omitted from the object if its value is empty,
// as defined above.
Field int `json:"myName,omitempty"`

// Field appears in JSON as key "Field" (the default), but
// the field is skipped if empty.
// Note the leading comma.
Field int `json:",omitempty"`
```

`omitempty`这个是字段的数据为空时，在`JSON`中省略这个字段。为的是节省数据空间，`Protobuf`编译器生成的结构体代码中每个字段标签中都有`omitempty`。但是在`Api`开发中这个不常用，因为字段不固定对前端很不友好。

对`Protobuf`不了解的可以看我之前写的文章《[Protobuf语言指南](https://mp.weixin.qq.com/s?__biz=MzUzNTY5MzU2MA==&mid=2247483855&idx=1&sn=5f41a7ec8b2f0634663933397d9727dc&chksm=fa80d058cdf7594e70b018e560ee08278a3f6722dc5502aab27d85cf6cd04a871ead94fd80af&token=1432641249&lang=zh_CN#rd)》。

结构体字段标签的`json`注解中都不加`omitempty`后还遇到一种情况，就是数据类型为切片的字段在数据为空的时候会被`JSON`编码为`null`而不是`[]`。这个前端经常会问我没数据的时候能不能不要返回`null`，每回还要多写一个判断。我的说辞都是不能，其实规范点讲是应该返回`[]`的知识我是我自己没找到到解决方法。作为一个在写代码上有强迫症的人，这个问题还是想搞明白的，好在有一天在`StackOverflow`上看到一个答案，才发现是编码的疏忽导致的。

### 解决空切片在JSON里被编码成null

因为切片的零值为`nil`，无指向内存的地址，所以当以这种形式定义`var f []int`初始化`slice`后，在JSON中将其编码为`null`，如果想在 JSON 中将空 slice 编码为`[]`则需用make初始化 slice为其分配内存地址：

运行下面的例子可以看出两点的区别：

```
package main

import (
    "encoding/json"
    "fmt"
)

type Person struct {
    Friends []string
}

func main() {
    var f1 []string
    f2 := make([]string, 0)

    json1, _ := json.Marshal(Person{f1})
    json2, _ := json.Marshal(Person{f2})

    fmt.Printf("%s\n", json1)
    fmt.Printf("%s\n", json2)
}
```

输出：

```
{"Friends":null}
{"Friends":[]}
```

其实导致这个问题的原因是Go的`append`函数（甩锅），我们都知道引用类型的变量定义后如果没初始化他们的值是`nil`，无指向内存的地址，是无法直接使用的。但是`append`函数在给切片追加元素时会判断切片是否已初始化，没有的话会帮其初始化分配底层数组。我的习惯是先声明切片，然后再在下面的循环代码中向切片追加元素。但是如果循环没有执行，比如你从数据库没查出数据，就会导致对应切片字段在无数据时返回的是`nil`然后被`JSON`编码成了`null`。所以这个算是一个经验总结出来的`Tip`吧在写代码时大家一定要注意了。

这就是我在开发时把数据编码成`JSON`格式时遇到的三个问题和相应的解决方法。加上之前写的[解析JSON的文章](https://zhuanlan.zhihu.com/p/80442700)，两个文章加起来差不多就能汇总日常开发中关于`encoding/json`库使用的各种问题了。