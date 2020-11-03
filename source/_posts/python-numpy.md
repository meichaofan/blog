---
title: NumPy介绍
date: 2019-05-31 00:23:41
tags:
- python
- numpy
---

## NumPy
在学习深度学习时，NumPy的数组类提供了很多关于操作数组和矩阵的便捷方法。

---

## 2019/5/30
### 1.1 导入NumPy
NumPy并不存在于标准版`Python`中，因此，首先需要导入NumPy库

```
>>> import numpy as np
```

### 1.2 生成NumPy数组
使用`np.array()`方法。`np.array()`接受Python的**列表**作为参数，生成NumPy数组（`num.ndarray`）。

```
>>> x=np.array([1.0,2.0,3.0])
>>> print(x)
[1. 2. 3.]
>>> type(x)
<class 'numpy.ndarray'>
```

### 1.3 NumPy的算术运算

```
>>> x = np.array([1.0, 2.0, 3.0])
>>> y = np.array([2.0, 4.0, 6.0])
>>> x + y # 对应元素的加法
array([ 3., 6., 9.])
>>> x - y
array([ -1., -2., -3.])
>>> x * y # element-wise product
array([ 2., 8., 18.])
>>> x / y
array([ 0.5, 0.5, 0.5])
```

需要注意，数组x和数组y的元素个数相同，如果元素个数不同，程序会报错。

NumPy数组不仅可以进行**对应**元素的运算，也可以和单一的数值（标量）进行运算，这个功能叫**广播**。

```
>>> x = np.array([1.0, 2.0, 3.0])
>>> x / 2.0
array([ 0.5, 1. , 1.5])
```



### NumPy的N维数组

NumPy可以生成多维数组

```
>>> A = np.array([[1, 2], [3, 4]])
>>> print(A)
[[1 2]
[3 4]]
>>> A.shape #查看数组形状
(2, 2)
>>> A.dtype #查看数组元素的数据类型
dtype('int64')
```

```
>>> B = np.array([[3, 0],[0, 6]])
>>> A + B   #两个二维数组相加
array([[ 4, 2],
[ 3, 10]])
>>> A * B    #两个二维数组相乘，对应位置的元素进行算术运算
array([[ 3, 0],
[ 0, 24]])
```

```
#广播
>>>print(A)
[[1 2]
 [3 4]]
>>>A*10
array([[ 10, 20],
[ 30, 40]])
```

### 访问元素

元素的索引从0开始。对各个元素的访问如下：

```
>>> X = np.array([[51, 55], [14, 19], [0, 4]])
>>> print(X)
[[51 55]
[14 19]
[ 0 4]]
>>> X[0] # 第0行
array([51, 55])
>>> X[0][1] # (0,1)的元素
55
```

也可以用for语句访问各个元素

```
>>> for row in X:
... print(row)
...
[51 55]
[14 19]
[0 4]
```

除了前面介绍的索引操作，NumPy还可以使用数组访问各个元素

```
>>> X = X.flatten() # 将X转换为一维数组
>>> print(X)
[51 55 14 19 0 4]
>>> X[np.array([0, 2, 4])] # 获取索引为0、 2、 4的元素
array([51, 14, 0])
```

 运用这个标记法，可以获取满足一定条件的元素。例如，要从 X中抽出大于15的元素，可以写成如下形式。

```
>>> X > 15
array([ True, True, False, True, False, False], dtype=bool)
>>> X[X>15]
array([51, 55, 19])
```

对NumPy数组使用不等号运算符等（上例中是 X > 15）,结果会得到一个布尔型的数组。上例中就是使用这个布尔型数组取出了数组的各个元素（取出 True对应的元素）。 