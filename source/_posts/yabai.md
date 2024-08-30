---
title: yabai
enlink: mac-yabai
date: 2021-07-01 11:00:07
categories:
- 工具
- 软件记录
tags:
- mac
- window-manager
---
### 启动命令
```bash
brew services restart skhd

brew services restart yabai

brew services restart spacebar

sudo yabai --load-sa
```


## 快捷键规划

### window

| 快捷键  | 描述                 |
| :--     | :--                  |
| alt - h | focus west window    |
| alt - j | focus south window   |
| alt - k | focus north window   |
| alt - l | focus east window    |

### desktop

| 快捷键         | 描述                            |
| :--            | :--                             |
| alt - 1        | focus desktop 1                 |
| alt - n        | focus next desktop              |
| alt - p        | focus previous desktop          |
| ctrl - alt - 1 | move window to desktop 1        |
| ctrl - alt - n | move window to next desktop     |
| ctrl - alt - p | move window to previous desktop |

### create/quit

| 快捷键         | 描述                                                      |
| :--            | :--                                                       |
| alt - w        | close current window, and focus left window               |
| alt - q        | close current desktop, and focus recent desktop           |
| alt - c        | create desktop                                            |
| alt - ctrl - c | create desktop and move current window to the new desktop |


