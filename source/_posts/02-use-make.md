---
title: 使用make
date: 2019-04-09 02:12:59
tags: 
- C
- Makefile
---

## 使用make
使用make的第一个阶段，就是用它已知的方法来构建程序。make预置了一些知识，来从其它文件构建多种文件。在上一个练习中，有如下操作：

```
$ make hello-world
$ CFLAG="-Wall" make hello-world
```

第一条命令中你告诉make，“我想创建名为hello-word的文件”。于是，make执行了下面的动作：
* 文件hello-world存在吗？
* 没有的话。好的，那有没有其它文件是以hello-world开头的？
* 有，叫做hello-world.c。我知道如何构建.c文件吗？
* 知道。我会运行 `cc hello-world.c -o hello-world` 来构建它。
* 我将使用 `cc` 从 hello-world.c 文件为你构建 hello-world

上面的第二条命令，是向 make 命令传递“修改器”的途径。这个例子中，我执行了 `CFLAGS="-Wall" make hello-world`，它会给make使用的 `cc` 命令添加 -Wall 选项。这行命令告诉编译器要报告所有的警告。

## Makefile编写

创建文件并写入一下内容

```
CFLAGS=-Wall -g

clean:
    rm -rf hello-world
```

将文件在你当前文件夹下保存为Makefile。Make会自动假设当前文件夹下有一个叫Makefile的文件，并且会执行它。

首先我们在文件中设置CFLAGS,所以之后都不用再设置了。并且添加了-g标识来获取调试信息。接着我们写了一个`clean`的部门，它告诉make如何清理我们的小项目。

请确保，你的makefile文件和hello-world.c在同一个目录下，hello-world.c内容如下：

```
int main(int argc,char *argv[]){
    puts("Hello world.\nnice to see you!");
    return 0;
}
```

之后可以执行命令:

```
$ make clean
$ make hello-world
```

如果代码正常执行，你应该看到下面这些内容

```
[root@www practice02]# make clean
rm -f hello-world
[root@www practice02]# make hello-world
cc -Wall -g    hello-world.c   -o hello-world
hello-world.c: In function ‘main’:
hello-world.c:2:5: warning: implicit declaration of function ‘puts’ [-Wimplicit-function-declaration]
     puts("Hello world.\nnice to see you!");
     ^
```

你可以看出来,我执行了`make clean`，它告诉make执行我们的`clean`目标。在看makefile，发现`clean`下面有一些shell命令。你可以在此处输入任意多的命令，所以它是一个非常棒的自动化工具。

>注: 
>如果你修改了 ex1.c ，添加了 #include\<stdio\> ，输出中的关于 puts 的警告就会消失（这其实应该算作一个错误） 。我这里有警告是因为我并没有去掉它

## 附加题
* 创建目标 all:hello-world,可以用单个命令make构建hello-world
```
CFLAGS=-Wall -g

all: hello-world

clean:
    rm -f hello-world

```

* 阅读 man make 来了解关于他的更多信息。
* 阅读 man cc 来了解关于 -Wall 和 -g 行为的更多信息。
