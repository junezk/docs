# Golang使用github.com/spf13/viper绑定命令行参数

给出一个例子viper绑定命令行选项的用法。

因此viper读取设置值的优先顺序是：

1. 命令行选项
2. 环境变量
3. 配置文件

```go
package main

import (
    "fmt"
    "os"
    "strings"

    "github.com/spf13/cobra"
    "github.com/spf13/viper"
)

const ProgramName = "testcmd"

var (
    address     string
)

func Cmd(programName string) *cobra.Command {
    var cmd = &cobra.Command{
        Use:   programName,
        Short: fmt.Sprintf("Sample %s program", programName),
        Long:  fmt.Sprintf("Long name of sample %s program", programName),
        RunE: func(cmd *cobra.Command, args []string) error {
            fmt.Printf("viper[server.address]=[%s]\n", viper.GetString("server.address"))
            return nil
        },
    }

    cmd.Flags().StringVarP(&address,  "address", "d", "",    "Target address")


    viper.SetConfigFile("config/config.yaml")
    viper.ReadInConfig()

    viper.SetEnvPrefix("t")
    viper.AutomaticEnv()
    viper.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))
    viper.BindPFlag("server.address", cmd.Flags().Lookup("address"))

    return cmd
}

func main() {
    cmd := Cmd(ProgramName)
    if cmd.Execute() != nil {
        os.Exit(1)
    }
}
```

函数`viper.BindPFlag`负责把viper项(`server.address`)和命令行参数(`address`)关联在一起。

假设有配置文件

```ruby
$ cat config/config.yaml
server:
  address: config_address
```

运行：

```go
$ go run main.go
viper[server.address]=[config_address]

$ T_SERVER_ADDRESS=env_address go run main.go
viper[server.address]=[env_address]

$ T_SERVER_ADDRESS=env_address go run main.go --address command_address
viper[server.address]=[command_address]
```

从这个例子可以看出命令行中的设置具有最高的优先级，然后是环境变量，最后才是配置文件。