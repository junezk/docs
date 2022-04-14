# [golang常用库：字段参数验证库-validator使用](https://www.cnblogs.com/jiujuan/p/13823864.html)

## 一、背景[#](https://www.cnblogs.com/jiujuan/p/13823864.html#3479039843)

在平常开发中，特别是在web应用开发中，为了验证输入字段的合法性，都会做一些验证操作。比如对用户提交的表单字段进行验证，或者对请求的API接口字段进行验证，验证字段的合法性，保证输入字段值的安全，防止用户的恶意请求。

一般的做法是用正则表达式，一个字段一个字段的进行验证。一个一个字段验证的话，写起来比较繁琐。那有没更好的方法，进行字段的合法性验证？有， 这就是下面要介绍的 [validator](https://github.com/go-playground/validator) 这个验证组件。

代码地址：
https://github.com/go-playground/validator

文档地址：
https://github.com/go-playground/validator/blob/master/README.md

## 二、功能介绍[#](https://www.cnblogs.com/jiujuan/p/13823864.html#3672075756)

这个验证包 [github.com/go-playground/validator](https://github.com/go-playground/validator) 验证功能非常多。

### 标记之间特殊符号说明[#](https://www.cnblogs.com/jiujuan/p/13823864.html#1963847459)

- 逗号( `,`)：把多个验证标记隔开。`注意`：隔开逗号之间不能有空格, `validate:"lt=0,gt=100"`，逗号那里不能有空格，否则panic
- 横线( `-` )：跳过该字段不验证
- 竖线( `|` )：使用多个验证标记，但是只需满足其中一个即可
- required：表示该字段值必输设置，且不能为默认值
- omitempty：如果字段未设置，则忽略它

### 范围比较验证[#](https://www.cnblogs.com/jiujuan/p/13823864.html#1163903440)

> doc: https://github.com/go-playground/validator/blob/master/README.md#comparisons

范围验证: 切片、数组和map、字符串，验证其长度；数值，验证大小范围

- lte：小于等于参数值，`validate:"lte=3"` (小于等于3)
- gte：大于等于参数值，`validate:"lte=120,gte=0"` (大于等于0小于等于120)
- lt：小于参数值，`validate:"lt=3"` (小于3)
- gt：大于参数值，`validate:"lt=120,gt=0"` (大于0小于120)
- len：等于参数值，`validate:"len=2"`
- max：最大值，小于等于参数值，`validate:"max=20"` (小于等于20)
- min：最小值，大于等于参数值，`validate:"min=2,max=20"` (大于等于2小于等于20)
- ne：不等于，`validate:"ne=2"` (不等于2)
- oneof：只能是列举出的值其中一个，这些值必须是数值或字符串，以空格分隔，如果字符串中有空格，将字符串用单引号包围，`validate:"oneof=red green"`

例子：

```go
Copytype User struct {
    Name string `json:"name" validate:"min=0,max=35"`
    Age  unit8  `json:"age" validate:"lte=90,gte=0"`
}
```

更多功能请参看文档 [validator comparisons doc](https://github.com/go-playground/validator/blob/master/README.md#comparisons)

### 字符串验证[#](https://www.cnblogs.com/jiujuan/p/13823864.html#1860812930)

> doc: https://github.com/go-playground/validator/blob/master/README.md#strings

- contains：包含参数子串，`validate:"contains=tom"` (字段的字符串值包含tom)
- excludes：包含参数子串，`validate:"excludes=tom"` (字段的字符串值不包含tom)
- startswith：以参数子串为前缀，`validate:"startswith=golang"`
- endswith：以参数子串为后缀，`validate:"startswith=world"`

例子：

```go
Copytype User struct { 
    Name string `validate:"contains=tom"` 
    Age int `validate:"min=1"`
}
```

更多功能请参看文档 [validator strings doc](https://github.com/go-playground/validator/blob/master/README.md#strings)

### 字段验证[#](https://www.cnblogs.com/jiujuan/p/13823864.html#3759694446)

> doc: https://github.com/go-playground/validator/blob/master/README.md#fields

- eqcsfield：跨不同结构体字段验证，比如说 Struct1 Filed1，与结构体Struct2 Field2相等，

```go
Copytype Struct1 struct {
    Field1 string `validate:eqcsfield=Struct2.Field2`
    Struct2 struct {
        Field2 string 
    }
}
```

- necsfield：跨不同结构体字段不相等
- eqfield：同一结构体字段验证相等，最常见的就是输入2次密码验证

```go
Copytype User struct { 
    Name string `validate:"lte=4"` 
    Age int `validate:"min=20"` 
    Password string `validate:"min=10"`
    Password2 string `validate:"eqfield=Password"`
}
```

- nefield：同一结构体字段验证不相等

```go
Copytype User struct {
    Name string `validate:"lte=4"` 
    Age int `validate:"min=20"` 
    Password string `validate:"min=10,nefield=Name"`
}
```

- gtefield：大于等于同一结构体字段，`validate:"gtefiled=Field2"`
- ltefield：小于等于同一结构体字段

更多功能请参看文档：[validator Fields DOC](https://github.com/go-playground/validator/blob/master/README.md#fields)

### 网络验证[#](https://www.cnblogs.com/jiujuan/p/13823864.html#873417027)

> doc: https://github.com/go-playground/validator/blob/master/README.md#network

- ip：字段值是否包含有效的IP地址，`validate:"ip"`
- ipv4：字段值是否包含有效的ipv4地址，`validate:"ipv4"`
- ipv6：字段值是否包含有效的ipv6地址，`validate:"ipv6"`
- uri：字段值是否包含有效的uri，`validate:"uri"`
- url：字段值是否包含有效的uri，`validate:"url"`

更多功能请参看文档：[validator network DOC](https://github.com/go-playground/validator/blob/master/README.md#network)

### Format[#](https://www.cnblogs.com/jiujuan/p/13823864.html#301475669)

> doc: https://github.com/go-playground/validator/blob/master/README.md#format

- base64：字段值是否包含有效的base64值

更多功能请参看文档 [validator strings doc](https://github.com/go-playground/validator/blob/master/README.md#strings)

### 其他[#](https://www.cnblogs.com/jiujuan/p/13823864.html#2346051894)

> 请参看文档: https://github.com/go-playground/validator/blob/master/README.md#other

## 三、安装[#](https://www.cnblogs.com/jiujuan/p/13823864.html#1495047000)

go get:

> go get github.com/go-playground/validator/v10

在文件中引用validator包：

> import "github.com/go-playground/validator/v10"

## 四、validator使用[#](https://www.cnblogs.com/jiujuan/p/13823864.html#784698480)

> 文档：https://github.com/go-playground/validator/blob/master/README.md#examples

### 例子1：验证单个字段变量值[#](https://www.cnblogs.com/jiujuan/p/13823864.html#932298499)

validation1.go

```go
Copypackage main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

func main() {
	validate := validator.New()

	var boolTest bool
	err := validate.Var(boolTest, "required")
	if err != nil {
		fmt.Println(err)
	}
	var stringTest string = ""
	err = validate.Var(stringTest, "required")
	if err != nil {
		fmt.Println(err)
	}

	var emailTest string = "test@126.com"
	err = validate.Var(emailTest, "email")
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println("success") // 输出： success。 说明验证成功
	}

	emailTest2 := "test.126.com"
	errs := validate.Var(emailTest2, "required,email")
	if errs != nil {
		fmt.Println(errs) // 输出: Key: "" Error:Field validation for "" failed on the "email" tag。验证失败
	}

	fmt.Println("\r\nEnd!!")
    
    
}
```

运行输出：

> go run simple1.go
> Key: '' Error:Field validation for '' failed on the 'required' tag
> Key: '' Error:Field validation for '' failed on the 'required' tag
> success
> Key: '' Error:Field validation for '' failed on the 'email' tag
>
> End!!

### 例子2：验证结构体struct[#](https://www.cnblogs.com/jiujuan/p/13823864.html#244574535)

> from：[struct validate](https://github.com/go-playground/validator/blob/master/_examples/simple/main.go)

validation_struct.go，这个程序还列出了效验出错字段的一些信息，

```go
Copypackage main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

type User struct {
	FirstName string     `validate:"required"`
	LastName  string     `validate:"required"`
	Age       uint8      `validate:"gte=0,lte=130"`
	Email     string     `validate:"required,email"`
	Addresses []*Address `validate:"required,dive,required"`
}

type Address struct {
	Street string `validate:"required"`
	City   string `validate:"required"`
	Planet string `validate:"required"`
	Phone  string `validate:"required"`
}

func main() {
	address := &Address{
		Street: "Eavesdown Docks",
		Planet: "Persphone",
		Phone:  "none",
	}

	user := &User{
		FirstName: "Badger",
		LastName:  "Smith",
		Age:       135,
		Email:     "Badger.Smith@gmail.com",
		Addresses: []*Address{address},
	}

	validate := validator.New()
	err := validate.Struct(user)
	if err != nil {
		fmt.Println("=== error msg ====")
		fmt.Println(err)

		if _, ok := err.(*validator.InvalidValidationError); ok {
			fmt.Println(err)
			return
		}

		fmt.Println("\r\n=========== error field info ====================")
		for _, err := range err.(validator.ValidationErrors) {
           // 列出效验出错字段的信息
			fmt.Println("Namespace: ", err.Namespace())
			fmt.Println("Fild: ", err.Field())
			fmt.Println("StructNamespace: ", err.StructNamespace())
			fmt.Println("StructField: ", err.StructField())
			fmt.Println("Tag: ", err.Tag())
			fmt.Println("ActualTag: ", err.ActualTag())
			fmt.Println("Kind: ", err.Kind())
			fmt.Println("Type: ", err.Type())
			fmt.Println("Value: ", err.Value())
			fmt.Println("Param: ", err.Param())
			fmt.Println()
		}

		// from here you can create your own error messages in whatever language you wish
		return
	}
}
```

运行 输出：

> $ go run validation_struct.go
> === error msg ====
> Key: 'User.Age' Error:Field validation for 'Age' failed on the 'lte' tag
> Key: 'User.Addresses[0].City' Error:Field validation for 'City' failed on the 'required' tag
>
> =========== error field info ====================
> Namespace: User.Age
> Fild: Age
> StructNamespace: User.Age
> StructField: Age
> Tag: lte
> ActualTag: lte
> Kind: uint8
> Type: uint8
> Value: 135
> Param: 130
>
> Namespace: User.Addresses[0].City
> Fild: City
> StructNamespace: User.Addresses[0].City
> StructField: City
> Tag: required
> ActualTag: required
> Kind: string
> Type: string
> Value:
> Param:

还可以给字段加一些其他tag信息，方面form，json的解析，如下：

```go
Copytype User struct {
    FirstName string     `form:"firstname" json:"firstname" validate:"required"`
	LastName  string     `form:"lastname" json:"lastname" validate:"required"`
	Age       uint8      ` form:"age" json:"age"validate:"gte=0,lte=130"`
	Email     string     ` form:"email" json:"email" validate:"required,email"`
}
```

### 例子2.2：验证slice map[#](https://www.cnblogs.com/jiujuan/p/13823864.html#4286407902)

#### slice[#](https://www.cnblogs.com/jiujuan/p/13823864.html#1343427171)

slice验证中用到一个tag关键字 `dive` , 意思深入一层验证。

validate_slice.go

```go
Copypackage main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

func main() {
	sliceone := []string{"123", "onetwothree", "myslicetest", "four", "five"}

	validate := validator.New()
	err := validate.Var(sliceone, "max=15,dive,min=4")
	if err != nil {
		fmt.Println(err)
	}

	slicetwo := []string{}
	err = validate.Var(slicetwo, "min=4,dive,required")
	if err != nil {
		fmt.Println(err)
	}
}
```

运行输出：

> $ go run validate_slice.go
> Key: '[0]' Error:Field validation for '[0]' failed on the 'min' tag
> Key: '' Error:Field validation for '' failed on the 'min' tag

说明：

```go
Copysliceone := []string{"123", "onetwothree", "myslicetest", "four", "five"}
validate.Var(sliceone, "max=15,dive,min=4")
```

> 第二个参数中tag关键字 `dive` 前面的 max=15，验证 [] , 也就是验证slice的长度，`dive` 后面的 min=4，验证slice里的值长度，也就是说 dive 后面的 tag 验证 slice 的值

**那如果是二维slice验证呢**？如：

```go
Copyslicethree := [][]string{}
validate.Var(slicethree, "min=2,dive,len=2,dive,required")

validate.Var(slicethree, "min=2,dive,dive,required")
```

说明：

> 这里有2个 dive，刚好深入到二维slice，但他们也有不同之处，第二个表达式的第一个dive后没有设置tag。
> 第一个验证表达式：
> min=2：验证第一个 [] 方括号的值长度 ;
> len=2：验证第二个 []string 长度;
> required：验证slice里的值

> 第二个验证表达式：
> min=2：验证第一个 [] 方括号的值长度 ;
> dive： 后没有设置tag值，**不验证第二个 []string** ;
> required： 验证slice里的值

#### map[#](https://www.cnblogs.com/jiujuan/p/13823864.html#3610025938)

map的验证中也需要tag关键字 `dive`， 另外，它还有 `keys` 和 `endkeys` 两tag，验证这2个tag之间map的 key，而不是value值。

validate_map.go

```go
Copypackage main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

func main() {
	var mapone map[string]string

	mapone = map[string]string{"one": "jimmmy", "two": "tom", "three": ""}

	validate := validator.New()
	err := validate.Var(mapone, "gte=3,dive,keys,eq=1|eq=2,endkeys,required")
	if err != nil {
		fmt.Println(err)
	}
}
```

运行输出：

> $ go run validate_map.go
> Key: '[three]' Error:Field validation for '[three]' failed on the 'eq=1|eq=3' tag
> Key: '[three]' Error:Field validation for '[three]' failed on the 'required' tag
> Key: '[one]' Error:Field validation for '[one]' failed on the 'eq=1|eq=3' tag
> Key: '[two]' Error:Field validation for '[two]' failed on the 'eq=1|eq=3' tag

说明：

> gte=3：验证map自己的长度；
> dive后的 keys,eq=1|eq=2,endkeys：验证map的keys个数，也就是验证 [] 里值。上例中定义了一个string，所以明显报了3个错误。
> required：验证 map的值value

**那嵌套map怎么验证**？
如：map[[3]string]string，和上面slice差不多，使用多个 `dive`

```go
Copyvar maptwo map[[3]string]string{}
validate.Var(maptwo, "gte=3,dive,keys,dive,eq=1|eq=3,endkeys,required")
```

说明：

> gte=3： 验证map的长度；
> keys,dive,eq=1|eq=3,endkeys：keys和endkeys中有一个dive(深入一级)，验证map中key的数组每一个值
> required： 验证map的值

### 用户自定义函数验证[#](https://www.cnblogs.com/jiujuan/p/13823864.html#3101280756)

用户自定义函数验证字段是否合法，效验是否正确。

### 例子3: 通过字段tag自定义函数[#](https://www.cnblogs.com/jiujuan/p/13823864.html#823576614)

##### validate.RegisterValidation

customer_tag.go：

```go
Copypackage main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

type User struct {
	Name string `form:"name" json:"name" validate:"required,CustomerValidation"` //注意：required和CustomerValidation之间不能有空格，否则panic。CustomerValidation：自定义tag-函数标签
	Age  uint8  ` form:"age" json:"age" validate:"gte=0,lte=80"`                 //注意：gte=0和lte=80之间不能有空格，否则panic
}

var validate *validator.Validate

func main() {
	validate = validator.New()
	validate.RegisterValidation("CustomerValidation", CustomerValidationFunc) //注册自定义函数，前一个参数是struct里tag自定义，后一个参数是自定义的函数

	user := &User{
		Name: "jimmy",
		Age:  86,
	}

	fmt.Println("first value: ", user)
	err := validate.Struct(user)
	if err != nil {
		fmt.Printf("Err(s):\n%+v\n", err)
	}

	user.Name = "tom"
	user.Age = 29
	fmt.Println("second value: ", user)
	err = validate.Struct(user)
	if err != nil {
		fmt.Printf("Err(s):\n%+v\n", err)
	}
}

// 自定义函数
func CustomerValidationFunc(f1 validator.FieldLevel) bool {
    // f1 包含了字段相关信息
    // f1.Field() 获取当前字段信息
    // f1.Param() 获取tag对应的参数
    // f1.FieldName() 获取字段名称
    
	return f1.Field().String() == "jimmy"
}
```

运行输出：

> $ go run customer.go
> first value: &{jimmy 86}
> Err(s):
> Key: 'User.Age' Error:Field validation for 'Age' failed on the 'lte' tag
> second value: &{tom 29}
> Err(s):
> Key: 'User.Name' Error:Field validation for 'Name' failed on the 'CustomerValidation' tag

**注意：

上面代码`user struct`定义中 ，`validate`里的required和CustomerValidation之间不能有空格，否则运行时报panic错误：`panic: Undefined validation function ' CustomerValidation' on field 'Name'`

### 例子4：自定义函数-直接注册函数1[#](https://www.cnblogs.com/jiujuan/p/13823864.html#3994425023)

不通过字段tag自定义函数，直接注册函数。

##### RegisterStructValidation

> https://github.com/go-playground/validator/blob/master/_examples/struct-level/main.go

customer1.go

```go
Copypackage main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

type User struct {
	FirstName      string `json:firstname`
	LastName       string `json:lastname`
	Age            uint8  `validate:"gte=0,lte=130"`
	Email          string `validate:"required,email"`
	FavouriteColor string `validate:"hexcolor|rgb|rgba"`
}

var validate *validator.Validate

func main() {
	validate = validator.New()

	validate.RegisterStructValidation(UserStructLevelValidation, User{})

	user := &User{
		FirstName:      "",
		LastName:       "",
		Age:            30,
		Email:          "TestFunc@126.com",
		FavouriteColor: "#000",
	}

	err := validate.Struct(user)
	if err != nil {
		fmt.Println(err)
	}
}

func UserStructLevelValidation(sl validator.StructLevel) {
	user := sl.Current().Interface().(User)

	if len(user.FirstName) == 0 && len(user.LastName) == 0 {
		sl.ReportError(user.FirstName, "FirstName", "firstname", "firstname", "")
		sl.ReportError(user.LastName, "LastName", "lastname", "lastname", "")
	}
}
```

运行输出：

> $ go run customer1.go
> Key: 'User.FirstName' Error:Field validation for 'FirstName' failed on the 'firstname' tag
> Key: 'User.LastName' Error:Field validation for 'LastName' failed on the 'lastname' tag

### 例子5：自定义函数-直接注册函数2[#](https://www.cnblogs.com/jiujuan/p/13823864.html#3237313568)

##### RegisterCustomTypeFunc

> https://github.com/go-playground/validator/blob/master/_examples/custom/main.go

[validate.RegisterCustomTypeFunc](https://github.com/go-playground/validator/blob/d6b17fd90bd4dd9d16a594c3035ceadc3de0193a/validator_instance.go#L241")：验证类型的自定义函数

customer2.go：

```go
Copypackage main

import (
	"database/sql"
	"database/sql/driver"
	"fmt"
	"reflect"

	"github.com/go-playground/validator/v10"
)

type DbBackedUser struct {
	Name sql.NullString `validate:"required"`
	Age  sql.NullInt64  `validate:"required"`
}

var validate *validator.Validate

func main() {
	validate = validator.New()

	validate.RegisterCustomTypeFunc(ValidateValuer, sql.NullString{}, sql.NullInt64{}, sql.NullBool{}, sql.NullFloat64{})

	// build object for validation
	x := DbBackedUser{Name: sql.NullString{String: "", Valid: true}, Age: sql.NullInt64{Int64: 0, Valid: false}}

	err := validate.Struct(x)
	if err != nil {
		fmt.Printf("Err(s):\n%+v\n", err)
	}
}

func ValidateValuer(field reflect.Value) interface{} {
	if valuer, ok := field.Interface().(driver.Valuer); ok {
		val, err := valuer.Value()
		if err == nil {
			return val
		}
		// handle the error how you want
	}
	return nil
}
```

运行输出：

> $ go run customer.go
> Err(s):
> Key: 'DbBackedUser.Name' Error:Field validation for 'Name' failed on the 'required' tag
> Key: 'DbBackedUser.Age' Error:Field validation for 'Age' failed on the 'required' tag

**注意，这个函数**：
[RegisterCustomTypeFunc](https://github.com/go-playground/validator/blob/d6b17fd90bd4dd9d16a594c3035ceadc3de0193a/validator_instance.go#L241")，它上面有2行注释：

> // RegisterCustomTypeFunc registers a CustomTypeFunc against a number of types
> //
> // NOTE: this method is not thread-safe it is intended that these all be registered prior to any validation

它是一个验证数据类型自定义函数，NOTE:这个方法不是线程安全的

### 例子6：两字段比较[#](https://www.cnblogs.com/jiujuan/p/13823864.html#2071381690)

**两个字段比较，有一种是密码比较验证**，用户注册时候验证2次密码输入是否相同。用tag `eqfield` 比较两字段。。

verify_pwd.go:

```go
Copypackage main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

// 注册用户 user struct
type User struct {
	UserName  string `json:"username" validate:"lte=14,gte=4"`
	Password  string `json:"password" validate:"max=20,min=6"`
	Password2 string `json:"password2" validate:"eqfield=Password"`
}

func main() {
	validate := validator.New()

	user1 := User{
		UserName:  "jim",
		Password:  "123456",
		Password2: "12345",
	}
	fmt.Println("validate user1 value: ", user1)
	err := validate.Struct(user1)
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println("====================")

	user2 := User{
		UserName:  "jimy",
		Password:  "123456",
		Password2: "123456",
	}
	fmt.Println("validate user2 value: ", user2)
	err = validate.Struct(user2)
	if err != nil {
		fmt.Println(err)
	}
}
```

运行输出：

> $ go run verify_pwd.go
> validate user1 value: {jim 123456 12345}
> Key: 'User.UserName' Error:Field validation for 'UserName' failed on the 'gte' tag
> Key: 'User.Password2' Error:Field validation for 'Password2' failed on the 'eqfield' tag
> ====================
> validate user2 value: {jimy 123456 123456}

**还有一种是2变量字段比较**，见下面例子 eq_field.go:

```go
Copypackage main

import (
	"fmt"

	"github.com/go-playground/validator/v10"
)

func main() {
	field1 := "tom"
	field2 := "jimmy"

	validate := validator.New()

	fmt.Println("tag nefield: ")
	err := validate.VarWithValue(field1, field2, "nefield")
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println("correct")
	}

	fmt.Println("===========================")

	fmt.Println("tag eqfield: ")
	err = validate.VarWithValue(field1, field2, "eqfield")
	if err != nil {
		fmt.Println(err)
	}
}
```

运行输出：

> $ go run eq_field.go
> tag nefield:
> correct
> ===========================
> tag eqfield:
> Key: '' Error:Field validation for '' failed on the 'eqfield' tag

### 例子7：翻译/自定义字段错误[#](https://www.cnblogs.com/jiujuan/p/13823864.html#1985938529)

- [universal-translator](https://github.com/go-playground/universal-translator/tree/master/_examples)
- [i10n](https://github.com/go-playground/locales)

```go
Copypackage main

import (
	"fmt"
	"strings"

	"github.com/go-playground/locales/en"
	"github.com/go-playground/locales/zh"
	ut "github.com/go-playground/universal-translator"
	"github.com/go-playground/validator/v10"

	zhtrans "github.com/go-playground/validator/v10/translations/zh"
	// entrans "github.com/go-playground/validator/v10/translations/en"
)

type Student struct {
	Name  string `validate:required`
	Email string `validate:"email"`
	Age   int    `validate:"max=30,min=12"`
}

func main() {
	en := en.New() //英文翻译器
	zh := zh.New() //中文翻译器

	// 第一个参数是必填，如果没有其他的语言设置，就用这第一个
	// 后面的参数是支持多语言环境（
	// uni := ut.New(en, en) 也是可以的
	// uni := ut.New(en, zh, tw)
	uni := ut.New(en, zh)
	trans, _ := uni.GetTranslator("zh") //获取需要的语言

	student := Student{
		Name:  "tom",
		Email: "testemal",
		Age:   40,
	}
	validate := validator.New()

	zhtrans.RegisterDefaultTranslations(validate, trans)

	err := validate.Struct(student)
	if err != nil {
		// fmt.Println(err)

		errs := err.(validator.ValidationErrors)
		fmt.Println(removeStructName(errs.Translate(trans)))
	}
}

func removeStructName(fields map[string]string) map[string]string {
	result := map[string]string{}

	for field, err := range fields {
		result[field[strings.Index(field, ".")+1:]] = err
	}
	return result
}
```

运行输出：

> $ go run customer_err_info3.go
> map[Age:Age必须小于或等于30 Email:Email必须是一个有效的邮箱]

## 五、参考[#](https://www.cnblogs.com/jiujuan/p/13823864.html#265064922)

- https://github.com/go-playground/validator/blob/master/README.md
- https://github.com/go-playground/validator/tree/master/_examples
- https://github.com/go-playground/universal-translator/tree/master/_examples
- https://github.com/go-playground/validator/issues/633
- [validator translator](https://www.liwenzhou.com/posts/Go/validator_usages/#autoid-1-0-2)