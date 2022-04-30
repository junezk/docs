# Gin的数据验证总结

gin框架使用[github.com/go-playground/validator](https://github.com/go-playground/validator)进行参数校验，目前已经支持`github.com/go-playground/validator/v10`了，我们需要在定义结构体时使用 `binding` tag标识相关校验规则，可以查看[validator文档](https://godoc.org/github.com/go-playground/validator#hdr-Baked_In_Validators_and_Tags)查看支持的所有 tag。

## 基本示例

```go
type SignUpParam struct {
	Age        uint8  `json:"age" binding:"gte=1,lte=130"`
	Name       string `json:"name" binding:"required"`
	Email      string `json:"email" binding:"required,email"`
	Password   string `json:"password" binding:"required"`
	RePassword string `json:"re_password" binding:"required,eqfield=Password"`
}

var u SignUpParam
if err := c.ShouldBind(&u); err != nil {
    c.JSON(http.StatusOK, gin.H{
        "msg": err.Error(),
    })
    return
}
```

validator默认的错误信息为英文，库本身是支持国际化的，借助相应的语言包可以实现校验错误提示信息的自动翻译。中文语言包为`github.com/go-playground/validator/v10/translations/zh`。

## 功能介绍

### 标记之间特殊符号说明

- 逗号( `,`)：把多个验证标记隔开。`注意`：隔开逗号之间不能有空格, `validate:"lt=0,gt=100"`，逗号那里不能有空格，否则panic
- 横线( `-` )：跳过该字段不验证
- 竖线( `|` )：使用多个验证标记，但是只需满足其中一个即可
- required：表示该字段值必输设置，且不能为默认值
- omitempty：如果字段未设置，则忽略它

### 范围比较验证

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

### 字符串验证

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

### 字段验证

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

### 网络验证

- ip：字段值是否包含有效的IP地址，`validate:"ip"`
- ipv4：字段值是否包含有效的ipv4地址，`validate:"ipv4"`
- ipv6：字段值是否包含有效的ipv6地址，`validate:"ipv6"`
- uri：字段值是否包含有效的uri，`validate:"uri"`
- url：字段值是否包含有效的uri，`validate:"url"`

### Format

- base64：字段值是否包含有效的base64值

## 用户自定义函数验证

### 自定义字段校验方法

例如：为`SignUpParam`添加一个需要使用自定义校验方法`checkDate`做参数校验的字段`Date`。

```go
type SignUpParam struct {
	Age        uint8  `json:"age" binding:"gte=1,lte=130"`
	Name       string `json:"name" binding:"required"`
	Email      string `json:"email" binding:"required,email"`
	Password   string `json:"password" binding:"required"`
	RePassword string `json:"re_password" binding:"required,eqfield=Password"`
	// 需要使用自定义校验方法checkDate做参数校验的字段Date
	Date       string `json:"date" binding:"required,datetime=2006-01-02,checkDate"`
}
```

其中`datetime=2006-01-02`是内置的用于校验日期类参数是否满足指定格式要求的tag。 如果传入的`date`参数不满足`2006-01-02`这种格式就会提示如下错误：

```bash
{"msg":{"date":"date的格式必须是2006-01-02"}}
```

针对date字段除了内置的`datetime=2006-01-02`提供的格式要求外，假设我们还要求该字段的时间必须是一个未来的时间（晚于当前时间），像这样针对某个字段的特殊校验需求就需要我们使用自定义字段校验方法了。

首先我们要在需要执行自定义校验的字段后面添加自定义tag，这里使用的是`checkDate`，注意使用英文分号分隔开。

```go
// customFunc 自定义字段级别校验方法
func customFunc(fl validator.FieldLevel) bool {
	date, err := time.Parse("2006-01-02", fl.Field().String())
	if err != nil {
		return false
	}
	if date.Before(time.Now()) {
		return false
	}
	return true
}
```

定义好了字段及其自定义校验方法后，就需要将它们联系起来并注册到我们的校验器实例中。

```go
// 在校验器注册自定义的校验方法
if err := v.RegisterValidation("checkDate", customFunc); err != nil {
	return err
}
```

这样，我们就可以对请求参数中`date`字段执行自定义的`checkDate`进行校验了。

validator 包还支持下面自定义方法，详情查看文档。

- 自定义验证错误显示语言
- 自定义结构验证函数
- 自定义错误提示





