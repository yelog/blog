---
title: 用 neovim 写 markdown 是一种什么样的体验(含技巧)
enlink: write-markdown-in-neovim-experience-and-tips
date: 2024-08-02 15:10:59
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

插件管理器推荐 `lazy.nvim`, 如下配置也是基于 `lazy.nvim` 的, 其他的插件管理器配置也相似

很多使用技巧是基于插件和配置的, 所以本文的大纲如下:

1. 样式
2. 效率
3. Snippet

# 样式

## 全局主题

使用的是 [folke/tokyonight.nvim](https://github.com/folke/tokyonight.nvim) 主题

```lua
{
  "folke/tokyonight.nvim",
  config = function()
    require("tokyonight").setup({
      style = "night",        -- The theme comes in three styles, `storm`, `moon`, a darker variant `night` and `day`
      light_style = "day",    -- The theme is used when the background is set to light
      transparent = true,     -- Enable this to disable setting the background color
      terminal_colors = true, -- Configure the colors used when opening a `:terminal` in Neovim
      styles = {
        comments = { italic = true },
        keywords = { italic = true },
        functions = {},
        variables = {},
        sidebars = "transparent",       -- style for sidebars, see below
        floats = "transparent",         -- style for floating windows
      },
      sidebars = { "qf", "help" },      -- Set a darker background on sidebar-like windows. For example: `["qf", "vista_kind", "terminal", "packer"]`
      day_brightness = 0.3,             -- Adjusts the brightness of the colors of the **Day** style. Number between 0 and 1, from dull to vibrant colors
      hide_inactive_statusline = false, -- Enabling this option, will hide inactive statuslines and replace them with a thin border instead. Should work with the standard **StatusLine** and **LuaLine**.
      dim_inactive = true,              -- dims inactive windows
      lualine_bold = true,              -- When `true`, section headers in the lualine theme will be bold
      on_colors = function(colors)
        colors.border = "#ef9020"
      end,
      on_highlights = function(hl, c)
        hl.Visual = { bg = "#6D6BC8" }
      end,
    })
    vim.cmd([[colorscheme tokyonight]])
  end,
}
```

## 代码块高亮

使用的是 [tpope/vim-markdown](https://github.com/tpope/vim-markdown) 提供的代码块高亮功能

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

如下图所示, 左边是原始 markdown 文本, 右边是开启了 `vim-markdown` 插件的效果:

![vim-markdown](https://cdn.jsdelivr.net/gh/yelog/assets/images/202408301733036.png)

## 终端实时渲染

使用的是 [yelog/marklive.nvim](https://github.com/yelog/marklive.nvim) 进行终端样式实时渲染

```lua
{
    "yelog/marklive.nvim",
    dependencies = { 'nvim-treesitter/nvim-treesitter' },
    lazy = true,
    ft = "markdown",
    dev = true,
},

```

效果如下:

![marklive.nvim](https://cdn.jsdelivr.net/gh/yelog/assets/images/202408301903047.png)

## 实时浏览器预览

[iamcco/markdown-preview.nvim](https://github.com/iamcco/markdown-preview.nvim), 这是一个 `Markdown` 实时预览插件, 支持 `Github` 风格的 `Markdown`, 也支持数学公式渲染. 安装方法如下:

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

![markdown-preview](https://cdn.jsdelivr.net/gh/yelog/assets/images/202408301727428.gif)

# 效率插件

## 普通列表

[bullets-vim/bullets.vim](https://github.com/bullets-vim/bullets.vim), 这是一个辅助写 list 的插件, 支持有序列表, 无序列表, 任务列表等等, 可以缩进, 新增

```lua
{
    "bullets-vim/bullets.vim",
    config = function()
      vim.g.bullets_enabled_file_types = { "markdown", "text", "gitcommit", "scratch" }
    end,
}
```

效果如下:

![bullets.vim](https://cdn.jsdelivr.net/gh/yelog/assets/images/202408301740833.gif)


## 任务列表

[tenxsoydev/vim-markdown-checkswitch](https://github.com/tenxsoydev/vim-markdown-checkswitch), 任务状态切换插件

```lua
{
    "tenxsoydev/vim-markdown-checkswitch",
        config = function()
        vim.g.md_checkswitch_style = "cycle"
    end,
}
```

效果如下, 可以通过 `:CheckSwitch` 命令切换任务状态

![CheckSwitch](https://cdn.jsdelivr.net/gh/yelog/assets/images/202408311644937.gif)

## 表格

[dhruvasagar/vim-table-mode](https://github.com/dhruvasagar/vim-table-mode), 这是一个支持在 `vim/neovim` 中编辑 `markdown` 表格的插件, 支持高亮, 自动对齐, 自动生成等

```lua
{
    "dhruvasagar/vim-table-mode",
    config = function()
      vim.cmd([[
        augroup markdown_config
          autocmd!
          autocmd FileType markdown nnoremap <buffer> <M-s> :TableModeRealign<CR>
        augroup END
      ]], false)
      vim.g.table_mode_sort_map = '<leader>mts'
    end
}, --> table mode

```

效果如下

![vim-table-mode](https://cdn.jsdelivr.net/gh/yelog/assets/images/202408301748302.gif)


# Snippet

```snippets
# https://github.com/honza/vim-snippets/blob/master/snippets/markdown.snippets
snippet h1
	# ${1}

	${2}
snippet h2
	## ${1}

	${2}
snippet h3
	### ${1}

	${2}
snippet h4
	#### ${1}

	${2}
snippet h5
	##### ${1}

	${2}
snippet h6
	##### ${1}

	${2}
snippet h6
	##### ${1}

	${2}

snippet l
	[${1}](${2}) ${3}

snippet !
	![${1}](${2}) ${3}

snippet *
	*${1}* ${2}
snippet **
	**${1}** ${2}

snippet code
	\`${1}\` ${2}
snippet codeblock
	\`\`\`${1}
	${2}
	\`\`\`
	${3}

snippet tb
	| ${1} | ${2} | ${3} |
	| ---- | ---- | ---- |
	| ${4} | ${5} | ${6} |

snippet tb2
	| ${1} | ${2} |
	| ---- | ---- |
	| ${4} | ${5} |


snippet info
	---
	title: ${1:`expand('%:t:r')`}
	enlink: ${2}
	date: `strftime('%Y-%m-%d %H:%M:%S')`
	categories:
	- ${3}
	tags:
	- ${4}
	---

snippet callout
	> [!${1:NOTE}] ${2:title}
	> ${3:content}
```







