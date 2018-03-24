---
title: "kubernetes feature gates"
date: 2018-03-24T22:31:13+08:00
tags: [
    "kubenetes",
]
---

kubernetes 1.8开始支持[local ephemeral storage](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#local-ephemeral-storage-alpha-feature)

在1.8和1.9系列中还只是alpha特性，为了开启特性，需要在master的三个组件和`kubelet`的启动参数中设置`feature-gates`参数

我启动时传入的参数是`AllAlpha=true`，在打开这个开关后`kubelet`就无法加入到集群中，查看`kubelet`的日志，发现如下报错`will not sync ConfigOK condition, error: nodes <node> not found`，并且，一个小时后自动恢复

`google`发现是有`DynamicKubeletConfig`这个特性会牵涉到ConfigOK condition这个特性，原来是在`feature-gates`的设置中同时打开了`DynamicKubeletConfig`特性，这个特性用来在[运行时动态的修改`kubelet`的配置](https://kubernetes.io/docs/tasks/administer-cluster/reconfigure-kubelet/)，这个特性会让`kubelet`定时的从configmap中获取最新的配置，如果configmap中没有就会报上面的错，直到超时