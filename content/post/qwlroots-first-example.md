---
title: "为qwlroots添加了一个示例（第一个）"
date: 2022-11-29T19:38:00+08:00
draft: false
authors: ["zccrs"]
tags: ["qwlroots"]
---

今天达成了一个小的里程碑，终于能用 qwlroots 实现一个 Wayland Compositor 的例子了。

<!--more-->

代码[在这里](https://github.com/vioken/qwlroots/tree/main/examples/tinywl)，tinywl 跟 wlroots 的 tinywl 的效果一样，本身就是参考它的代码写的，不过相比它解决了一个问题：`用鼠标拖动窗口时，如果窗口已经是最小了，没法再缩小，再拖动鼠标会导致窗口被移动`。这个问题我很早之前就在 wlroots 的 tinywl 中发现了，但是一直懒得处理，bug 本身很简单（原因是 tinywl 的作者偷懒，在用鼠标 resizing 窗口时没有考虑窗口的`最小大小`和`最大大小`属性，导致计算出的新大小无法生效，因此体现出的效果就是窗口大小没变，但是位置变化了），今天趁这个机会先帮 wlroots 做了修复：https://gitlab.freedesktop.org/wlroots/wlroots/-/merge_requests/3896 ，这个代码很简单，应该很快能合入。

qwlroots 的 tinywl 的演示录屏：

[录屏 2022-11-29 18-50-08.webm](https://user-images.githubusercontent.com/13449038/204519293-5d73c2ca-6416-4c04-8869-9d90aeb91aba.webm)
