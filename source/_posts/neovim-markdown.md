---
title: 用 neovim 写 markdown 是一种什么样的体验(含技巧)
enlink: write-markdown-in-neovim-experience-and-tips
date: 2024-08-02 15:10:59
hidden: true
categories:
- 工具
- vim
tags:
- vim
- neovim
- markdown
- editor
---

# 前言

从上大学开始使用 `vim` 已有 12 年, 最近三四年开始深度使用 `vim/neovim`, 包括写代码, 写文档, 写博客等等. 从这篇文章开始, 我将会记录和分享关于 `vim` 的一些使用技巧, 配置和插件.

很多使用技巧是基于插件和配置的, 所以本文的大纲如下:

1. 推荐插件
2. 配置
3. 写 markdown 的技巧

# 推荐插件


插件管理器推荐 `lazy.nvim`, 如下配置也是基于 `lazy.nvim` 的, 其他的插件管理器配置也相似

## 1. iamcco/markdown-preview.nvim

这是一个 markdown 实时预览插件, 支持 github 风格的 markdown, 也支持数学公式渲染. 安装方法如下:

```lua
{
    "iamcco/markdown-preview.nvim",
    cmd = { "MarkdownPreviewToggle", "MarkdownPreview", "MarkdownPreviewStop" },
    ft = { "markdown" },
    build = function()
      vim.fn["mkdp#util#install"]()
      vim.g.mkdp_theme = 'light'
    end,
}
```

添加了上面的插件后, 可以使用 `:MarkdownPreview` 命令就会自动弹出默认浏览器, 进行 markdown 实时预览.

效果如下:


## 2. tpope/vim-markdown

这是一个 markdown 的语法高亮插件, 安装方法如下:

```lua
{
    "tpope/vim-markdown",
    config = function()
      -- tpope/vim-markdown
      vim.g.markdown_syntax_conceal = 0
      vim.g.markdown_fenced_languages =
      { "html", "python", "bash=sh", "json", "java", "js=javascript", "sql", "yaml", "xml", "Dockerfile", "Rust",
        "swift", "javascript", 'lua' }
    end,
}, --> syntax highlighting and filetype plugins for Markdown
```










