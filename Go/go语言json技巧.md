# [go语言json技巧](https://www.liwenzhou.com/posts/Go/json_tricks_in_go/)

本文总结了我平时在项目中遇到的那些关于go语言JSON数据与结构体之间相互转换的问题及解决办法。

[TOC]

### 基本的序列化

首先我们来看一下Go语言中`json.Marshal()`（序列化）与`json.Unmarshal`（反序列化）的基本用法。

```go
type Person struct {
	Name   string
	Age    int64
	Weight float64
}

func main() {
	p1 := Person{
		Name:   "七米",
		Age:    18,
		Weight: 71.5,
	}
	// struct -> json string
	b, err := json.Marshal(p1)
	if err != nil {
		fmt.Printf("json.Marshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("str:%s\n", b)
	// json string -> struct
	var p2 Person
	err = json.Unmarshal(b, &p2)
	if err != nil {
		fmt.Printf("json.Unmarshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("p2:%#v\n", p2)
}
```

输出：

```bash
str:{"Name":"七米","Age":18,"Weight":71.5}
p2:main.Person{Name:"七米", Age:18, Weight:71.5}
```

### 结构体tag介绍

`Tag`是结构体的元信息，可以在运行的时候通过反射的机制读取出来。 `Tag`在结构体字段的后方定义，由一对**反引号**包裹起来，具体的格式如下：

```bash
`key1:"value1" key2:"value2"`
```

结构体tag由一个或多个键值对组成。键与值使用**冒号**分隔，值用**双引号**括起来。同一个结构体字段可以设置多个键值对tag，不同的键值对之间使用**空格**分隔。

### 使用json tag指定字段名

序列化与反序列化默认情况下使用结构体的字段名，我们可以通过给结构体字段添加tag来指定json序列化生成的字段名。

```go
// 使用json tag指定序列化与反序列化时的行为
type Person struct {
	Name   string `json:"name"` // 指定json序列化/反序列化时使用小写name
	Age    int64
	Weight float64
}
```

### 忽略某个字段

如果你想在json序列化/反序列化的时候忽略掉结构体中的某个字段，可以按如下方式在tag中添加`-`。

```go
// 使用json tag指定json序列化与反序列化时的行为
type Person struct {
	Name   string `json:"name"` // 指定json序列化/反序列化时使用小写name
	Age    int64
	Weight float64 `json:"-"` // 指定json序列化/反序列化时忽略此字段
}
```

### 忽略空值字段

当 struct 中的字段没有值时， `json.Marshal()` 序列化的时候不会忽略这些字段，而是默认输出字段的类型零值（例如`int`和`float`类型零值是 0，`string`类型零值是`""`，对象类型零值是 nil）。如果想要在序列序列化时忽略这些没有值的字段时，可以在对应字段添加`omitempty` tag。

举个例子：

```go
type User struct {
	Name  string   `json:"name"`
	Email string   `json:"email"`
	Hobby []string `json:"hobby"`
}

func omitemptyDemo() {
	u1 := User{
		Name: "七米",
	}
	// struct -> json string
	b, err := json.Marshal(u1)
	if err != nil {
		fmt.Printf("json.Marshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("str:%s\n", b)
}
```

输出结果：

```go
str:{"name":"七米","email":"","hobby":null}
```

如果想要在最终的序列化结果中去掉空值字段，可以像下面这样定义结构体：

```go
// 在tag中添加omitempty忽略空值
// 注意这里 hobby,omitempty 合起来是json tag值，中间用英文逗号分隔
type User struct {
	Name  string   `json:"name"`
	Email string   `json:"email,omitempty"`
	Hobby []string `json:"hobby,omitempty"`
}
```

此时，再执行上述的`omitemptyDemo`，输出结果如下：

```bash
str:{"name":"七米"} // 序列化结果中没有email和hobby字段
```

### 忽略嵌套结构体空值字段

首先来看几种结构体嵌套的示例：

```go
type User struct {
	Name  string   `json:"name"`
	Email string   `json:"email,omitempty"`
	Hobby []string `json:"hobby,omitempty"`
	Profile
}

type Profile struct {
	Website string `json:"site"`
	Slogan  string `json:"slogan"`
}

func nestedStructDemo() {
	u1 := User{
		Name:  "七米",
		Hobby: []string{"足球", "双色球"},
	}
	b, err := json.Marshal(u1)
	if err != nil {
		fmt.Printf("json.Marshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("str:%s\n", b)
}
```

匿名嵌套`Profile`时序列化后的json串为单层的：

```bash
str:{"name":"七米","hobby":["足球","双色球"],"site":"","slogan":""}
```

想要变成嵌套的json串，需要改为具名嵌套或定义字段tag：

```go
type User struct {
	Name    string   `json:"name"`
	Email   string   `json:"email,omitempty"`
	Hobby   []string `json:"hobby,omitempty"`
	Profile `json:"profile"`
}
// str:{"name":"七米","hobby":["足球","双色球"],"profile":{"site":"","slogan":""}}
```

想要在嵌套的结构体为空值时，忽略该字段，仅添加`omitempty`是不够的：

```go
type User struct {
	Name     string   `json:"name"`
	Email    string   `json:"email,omitempty"`
	Hobby    []string `json:"hobby,omitempty"`
	Profile `json:"profile,omitempty"`
}
// str:{"name":"七米","hobby":["足球","双色球"],"profile":{"site":"","slogan":""}}
```

还需要使用嵌套的结构体指针：

```go
type User struct {
	Name     string   `json:"name"`
	Email    string   `json:"email,omitempty"`
	Hobby    []string `json:"hobby,omitempty"`
	*Profile `json:"profile,omitempty"`
}
// str:{"name":"七米","hobby":["足球","双色球"]}
```

### 不修改原结构体忽略空值字段

我们需要json序列化`User`，但是不想把密码也序列化，又不想修改`User`结构体，这个时候我们就可以使用创建另外一个结构体`PublicUser`匿名嵌套原`User`，同时指定`Password`字段为匿名结构体指针类型，并添加`omitempty`tag，示例代码如下：

```go
type User struct {
	Name     string `json:"name"`
	Password string `json:"password"`
}

type PublicUser struct {
	*User             // 匿名嵌套
	Password *struct{} `json:"password,omitempty"`
}

func omitPasswordDemo() {
	u1 := User{
		Name:     "七米",
		Password: "123456",
	}
	b, err := json.Marshal(PublicUser{User: &u1})
	if err != nil {
		fmt.Printf("json.Marshal u1 failed, err:%v\n", err)
		return
	}
	fmt.Printf("str:%s\n", b)  // str:{"name":"七米"}
}
```

### 优雅处理字符串格式的数字

有时候，前端在传递来的json数据中可能会使用字符串类型的数字，这个时候可以在结构体tag中添加`string`来告诉json包从字符串中解析相应字段的数据：

```go
type Card struct {
	ID    int64   `json:"id,string"`    // 添加string tag
	Score float64 `json:"score,string"` // 添加string tag
}

func intAndStringDemo() {
	jsonStr1 := `{"id": "1234567","score": "88.50"}`
	var c1 Card
	if err := json.Unmarshal([]byte(jsonStr1), &c1); err != nil {
		fmt.Printf("json.Unmarsha jsonStr1 failed, err:%v\n", err)
		return
	}
	fmt.Printf("c1:%#v\n", c1) // c1:main.Card{ID:1234567, Score:88.5}
}
```

### 整数变浮点数

在 JSON 协议中是没有整型和浮点型之分的，它们统称为number。json字符串中的数字经过Go语言中的json包反序列化之后都会成为`float64`类型。下面的代码便演示了这个问题：

```go
func jsonDemo() {
	// map[string]interface{} -> json string
	var m = make(map[string]interface{}, 1)
	m["count"] = 1 // int
	b, err := json.Marshal(m)
	if err != nil {
		fmt.Printf("marshal failed, err:%v\n", err)
	}
	fmt.Printf("str:%#v\n", string(b))
	// json string -> map[string]interface{}
	var m2 map[string]interface{}
	err = json.Unmarshal(b, &m2)
	if err != nil {
		fmt.Printf("unmarshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("value:%v\n", m2["count"]) // 1
	fmt.Printf("type:%T\n", m2["count"])  // float64
}
```

这种场景下如果想更合理的处理数字就需要使用`decoder`去反序列化，示例代码如下：

```go
func decoderDemo() {
	// map[string]interface{} -> json string
	var m = make(map[string]interface{}, 1)
	m["count"] = 1 // int
	b, err := json.Marshal(m)
	if err != nil {
		fmt.Printf("marshal failed, err:%v\n", err)
	}
	fmt.Printf("str:%#v\n", string(b))
	// json string -> map[string]interface{}
	var m2 map[string]interface{}
	// 使用decoder方式反序列化，指定使用number类型
	decoder := json.NewDecoder(bytes.NewReader(b))
	decoder.UseNumber()
	err = decoder.Decode(&m2)
	if err != nil {
		fmt.Printf("unmarshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("value:%v\n", m2["count"]) // 1
	fmt.Printf("type:%T\n", m2["count"])  // json.Number
	// 将m2["count"]转为json.Number之后调用Int64()方法获得int64类型的值
	count, err := m2["count"].(json.Number).Int64()
	if err != nil {
		fmt.Printf("parse to int64 failed, err:%v\n", err)
		return
	}
	fmt.Printf("type:%T\n", int(count)) // int
}
```

`json.Number`的源码定义如下：

```go
// A Number represents a JSON number literal.
type Number string

// String returns the literal text of the number.
func (n Number) String() string { return string(n) }

// Float64 returns the number as a float64.
func (n Number) Float64() (float64, error) {
	return strconv.ParseFloat(string(n), 64)
}

// Int64 returns the number as an int64.
func (n Number) Int64() (int64, error) {
	return strconv.ParseInt(string(n), 10, 64)
}
```

我们在处理number类型的json字段时需要先得到`json.Number`类型，然后根据该字段的实际类型调用`Float64()`或`Int64()`。

### 自定义解析时间字段

Go语言内置的 json 包使用 `RFC3339` 标准中定义的时间格式，对我们序列化时间字段的时候有很多限制。

```go
type Post struct {
	CreateTime time.Time `json:"create_time"`
}

func timeFieldDemo() {
	p1 := Post{CreateTime: time.Now()}
	b, err := json.Marshal(p1)
	if err != nil {
		fmt.Printf("json.Marshal p1 failed, err:%v\n", err)
		return
	}
	fmt.Printf("str:%s\n", b)
	jsonStr := `{"create_time":"2020-04-05 12:25:42"}`
	var p2 Post
	if err := json.Unmarshal([]byte(jsonStr), &p2); err != nil {
		fmt.Printf("json.Unmarshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("p2:%#v\n", p2)
}
```

上面的代码输出结果如下：

```go
str:{"create_time":"2020-04-05T12:28:06.799214+08:00"}
json.Unmarshal failed, err:parsing time ""2020-04-05 12:25:42"" as ""2006-01-02T15:04:05Z07:00"": cannot parse " 12:25:42"" as "T"
```

也就是内置的json包不识别我们常用的字符串时间格式，如`2020-04-05 12:25:42`。

不过我们通过实现 `json.Marshaler`/`json.Unmarshaler` 接口实现自定义的事件格式解析。

```go
type CustomTime struct {
	time.Time
}

const ctLayout = "2006-01-02 15:04:05"

var nilTime = (time.Time{}).UnixNano()

func (ct *CustomTime) UnmarshalJSON(b []byte) (err error) {
	s := strings.Trim(string(b), "\"")
	if s == "null" {
		ct.Time = time.Time{}
		return
	}
	ct.Time, err = time.Parse(ctLayout, s)
	return
}

func (ct *CustomTime) MarshalJSON() ([]byte, error) {
	if ct.Time.UnixNano() == nilTime {
		return []byte("null"), nil
	}
	return []byte(fmt.Sprintf("\"%s\"", ct.Time.Format(ctLayout))), nil
}

func (ct *CustomTime) IsSet() bool {
	return ct.UnixNano() != nilTime
}

type Post struct {
	CreateTime CustomTime `json:"create_time"`
}

func timeFieldDemo() {
	p1 := Post{CreateTime: CustomTime{time.Now()}}
	b, err := json.Marshal(p1)
	if err != nil {
		fmt.Printf("json.Marshal p1 failed, err:%v\n", err)
		return
	}
	fmt.Printf("str:%s\n", b)
	jsonStr := `{"create_time":"2020-04-05 12:25:42"}`
	var p2 Post
	if err := json.Unmarshal([]byte(jsonStr), &p2); err != nil {
		fmt.Printf("json.Unmarshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("p2:%#v\n", p2)
}
```

### 自定义MarshalJSON和UnmarshalJSON方法

上面那种自定义类型的方法稍显啰嗦了一点，下面来看一种相对便捷的方法。

首先你需要知道的是，如果你能够为某个类型实现了`MarshalJSON()([]byte, error)`和`UnmarshalJSON(b []byte) error`方法，那么这个类型在序列化（MarshalJSON）/反序列化（UnmarshalJSON）时就会使用你定制的相应方法。

```go
type Order struct {
	ID          int       `json:"id"`
	Title       string    `json:"title"`
	CreatedTime time.Time `json:"created_time"`
}

const layout = "2006-01-02 15:04:05"

// MarshalJSON 为Order类型实现自定义的MarshalJSON方法
func (o *Order) MarshalJSON() ([]byte, error) {
	type TempOrder Order // 定义与Order字段一致的新类型
	return json.Marshal(struct {
		CreatedTime string `json:"created_time"`
		*TempOrder         // 避免直接嵌套Order进入死循环
	}{
		CreatedTime: o.CreatedTime.Format(layout),
		TempOrder:   (*TempOrder)(o),
	})
}

// UnmarshalJSON 为Order类型实现自定义的UnmarshalJSON方法
func (o *Order) UnmarshalJSON(data []byte) error {
	type TempOrder Order // 定义与Order字段一致的新类型
	ot := struct {
		CreatedTime string `json:"created_time"`
		*TempOrder         // 避免直接嵌套Order进入死循环
	}{
		TempOrder: (*TempOrder)(o),
	}
	if err := json.Unmarshal(data, &ot); err != nil {
		return err
	}
	var err error
	o.CreatedTime, err = time.Parse(layout, ot.CreatedTime)
	if err != nil {
		return err
	}
	return nil
}

// 自定义序列化方法
func customMethodDemo() {
	o1 := Order{
		ID:          123456,
		Title:       "《七米的Go学习笔记》",
		CreatedTime: time.Now(),
	}
	// 通过自定义的MarshalJSON方法实现struct -> json string
	b, err := json.Marshal(&o1)
	if err != nil {
		fmt.Printf("json.Marshal o1 failed, err:%v\n", err)
		return
	}
	fmt.Printf("str:%s\n", b)
	// 通过自定义的UnmarshalJSON方法实现json string -> struct
	jsonStr := `{"created_time":"2020-04-05 10:18:20","id":123456,"title":"《七米的Go学习笔记》"}`
	var o2 Order
	if err := json.Unmarshal([]byte(jsonStr), &o2); err != nil {
		fmt.Printf("json.Unmarshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("o2:%#v\n", o2)
}
```

输出结果：

```bash
str:{"created_time":"2020-04-05 10:32:20","id":123456,"title":"《七米的Go学习笔记》"}
o2:main.Order{ID:123456, Title:"《七米的Go学习笔记》", CreatedTime:time.Time{wall:0x0, ext:63721678700, loc:(*time.Location)(nil)}}
```

### 使用匿名结构体添加字段

使用内嵌结构体能够扩展结构体的字段，但有时候我们没有必要单独定义新的结构体，可以使用匿名结构体简化操作：

```go
type UserInfo struct {
	ID   int    `json:"id"`
	Name string `json:"name"`
}

func anonymousStructDemo() {
	u1 := UserInfo{
		ID:   123456,
		Name: "七米",
	}
	// 使用匿名结构体内嵌User并添加额外字段Token
	b, err := json.Marshal(struct {
		*UserInfo
		Token string `json:"token"`
	}{
		&u1,
		"91je3a4s72d1da96h",
	})
	if err != nil {
		fmt.Printf("json.Marsha failed, err:%v\n", err)
		return
	}
	fmt.Printf("str:%s\n", b)
	// str:{"id":123456,"name":"七米","token":"91je3a4s72d1da96h"}
}
```

### 使用匿名结构体组合多个结构体

同理，也可以使用匿名结构体来组合多个结构体来序列化与反序列化数据：

```go
type Comment struct {
	Content string
}

type Image struct {
	Title string `json:"title"`
	URL   string `json:"url"`
}

func anonymousStructDemo2() {
	c1 := Comment{
		Content: "永远不要高估自己",
	}
	i1 := Image{
		Title: "赞赏码",
		URL:   "https://www.liwenzhou.com/images/zanshang_qr.jpg",
	}
	// struct -> json string
	b, err := json.Marshal(struct {
		*Comment
		*Image
	}{&c1, &i1})
	if err != nil {
		fmt.Printf("json.Marshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("str:%s\n", b)
	// json string -> struct
	jsonStr := `{"Content":"永远不要高估自己","title":"赞赏码","url":"https://www.liwenzhou.com/images/zanshang_qr.jpg"}`
	var (
		c2 Comment
		i2 Image
	)
	if err := json.Unmarshal([]byte(jsonStr), &struct {
		*Comment
		*Image
	}{&c2, &i2}); err != nil {
		fmt.Printf("json.Unmarshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("c2:%#v i2:%#v\n", c2, i2)
}
```

输出：

```go
str:{"Content":"永远不要高估自己","title":"赞赏码","url":"https://www.liwenzhou.com/images/zanshang_qr.jpg"}
c2:main.Comment{Content:"永远不要高估自己"} i2:main.Image{Title:"赞赏码", URL:"https://www.liwenzhou.com/images/zanshang_qr.jpg"}
```

### 处理不确定层级的json

如果json串没有固定的格式导致不好定义与其相对应的结构体时，我们可以使用`json.RawMessage`原始字节数据保存下来。

```go
type sendMsg struct {
	User string `json:"user"`
	Msg  string `json:"msg"`
}

func rawMessageDemo() {
	jsonStr := `{"sendMsg":{"user":"q1mi","msg":"永远不要高估自己"},"say":"Hello"}`
	// 定义一个map，value类型为json.RawMessage，方便后续更灵活地处理
	var data map[string]json.RawMessage
	if err := json.Unmarshal([]byte(jsonStr), &data); err != nil {
		fmt.Printf("json.Unmarshal jsonStr failed, err:%v\n", err)
		return
	}
	var msg sendMsg
	if err := json.Unmarshal(data["sendMsg"], &msg); err != nil {
		fmt.Printf("json.Unmarshal failed, err:%v\n", err)
		return
	}
	fmt.Printf("msg:%#v\n", msg)
	// msg:main.sendMsg{User:"q1mi", Msg:"永远不要高估自己"}
}
```

### 参考链接

- https://stackoverflow.com/questions/25087960/json-unmarshal-time-that-isnt-in-rfc-3339-format
- https://colobu.com/2017/06/21/json-tricks-in-Go/
- https://stackoverflow.com/questions/11066946/partly-json-unmarshal-into-a-map-in-go
- http://choly.ca/post/go-json-marshalling/