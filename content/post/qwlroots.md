---
title: "关于qwlroots"
date: 2022-11-25T20:52:00+08:00
draft: false
tags: ["qwlroots"]
---

```
这是一个时光回溯的记录，需要把时间退回到 2022年11月5日再接着看
```

创建[qwlroots](https://github.com/vioken/qwlroots/)是觉得Wayland的wl_signal和Qt的信号机制挺般配的，再加上单纯的C接口用起来有点不顺手，为了今后能顺利重构waylib，所以先着手搞这个项目，和waylib那种封装不同的是，qwlroots对wlroots的封装不需要考虑逻辑和结构，只需要把接口层做封装，相对来说可以快速搞。
