---
title: "for...range和闭包"
date: 2017-12-30T01:32:53+08:00
tags: [
    "golang",
]
---

今天遇到了一个有趣的问题，涉及到了golang编程的三个知识点，`for...range`、`闭包`和`goroutine`

为了简化问题，写了一段示例代码来做分析：
```golang
package main

import "time"

func main() {
    ss := []string{
        "haha",
        "hehe",
        "xixi",
    }

    for _, s := range ss {
        go func() {
            print(s, "\n")
        }()
    }

    time.Sleep(1 * time.Second)
}
```
如果我们无意中写出这种代码，心里预期的输出可能是三个不同的字符串，当然，顺序可能是不固定的，这是`goroutine`的特性决定的，我们这里就不做说明了，但实际上着代码的输出会是
```
xixi
xixi
xixi
```

如果我们将代码修改为
```golang
package main

import "time"

func main() {
    ss := []string{
        "haha",
        "hehe",
        "xixi",
    }

    for _, s := range ss {
        go func(s1 string) {
            print(s1, "\n")
        }(s)
        // time.Sleep(1 * time.Microsecond)
    }

    time.Sleep(1 * time.Second)
}
```
则可以得到我们期望的输出
```
haha
hehe
xixi
```

原因在于`for...range`的赋值方式和`闭包`概念：
### fro...range
```
The iteration variables may be declared by the "range" clause using a form of short 
variable declaration (:=). In this case their types are set to the types of the 
respective iteration values and their scope is the block of the "for" statement; they 
are re-used in each iteration. If the iteration variables are declared outside the "for" 
statement, after execution their values will be those of the last iteration.
```
这是`goang`官方对`for...range`中迭代变量--也就是我们代码中的`s`--的说明
也就是说
```golang
for _, s := range ss {
    //TODO:
}
```
和
```golang
var s string
for _, s = range ss {
    //TODO:
}
```
这两种写法是同样的效果，但是第二种写法相对来说就很好理解--`for...range`的迭代中不会重复分配迭代变量，而是会重复利用第一次生成的或者传入的迭代变量，`for...range`还有[另外一个坑](https://github.com/golang/go/wiki/Range#gotchas)，在此不做赘述
### 闭包
简单点讲，引用了全局变量的函数就是典型的`闭包`[^1]，这个函数在不同场景调用会产生不同的效果（即使收到了同样的输入参数），也可以称之为*闭包在运行时可以有多个实例*

所以我们第二段代码中，通过给匿名函数增加了一个输入变量，这样虽然我们仍在使用`for...range`，但是没有了`闭包`，出现一个传值调用的过程，最终得到我们期望的输出

[^1]:[参见wiki](https://zh.wikipedia.org/wiki/%E9%97%AD%E5%8C%85_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6))