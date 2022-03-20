# golang系列——实战http上传文件

前面已经讲述了构建服务端和客户端并进行简单的数据交换，本文将实现从客户端上报图片到服务端并保存。

直接对前文的客户端进行改造后如下：

```go
package main

import (
    "io/ioutil"
    "log"
    "net/http"
    "os"
    "strings"
    "sync"
    "time"
)

var wc sync.WaitGroup

//SendData sends data to server.
func SendData(c *http.Client, url string, method string, filePath string) {
    defer wc.Done()

    if c == nil {
        log.Fatalln("client is nil")
    }
    if method == "POST" {
        boundary := "ASSDFWDFBFWEFWWDF" //可以自己设定，需要比较复杂的字符串作
        var data []byte
        if _, err := os.Lstat(filePath); err == nil {
            file, _ := os.Open(filePath)
            defer file.Close()

            data, _ = ioutil.ReadAll(file)
        } else {
            log.Fatal("file not exist")
        }

        picData := "--" + boundary + "\n"
        picData = picData + "Content-Disposition: form-data; name=\"userfile\"; filename=" + filePath + "\n"
        picData = picData + "Content-Type: application/octet-stream\n\n"
        picData = picData + string(data) + "\n"
        picData = picData + "--" + boundary + "\n"
        picData = picData + "Content-Disposition: form-data; name=\"text\";filename=\"1.txt\"\n\n"
        picData = picData + string("data=ali") + "\n"
        picData = picData + "--" + boundary + "--"

        req, err := http.NewRequest(method, url, strings.NewReader(picData))
        req.Header.Set("Content-Type", "multipart/form-data; boundary=" + boundary)
        if err == nil {
            if rep, err := c.Do(req); err == nil {
                content, _ := ioutil.ReadAll(rep.Body)
                log.Println("get response: " + string(content))
                rep.Body.Close()
            }
        }
    } else if method == "GET" {
        //TODO get data from server
    }
}

func main() {
    client := &http.Client{
        Timeout: time.Second * 3,
    }
    postImgPath := "1.png"
    method := "POST"
    url := "http://127.0.0.1:8000/postdata"
    wc.Add(1)

    go SendData(client, url, method, postImgPath)

    wc.Wait()
}
```

在客户端代码中，有几点需要说明一下：

**multipart/form-data**：在请求头中的Content-Type字段应该设置成**multipart/form-data**，利用表单的形式上传文件。同时还应该加上**boundary**的内容，**boundary**可以自己指定。

**boundary**：如何大家和本文一样自己构建请求体，需要利用**boundary**进行数据分隔，分隔的格式有严格的要求，下面给出一个范例。其中name字段是表单中该元素的名词，filename是内容的名称(可以认为是文件名)，二者是有区别的；另外具体的内容，如1.png的内容与前一行必须有一个空行(回车)；boundary加上字符"--"作为分隔符，而结束的boundary还应该在末尾加上字符"--"，

```text
--boundary  //分割符 
Content-Disposition: form-data; name="userfile"; filename="1.png"  
Content-Type: application/octet-stream  

1.png的内容  
--${bound}  
Content-Disposition: form-data; name="text"; filename="username"  

name=Tom
--boundary--
```

随后再编写server端，代码如下所示

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
    "strings"
    "time"
)

//DownloadFile download file from client to local.
func DownloadFile(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case "GET":
        fmt.Println("GET")
        w.Write([]byte(string("hi, get successful")))
    case "POST":
        fmt.Println("POST")
        r.ParseForm() //解析表单
        imgFile, _, err := r.FormFile("userfile")//获取文件内容
        if err != nil {
            log.Fatal(err)
        }
        defer imgFile.Close()

        imgName := ""
        files := r.MultipartForm.File //获取表单中的信息
        for k, v := range files {
            for _, vv := range v {
                fmt.Println(k + ":" + vv.Filename)//获取文件名
                if strings.Index(vv.Filename, ".png") > 0 {
                    imgName = vv.Filename
                }
            }
        }

        saveFile, _ := os.Create(imgName)
        defer saveFile.Close()
        io.Copy(saveFile, imgFile) //保存

        w.Write([]byte("successfully saved"))
    default:
        fmt.Println("default")
    }
}

func main() {
    server := &http.Server{
        Addr:         "127.0.0.1:8000",
        ReadTimeout:  2 * time.Second,
        WriteTimeout: 2 * time.Second,
    }
    mux := http.NewServeMux()
    mux.HandleFunc("/postdata", DownloadFile)
    server.Handler = mux
    server.ListenAndServe()
}
```

server端主要是响应请求并存储文件，相关注释见代码。理论上可以上传任意格式、任意大小的文件，例如我在本机测试成功上传40M的mp4文件，然而由于网络、传输效率、成功率等考虑，大文件可以分多块上传，大家可以自行探究，后期如果有时间，我将再进行详细讨论。

目前的很多golang框架都能够方便的实现文件的上传下载等功能，本文主要想通过实现http的文件传输了解具体的实现过程和其中的网络知识，而在实际项目中，大家可以基于各类成熟框架进行开发。