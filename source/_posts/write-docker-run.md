---
title: 构建实现run命令的容器
date: 2019-04-24 23:24:32
tags:
- docker
- runC
---

实现`run`命令

构建一个简单版本的`run`命令，类似于`docker run -it [command]`，为了了解`Docker`启动容器的原理，该简单版本的实现参考了`runC`的实现。

---

1.目前的代码文件结构如下：
```
.
├── container
│   ├── container_process.go
│   └── init.go
├── main_command.go
├── main.go
├── README.md
└── run.go
```
2.首先，来看一下入口`main`文件
```
package main

import (
    "github.com/urfave/cli"
    "github.com/Sirupsen/logrus"
    "os"
)

const usage = `my docker is a simple container runtime implement`

//mydocker run -it /bin/bash

func main() {
    app := cli.NewApp()
    app.Name = "mydocker"
    app.Usage = usage

    app.Commands = []cli.Command{
        initCommand,
        runCommand,
    }

    //设置日志格式
    app.Before = func(context *cli.Context) error {
        logrus.SetFormatter(&logrus.JSONFormatter{})
        logrus.SetOutput(os.Stdout)
        return nil;
    }

    if err := app.Run(os.Args); err != nil {
        logrus.Fatal(err)
    }
}
```
使用 github.com/urfave/cli 提供的命令行工具, 该工具的用法, [点此](/2019/04/22/introducing-urfave-cli/)。定义mydocker的两个基本命令,`initCommand`和`runCommand`，在app.Before内初始化一下`logrus`的日志配置。下面看一下，具命令的定义。

3.`runCommand` & `initCommand` 定义
```
//这里定义`runCommand`的`Flags`，其作用类似于命令时使用 -- 来指定参数

var runCommand = cli.Command{
    Name: "run",
    Usage: `Create a container with namespace and cgroups limit
           mydocker run -it [command]`,
    Flags: []cli.Flag{
        cli.BoolFlag{
            Name:  "it",
            Usage: "enable tty",
        },
    },
    /**
    这里是run命令执行的真正函数
    1. 判断参数是否包含command
    2. 获取用户指定的command
    3. 调用 Run function 去准备启动容器
     */
    Action: func(context *cli.Context) error {
        if len(context.Args()) < 1 {
            return fmt.Errorf("Missing container command")
        }
        cmd := context.Args().Get(0)
        tty := context.Bool("it")
        Run(tty, cmd)
        return nil
    },
}

//定义initCommand的具体操作，此操作为内部方法，禁止外部调用
var initCommand = cli.Command{
    Name:  "init",
    Usage: "Init container process run user's process in container, Do not call it outside",
    /**
    1.获取传递过来的command参数
    2.执行容器初始化操作
     */
    Action: func(context *cli.Context) error {
        logrus.Infof("init come on")
        cmd := context.Args().Get(0)
        logrus.Infof("command %s", cmd)
        err := container.RunContainerInitProcess(cmd, nil)
        return err
    },
}
```

4.先来看一下`Run`和`NewParentProcess`做了哪些事情。
这里是父进程，也就是当前进程执行的内容。
* 这里的`/proc/self/exe`的调用中，`/proc/self`指的是当前运行进程自己的环境，`exec`其实就是自己调用了自己，使用这种方式对创建出来的进程初始化。
* 后面的`args`是参数，其中`init`是传递给本进程的第一个参数，在本例中，其实就会去调用`initCommand`去初始化进程的一些环境和资源。`./mydocker init [command]` 
* 下面的`clone`参数就是去fork出来一个新的进程，并且使用了`namespace`隔离新创建的进程和外部环境。
* 如果用户指定了 `-it` 参数，就需要把当前进程的输入输出导入到标准的输入输出上

```
// conatiner_process.go
package container

import (
    "os/exec"
    "syscall"
    "os"
)

func NewParentProcess(tty bool, command string) *exec.Cmd {
    args := []string{"init", command}
    cmd := exec.Command("/proc/self/exe", args...)
    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS | syscall.CLONE_NEWNET | syscall.CLONE_NEWIPC,
    }
    if tty {
        cmd.Stdin = os.Stdout
        cmd.Stdout = os.Stdout
        cmd.Stderr = os.Stderr
    }
    return cmd
}
```
```
// run.go
package main

import (
    "mydocker/container"
    "github.com/Sirupsen/logrus"
    "os"
)

//启动init进程
func Run(tty bool, command string) {
    parent := container.NewParentProcess(tty, command)
    if err := parent.Run(); err != nil {
        logrus.Error(err)
    }
    parent.Wait()
    os.Exit(-1)
}
```

5.那么`init`函数里面发生了什么呢？

这里的`init`函数是在容器内执行的，也就是说代码执行到这里，容器所在的进程其实已经创建出来了，这是本容器执行的第一个进程。

使用`mount`先去挂载`proc`文件系统，以便后面通过`ps`命令去查看当前容器内进程情况。

```
// init.go 
package container

import (
    "github.com/Sirupsen/logrus"
    "syscall"
    "os"
)

func RunContainerInitProcess(command string, args []string) error {
    logrus.Infof("command %s", command)
    defaultMountFlags := syscall.MS_NOEXEC | syscall.MS_NOSUID | syscall.MS_NODEV
    syscall.Mount("proc", "/proc", "proc", uintptr(defaultMountFlags), "")
    argv := []string{command}
    if err := syscall.Exec(command, argv, os.Environ()); err != nil {
        logrus.Errorf(err.Error())
    }
    return nil
}
```
这里`MountFlag`意思如下：
* MS_NOEXEC在本文件系统中不允许运行其它程序
* MS_NOSUID在本文件系统中运行程序的时候，不允许`set-user-ID`或`set-group-ID`
* MS_NODEV这个参数自Linux2.4以来，所有mount的系统都会默认设定的

**本函数最后的syscall.Exec，这个系统调用实现了完成初始化并将用户进程运行起来的操作。**下面解释一下这句话的神奇之处。

首先，使用`Docker`创建起来一个容器后，会发现容器内的第一个进程，也就是PID为1的那个进程，是指定的前台进程。但是，根据前面所讲，容器启动后的第一个进程不是用户进程，而是`init`初始化的进程。这个时候通过`ps`命令就会发现，容器内的第一进程变成了自己的`init`，这个和预想的不一样。你可能回想，大不了把第一个`init`进程给`kill`掉。但是PID为1的进程是不能被`kill`掉的，如果该进程被`kill`掉，我们的容器也就退出了。那么有什么办法？这里execve系统调用就可以大显神威了。

syscall.Exec这个方法，其实最终调用了Kernel的int execve(const char *filename,const *const argv[],char *const envp[]);这个系统函数。它的作用是执行当前的filename对应的程序，会覆盖当前进程的镜像、数据和堆栈等信息，包括PID，这些都会将要运行的进程覆盖掉。也就是说，调用这个方法，将用户指定的进程运行起来，把最初的`init`进程给替换掉，这样当进入到容器内部的时候，就会发现容器内的第一个程序就是我们指定的进程了[command]。这其实也是目前Docker使用的容器引擎`runC`的实现方式之一。

6.流程图如图所示:
<div align="center"><img src="https://github.com/meichaofan/static-file/blob/master/docker/runc.png?raw=true
" width = 50% height = 50% /></div>
<div align="center">图1：mydocker 启动流程</div>

7.下面编译运行一下。
>
#使用 go build，在mydocker目录下进行编译
>
#使用 `./mydocker run -it /bin/sh` 命令，其中 `-it` 表示想要以交互的形式运行容器， `/bin/bash` 为指定容器的第一个进程。
```
root@ubuntu1:/home/meichaofan/peek-a-boo/src/mydocker# ./mydocker run -it /bin/sh
{"level":"info","msg":"init come on","time":"2019-04-24T21:09:35-07:00"}
{"level":"info","msg":"command /bin/sh","time":"2019-04-24T21:09:35-07:00"}
{"level":"info","msg":"command /bin/sh","time":"2019-04-24T21:09:35-07:00"}
# ps aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.0  0.0   4508   756 pts/0    S    21:09   0:00 /bin/sh
root          5  0.0  0.1  39104  3188 pts/0    R+   21:09   0:00 ps aux
```
在容器运行 `ps -aux` 时，可以发现 `/bin/sh` 进程是容器内的第一个进程，PID=1。而 `ps -aux` 是PID为1的进程创建出来的。

> 

这里的 `/bin/sh` 是一个会在前台一直运行的进程。如果指定一个运行就退出的进程会是什么效果。
```
root@ubuntu1:/home/meichaofan/peek-a-boo/src/mydocker# ./mydocker run -it /bin/ls
{"level":"info","msg":"init come on","time":"2019-04-24T21:19:05-07:00"}
{"level":"info","msg":"command /bin/ls","time":"2019-04-24T21:19:05-07:00"}
{"level":"info","msg":"command /bin/ls","time":"2019-04-24T21:19:05-07:00"}
container  main_command.go  main.go  mydocker  README.md  run.go
```
由于没有`chroot`，所以目前的系统文件是继承宿主主机的系统文件，运行了一下 `ls` 命令，当容器启动起来以后，打印出了当前目录内容，然后便退出了，这个结果和Docker要求容器必须有一个一直在前台运行的进程的要求是一致的。

