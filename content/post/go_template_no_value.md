---
title: "go template <no value>处理"
date: 2018-04-07T23:39:33+08:00
tags: [
    "golang",
    "golang template",
]
---

在使用go的template库的时候，碰到一个需求：如果模板中使用了元数据中没有的字段，希望在渲染后看到的是空或者一个我们期望的默认值，但是得到的最直接的结果是`<no value>`

在解决的过程中发现了一个函数[option](https://golang.org/pkg/text/template/#Template.Option)，这个函数可以设置template在渲染过程中的行为，这样我们就可以在模板中加入一些逻辑判断来实现我们的期望目标

```golang template
endpoint: {{ if eq (print .Property.endpoint) "" -}}
          {{ (index .Hosts.master 0).IP }}:6443
          {{- else -}}
          {{ .Property.endpoint }}
          {{- end }}
```
上面的模板可以达到--如果有设置endpoint，则使用设置的endpoint，如果没有，则使用第一台主机的6443端口作为endpoint--期望的效果