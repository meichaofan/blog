---
title: go 初始化顺序
date: 2019-06-30 15:00:59
tags:
---

### 初始化顺序

先执行import包的每个文件的**常量**和**变量**，然后是**init**函数，最后执行**main**函数，当**main**函数执行结束程序退出。

![go 初始化顺序](http://image.huany.top/hexo/go/go-initialization-sequence.png)

