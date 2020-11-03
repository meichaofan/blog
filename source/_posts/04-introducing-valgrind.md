---
title: C程序分析工具 valgrind
date: 2019-04-18 00:59:44
tags:
- C
- valgrind
---

现在介绍另外一个工具，在学习C的过程中，要习惯性的使用，它就是 Valgrind 。Valgrind是一个运行你的程序的程序，并且随后会报告所有你犯下的可怕错误。

## 安装 Valgrind

有两种安装方式，在Centos中，可以通过yum install -y Valgrind方式安装，另外是下载Valgrind源码安装。

这里演示源码安装方式

```
1) 下载Valgrind源码包
wget wget https://sourceware.org/pub/valgrind/valgrind-3.15.0.tar.bz2

2) 解压
tar jxvf valgrind-3.15.0.tar.bz2

3) configure
./configure

4) 编译
make

5) 安装
make install
```

## 使用Valgrind

这里写一个带有错误的C程序，待会儿让Valgrind来运行一下,error.c源码如下：

```
#include<stdio.h>

int main(){
    int age = 12;
    int height;

    printf("I am %d years old.\n");
    printf("I am %d inches tall.\n",height);

    return 0;
}
```

当前程序有两个错误：
* 没有初始化height变量
* 没有将age变量传入第一个printf函数

使用make构建

```
[root@www practice04]# make
cc -Wall -g    error.c   -o error
error.c: In function ‘main’:
error.c:7:5: warning: format ‘%d’ expects a matching ‘int’ argument [-Wformat=]
     printf("I am %d years old.\n");
     ^
error.c:4:9: warning: unused variable ‘age’ [-Wunused-variable]
     int age = 12;
         ^
error.c:8:11: warning: ‘height’ is used uninitialized in this function [-Wuninitialized]
     printf("I am %d inches tall.\n",height);
           ^
```

使用Valgrind来运行error程序
```
==17432== Memcheck, a memory error detector
==17432== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==17432== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==17432== Command: ./error
==17432== 
I am -16775928 years old.
==17432== Conditional jump or move depends on uninitialised value(s)
==17432==    at 0x4E7C9F2: vfprintf (in /usr/lib64/libc-2.17.so)
==17432==    by 0x4E86878: printf (in /usr/lib64/libc-2.17.so)
==17432==    by 0x400561: main (error.c:8)
==17432== 
==17432== Use of uninitialised value of size 8
==17432==    at 0x4E7BE8B: _itoa_word (in /usr/lib64/libc-2.17.so)
==17432==    by 0x4E7CF05: vfprintf (in /usr/lib64/libc-2.17.so)
==17432==    by 0x4E86878: printf (in /usr/lib64/libc-2.17.so)
==17432==    by 0x400561: main (error.c:8)
==17432== 
==17432== Conditional jump or move depends on uninitialised value(s)
==17432==    at 0x4E7BE95: _itoa_word (in /usr/lib64/libc-2.17.so)
==17432==    by 0x4E7CF05: vfprintf (in /usr/lib64/libc-2.17.so)
==17432==    by 0x4E86878: printf (in /usr/lib64/libc-2.17.so)
==17432==    by 0x400561: main (error.c:8)
==17432== 
==17432== Conditional jump or move depends on uninitialised value(s)
==17432==    at 0x4E7CF54: vfprintf (in /usr/lib64/libc-2.17.so)
==17432==    by 0x4E86878: printf (in /usr/lib64/libc-2.17.so)
==17432==    by 0x400561: main (error.c:8)
==17432== 
==17432== Conditional jump or move depends on uninitialised value(s)
==17432==    at 0x4E7CABD: vfprintf (in /usr/lib64/libc-2.17.so)
==17432==    by 0x4E86878: printf (in /usr/lib64/libc-2.17.so)
==17432==    by 0x400561: main (error.c:8)
==17432== 
==17432== Conditional jump or move depends on uninitialised value(s)
==17432==    at 0x4E7CB40: vfprintf (in /usr/lib64/libc-2.17.so)
==17432==    by 0x4E86878: printf (in /usr/lib64/libc-2.17.so)
==17432==    by 0x400561: main (error.c:8)
==17432== 
I am 0 inches tall.
==17432== 
==17432== HEAP SUMMARY:
==17432==     in use at exit: 0 bytes in 0 blocks
==17432==   total heap usage: 0 allocs, 0 frees, 0 bytes allocated
==17432== 
==17432== All heap blocks were freed -- no leaks are possible
==17432== 
==17432== Use --track-origins=yes to see where uninitialised values come from
==17432== For lists of detected and suppressed errors, rerun with: -s
==17432== ERROR SUMMARY: 6 errors from 6 contexts (suppressed: 0 from 0)
```

Valgrind运行并分析程序，返回的数据分成三部分，第一部分是Vargrind版本信息,第二部分报告程序的相关错误信息，第三部分会生成一个简短报告，告诉你你的程序有多烂。

## 附加题

* 根据Valgrind错误信息，修复程序

```
==17515== Memcheck, a memory error detector
==17515== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==17515== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==17515== Command: ./error
==17515== 
I am 12345 years old.
I am 72 inches tall.
==17515== 
==17515== HEAP SUMMARY:
==17515==     in use at exit: 0 bytes in 0 blocks
==17515==   total heap usage: 0 allocs, 0 frees, 0 bytes allocated
==17515== 
==17515== All heap blocks were freed -- no leaks are possible
==17515== 
==17515== For lists of detected and suppressed errors, rerun with: -s
==17515== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```
