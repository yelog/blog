---
title: linux下修改按键ESC<=>CAPSLOCK和Control=>ALT_R
permalink: linux下修改按键ESC<=>CAPSLOCK和Control=>ALT_R
date: 2017-10-20 17:38:49
categories:
- 运维
tags:
- keybord
- vim
- emacs
- linux
---
使用 `vim` 过程中发现 `esc` 和 `ctrl` 按键很难按，小拇指没有那么长啊～～，而 `caps_lock` 和 `alt_r`(右alt) 很少用。

本教程将 `esc` 和 `caps_lock` 两个按键交换， `alt_r`(右alt) 改为 `ctrl`。

## 一、 esc 与 caps_lock 按键交换
①. 创建 `.xmodmaprc` 文件。
②. 加入以下内容：
```bash
remove Lock = Caps_Lock
add Lock = Escape
keysym Caps_Lock = Escape
keysym Escape = Caps_Lock
```
③. 执行 `xmodmap .xmodmaprc` 使之生效。
## 二、 将 右alt 改为 ctrl
①. 查看需要修改键位的 keysym
通过 `xev | grep keycode` 获取右 `alt` 的 keysym 为 `Alt_R`。如下图所示：
![通过xev获取右alt的keysym](http://img.xiangzhangshugongyi.com/FvuqjLi5czeBluMTyIfv_xUOcu5k.png)

②. 查看 `Alt_R` 是哪个 modifier 使用的
通过 `xmodmap -pm` 查看，发现 `Alt_R` 是作为 modifier `mod1` 使用的。如下图所示：
![查看 Alt_R 是作为 mode1 使用的](http://img.xiangzhangshugongyi.com/Fib8QjT-Ccx30DCf2rF4WkzHsbOH.png)

③. 修改 modifier
```bash
xmodmap -e 'remove mod1 = Alt_R' # 解除原来绑定
xmodmap -e 'add control = Alt_R' # 作为 control 使用
```
