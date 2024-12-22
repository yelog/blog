---
title: 2024年MacOS终端大比拼
enlink: macos-terminal
date: 2024-06-23 15:54:00
categories:
- 工具
- 软件记录
tags:
- mac
- terminal
- efficiency
---

## 最流行的终端


## 横评

| capability   | Kitty | Alacritty | WezTerm | iTerm2 | Native |
| ----         | ----  | ----      | ----    | ----   | ----   |
| key-bind     | ✅    | ✅        | ✅      | ✅     | ❌     |
| vim-mode     | ❌    | ✅        | ✅      | ❌     | ❌     |
| Cursor Trail | ✅    | ❌        | ❌      | ❌     | ❌     |
| show-image   | ✅    | ❌        | ❌      | ❌     | ❌     |

## log

### 2024-11-17

> `Kitty` -> `WezTerm`

- 发现 `WezTerm` 的 `cmd+k` 不能用是因为写了一个测试出的绑定键导致的
- `Kitty` 快捷键经常失效

### 2024-10-31
> `WezTerm` -> `Kitty`

- `Kitty` support [Add Cursor Trail Feature to Enhance Cursor Visibility](https://github.com/kovidgoyal/kitty/pull/7970)
- support `command-k` 用于绑定 `avante.nvim`

### 2024-06-27

> `Kitty` -> `WezTerm`

`Kitty` 绑定 `cmd-shift-f` 在 tmux 下无法使用， 且没有 `vim-mode`


### 2024-06-26

> `Alacritty` -> `Kitty`

因为在 `vim` 的 `normal` 模式下, 如果是中文输入法, 输入的内容会出现在输入法的候选框内, 然后按 `<CAPS>` 按键切换输入法, 候选框中的输入的字母, 会以 insert 的方式输出到光标所在的位置, 这个问题在 `WezTerm` 和 `Kitty` 中没有出现.

> 注意: `Kitty` 的 `map cmd+1 send_key cmd+1` 能够正常映射到 `NeoVim` 中进行 `maps.n["<D-1>"] = { "<cmd>Neotree left toggle<cr>", desc = "Toggle Explorer" }` 绑定, 但是开启 tmux 后, `cmd+1` 映射到 `NeoVim` 中就不行了

### 2024-06-20

> `WezTerm` -> `Alacritty`

