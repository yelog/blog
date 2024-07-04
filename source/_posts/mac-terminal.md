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

| capability | Kitty | Alacritty | WezTerm | iTerm2 | Native |
| ----       | ----  | ----      | ----    | ----   |        |
| key-bind   |       |           |         |        |        |


## log

### 2024-06-27

> 放弃 `Kitty` -> 转为 `WezTerm`

`Kitty` 绑定 `cmd-shift-f` 在 tmux 下无法使用， 且没有 `vim-mode`


### 2024-06-26

> 放弃使用 `Alacritty` -> 转为 `Kitty`

因为在 `vim` 的 `normal` 模式下, 如果是中文输入法, 输入的内容会出现在输入法的候选框内, 然后按 `<CAPS>` 按键切换输入法, 候选框中的输入的字母, 会以 insert 的方式输出到光标所在的位置, 这个问题在 `WezTerm` 和 `Kitty` 中没有出现.

> 注意: `Kitty` 的 `map cmd+1 send_key cmd+1` 能够正常映射到 `NeoVim` 中进行 `maps.n["<D-1>"] = { "<cmd>Neotree left toggle<cr>", desc = "Toggle Explorer" }` 绑定, 但是开启 tmux 后, `cmd+1` 映射到 `NeoVim` 中就不行了

### 2024-06-20

> 放弃 `WezTerm` -> 转为 `Alacritty`

