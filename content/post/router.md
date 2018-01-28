---
title: "HttpRouter路由"
date: 2018-01-28T23:55:19+08:00
tags: [
    "golang",
    "gin",
]
---

`Gin`这个web框架使用`HttpRouter`作为router

`HttpRouter`使用`radix tree`[^radix tree]作为保存Http Path的数据结构

[insertChild](https://github.com/julienschmidt/httprouter/blob/master/tree.go#L212-L323)和[addRoute](https://github.com/julienschmidt/httprouter/blob/master/tree.go#L80-L210)是最主要的两个方法

这两个方法的主要特点是：

1. 路径分拆保存，比如：

    /abcxyz和/abcde会保存成
    ```
    |-abc
    |   |-de
    |   |-xyz
    ```
2. 通配符不能和确定字段同时存在，比如：

    /abcxyz和/:abc，如果同时有这两个路径，运行时会panic

[^radix tree]:[radix tree](https://en.wikipedia.org/wiki/Radix_tree)