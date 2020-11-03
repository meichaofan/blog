---
title: go sync.Once
date: 2019-07-26 00:30:27
tags: go
---

`sync.Once` 可以实现单例模式，确保`sync.Once.Do(f func())`只会被执行一次，可以初始化某个实例单例。

sync.Once表示只执行一次的函数。要做到这一点，就需要以下两点要求：

- 计数器，统计函数执行的次数
- 线程安全，保障在多G情况下，函数仍然只执行一次，比如锁。即对计时器的修改是线性安全的。

### 源码

下面是sync.Once源码，源码不长，但值得分析

```
package sync

import (
    "sync/atomic"
)

// Once is an object that will perform exactly one action.
type Once struct {
    m    Mutex
    done uint32
}

func (o *Once) Do(f func()) {
    if atomic.LoadUint32(&o.done) == 1 {
        return
    }
    // Slow-path.
    o.m.Lock()
    defer o.m.Unlock()
    if o.done == 0 {
        defer atomic.StoreUint32(&o.done, 1)
        f()
    }
}
```

**Once结构体证明了之前的猜想，果然有两个变量。**

Do方法相对简单，但是也是也可以学习的地方。

- 1.原子操作判断o.done是否为1，若为1，表示f已经执行过，直接返回
- 2.加锁，保证互斥访问
- 3.若o.done为0，表示f未执行，执行f函数，对o.done原子赋值。
