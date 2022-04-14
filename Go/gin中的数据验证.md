# gin中的数据验证

`gin`中有自带验证，定义一个验证结构体，让他能够直接定义。

## gin自带数据验证

```go
type Login struct {
	Username string `form:"username" json:"username" binding:"required"`
	Password string `form:"password" json:"password" binding:"required"`
}
```

写一个服务器，并且添加验证器：

```go
func main(){
		s := gin.Default()
		s.Get("/", func (context *gin.Context) {
			var l Login
			if err := context.ShouldBindWith(&l,binding.Query); err != nil {
				context.JSON(200,gin.H{
					"data":nil,
					"message":err.Error()
				})
				return
			}
		})
}
```

上面的代码是一个正常的验证器代码。

## validator数据验证包

但是有一个验证器更厉害，可以直接翻译成中文。

安装这个库：

```bash
go get github.com/go-playground/validator/v10
```

首先，初始化一个验证器中文错误库的函数：

```go
import (
        "fmt"
	"github.com/gin-gonic/gin/binding"
	"github.com/go-playground/locales/en"
	"github.com/go-playground/locales/zh"
	ut "github.com/go-playground/universal-translator"
	"github.com/go-playground/validator/v10"
	enTranslations "github.com/go-playground/validator/v10/translations/en"
	zhTranslations "github.com/go-playground/validator/v10/translations/zh"
)

var Trans ut.Translator // 全局验证器

func ValidatorTrans(locale string) (err error) {
	// 修改gin框架中的Validator引擎属性，实现自定制
	if v, ok := binding.Validator.Engine().(*validator.Validate); ok {

		zhT := zh.New() // 中文翻译器
		enT := en.New() // 英文翻译器
		uni := ut.New(enT, zhT, enT)

		// locale 通常取决于 http 请求头的 'Accept-Language'
		var ok bool
		// 也可以使用 uni.FindTranslator(...) 传入多个locale进行查找
		Trans, ok = uni.GetTranslator(locale)
		if !ok {
			return fmt.Errorf("uni.GetTranslator(%s) failed", locale)
		}

		// 注册翻译器
		switch locale {
		case "en":
			err = enTranslations.RegisterDefaultTranslations(v, Trans)
		case "zh":
			err = zhTranslations.RegisterDefaultTranslations(v, Trans)
		default:
			err = enTranslations.RegisterDefaultTranslations(v,Trans)
		}
		return
	}
	return
}
```

接着在服务里面初始化它，并使用：

```diff
func main(){
		s := gin.Default()
+		if err := inits.ValidatorTrans("zh"); err != nil {
+			fmt.Printf("inits trans failed, err:%v\\n", err)
+			return
+		}
		s.Get("/", func (context *gin.Context) {
			var l Login
			if err := context.ShouldBindWith(&l,binding.Query); err != nil {
+				if  errs, ok := err.(validator.ValidationErrors); ok {
+					context.JSON(200,gin.H{
+						"data":nil,
+						"message":errs.Translate(global.Trans),
+					})
+					return
				}
			}
		})
		s.Run(":8080")
}
```

这样，一个验证器就做好了。

## gin中如何自定义验证器



```go
type Booking struct {
	// 包含绑定和验证的数据,bookabledate就是自定义的验证函数
	CheckIn time.Time `form:"check_in" binding:"required,bookabledate" time_format:"2006-01-02"`
	// gtfield=CheckIn只对数字或时间有效，参考官网链接
	// https://pkg.go.dev/github.com/go-playground/validator/v10#section-readme
	CheckOut time.Time `form:"check_out" binding:"required,gtfield=CheckIn,bookabledate" time_format:"2006-01-02"`
}

// 自定义验证器
var bookableDate validator.Func = func(fl validator.FieldLevel) bool {
	if date, ok := fl.Field().Interface().(time.Time); ok {
		today := time.Now()
		if today.After(date) {  // 输入的日期必须大于今天的日期，否则验证失败
			return false
		}
	}
	return true
}

// 使用自定义数据验证
func main() {
	router := gin.Default()

	// 注册验证器
	validate, ok := binding.Validator.Engine().(*validator.Validate)
	if ok {
		validate.RegisterValidation("bookabledate", bookableDate)
	}

	router.GET("/bookable", getBookable)
	router.Run()
}

func getBookable(context *gin.Context) {
	var book Booking
	if err := context.ShouldBindWith(&book, binding.Query); err == nil {
		context.JSON(200, gin.H{"message": "book date is valid"})
	} else {
		context.JSON(http.StatusBadRequest, gin.H{"err": err.Error()})
	}
}
```

