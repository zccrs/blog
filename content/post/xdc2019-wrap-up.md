---
title: "翻译：记 2019 年的 X.org 开发者研讨会 - emersion"
date: 2021-03-11T11:03:00+08:00
draft: false
authors: ["zccrs"]
tags: ["xdc", "翻译"]
---


原文：https://emersion.fr/blog/2019/xdc2019-wrap-up

# XDC 2019 wrap-up

 &gt; 2019-10-10

这篇文章是我从加拿大蒙特利尔飞往赫尔辛基的路上写的，我刚参加完 X.Org 的开发者研讨会。从去年西班牙那次会议以来，这是我第二次参加 XDC（译者注：X.Org Developer’s Conference）。

今年的研讨会非常的给力。我遇到了很多一起工作过的同志，他们来自四面八方的各个组织和项目。会议内容也很牛，我会尝试总结一下这次讨论的一些事情。

# libliftoff

前一阵子，我搞了一个新项目，名字叫 [libliftoff](https://github.com/emersion/libliftoff)。它是我在这次会议上的亮点，所以接下来我会详细的介绍它。带着问题往下看：“libliftoff 是用来解决什么问题的呢？”

# libliftoff 是什么

它是一个合成器的中间层，负责从 clients 中获取 buffers，把它们绘制在一个独立的 buffer 上，并把这个 buffer 显示到屏幕上。假设我现在打开了一个“文本编辑器”和一个“终端”，合成器会把它们的窗口 buffer 复制到屏幕的 buffer 上，一般是用 OpenGL 做这个事。

不过，复制 buffers 的操作很浪费资源，因为它会做很多事情，比如：它总是进行 alpha 混合、在不同格式之间转换、让渲染引擎一直干活（注：GPU 在执行 OpenGL/Vulkan 的命令）等等，这些操作不仅占用时间，而且也会增加耗电量。

为了改善性能，许多 GPU 都提供了硬件实现的“叠加平面（planes）”支持（下文直接称为“叠加平面”）。“叠加平面”可以作为显示引擎负责合成工作，这种行为称为`直接显示`，可以避免全让合成器做复制 buffers 的活。

在 Android 上，硬件“叠加平面”在“Hardware Composer”组件中用的很广。许多 Wayland 合成器也会为光标使用“叠加平面”，但是用在其它场景的情况还是很少见。Weston（译者注：一个 Wayland 合成器） 是仅有的使用“叠加平面”的合成器之一。

使用“叠加平面”并不是一件容易的事情，它有一些限制，比如：不支持 Alpha 通道、不是任意的 buffer 都可以用。虽然这些限制往往会让人觉得很扯淡，但它是由硬件特性决定的。举个例子：对于某些格式的 buffers，Intel 的硬件只能将其放在“叠加平面”的偶数坐标位置、某些 buffers 是在内存上分配的，“叠加平面”干脆就不支持这一类的 buffer、另外，显示设备一般都有带宽限制，在“叠加平面”上使用过大的 buffer 时会失败、某些 ARM 设备上，多个“叠加平面”之间不支持重叠。

目前为止，关于硬件的这些限制信息，合成器程序还查询不了，因为硬件之间的差异性很大，所以很难设计一个能查询硬件限制信息的 API，甚至在某些新出 GPU 上还会有更奇怪的限制。唯一的方法是搞一些前置工作，用它做一些尝试性的使用。

因此，设计和实现一个能有效的使用“叠加平面”的代码是一个相当复杂的事情。我一直在想，共享这些代码，是不是会对合成器使用“叠加平面”有所帮助，于是我就奔着这个目标设计了 libliftoff。

# libliftoff 的研讨会

![图一](https://sr.ht/Fd_h.jpg)

我在 XDC 搞了一个[研讨会](https://xdc2019.x.org/event/5/contributions/583/)，讨论关于 libliftoff 的事情，目的是把合成器和驱动专家凑一块，一起商量如何才让合成器有效的用上“叠加平面”。

libliftoff 的受欢迎程度让我很惊讶，连桌子附近都坐满了人：

- 在合成器这边，有 wlroots 团队的人（Drew DeVault, Scott Anderson 和我自己），KDE 的 Roman Gilg，Weston 的 Daniel Stone，还有 X server 的 Keith Packard。
- 在驱动这边，有很多来自于不同厂商的开发者：有 AMD、Arm、Google、Nvidia、Qualcomm 等等。

大家貌似都很兴奋，为 libliftoff 提了很多宝贵的意见。我在此感谢所有参与研讨会的人！

短期的计划是，让 libliftoff 为合成器提供能实际使用的功能（实验性的），不过有个事我还需要弄清楚，那就是应该怎么样支持多个 output：因为每个 output 都有自己的时序，所以，如何将“叠加平面”从一个 output 迁移到另一个 output 是一件很难搞的问题。最终，如果能实现这样的功能就完美了：为每一个图层分配一个优先级，并把那些频繁在别的图层之前更新的图层放在同一个“叠加平面”上。

另外也讨论了一些长期计划。

首先，我们想搞定一个跟内存相关的问题：clients 往往会在内存上分配 buffer，这些 buffer 没法直接用在“叠加平面”上。通常，合成器都会给一些提示信息：“你现在正在使用 `Y_TILED`，所以我不能将你用在“叠加平面”上，但是如果你能使用 `X_TILED` 我就能把你用在“叠加平面”上”，这样 client 就可以决定切换它的 buffer 格式。我大概在一年前给 Wayland 提了一个[补丁](https://patchwork.freedesktop.org/series/52370/)，这个补丁新增的信息能被 libliftoff 识别出来，在这种情况下，合成器可以把这个信息转发给 clients。

接下来是关于内核 API 的问题，现在只有一个办法能让我们知道是否可以使用“叠加平面”，那就是用 atomic 相关的 test 接口测试（译者注：atomic 是 DRM 提供的一系列原子操作接口，支持事务），这个做法相当的不清真，因为我们必须得遍历尝试多个组合条件，基本上是暴力求解。除此之外，比较好的做法都要跟具体的硬件相关。

为了搞定这个问题，我们必须让内核提供更多的信息，但是因为不同硬件的差异性很大，设计一个足够通用的 API 将是一个很蛋疼的事情，另一个解决方案是在 libliftoff 中添加特定厂商的插件，允许每个驱动添加自己的代码，以此更好的匹配自己的硬件。目前来说，貌似这是最好的方案了。

这是本次讨论的总结([PPT](https://fs.emersion.fr/protected/presentations/present.html?src=libliftoff-xdc2019/index.md))：

[视频：XDC 2019 - Day 3](https://youtu.be/JIry8jpbPUY)

Scott Anderson 也写了一些[总结](https://lists.freedesktop.org/archives/wayland-devel/2019-October/040924.html)，是关于 Wayland 协议的一些想法的详细介绍。

# 分配器

在三年前的 XDC 上，Nvidia 的 James Jones 提出了这个[分配器项目](https://www.x.org/wiki/Events/XDC2016/Program/jones_unix_device_mem_alloc/)，用于解决 GBM/EGLStreams 现有的问题。今年，他做了一个关于 GBM、Nouveau 和 transitions 的[新的分享](https://xdc2019.x.org/event/5/contributions/335/)。跟之前的提案相比，这个提案的目的是在现有 API 的基础上建立一种叫 transition 的机制。

[视频：XDC 2019 - Day 2](https://youtu.be/HYa4UvVtMOE)

transition 可以用来减少渲染引擎的带宽占用。某些 GPU 支持使用压缩的 buffer，与未压缩的 buffer 相比，GPU 可以在这些压缩后的 buffer 上直接执行 OpenGL/Vulkan 的操作，这样做效率很高。所以，我们应该在渲染时使用压缩后的 buffers。

不过，压缩后的 buffers 不能直接用在“叠加平面”上，需要先将它们解压才能使用，有一个方法是把它解压为一个新的 buffer，但是这样需要复制数据，速度会很慢。

在某些 GPU 上有个更好的方法，它们支持在“叠加平面”中直接解压 buffer。这指的是：当一个 Wayland client 在压缩后的 buffer 上渲染，如果这个 buffer 可以在“叠加平面”中解压，并且之后可以直接使用它，那我们就将这种“buffer 就地解压的程序”称为 transition。

之后我们在一个研讨会中继续探讨 transitions，讨论了应该在何时何地进行 transition 操作。想法一是：如果合成器准备把一个 buffer 用到“叠加平面”中，那就在合成器中做这个事，可以用 libliftoff 判断什么时候应该使用 transition。另一个想法是：在 client 把 buffer 提交给合成器之前做这个事，client 需要从合成器那获取一个信息，以便知道应该什么时候执行操作（这就是我之前提到的那个 [Wayland 协议的补丁](https://patchwork.freedesktop.org/series/52370/)可以做的事情）。

这是这次研讨会的总结：

[视频：XDC 2019 - Day 3](https://youtu.be/JIry8jpbPUY)

# 可变刷新率（VRR）

AMD 的 Harry Wentland 搞了一个关于“自适应同步”（DisplayPort 的技术）、“可变刷新率”（HDMI 的技术）和“FreeSync”（AMD 的技术）的研讨会。所有这些技术都是为了适配屏幕的功能，允许它从“固定为 60Hz”的模式变成“可以稍微等待下一帧”的模式（译者注：动态刷新率的模式）。

[视频：XDC 2019 - Day 2](https://youtu.be/HYa4UvVtMOE)

这东西的主要使用场景是游戏应用。游戏通常是用动态刷新率的方式更新画面（依赖于场景的复杂度），下一帧可以更快也可以更慢。游戏还希望能降低延迟，即避免渲染到在屏幕上显示的时间差过长。

在屏幕固定刷新率的情况下，如果有一帧的渲染用时稍微长了一点，它错过了显示器的刷新时机，这样就会导致这一帧的显示有些滞后，而 VRR 则允许增加屏幕刷新时的等待时间，以免丢帧。

另一种场景是视频播放。视频有固定的帧率，但是通常会与屏幕的刷新率不同，视频播放器需要用帧插值的方式，才能适应屏幕的刷新率，而 VRR 则允许降低刷新率以匹配能完美播放视频的时序。

屏幕内容通常都是处于静止状态，在这种情况下，VRR 还可以降低电量消耗。比如用户在使用文本编辑器时，完全不需要 60FPS，这时候合成器可以降低屏幕的刷新率。

游戏的使用场景比较简单，合成器可以比 deadline 稍微晚一点点提交帧，硬件可以搞定这种情况。其它场景需要做更多的工作，要想把 VRR 用在视频播放器上，我们需要某种定时提交帧的 API。要想用 VRR 做节能，合成器要能支持帧率改变时的平滑过渡，否则屏幕会有闪烁问题（这是支持 VRR 功能的屏幕的限制）。

在我看来，我们目前应该把精力放在 Wayland 协议对游戏场景的支持上，因为搞其它两种场景需要做的事情太多了，它们不仅需要更多的新 API（内核和 Wayland 的都需要），还需要做很多的实验工作。

# Chamelium

我在 Intel 实习期间，参与了一个叫 Chamelium 的项目，它是一个基础的屏幕模拟器，你可以通过网络向它发送命令，它已经被用在了 ChromeOS 的 i915 显卡环境的 CI 服务中。

我搞了一个关于这个项目以及我在项目内的工作的研讨会：

[视频：XDC 2019 - Day 3](https://youtu.be/JIry8jpbPUY)

除了这三个主题，我们还讨论了很多其它的东西，但是不能全塞到这一篇博客中讲，所以就说到这吧。感谢所有参加 XDC 的人，感谢 Mark Filion 组织了这次活动，还要感谢所有赞助商，没有你们的赞助就不会有的这次活动！
