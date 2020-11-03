---
title: 介绍urfave/cli 一款构建命令行app的go包
date: 2019-04-22 22:53:29
tags:
- cli
- go
---

一个简单，快速构建基于命令行应用的工具包
https://github.com/urfave/cli

---

### 安装
```
go get github.com/urfave/cli
```

### 使用

1.Cli的哲学之一是API应该有趣和充满惊喜，所以cli应用程序可以做到在`main()`函数里只有一行代码。

```
package main

import (
    "github.com/urfave/cli"
    "os"
    "log"
)

func main() {
    err := cli.NewApp().Run(os.Args)
    if err != nil {
        log.Fatal(err)
    }
}
```
1.1.运行并输出一些默认信息
```
NAME:
   one - A new cli application

USAGE:
   one [global options] command [command options] [arguments...]

VERSION:
   0.0.0

COMMANDS:
     help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h     show help
   --version, -v  print the version
```

2.设置执行动作，和输出一些帮助文档
```
package main

import (
    "github.com/urfave/cli"
    "fmt"
    "os"
    "log"
)

func main() {
    app := cli.NewApp()
    app.Name = "boom"
    app.Usage = "make an explosive entrance"
    app.Action = func(c *cli.Context) error {
        fmt.Println("boom! I say!")
        return nil
    }

    err := app.Run(os.Args)
    if err != nil {
        log.Fatal(err)
    }
}
```
2.1.编译并运行 go build . && ./main
```
meichaofan@ubuntu1:~/peek-a-bow/src/cli/two$ go run main.go
boom! I say!
```
2.2.运行 ./main -h , 可以看到，显示有相关文档
```
meichaofan@ubuntu1:~/peek-a-bow/src/cli/two$ ./main -h
NAME:
   boom - make an explosive entrance

USAGE:
   main [global options] command [command options] [arguments...]

VERSION:
   0.0.0

COMMANDS:
     help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h     show help
   --version, -v  print the version
```
3.支持参数
```
package main

import (
    "github.com/urfave/cli"
    "fmt"
    "os"
    "log"
)

func main() {
    app := cli.NewApp()

    app.Action = func(ctx *cli.Context) error {
        fmt.Printf("Hello %q\n", ctx.Args().Get(0))
        return nil
    }

    err := app.Run(os.Args)
    if err != nil {
        log.Fatal(err)
    }
}
```
3.1.输出
```
meichaofan@ubuntu1:~/peek-a-bow/src/cli/three$ ./main huanhuan
Hello "huanhuan"
```

4.支持标识位 flags
```
package main

import (
    "os"
    "log"
    "github.com/urfave/cli"
    "fmt"
)

func main() {
    app := cli.NewApp()

    app.Flags = []cli.Flag{
        cli.StringFlag{
            Name:  "lang",
            Value: "english",
            Usage: "language for the greeting",
        },
    }

    app.Action = func(ctx *cli.Context) error {

        name := "huanhuan"
        if ctx.NArg() > 0 {
            name = ctx.Args().Get(0)
            fmt.Printf("there have %d args",len(ctx.Args()))
        }

        if ctx.String("lang") == "spanish" {
            fmt.Println("Hala", name)
        } else {
            fmt.Println("Hello", name)
        }

        return nil
    }

    err := app.Run(os.Args)
    if err != nil {
        log.Fatal(err)
    }
}
```
4.1.注：Flag和argumens要分清楚 ./main --lang spanish aa bb cc ，其中有3个参数，1个标识
```
meichaofan@ubuntu1:~/peek-a-bow/src/cli/four$ ./main --lang spanish aa bb cc
there have 3 args
Hala aa
```

5.还可以将flag值注入到变量中，通过变量来判断
```
package main

import (
  "log"
  "os"
  "fmt"

  "github.com/urfave/cli"
)

func main() {
  var language string

  app := cli.NewApp()

  app.Flags = []cli.Flag {
    cli.StringFlag{
      Name:        "lang",
      Value:       "english",
      Usage:       "language for the greeting",
      Destination: &language,
    },
  }

  app.Action = func(c *cli.Context) error {
    name := "someone"
    if c.NArg() > 0 {
      name = c.Args()[0]
    }
    if language == "spanish" {
      fmt.Println("Hola", name)
    } else {
      fmt.Println("Hello", name)
    }
    return nil
  }

  err := app.Run(os.Args)
  if err != nil {
    log.Fatal(err)
  }
}
```
6.指定flag的值，这些值的占位符用后引号表示。
```
package main

import (
  "log"
  "os"

  "github.com/urfave/cli"
)

func main() {
  app := cli.NewApp()

  app.Flags = []cli.Flag{
    cli.StringFlag{
      Name:  "config, c",
      Usage: "Load configuration from `FILE`",
    },
  }

  err := app.Run(os.Args)
  if err != nil {
    log.Fatal(err)
  }
}
```
6.1.我们将会在帮助中看到:
```
--config FILE, -c FILE   Load configuration from FILE
```
