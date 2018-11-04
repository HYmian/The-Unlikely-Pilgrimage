---
title: "dnf小技巧"
date: 2018-11-04T17:54:14+08:00
tags: [
    "fedora",
    "dnf",
]
---

最近想做一件事，安装一个全新的`fedora`环境后，快速的自动把我的习惯配置给初始化，所以开始翻以前的安装历史，准备做一个脚本，这中间发现了几个有意思的`dnf`命令

## dnf history [info]
![dnf history](/img/dnf_history.png)

`history`用来查询dnf对package的改动历史，但是，就像上图所示，history的展示长度有限，很多时候执行的命令被截断了，这时候就需要`dnf history info`了：

![dnf history info](/img/dnf_history_info.png)

可以看到一条命另中安装的所有包，info后面跟的数字即为history输出中的第一列

## dnf repo[list/-packages]
`dnf repolist`可以查看当前dnf的所有repository

`dnf repository-packages <repo-name> list`可以查看某个repository提供的所有package

![dnf repo](/img/dnf_history_info.png)

## dnf repoquery
简单的`dnf repoquery`只是用来在repository查询某个package

但是用`dnf repoquery <package-name> -l`就可以看到package包含的所有文件了