---
title: go-date-time-conver
date: 2019-06-30 17:00:56
tags: go
---

```
package main

import(
    "fmt"
    "time"
)

func main() {
    datetime := "2015-01-01 00:00:00"  //待转化为时间戳的字符串

    //日期转化为时间戳
    timeLayout := "2006-01-02 15:04:05"  //转化所需模板
    loc, _ := time.LoadLocation("Local")    //获取时区
    tmp, _ := time.ParseInLocation(timeLayout, datetime, loc)
    timestamp := tmp.Unix()    //转化为时间戳 类型是int64
    fmt.Println(timestamp)
    
    //时间戳转化为日期
    datetime = time.Unix(timestamp, 0).Format(timeLayout)
    fmt.Println(datetime)
}
```
输出
```
1420041600
2015-01-01 00:00:00
```
