---
title: Github Copilot 使用技巧(完结)
enlink: github-copilot-skill
date: 2024-11-06 12:44:51
categories:
- AI
tags:
- ai
- copilot
- code
---

# 0 前言

从 [Github Copilot](https://github.com/features/copilot) 内测申请, 到后来作为体验小组成员, 推动公司统一购买, 已经使用了较长的时间. 积累的一些使用技巧, 我会在这边文章中进行分享, 如果有不对或需要补充的地方, 欢迎在评论区指出.

我会结合一些实际使用的例子从如下几个方面来分享:

- 代码补全
- Inline Chat 修改代码
- Chat 修改代码

> 本文的示例会使用 `Idea`, `VSCode`, `NeoVim` 等编辑器, 但是 `Copilot` 的使用方式是一样的.

# 1 代码补全

代码补全作为 `Copilot` 的核心功能, 也是刚开始使用时, 感知最明显的功能. 开启 `Copilot` 后, 直接开始编写代码, 会发现 `Copilot` 会根据你的输入, 生成一些代码片段, 你可以选择使用, 也可以继续输入, `Copilot` 会根据你的输入, 继续生成代码.


## 1.1 通过上下文, 生成代码补全

如下, 我们在 `TypeScript` 中有一个文件用于写 `API` 的地方, 当我们准备添加一个 `delAttribute` 方式时, 我们如数 `export const delAttribute` 之后, 就提示了代码, 可以按 `Tab` 或者设置快捷键让提示代码上屏

![Copilot Hint](https://cdn.jsdelivr.net/gh/yelog/assets/images/202411271034673.gif)

## 1.2 通过注释, 生成代码补全

我们还可以通过写注释的方式, 让 `Copilot` 更加精准的给我们生成提示代码, 如下在 `Vue3` 工程中, 需要通过计算属性的方式, 做一些处理, 我们可以现将注释写出来, 然后 `Copilot` 会根据注释, 生成代码 

![Hint from comments](https://cdn.jsdelivr.net/gh/yelog/assets/images/202411271054707.gif)

# 2 Inline Chat 修改代码

选中代码段, 然后打开 `Copilot Inline Chat`, 输入需要修改的问题并回车, `Copilot` 就会直接按照描述来修改代码

如下图, 我们在国际化文件中添加了中文, 然后复制到西班牙语的文件中, 需要修改为西班牙语, 我们可以直接调用 `Copilot Inline Chat` 来修改

![Inline Chat](https://cdn.jsdelivr.net/gh/yelog/assets/images/202411271103290.gif)


# 3 Chat 修改代码

我们可以直接在 `Chat` 聊天框中, 输入需要修改的问题, `Copilot` 会根据你的描述, 生成代码, 如果不满意, 可以继续聊天, 返回满意的代码后, 点击 `Accept` 就可以将代码应用到编辑器中.

![Github Chat](https://cdn.jsdelivr.net/gh/yelog/assets/images/202412021204553.png)

# 4 最后

使用 `Copilot` 的这两年, 对于我来说, 并没有提高太多的开发效率, 但是已经离不开 `Copilot` 了, 因为它带来的舒适地开发体验, 对开发人员来说更加重要.




