# Golang使用github.com/spf13/viper处理配置参数

一个简单例子使用viper读取配置信息。



```go
package main

import (
    "fmt"
    "strings"

    "github.com/spf13/viper"

)

func main() {
    //viper.SetConfigFile("config/config.yaml")
    viper.AddConfigPath("config")
    viper.SetConfigType("yaml")
    viper.SetConfigName("config")

    if err := viper.ReadInConfig(); err == nil {
        fmt.Println("Using config file:", viper.ConfigFileUsed())
    }

    // For environment variables.
    viper.AutomaticEnv()
    viper.SetEnvPrefix("TEST")
    replacer := strings.NewReplacer(".", "_")
    viper.SetEnvKeyReplacer(replacer)

    key := "A1.B"
    fmt.Printf("v[%s]=[%s]\n", key, viper.GetString(key))
    key  = "A2.B"
    fmt.Printf("v[%s]=[%s]\n", key, viper.GetString(key))
}
```

函数说明：

1. viper.SetConfigFile 直接设置配置文件
2. viper.AddConfigPath/viper.SetConfigType/viper.SetConfigName
    通过设置多个路径一次来搜索配置文件，直到搜到一个位置。从函数名可以看出，viper只能支持一个配置文件，一种类型，可以包括多个搜索路径。
3. viper.AutomaticEnv是否去读取环境变量
    如果设置了，那么会优先读取环境变量，即使配置文件中已经定义了，环境变量会覆盖配置文件的值。
4. viper.SetEnvPrefix/viper.SetEnvKeyReplacer设置环境变量的读取规则。
    首先点号('.')是不能出现在环境变量的，所以必须用viper.SetEnvKeyReplacer把点号('.')映射成下划线('_')，然后viper.SetEnvPrefix可以设置一个环境变量的前缀。

举个例子来说：

1. viper.GetString("AA") // 直接读取环境变量AA
2. viper.GetString("AA.BB") // 试图读取环境变量AA.BB
    但是AA.BB格式是不允许出现在环境变量的，所以可能使用AA_BB环境变量，然后使用viper.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))
3. viper.GetString("AA.BB")直接读取的环境变量AA_BB，但是有时我们希望不要读取其他进程的，不是为当前进程设置的环境变量，这样我们需要把当前进程的环境变量加一个统一的前缀，例如TEST，所以环境变量TEST_AA_BB的值就是viper.GetString("AA.BB")最终取到的值。