---
title: "开发DDE的屏保"
date: 2019-09-03T20:41:00+08:00
draft: false
tags: ["deepin", "DDE"]
---

## 前言

还记得刚推出屏幕保护功能那会儿，我偶逛论坛，围观大家对这个功能的评价。其中让我印象最深的一句话就是：“一股Windows98风”，总之，评价总结出来就是一个字：“吃藕”。

大家追求美好事物的诚挚之心深深地打动了我，而且，我个人做事情的风格是喜欢未雨绸缪，在屏幕保护程序开发之初，就已经定好了易于扩展的架构，所以我当时就下定了决心，为大家开发一个非Windows98风格的屏保。

为了达到绝对“非Windows98”的目的，我特意选择了Windows10中的默认屏保作为参考，在无数个周末的战斗下，最终成功将名为“泡泡”的屏保应用发布到了商店（项目地址：[https://github.com/zccrs/screensaver-pp](https://github.com/zccrs/screensaver-pp) ）。

本着“授人以鱼不如授人以渔”的理念，特地整理了这篇文章协助大家开发一款属于自己的时尚屏保应用。

## 正文

在Linux+X11生态环境中，xscreensaver是最“流行”的屏幕保护程序，有着非常多的屏保资源，所以deepin-screensaver必然要兼容它的资源。

但是，xscreensaver对屏保资源的扩展方式并不符合deepin的开发理念，因此，deepin-screensaver实现了一套全新的屏保扩展方式。

支持使用Qt qml模块编写屏保应用，一个标准的屏保应用只需要包含一个 "xx.rcc" 文件，将文件安装到 /usr/lib/deepin-screensaver/resources目录。

rcc 格式是一个编译之后的Qt资源文件，在这个资源文件中至少要包含两个文件：qml代码文件、屏保封面图。

![image](https://user-images.githubusercontent.com/13449038/218910865-688f689f-6d0c-49dd-b916-5b6a75bd354c.png)

图中文件名括号内为其别名，也就是屏保主应用加载文件时能读取到的文件名。

* qml代码文件：屏保应用的代码入口，会被屏保主程序加载显示
* 屏保封面图：设置屏保入口显示的预览图，支持svg png jpeg bmp等格式
所有的文件必须以特定的目录结构组织到一个Qt资源文件（qrc文件），以“泡泡”屏保为例：qml.qrc 为其资源文件，包含三个前缀路径

* /deepin-screensaver/modules：放置屏保应用的主qml文件，此路径下的所有qml文件都会被当做一个独立的屏保应用，因此，项目中的其它文件需要额外建立新的前缀放置
* /deepin-screensaver/modules/cover：放置屏保应用封面图文件，文件名称必须和modules目录中的qml文件一致，且包含它的 ".qml" 后缀。如图上，qml文件全名为："pp.qml"，封面图全名为："pp.qml.svg"。
* /deepin-screensaver/modules/pp：此前缀不是必须的，用于放置项目中的其它文件。为了不与其它项目产生冲突，建议使用项目名作为目录名称
资源文件最好以项目名称命名，避免和其它屏保应用冲突。另外，大家可能已经发现了，这三个前缀都有一个共同点，那就是以 "/deepin-screensaver/modules"开头，的确，这是一个格式要求，不能随意更改路径。

主qml文件作为屏保应用的入口，它的根元素一定要设置

```plain
anchors.fill: parent
```
这样才能确保屏保应用充满整个屏幕。在多屏的情况下下，会创建多个窗口示例，可根据屏幕绘制不同的屏保内容。
项目编译其实很简单，只需要使用Qt提供的rcc命令将qrc文件编译为rcc文件即可，使用qmake构建系统时，可以在pro文件中调用以下命令：

```plain
system(rcc --binary $$_PRO_FILE_PWD_/xx.qrc -o $$_PRO_FILE_PWD_/xx.rcc)
```
当然，最后不要忘记将 xx.rcc 文件安装到deepin-screensaver所要求的目录。做完这所有的步骤后，回到桌面，在右键菜单中选择“壁纸与屏保”，切换到屏保设置后即可看到新添加的屏保应用。
另外，deepin-screensaver为qml提供了获取当前屏幕截图的接口，只需要为Image项指定特定的路径即可：

```plain
Image {
    anchors.fill: parent
    source: "image://deepin-screensaver/screen/" + Screen.name
}
```
由于要获取屏幕名称，上述代码需要 "import QtQuick.Window 2.2" 使用
## 后记

屏保封面图最佳比例为：8:5，推荐使用svg格式，以更好的适应高分屏缩放。

推荐大家使用Qt Creator作为项目的开发工具，可以方便的编辑 qrc 文件。

泡泡屏保是一个完整的demo，有任何疑问的地方都可以以其作为参考

## 参考

* “泡泡”屏保项目：[https://github.com/zccrs/screensaver-pp](https://github.com/zccrs/screensaver-pp)
* Qt资源文件：[https://doc.qt.io/qt-5/resources.html](https://doc.qt.io/qt-5/resources.html)
 

