---
title: 2022年我在使用这些vim插件
enlink: vim-plugs-2022
date: 2022-06-27 15:07:39
hidden: true
categories: 运维
tags:
- vim
- neovim
---

## 前言
从第一次接触 `vim` 已逾期 10 年, 期间大部分都是一些简单操作,
最近一两年开始深度使用 `vim`, 目前使用 `neovim` 版本.
本文将记录一些笔者觉得好用的一些 `Plugin`, 本文也将持续更新.

> 注意: 笔者使用的插件管理器是 [vim-plug](https://github.com/junegunn/vim-plug),
所以以下示例都是基于 `vim-plug` 来写的.

## Goto/Open
### vim-open-url
[vim-open-url](https://github.com/dhruvasagar/vim-open-url)
可以用浏览器打开光标下的 url. 

- `gB` 用默认浏览器打开光标下的 url
- `g<CR>` 使用默认搜索引擎搜索光标下的单词
- `gG` 使用 Google 搜索光标下的单词
- `gW` 使用 Wikipedia 搜索光标下的单词

```vim
Plug 'dhruvasagar/vim-open-url'
```

## Auto Complete
### neoclide/coc.nvim

