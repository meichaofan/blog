---
title: Go slice实现原理
date: 2019-06-30 17:27:15
tags: go
---
### 1 前言

Slice又称为动态数组，低层依托于数组实现。可以方便的进行扩容，传递等。实际使用过程中，比数组更灵活。

### 2 Slice实现原理

Slice依托于数组实现，低层数组对用户屏蔽，在低层数组容量不足时可以实现自动重分配并生成新的Slice。

### 2.1 Slice的数据结构

源码包中`src/runtime/slice.go`定义了Slice的数据结构

```
type slice struct{
    array unsafe.Pointer
    len int
    cap int
}
```

从数据结构看，slice其实就是一个结构体，array指针指向低层数组，len表示切片长度，cap表示低层数组容量。

### 2.2 使用make创建Slice

使用**make**关键字创建Slice时，可以同时制定长度和容量，创建时低层会分配一个数组，数组的长度即容量。

例如，语句 `slice:=make([]int,5,10)`所创建的slice，结构如下图所示:

![](http://image.huany.top/hexo/go/make_slice_01.png)

该Slice的长度为5，可以使用slice[0]~slice[4]来操作里面的元素，capacity为10，表示后续向slice添加新的元素时可以不必重新分配内存，直接使用预留内存即可。

### 2.3 使用数组创建Slice

使用数组来创建Slice时，Slice将于原数组共用一部分内存。例如，语句`slice:=array[5:7]`所创建的Slice，结构如下图所示:

![](http://image.huany.top/hexo/go/make_slice_02.jpg)

切片从数组array[5]开始，到数组array[7]结束（不含array[7]）,即切片长度为2，数组后面的内容都作为切片的预留内存，即capacity为5。

数组和切片操作可能作用于同一块内存，这也是使用过程中需要注意的地方。

### 2.4 Slice扩容

使用append向Slice追加元素时，如果Slice空间不足，将会触发Slice扩容，扩容实际上重新一配一块更大的内存，将原Slice数据拷贝进新Slice，然后返回新Slice，扩容后再将数据追加进去。

例如，当向一个capacity为5，且length也为5的Slice再次追加1个元素时，就会发生扩容，如图所示：

![](http://image.huany.top/hexo/go/make_slice_03.jpg)

扩容操作只关心容量，会把原Slice数据拷贝到新Slice，追加数据由append在扩容结束后完成。上图可见，扩容后新的Slice长度仍然是5，但容量由5提升到了10，原Slice的数据也都拷贝到了新Slice指向的数组中。

扩容容量的选择遵循以下规则：
- 如果原Slice容量小于1024，则新Slice容量将扩大为原来的2倍；
- 如果原Slice容量大于等于1024，则新Slice容量将扩大为原来的1.25倍；

使用append()向Slice添加一个元素的实现步骤如下：
1. 假如Slice容量够用，则将新元素追加进去，Slice.len++，返回原Slice
2. 原Slice容量不够，则将Slice先扩容，扩容后得到新Slice
3. 将新元素追加进新Slice，Slice.len++，返回新的Slice。

### 2.5 Slice Copy

使用copy()内置函数拷贝两个切片时，会将源切片的数据逐个拷贝到目的切片指向的数组中，拷贝数量取两个切片长度的最小值。例如长度为10的切片拷贝到长度为5的切片时，将会拷贝5个元素。也就是说，copy过程中不会发生扩容。

### 2.6 特殊切片

跟据数组或切片生成新的切片一般使用`slice := array[start:end]`方式，这种新生成的切片并没有指定切片的容量，实际上新切片的容量是从start开始直至array的结束。

比如下面两个切片，长度和容量都是一致的，使用共同的内存地址：

```
sliceA := make([]int, 5, 10)
sliceB := sliceA[0:5]
```

根据数组或切片生成切片还有另一种写法，即切片同时也指定容量，即`slice[start:end:cap]`, 其中cap即为新切片的容量，当然容量不能超过原切片实际值，如下所示：

```
sliceA := make([]int, 5, 10)  //length = 5; capacity = 10
sliceB := sliceA[0:5]         //length = 5; capacity = 10
sliceC := sliceA[0:5:5]       //length = 5; capacity = 5
```

### 3 总结

- 每个切片都指向一个底层数组
- 每个切片都保存了当前切片的长度、底层数组可用容量
- 使用len()计算切片长度时间复杂度为O(1)，不需要遍历切片
- 使用cap()计算切片容量时间复杂度为O(1)，不需要遍历切片
- 通过函数传递切片时，不会拷贝整个切片，因为切片本身只是个结构体而矣
- 使用append()向切片追加元素时有可能触发扩容，扩容后将会生成新的切片
