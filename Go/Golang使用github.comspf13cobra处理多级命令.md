# Golang使用github.com/spf13/cobra处理多级命令

## 基础用法

```go
package main

import (
    "fmt"
    "os"

    "github.com/spf13/cobra"
)

const ProgramName = "testcmd"

var (
    address     string
    tlsEnabled  bool
)

func Cmd(programName string) *cobra.Command {
    var cmd = &cobra.Command{
        Use:   programName,
        Short: fmt.Sprintf("Sample %s program", programName),
        Long:  fmt.Sprintf("Long name of sample %s program", programName),
        RunE: func(cmd *cobra.Command, args []string) error {
            fmt.Printf("Welcome to %s world with address=[%s], tls=[%t]\n", programName, address, tlsEnabled)
            if len(args) > 0 {
                return fmt.Errorf("unexpected args are met: %v", args)
            }
            return nil
        },
    }

    cmd.Flags().StringVarP(&address,  "address", "d", "",    "Target address")
    cmd.Flags().BoolVarP(&tlsEnabled, "tls",     "",  false, "Whethere TLS is enabled")

    return cmd
}

func main() {
    cmd := Cmd(ProgramName)
    if cmd.Execute() != nil {
        os.Exit(1)
    }
}
```

对比golang原始的flag模块，这个格式处理的更加简洁。

## 多级命令

假设主程序包含两个子命令version和mysub，然后mysub又包含一个子命令mygrandsub，等于是主程序的孙子命令。

```ruby
|--mygrog
    |-- version
    |-- mysub
          |-- mygrandsub
```

文件目录结构为：



```cpp
HOME
|-- main.go
|-- cmd
      |-- cmd.go           # define commons shared by all subcommands
      |-- version.go       # define subcommand: version
      |-- mysub.go         # define subcommand: mysub
      |-- mygrandsub.go    # define grandsubcommand as subcommand of mysub
```

这里把所有的子命令都放在一个package目录下面。

main.go



```go
package main

import (
    "os"

    "github.com/spf13/cobra"

    "cmd"
)

const ProgramName = "myprog"

var loggingLevel string

func main() {
    var mainCmd = &cobra.Command{Use: ProgramName}

    // Define persistent command-line flags that are valid for commands and subcommands.
    mainCmd.PersistentFlags().StringVarP(&loggingLevel, "logging-level", "l", "info", "Global log-level setting")

    mainCmd.AddCommand(cmd.VersionCmd(ProgramName))
    mainCmd.AddCommand(cmd.MySubCmd())

    // On failure Cobra prints the usage message and error string, so we only
    // need to exit with a non-0 status
    if mainCmd.Execute() != nil {
        os.Exit(1)
    }
}
```

- 主程序定义了一个命令行参数logging-level，所有的子命令都能用。
- 主程序添加进两个子命令：version和mysub

cmd.go



```go
package cmd

import (
    "log"

    "github.com/spf13/cobra"
    "github.com/spf13/pflag"
)

var (
    cmdName      string
    cmdVersion   int
)

var flagset *pflag.FlagSet

func init() {
    resetFlags()
}

// resetFlags init a potential flag set, it could be picked up optionally by an individual command
func resetFlags() {
    flagset = &pflag.FlagSet{}

    flagset.StringVarP(&cmdName, "name",     "n", "",   "Name of the cmd")
    flagset.IntVarP(&cmdVersion, "version",  "v", 1,    "Version of the cmd")
}

// add flagset pre-defined flag to given cmd by flag name
func attachFlags(cmd *cobra.Command, flagNames []string) {
    cmdFlags := cmd.Flags()
    for _, flagName := range flagNames {
        if flag := flagset.Lookup(flagName); flag != nil {
            cmdFlags.AddFlag(flag)
        } else {
            log.Fatalf("Could not find flag '%s' to attach to command '%s'", flagName, cmd.Name())
        }
    }
}
```

version.go



```go
package cmd

import (
    "fmt"
    "runtime"

    "github.com/spf13/cobra"
)

const Version = 1

func VersionCmd(programName string) *cobra.Command {
    var cobraCommand = &cobra.Command{
        Use:   "version",
        Short: "Print fabric peer version.",
        Long:  `Print current version of the fabric peer server.`,
        Run: func(cmd *cobra.Command, args []string) {
            // Parsing of the command line is done so silence cmd usage
            cmd.SilenceUsage = true
            fmt.Print(GetInfo(programName))
        },
    }
    return cobraCommand
}

// GetInfo returns version information for the peer
func GetInfo(programName string) string {
    return fmt.Sprintf("%s:\n Version: %d\n Go version: %s\n OS/Arch: %s\n",
        programName, Version, runtime.Version(), fmt.Sprintf("%s/%s", runtime.GOOS, runtime.GOARCH))
}
```

mysub.go



```go
package cmd

import (
    "fmt"
    "log"

    "github.com/spf13/cobra"
)

const MySubName = "mysub"

var (
    mysubTLS        bool
)

func MySubCmd() *cobra.Command {
    var cmd *cobra.Command = &cobra.Command {
        Use:    MySubName,
        Short:  fmt.Sprintf("Query using the specified %s", MySubName),
        Long:   fmt.Sprintf("Query using the long specified name %s", MySubName),
        RunE:   func(cmd *cobra.Command, args []string) error {
            log.Printf("%s:RunE: Entry", MySubName)

            if len(args) != 0 {
                return fmt.Errorf("trailing args detected")
            }

            log.Printf("%s:RunE: Exit", MySubName)
            return nil
        },
    }

    // assemble flag from predefined flagset
    flags := []string{ "name", "version" }
    attachFlags(cmd, flags)

    // assemble additional flags that are not in flagsSet
    cmd.Flags().BoolVarP(&mysubTLS,     "tls",  "", false, "Whethere TLS is enabled or not")

    // add a subcommand
    cmd.AddCommand(MyGrandSubCmd())

    return cmd
}
```

- mysub定义了三个个本地命令行参数name, version, 和tls
- mysub定义了一个它的子命令mygrandsub，这相当于是主程序的孙子命令。

mygrandsub.go



```go
package cmd

import (
    "fmt"
    "log"

    "github.com/spf13/cobra"
)

const MyGrandSubName = "mygrandsub"

var (
    mygrandsubtimeout   int
)

func MyGrandSubCmd() *cobra.Command {
    var cmd *cobra.Command = &cobra.Command {
        Use:    MyGrandSubName,
        Short:  fmt.Sprintf("Query using the specified %s", MyGrandSubName),
        Long:   fmt.Sprintf("Query using the long specified name %s", MyGrandSubName),
        RunE:   func(cmd *cobra.Command, args []string) error {
            log.Printf("%s:RunE: Entry", MyGrandSubName)

            log.Printf("%s:RunE: Exit", MyGrandSubName)
            return nil
        },
    }

    // assemble flag from predefined flagset
    flags := []string{ "name", "version" }
    attachFlags(cmd, flags)

    // assemble additional flags that are not in flagsSet
    cmd.Flags().IntVarP(&mygrandsubtimeout, "timeout",  "t", 100, "Command execution timeout value in second")

    return cmd
}
```

- mygrandsub定义了三个个本地命令行参数name, version, 和timeout

看下运行结果：

- 主程序



```bash
$ go run main.go --help
Usage:
  myprog [command]

Available Commands:
  completion  generate the autocompletion script for the specified shell
  help        Help about any command
  mysub       Query using the specified mysub
  version     Print fabric peer version.

Flags:
  -h, --help                   help for myprog
  -l, --logging-level string   Global log-level setting (default "info")

Use "myprog [command] --help" for more information about a command.
```

- 子命令version



```bash
$ go run main.go version --help
Print current version of the fabric peer server.

Usage:
  myprog version [flags]

Flags:
  -h, --help   help for version

Global Flags:
  -l, --logging-level string   Global log-level setting (default "info")
```

- 子命令mysub



```csharp
$ go run main.go mysub --help
Query using the long specified name mysub

Usage:
  myprog mysub [flags]
  myprog mysub [command]

Available Commands:
  mygrandsub  Query using the specified mygrandsub

Flags:
  -h, --help          help for mysub
  -n, --name string   Name of the cmd
      --tls           Whethere TLS is enabled or not
  -v, --version int   Version of the cmd (default 1)

Global Flags:
  -l, --logging-level string   Global log-level setting (default "info")

Use "myprog mysub [command] --help" for more information about a command.
```

- 孙子命令mygrandsub



```csharp
$ go run main.go mysub mygrandsub --help
Query using the long specified name mygrandsub

Usage:
  myprog mysub mygrandsub [flags]

Flags:
  -h, --help          help for mygrandsub
  -n, --name string   Name of the cmd
  -t, --timeout int   Command execution timeout value in second (default 100)
  -v, --version int   Version of the cmd (default 1)

Global Flags:
  -l, --logging-level string   Global log-level setting (default "info")
```

