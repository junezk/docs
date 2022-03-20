# golang如何按行读取文本

golang如何按行读取文本

golang的库bufio.Scanner是非常方便用来处理文本文件。

下面的例子是按行读取文本文件。

```go
package main

import (
    "os"
    "log"
    "fmt"
    "bufio"
)

func main() {
    err := HandleText("a.txt")
    if err != nil {
        panic(err)
    }
}

func HandleText(textfile string) error {
    file, err := os.Open(textfile)
    if err != nil {
        log.Printf("Cannot open text file: %s, err: [%v]", textfile, err)
        return err
    }
    defer file.Close()

    scanner := bufio.NewScanner(file)
    for scanner.Scan() {
        line := scanner.Text()  // or
      //line := scanner.Bytes()

      //do_your_function(line)
        fmt.Printf("%s\n", line)
    }

    if err := scanner.Err(); err != nil {
        log.Printf("Cannot scanner text file: %s, err: [%v]", textfile, err)
        return err
    }

    return nil
}
```

bufio.Reader和bufio.Scanner的关系

bufio.Reader是go早期的版本也是用来处理文本，使用起来有一些不方便，例如需要处理行太长的问题，而bufio.Scanner是go1.1中新增加的功能，既然是新加的功能肯定是修正之前的不足，在使用上更加方便，比如就不用处理行太长的问题。

总之就是bufio.Scanner是后开发的模块，功能更强大，使用更方便。