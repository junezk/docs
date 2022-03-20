# 彻底解决Golang获取当前项目绝对路径问题

### **导读**

由于Golang是编译型语言（非脚本型语言），如果你想在Golang程序中获取当前执行目录将是一件非常蛋疼的事情。以前大家最折中的解决方案就是**通过启动传参或是环境变量将路径手动传递到程序**，而今天我在看日志库的时候发现了一种新的解决方案。

### **Go程序两种不同的执行方式**

用Go编写的程序有两种执行方式，`go run`和`go build`

- 通常的做法是`go run`用于本地开发，用一个命令中快速测试代码确实非常方便；
- 在部署生产环境时，我们会通过`go build`构建出二进制文件然后上传到服务器再去执行。

### **两种启动方式会产生什么问题？**

**那么两种启动方式下，获取到当前执行路径会产生什么问题？**

话不多说，我们直接上代码

我们编写获取当前可执行文件路径的方法

```go
package main

import (
	"fmt"
	"log"
	"os"
	"path/filepath"
)

func main() {
	fmt.Println("getCurrentAbPathByExecutable = ", getCurrentAbPathByExecutable())
}


// 获取当前执行程序所在的绝对路径
func getCurrentAbPathByExecutable() string {
	exePath, err := os.Executable()
	if err != nil {
		log.Fatal(err)
	}
	res, _ := filepath.EvalSymlinks(filepath.Dir(exePath))
	return res
}
```

首先通过`go run`启动

```text
D:\Projects\demo>go run main.go
getCurrentAbPathByExecutable =  C:\Users\XXX\AppData\Local\Temp\go-build216571510\b001\exe
```

再尝试`go build`执行

```text
D:\Projects\demo>go build & demo.exe
getCurrentAbPathByExecutable =  D:\Projects\demo
```

通过对比执行结果，我们发现两种执行方式，我们获取到了不同的路径。而且很明显，`go run`获取到的路径是错误的。

**原因：** 这是由于`go run`会将源代码编译到系统`TEMP`或`TMP`环境变量目录中并启动执行；而`go build`只会在当前目录编译出可执行文件，并不会自动执行。

我们可以简单理解为，`go run main.go`等价于`go build & ./main`

虽然两种执行方式最终都是一样的过程：**源码->编译->可执行文件->执行输出**，但他们的执行目录却完全不一样了。

### **新的方案诞生**

这是在我今天查看服务日志(`zap`库)的时候，突然反应过来一件事情。比如下面是一条简单的日志，而服务是通过`go run`启动的，但日志库却把我正确的程序路径`D:/Projects/te-server/modules/es/es.go:139`给打印出来了

```text
2021-03-26 17:47:06    D:/Projects/te-server/modules/es/es.go:139  update es index {"index": "tags", "data": "[200 OK] {\"acknowledged\":true}"}
```

于是我马上去翻看`zap`源码，发现是通过`runtime.Caller()`实现的，其实所有Golang日志库都会有`runtime.Caller()`这个调用。

我开心的以为找到了最终答案，然后写代码试了下：

```text
package main

import (
	"fmt"
	"path"
	"runtime"
)

func main() {
	fmt.Println("getCurrentAbPathByCaller = ", getCurrentAbPathByCaller())

}

// 获取当前执行文件绝对路径（go run）
func getCurrentAbPathByCaller() string {
	var abPath string
	_, filename, _, ok := runtime.Caller(0)
	if ok {
		abPath = path.Dir(filename)
	}
	return abPath
}
```

首先在windows下面`go run` 和`go build`试一下

```text
D:\Projects\demo>go run main.go
getCurrentAbPathByCaller =  D:/Projects/demo


D:\Projects\demo>go build & demo.exe
getCurrentAbPathByCaller =  D:/Projects/demo
```

嗯~~ 结果完全正确！

然后我再把构建好的程序扔到linux再运行后，它把我windows的路径给打印出来了 --！

```text
[root@server app]# chmod +x demo 
[root@server app]# ./demo 
getCurrentAbPathByCaller =  D:/Projects/demo
```

没想到白白高兴一场，这个时候我就在想，既然`go run`时可以通过`runtime.Caller()`获取到正确的结果，`go build`时也可以通过` os.Executable()`来获取到正确的路径；

那如果我能判定当前程序是通过`go run`还是`go build`执行的，选择不同的路径获取方法，所有问题不就迎刃而解了吗。

### **区分程序是`go run`还是`go build`执行**

Go没有提供接口让我们区分程序是`go run`还是`go build`执行，但我们可以换个思路来实现：

根据`go run`的执行原理，我们得知它会源代码编译到系统`TEMP`或`TMP`环境变量目录中并启动执行；

那我们可以直接在程序中对比`os.Executable()`获取到的路径是否与环境变量`TEMP`设置的路径相同， 如果相同，说明是通过`go run`启动的，因为当前执行路径是在`TEMP`目录；不同的话自然是`go build`的启动方式。

下面是完整代码：

```text
package main

import (
	"fmt"
	"log"
	"os"
	"path"
	"path/filepath"
	"runtime"
	"strings"
)

func main() {
	fmt.Println("getTmpDir（当前系统临时目录） = ", getTmpDir())
	fmt.Println("getCurrentAbPathByExecutable（仅支持go build） = ", getCurrentAbPathByExecutable())
	fmt.Println("getCurrentAbPathByCaller（仅支持go run） = ", getCurrentAbPathByCaller())
	fmt.Println("getCurrentAbPath（最终方案-全兼容） = ", getCurrentAbPath())
}

// 最终方案-全兼容
func getCurrentAbPath() string {
	dir := getCurrentAbPathByExecutable()
	if strings.Contains(dir,getTmpDir())  {
		return getCurrentAbPathByCaller()
	}
	return dir
}

// 获取系统临时目录，兼容go run
func getTmpDir() string {
	dir := os.Getenv("TEMP")
	if dir == "" {
		dir = os.Getenv("TMP")
	}
	res, _ := filepath.EvalSymlinks(dir)
	return res
}

// 获取当前执行文件绝对路径
func getCurrentAbPathByExecutable() string {
	exePath, err := os.Executable()
	if err != nil {
		log.Fatal(err)
	}
	res, _ := filepath.EvalSymlinks(filepath.Dir(exePath))
	return res
}

// 获取当前执行文件绝对路径（go run）
func getCurrentAbPathByCaller() string {
	var abPath string
	_, filename, _, ok := runtime.Caller(0)
	if ok {
		abPath = path.Dir(filename)
	}
	return abPath
}
```

在windows执行

```text
D:\Projects\demo>go run main.go
getTmpDir（当前系统临时目录） =  C:\Users\XXX\AppData\Local\Temp
getCurrentAbPathByExecutable（仅支持go build） =  C:\Users\XXX\AppData\Local\Temp\go-build456189690\b001\exe
getCurrentAbPathByCaller（仅支持go run） =  D:/Projects/demo
getCurrentAbPath（最终方案-全兼容） =  D:/Projects/demo

D:\Projects\demo>go build & demo.exe
getTmpDir（当前系统临时目录） =  C:\Users\XXX\AppData\Local\Temp
getCurrentAbPathByExecutable（仅支持go build） =  D:\Projects\demo
getCurrentAbPathByCaller（仅支持go run） =  D:/Projects/demo
getCurrentAbPath（最终方案-全兼容） =  D:\Projects\demo
```

在windows编译后上传到Linux执行

```text
[root@server app]# pwd
/data/app
[root@server app]# ./demo 
getTmpDir（当前系统临时目录） =  .
getCurrentAbPathByExecutable（仅支持go build） =  /data/app
getCurrentAbPathByCaller（仅支持go run） =  D:/Projects/demo
getCurrentAbPath（最终方案-全兼容） =  /data/app
```

对比结果，我们可以看到，在不同的系统中，不同的执行方式，我们封装的`getCurrentAbPath`方法最终都输出的正确的结果，perfect！