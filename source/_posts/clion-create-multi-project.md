---
title: clion建立多级工作目录
date: 2019-07-31 23:42:39
tags: c
---

之前对Clion不熟悉，每次写项目都在主目录下，导致一个主目录只能写一个main函数，下面学习如何在CLion建立多级工程目录

---

### 操作：

在主工作目录下，有一个CMakeList.txt文件，内容如下：

![](http://image.huany.top/hexo/c/clion-main-cmakelist.png)

其中，`ADD_SUBDIRECTORY(function-point)`，这个是代表包含子目录的意思。

对于子目录`function-point`,也必须有一个CMakeList.txt文件，指定子目录中的可执行文件。

![](http://image.huany.top/hexo/c/clion-sub-makelist.png)



