---
layout: post
title: "AsyncDisplayKit教程：实现60 fps滚动"
date: 2014-11-24 19:28:13 +0800
comments: true
categories: 教程
---


本文译自：

###摘要：

FaceBook的Paper团队发布了其iOS UI框架[AsyncDisplayKit](https://github.com/facebook/AsyncDisplayKit)。该库旨在快速响应用户界面，将图片解码、布局以及渲染操作都放在了后台线程执行，故不会阻塞主线程。本教程将带你简单学习AsyncDisplayKit。

在这个教程中，你将学习到如何利用AsyncDisplayKit来优化UICollectionView的滚动问题。以此为例，你将学会如何在现有的项目中使用AsyncDisplayKit。

**提示**：*在开始本教程前，你必须对Swift、Core Animation 以及 Core Graphics有所掌握。*
*你可以在这里找到[Swift](http://www.raywenderlich.com/74438/swift-tutorial-a-quick-start)、[Core Animation](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html)以及[Core Graphics](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Introduction/Introduction.html)教程来更好的开始本文的学习。另外强烈建议大家看看WWDC 2012的视频“[iOS App Performance: Graphics and Animations](https://developer.apple.com/videos/wwdc/2012/?id=238)”，这将对你有很大的帮助。*

<!--more-->

###Getting Started
在开始学习前，看一下[AsyncDisplayKit’s intro](https://code.facebook.com/posts/721586784561674/introducing-asyncdisplaykit-for-smooth-and-responsive-apps-on-ios/),以了解AsyncDisplayKit。

当你准备好了之后，你可以在这里下载初始工程[`下载`](http://cdn1.raywenderlich.com/wp-content/uploads/2014/10/Layers-Start.zip)。该工程运行环境为Xcoed6.1以及iOS8.1SDK。

**提示**：*本教程中的源码是基于AsyncDisplayKit 1.0版本来写的，初始项目中已导入该版本的库文件。*

该APP主要用于浏览热带雨林发现的动物的基本信息，界面利用UICollectionView来制作。每个界面包含动物图片、名称、以及描述信息。卡片背景图片做了模糊处理，以突出文字描述。

![icon](/Users/zcx/octopress/source/images/20141124/1.png)

用Xcode打开`Layers.xcworkspace`工程。

本教程中请遵循以下几点来体验AsyncDisplayKit的强大之处：
* 请在真机运行。在模拟器上很难感受到。