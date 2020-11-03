---
title: 剑指Offer - 3.1 - 数组中重复的数字
date: 2019-07-26 23:10:17
tags:
---

在一个长度为n的数组里的所有数字都是0~n-1的范围内。数组中某些数字都是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是重复的数字2或者3.

---

#### 解题思路

我们注意到数组中的数字都在0~n-1的范围内。如果这个数组中没有重复的数字，那么当数字排序之后数字i将出现在下标为i的位置。由于数组中有重复的数字，有些位置可能存在多个数字，同时有些位置可能没有数字。

#### 代码实现

```go
package main

import (
    "fmt"
    "sort"
)

/**
n个数， 0~n-1之间，找出有重复的数字
*/
//1.排序
func getDuplicateNum1(numbers []int) []int {
    var ret []int
    i := 0
    length := len(numbers)
    sort.Ints(numbers)
    for i < length-1 {
        //fmt.Printf("i: %d\n", i)
        if numbers[i] == numbers[i+1] {
            ret = append(ret, numbers[i])
        }
        for i < length-1 && numbers[i] == numbers[i+1] {
            i++
        }
        i++
    }
    return ret
}

//2.借助map(hashtable)
func getDuplicateNum2(numbers []int) []int {
    var ret []int
    tmp := make(map[int]int)
    for i := 0; i < len(numbers); i++ {
        if count, ok := tmp[numbers[i]]; ok {
            tmp[numbers[i]] = count + 1
        } else {
            tmp[numbers[i]] = 1
        }
    }
    for k, v := range tmp {
        if v > 1 {
            ret = append(ret, k)
        }
    }
    return ret
}


//巧妙3：数组内自排序，时间复杂度0(n) , 空间复杂度是 O(1)
func getDuplicateNum3(numbers []int) int {
    length := len(numbers)
    i := 0
    for i < length {
        for i != numbers[i] {
            if numbers[i] == numbers[numbers[i]] && i != numbers[i] {
                return numbers[i]
            }
            numbers[i], numbers[numbers[i]] = numbers[numbers[i]], numbers[i]
        }
        i++
    }
    return length
}

func main() {
    numbers := []int{0, 1, 2, 2, 3, 3, 5}
    ret1 := getDuplicateNum1(numbers)
    fmt.Printf("%v\n", ret1)

    numbers = []int{0, 1, 2, 2, 3, 3, 5}
    ret2 := getDuplicateNum2(numbers)
    fmt.Printf("%v\n", ret2)

    numbers = []int{0, 1, 2, 2, 3, 3, 5}
    ret3 := getDuplicateNum3(numbers)
    fmt.Printf("%v\n", ret3)
}
```
