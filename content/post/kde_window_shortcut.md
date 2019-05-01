---
title: "Fedora KDE释放Alt+ Left Button的全局快捷键"
date: 2019-05-01T14:33:00+08:00
tags: [
    "Fedora",
]
categories: [
    "Fedora",
]
---

今天在`Fedora 29 KDE`上用vs code上写代码时遇到一个问题，vs code使用`Alt`+左键作为多光标选择的快捷键

但是在`Fedora`上这个快捷键默认是用来移动窗口的，在此记录下修改方法

System Settings -> Window Management -> Window Behaviour -> Window Actions ![](/img/kde_window_behaviour.png)

修改图中的`修饰建`为meta，或者将鼠标左键的动作设为无动作即可

## 参考资料
5. [reddit帖子](https://www.reddit.com/r/kde/comments/7bok16/how_can_i_change_the_behavior_of_altclick_on_kde/)
