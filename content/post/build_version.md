---
title: "仿kubeadm输出版本信息"
date: 2018-01-15T00:16:35+08:00
tags: [
    "golang",
]
---

一直很喜欢`kubeadm version`子命令输出的版本信息格式
```json
kubeadm version: &version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.1", GitCommit:"3a1c9449a956b6026f075fa3134ff92f7d55f812", GitTreeState:"clean", BuildDate:"2018-01-04T11:40:06Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
```

今天从`kubernetes`项目里扣出了一部分shell代码，并在代码中做了相应的修改，实现样式如下
```
 version: {Major:1 Minor:4 GitVersion:v1.4.8 GitCommit:56014371b47852cdef8579092cc790770ccb5579 GitTreeState:clean BuildDate:2018-01-14T16:22:27+0000 GoVersion:go1.8.5 Compiler:gc Platform:linux/amd64}
```

主要利用三点：

5. `go tool link`的`-X`选项，这个选项可以为一个`symbol`设置一个值，这个值可以在最终的构建物中访问到
5. 在`shell`中获取当前项目的`git`版本信息[^shell]
5. 利用`golang/runtime`获取编译器和系统架构信息

    ```golang
    GoVersion := runtime.Version()
    Compiler  := runtime.Compiler
    Platform  := fmt.Sprintf("%s/%s", runtime.GOOS, runtime.GOARCH)
    ```

最后在`version`子命令中输出包含这些信息的结构体即可

**这里还有一个小问题暂未解决，在做CI/CD时，如果用来构建的容器没有`git`命令会缺失信息，暂未想到如何解决**

[^shell]:[build.sh](https://gist.githubusercontent.com/HYmian/fe9fb6e64e2a7d2d452ad4a7fbeb9d1c/raw/3dfba631e79b52bb4ed548828edb8430faca5545/build.sh)