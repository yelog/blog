---
title: Mac神器-BTT(BetterTouchTool)不完全教程
enlink: Mac-BetterTouchTool
date: 2017-12-13 17:04:25
categories:
- 工具
- 软件记录
tags:
- mac
- efficiency
---
## 介绍
![BetterTouchTool](https://cdn.jsdelivr.net/gh/yelog/assets/images/Fm6s-GR0yVJgdndJwftdj9eXL7LB.png)
BetterTouchTool 是一款专为Mac用户开发的 窗口管理/`Trackpad`(触控板)/`Magic Mouse`(苹果鼠标)/`Keyboard`(键盘)/`TouchBar` 功能增强制作的软件。

这款软件不但可以设置全局的 手势/快捷键/TouchBar ，还可以给不同的应用定义不同的姿势，再配合上 Alfred 的 workflow，简直各种高难度姿势都能玩的出来。

本文主要介绍以下功能：
1. 窗口管理
2. 帮 Trackpad 定义各种姿势
3. 帮 Magic Mouse 定义各种姿势
4. 帮 Keyboard 定义各种姿势
5. 帮任何应用自定义 TouchBar

*本文以 macbook pro 2017 touchbar 版为例*
## 1. 窗口管理
这个功能无需过多配置，默认配置即可很好使用（和windows的理念相似）

- 将窗口移到左右边缘，最大化至半屏
- 将窗口移到上边缘，最大化至全屏

![窗口管理](https://cdn.jsdelivr.net/gh/yelog/assets/images/lhfTY9ysdWOzOOKprnh0X6MTfoT8.gif)
如果对默认配置不满意，也可以在如下图所示的位置来调整窗口展示：
![窗口管理配置](https://cdn.jsdelivr.net/gh/yelog/assets/images/FkluTBqX_n2UXj2ICsOE1tW1yFan.png)

## 2. 帮 Trackpad 定义各种姿势
### 姿势选择
在界面选择 Trackpad（触摸板） -> Add New Gesture（添加一个新姿势）

左边可以选择生效的范围：全局或者某个应用

![选择触摸板姿势](https://cdn.jsdelivr.net/gh/yelog/assets/images/FsPEIn9TlNOwHkWHgyD32snaT7bY.jpg)
如上图所示，姿势包括但不限于如：
1. 单指：左下角单击、单指轻拍右上角、单指轻拍上边中点
2. 双指：两个手指捏、张开两指以两指中心为圆轴逆时针、中指拍住中央食指轻拍面板、双指从上边缘下滑
3. 三指：三指轻拍、三指拍顶端、三指点击并向上滑、两指轻拍住，拍左、右二指固定拍住，左一下滑
4. 四指：四指双轻拍、中指无名小拍住，食单击、食中指无名拍住，小单击
5. 五指：五手指轻拍、五手指上滑

上面只是列一些典型，更多姿势可以在上图中浏览。

### 绑定功能
![定义姿势功能](https://cdn.jsdelivr.net/gh/yelog/assets/images/FjNWJgoefc34vGaUhruvWQr2R--5.png)

选择过姿势之后，也可以选择在按住某个功能键的时候才能使用（左下角）。

右边是绑定功能：快捷键或动作。

- 绑定快捷键举例：
比如 给chrome 设置 姿势（两指从触控板下边缘滑入），弹出开发者模式（快捷键绑定：command+option+i），如下图：
![给chrome设置姿势，弹出开发者模式](https://cdn.jsdelivr.net/gh/yelog/assets/images/Fnuaa2qtpdf28autM22l3RPRtCau.jpg)

- 绑定动作举例：
设置 在任何应用内，五指下滑 锁屏，如下图
![五指下滑锁屏](https://cdn.jsdelivr.net/gh/yelog/assets/images/FuTHBQJRvDVem9c14G608eXKtR7R.png)

## 3. 帮 Magic Mouse 定义各种姿势
这个功能设置和 Trackpad 设置 大同小异，所以这边就不多讲，直接图示几个功能。
![两指上滑呼出Mission Control](https://cdn.jsdelivr.net/gh/yelog/assets/images/FgwGsolKj59jVnOkgI5vb4zdxNwm.png)

我快捷键设置了 option+E 鼠标取词翻译（欧陆词典），然后绑定到双指轻拍鼠标，即可触发翻译。
![双指轻拍-取词翻译](https://cdn.jsdelivr.net/gh/yelog/assets/images/FsuJ3xU8k5ANQkuXdgpy0Xxkhdme.png)

## 4. 帮 Keyboard 定义各种姿势
这个功能比较简单，设置一些 键盘快捷键或录制案件序列 来触发 一些动作或者其他快捷键功能。

## 5. 帮任何应用自定义 TouchBar
这个重磅功能，可以帮助不支持touchbar的软件定制 TouchBar，是不是有点厉害。

下面就以我给 IntelliJ IDEA 定制 TouchBar 为例 (没有F1 ~ F12 功能键，debug真的很痛苦，这个软件真的是雪中送炭)，展示一下使用效果

![定制 TouchBar](https://cdn.jsdelivr.net/gh/yelog/assets/images/Fl0oSr-rNLr3f23_E_ZTlMEo46l1.png)

如上图所示，我给 IntelliJ IDEA 添加了 四个功能 step over/step into/resume/evaluate

添加完之后，切到 IntelliJ IDEA 软件中时，TouchBar 就显示我们添加的四个功能键， 如下图所示
![IntelliJ IDEA 定制 TouchBar](https://cdn.jsdelivr.net/gh/yelog/assets/images/Fi4MOmPsZDPj0Z2UEUxOqCbeIV1o.png)

## 最后

BTT还有其他很方便的功能，这盘就介绍到这里，等之后更新了 Alfred 的 workflow 开发指南之后，再一起更新一篇有意思的 BTT+Alfred 效率流。
