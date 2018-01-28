---
title: "gin-errorlog中间件"
date: 2018-01-18T23:30:47+08:00
tags: [
    "golang",
    "gin",
]
---

一直在用[gin](https://github.com/gin-gonic/gin)这个框架来做web开发，但是在开发过程中有一点繁琐的地方：
```golang
if err != nil {
    c.JSON(http.StatusInternalServerError, gin.H{
        "error": err.Error(),
    })
    glog.Error(err)
} else {
    c.JSON(http.StatusOK, hs)
}
```

本文就是讲如何利用``gin`的`middleware`特性来简化代码：

1. 在`gin`的`engine`中插入这个`middleware`

    ```golang
    r := gin.Default()
    r.Use(errorlog.ErrorResponse())
    ```

2. 在出现`error`的地方使用`c.AbortWithError(http.StatusInternalServerError, error)`返回即可

`ErrorResponse`代码见[这里](https://raw.githubusercontent.com/HYmian/gin-errorlog/master/error_log.go)

这里有一点需要注意
```golang
c.Writer.Header()["Content-Type"] = []string{"application/json; charset=utf-8"}
```
这一行代码需要在Next调用之前，因为，Next调用之后，返回的Http code以及被写入，无法再设置header，这是http协议的一个特性，参见[这里](http://engineering.pivotal.io/post/http-trailers/)