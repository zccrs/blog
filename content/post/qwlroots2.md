---
title: "简化连接wl_signal的方式"
date: 2022-11-25T21:07:00+08:00
draft: false
authors: ["zccrs"]
tags: ["qwlroots"]
---

```
这是一个时光回溯的记录，需要把时间退回到 2022年11月14日再接着看
```

今天为 QWBackend 增加了对 wlr_backend 信号的绑定：https://github.com/vioken/qwlroots/pull/4

<!--more-->

但是连接一个 wl_signal 实在是太麻烦了，至少要三行代码。幸好之前在写 Waylib 时做了一个封装，现在可以跟连接Qt的信号一样连接一个 wl_signal🌈。

![图片](https://user-images.githubusercontent.com/13449038/203991890-143f5d22-9a8e-49eb-a969-a2e11fc17102.png)
![图片](https://user-images.githubusercontent.com/13449038/203991972-26098939-502b-489b-aa76-423d6e399a88.png)
![图片](https://user-images.githubusercontent.com/13449038/203992391-a4b7f7d9-a3ee-4233-9209-03a9cb971614.png)
