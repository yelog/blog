---
title: 最强跳转插件 flash.nvim 在 ideavim 上使用是中什么体验
enlink: ideavim-flash
date: 2024-09-05 22:47:36
categories:
- 工具
- vim
tags:
- vim
- neovim
- ideavim
- idea
- jetbrain
- editor
---

# 前言

最近两三年都从 `Vim` 迁移到 `NeoVim` 之后, 使用到了非常多好用的插件, 尤其是跳转插件 [folke/flash.nvim](https://github.com/folke/flash.nvim) , 非常方便, 日常文档及一些软件开发(Web, rust, lua, python) 等已经在 `NeoVim` 下完成了

但是 `Java` 一直没有配置到像 `IntelijIdea` 那么方便, 所以 `Java` 的开发还是在 `IntelijIdea` 中完成, 好在有 `IdeaVim` 这个非常棒的插件, 大部分的 `Vim` 功能完成度非常高.

最大的缺点就是没有 `Vim`, `NeoVim` 的丰富的插件生态, 尤其是日常使用频率非常高的 `flash.nvim`, 所以就自己开发了一个在 `IdeaVim` 上的插件 [vim-flash](https://github.com/yelog/vim-flash)

>题外话: 本来叫 `ideavim-flash` 的, 在上传插件的时候, 因为存在关键字 `idea` 被驳回, 所以改名为 `vim-flash`.

# 安装

## 在线安装

插件市场安装: Setting -> Plugins -> Marketplace -> 搜索 `vim-flash`, 作者为 `yelog`, 然后点击安装.

![插件市场安装](https://cdn.jsdelivr.net/gh/yelog/assets/images/202409052305048.png)

## 离线安装

1. 到官方仓库的 `Release` 中下载 `vim-flash-xxx.zip` 包, 地址 [vim-flash-release](https://github.com/yelog/vim-flash/releases)
2. Idea -> Setting -> Plugins -> Install 旁边的齿轮 -> Install Plugin from disk -> 选择刚刚下载的 `vim-flash-xxx.zip` 包即可

![离线安装](https://cdn.jsdelivr.net/gh/yelog/assets/images/202409052310873.png)

# 配置

安装完成之后, 点击右下角的 `IdeaVim` 图标, 点击 `Open ~/.ideavimrc`

![编辑 ideavimrc](https://cdn.jsdelivr.net/gh/yelog/assets/images/202409052311037.png)

添加 `map s <Action>(flash.search)` 到最后一行, 然后点击右上角的刷新图标

![添加配置](https://cdn.jsdelivr.net/gh/yelog/assets/images/202409052313446.png)

# 使用和效果

## Normal Mode

打开一个文件, 按 `Esc` 进入 `IdeaVim` 的 `Normal` 模式下, 比如我们要定位到 `MarksCanvas` 这个单词, 我们可以依次按键盘的字母: `smarks`

1. 现在所有以包含 `marks` 的文字都高亮了, 并且后面跟着一个字母, 当我们按下某一个字母后, 就会发现光标到达了这个高亮处, 这就是这个插件的跳转功能.
2. 有一个高丽是橙色底的, 那是距离我们光标最近的位置, 当我们按下 **回车** 后, 光标会跳转到这个高亮处.

![vim-flash-normal-usage](https://cdn.jsdelivr.net/gh/yelog/assets/images/202409052324760.gif)

## Visual Mode

打开一个文件, 按 `v` 进入 `IdeaVim` 的 `Visual` 模式下, 我们可以通过类似于上面的跳转方式, 进行跳转选中


![vim-flash-vistual-usage](https://cdn.jsdelivr.net/gh/yelog/assets/images/202409052328220.gif)


# 结语

我是一个热爱技术, 崇尚效率和善于利用工具的人, 我会持续分享所得, 如果有收获, 请帮忙点赞, 评论,  点 `start`, 谢谢!!!


