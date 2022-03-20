# golang：类型断言的功用

看过很多别人写的类型断言，总感觉没那味，理解起来也不是很通透，首先我们需要知道类型断言是拿来干嘛的。

1. 首先我们需要知道在golang里面的函数参数可以为具体某一interface或者interface{}，这就是golang的设计哲学，万物起源interface{}，衍生出来specified interface，接下来就是基于衍生的specified interface衍生出来的现实生活中的实体--struct。

```go
//万恶起源
interface{}


// derivative specified interface 衍生接口
type animal interface{}
...
type tool interface{}


// specified struct
type animal01 struct{} , method01,method02....methodN
...
type animal0N struct{} , method01,method02....method0N

type tool01 struct{} , method01,method02....method0N
type tool0N struct{} , method01,method02....method0N

//定义一个function,参数为interface{}，那么所有实现了这个interface{}的struct都可以作为参数传进去
func typeAssert(interface{}){}

//定义另外一个function，参数为某一衍生借口，假若为animal interface;
//那么所有的animal0N的struct都可以作为参数，而tool0Nstruct则不行
func typeAssert(animal){} 
```

2.有了这一步之后，就有一个问题需要被解决，就是有0到N个实现了某一个衍生接口的struct，但是我又想对他们进行不一样的操作，这个时候就需要通过类型确定，或者类型断言来对他进行路由一样，golang里面是这样实现的，代码比较简单，如下所示：

```go
//由于这一个接口有多个实现类，所以当一个函数参数是以animal 接口为参数的话，则需要判断是具体那个实现类，这里就需要使用类型断言
func main() {
	// declare a dog
	prettyDog := dog{}
	typeAssert(prettyDog)
}

func typeAssert(a animal)  {
	// a为 animal接口实现类，但是我们想实现诸如：对不同的实现类又不一样的操作这个就需要根据不同的实现类来进行不同的操作
	switch a.(type) {
	case dog:
		fmt.Println("the is the dog")
		a.walk()
	case monkey:
		fmt.Println("this is the monkey")
		a.walk()
	}
}

//这里我们简单定义了个animal借口，有一个walk方法，然后有两个实现struct，dog和monkey
```

3.我觉得这个算是type assert的主要功用了吧，毕竟golang是面向接口编程的。