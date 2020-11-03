---
title: c 函数指针
date: 2019-07-26 00:10:42
tags: c
---

### 基本概念

首先，先不要把指向函数的指针认为太难了，它和普通的指针区别不是很大，只是定义形式上有所区别。

比如，对于一个指向整形的普通指针，定义形式如下：

```
int *p;
```

在定义中，指针变量的名称是p，符号`*`说明了p是一个指针，int说明这个指针指向的是整形变量。

那么，如果我们定义一个指向函数的指针，假设变量名称为p，比如它指向这样的一个函数，这个函数需要两个整数参数，其返回值也是整形参数，其定义如下：

```
int (*p)(int int);
```

对于这个定义分解一下，其中，p是变量的名称，符号`*`说明了p是一个指针，由于这个指针指向的是一个函数，所以在定义中必须体现函数的输入输出参数信息，那么最前面的int指的就是函数的返回值为int类型，后面的(int,int)则定义了该函数需要两个整形的输入参数。另外，必须将`*`与`p`用括号写成`(*p)`的形式，否则，由于括号的优先级大于`*`的优先级，去掉括号的话就成为另外一种意思了。

这样对比着理解，指向函数的指针，似乎与普通指针区别也不是太大。

### 指向函数指针例子

下面通过一个例子演示指向函数的指针的使用方法。

该例子的功能是，对于一个输入的一维数组，定义三个函数findMax、findMin和getAvg，分别实现查找该数组的最大值、最小值及计算该数组的平均值，这三个函数的输入输出参数完全相同。定义一个fun函数，在该函数的参数中，需要一个指针变量作参数，这个指针能够指向上面的三个函数。在主程序中，调用fun函数，根据传入不同的p值实现对输入的一维数组作不同的处理功能。

下面先看下几部分的实现代码吧。

#### findMax、findMin和getAvg代码实现

这三个函数对一维数组x，分别作求最大值、最小值及平均值的处理，并将其结果返回。C语言代码如下：

```
double findMax(double *x, int n) {
    double max = x[0];
    for (int i = 1; i < n; ++i) {
        if (max < x[i]) max = x[i];
    }
    return max;
}

double findMin(double *x, int n) {
    double min = x[0];
    for (int i = 1; i < n; ++i) {
        if (min > x[i]) min = x[i];
    }
    return min;
}

double getAvg(double *x, int n) {
    double sum = 0;
    for (int i = 0; i < n; ++i) {
        sum += x[i];
    }
    return sum / n;
}
```

这三个函数比较简单，函数原型完全一样，输入参数为一个指向double的指针x及x的元素个数n，输出参数也就是返回值是一个double型的数值。

#### fun函数的代码实现

该函数输入参数为3个，前两个为指向double的指针x及x的元素个数n，第三个为一个指向函数的指针类型，这个指针能够指向上面的三个函数。C语言代码如下：

```
double fun(double *x, int n, double (*p)(double *, int)) {
    return p(x, n);
}
```

那么，在主程序中可以调用该函数，只要输入不同的p值，就可以对输入的一维数组作不同的处理运算。

#### 主程序测试代码

主程序测试代码如下：

```
void mian(void)
{
    double x[5] = {1.1, 3.4, 4.5, 1.3, 5.6};
    fun(x, 5, findMax);
    fun(x, 5, findMin);
    fun(x, 5, getAvg);
}
```

#### 笨方法学C例子

```
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>

void die(const char *message) {
    if (errno) {
        perror(message);
    } else {
        printf("Error : %s\n", message);
    }
    exit(1);
}

typedef int (*compare_cb)(int a, int b);

int *bubble_sort(int *numbers, int count, compare_cb cmp) {
    int temp = 0;
    int i = 0;
    int j = 0;
    int *target = malloc(count * sizeof(int));

    if (!target) die("Memory error.");

    memcpy(target, numbers, count * sizeof(int));

    for (i = 0; i < count; ++i) {
        for (j = 0; j < count - 1; ++j) {
            if (cmp(target[j], target[j + 1]) > 0) {
                temp = target[j + 1];
                target[j + 1] = target[j];
                target[j] = temp;
            }
        }
    }
    return target;
}

int sorted_order(int a, int b) {
    return a - b;
}

int reverse_order(int a, int b) {
    return b - a;
}

int strange_order(int a, int b) {
    if (a == 0 || b == 0) {
        return 0;
    } else {
        return a % b;
    }
}

void test_sorting(int *numbers, int count, compare_cb cmp) {
    int i = 0;
    int *sorted = bubble_sort(numbers, count, cmp);
    if (!sorted) die("Failed to sort as request.");
    for (i = 0; i < count; i++) {
        printf("%d ", sorted[i]);
    }
    printf("\n");
    free(sorted);
}

int main(int argc, char *argv[]) {
    if (argc < 2) die("Usage: ext 4 3 1 5 6");
    int count = argc - 1;
    int i = 0;
    char **inputs = argv + 1;

    int *numbers = malloc(count * sizeof(int));
    if (!numbers) die("Memory error.");

    for (i = 0; i < count; ++i) {
        numbers[i] = atoi(inputs[i]);
    }

    test_sorting(numbers, count, sorted_order);
    test_sorting(numbers, count, reverse_order);
    test_sorting(numbers, count, strange_order);

    free(numbers);
}
```
