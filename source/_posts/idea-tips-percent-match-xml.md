---
title: ideavim 使用百分号%支持xml的对应标签跳转
enlink: idea-tips-percent-mach-xml
date: 2023-06-17 16:09:00
categories:
- 工具
- IDEA
tags:
- IntellijIDEA
- efficiency
- vim
---

## 前言

由于最近几年使用 vim 的频率越来越高, 所以在 idea 中也大量开始使用 vim 技巧, 在一年多前碰到个问题, 终于在最近解决了。

## 问题描述

在 idea 中, 在 normal 模式下, 使用 % 不能在匹配标签(xml/html等) 之间跳转

## 解决方案

在 `~/.ideavimrc` 中添加如下设置, 重启 idea 即可

```bash
set matchit
```

## 效果

![](https://cdn.jsdelivr.net/gh/yelog/assets/images/picgo_qiniu2023-06-17%2016.23.45.gif)

## 最后

最近会把一些加的 tips 分享出来，大家有什么建议和问题都可以在评论区讨论.
