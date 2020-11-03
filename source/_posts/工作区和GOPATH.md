---
title: 工作区和GOPATH
date: 2019-03-29 13:56:32
tags: go
---

我们学习Go语言时，第一件要做的是，就是根据自己电脑的操作系统和计算架构，从[Go语言官网](https://golang.google.cn)下载对应的二进制包，也就是拿来即用的安装包。

随后，**解压安装包**、**放置到某个目录**、**配置环境变量**，并在命令行输入 `go version` 来验证是否安装成功。

```shell
# 解压
[root@www package]# ls go1.12.1.linux-amd64.tar.gz 
go1.12.1.linux-amd64.tar.gz
[root@www package]# tar zxvf go1.12.1.linux-amd64.tar.gz -C /usr/local/

# 设置GOROOT、GOPATH、GOBIN环境变量
[root@www ~]# vim .bash_profile

PATH=$PATH:$HOME/bin:/usr/local/php/bin:/usr/local/go/bin
export PATH

export GOROOT=/usr/local/go
export GOPATH=/root/peek-a-bow
export GOBIN=/root/peek-a-bow/bin

# 创建工作目录
mkdir -p /root/peek-a-bow/{src,bin,pkg}
```

在整个安装过程中，需要配置3个环境变量，简单介绍一下:
* GOROOT: Go语言的安装目录
* GOPATH: 自定义的工作目录
* GOBIN: Go语言生成的可执行文件的目录

可以把GOPATH简单理解成Go语言的工作目录，它的值是一个目录的路径，也可以是多个目录的路径，每个目录都代表Go语言的一个工作区（workspace）。

我们需要利于这些工作区，去放置 Go 语言的源码文件（source file）,以及安装(install)后的归档文件（archive file）和可执行文件（executable file）。

事实上，由于Go语言项目在其生命周期内的所有操作（编码依赖管理、构建、测试、安装等）基本上都是围绕着GOPATH和工作区进行的。它的背后有3个知识点需要注意:

* 1.Go语言源码的组织是怎样的；
* 2.你是否了解源码安装后的结果；
* 3.你是否理解构建和安装Go程序的过程。

### 1.Go语言源码组织方式

Go语言是以代码包为基本组织单位的。所以说，Go语言源码的组织方式就是以环境变量GOPATH、工作区、src目录和代码包为主线的。

### 2.了解源码安装后的结果

源码文件在安装过程中，如果产生了归档文件，就会放进该工作区的pkg子目录；如果产生了可执行文件，就会放进该工作区的bin子目录。

### 3.理解构建和安装Go程序的过程

构建使用 `go build`,安装使用命令 `go install`。构建和安装代码包的时候都会执行编译、打包等操作。并且，这些操作产生的任何文件都会先被保存到某个临时的目录中。

如果构建的是**库源码**文件,那么操作结果只会保存在临时目录中，安装**库源码**文件，那么它的结果会被搬运到它所在工作区的pkg目录下的某个子目录中。

如果构建的是**命令源码**文件，那么它的操作结果文件会被搬运到源码文件所在的目录中。如果安装的是**命令源码**文件，那么结果文件会被搬运到它所在工作区的bin目录中，或者环境变量GOBIN指向的目录中。

