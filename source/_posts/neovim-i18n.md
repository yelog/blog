---
title: Neovim 插件 i18n.nvim 介绍
enlink: neovim-i18n
date: 2025-09-10 11:20:19
categories:
- 工具
- vim
tags:
- vim
- neovim
- i18n
- internationalization
---

# 前言

最近一直在使用 Neovim 做 vue3 的开发，其中使用了 [vue-i18n](https://github.com/intlify/vue-i18n) 作为国际化的解决方案，项目中有大量的国际化内容，需要统计管理、查询、提示。

正赶上最近 `Vibe Coding` 的概念比较火，就使用业务时间让 AI 帮忙写了这个国际化插件 [yelog/i18n.nvim](https://github.com/yelog/i18n.nvim)，主要功能有:
1. 实时预览国际化key
2. 国际化key的补全(集成 blink.cmp)
3. 国际化key定义的跳转
4. 国际化key不存在时提示 Diagnostic
5. 国际化key的统计
6. 国际化key的模糊搜索(集成 fzf.lua)

# 安装

推荐使用 `layz.nvim` 作为插件管理器，安装方式如下:

```lua
{
  'yelog/i18n.nvim',
  dependencies = {
    'ibhagwan/fzf-lua',
    'nvim-treesitter/nvim-treesitter'
  },
  config = function()
    require('i18n').setup({
      locales = { 'en', 'zh' },
      sources = {
        'src/locales/{locales}.json',
      }
    })
  end
}
```

其中 `locales` 作为你项目中的语言列表， `sources` 作为你项目中国际化文件的路径，`{locales}` 会被替换为 `locales` 中的语言列表。

`sources` 支持多个文件类型，变量路径等，具体可以参考 [REAEDME#Use Case](https://github.com/yelog/i18n.nvim?tab=readme-ov-file#-use-case)

# 使用介绍

## 实时预览国际化key

支持在国际化key使用的地方如 `t('common.hello')` 处，实时预览国际化内容。并且支持切换默认显示的语言，及是否显示国际化key

![实时预览国际化key](https://cdn.jsdelivr.net/gh/yelog/assets/images/202509101143188.gif)

## 国际化key的补全

在国际化key使用的地方如 `t('|')` 的竖线出，会集成 `blink.cmp` 进行补全显示

![补全国际化key](https://cdn.jsdelivr.net/gh/yelog/assets/images/202509101145231.png)

## 国际化key定义的跳转

在国际化 Key 使用的地方如 `t('common.hello')` 处，按 `gd` 可以跳转到国际化 key 的定义处。

在国际化key的定义处，按 `gd` 可以跳转到其他语言的定义处

![跳转定义](https://cdn.jsdelivr.net/gh/yelog/assets/images/202509101146459.gif)

## 国际化key不存在时提示 Diagnostic

如果在国际化key使用的地方使用了不存在的key，如 `t('common.hello1')`，会有 Diagnostic 提示

![diagnostic](https://cdn.jsdelivr.net/gh/yelog/assets/images/202509101147335.png)

## 国际化key的模糊搜索(集成 fzf.lua)

通过集成 fzf.lua 实现国际化key及默认语言翻译的模糊搜索。

![fzf.lua](https://cdn.jsdelivr.net/gh/yelog/assets/images/202509101148469.gif)

## 支持 help 提示

在国际化key使用的地方如 `t('common.hello')` 处，按 `<c-k>` 可以查看帮助提示

![help](https://cdn.jsdelivr.net/gh/yelog/assets/images/202509101150210.png)


# 最后

这个插件主要时为了满足自己的需求设计的，所以如何有任何建议和意见，欢迎提 [issue](https://github.com/yelog/i18n.nvim/issues)



