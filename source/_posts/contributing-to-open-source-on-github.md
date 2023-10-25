---
title: 如何给GitHub上的项目贡献代码
enlink: contributing-to-open-source-on-github
date: 2016-10-13 15:44:07
categories:
- 工具
- git
tags:
- Git
- GitHub
---
最近一直在使用 `hexo` 的一款主题 `yelee` ，但是发现它的代码块由于空行不占位导致的显示错位，所以就去GitHub上翻issue，果然有好多人都在反映这个问题，并且作者已经打上bug标签，事情应该就马上结束了，就去忙别的了。这两天又去逛了一下issue，发现这个bug仍然屹立在那里，强迫症又犯了，趁着今天工作不怎么忙，就把这个bug解决了。然后问题来了，怎么才能给作者贡献代码呢。
<!--more -->
## 准备工作
1. 首先通过 `git clone` 将项目克隆到本地（我早已拉下来，跳过此步骤）
2. `git pull` 拉取最新代码（将所有的change都同步到本地）
3. 将 原项目 `fork` 到 自己的github上,并复制代码url
4. 在本地添加第二个仓库地址：`git remote add [nickname] [your url]`

## 修改
1. 修改bug 或 新增功能
2. `git commit [file1] [file2] ... -m [message]` 本地提交代码

## 同步到github中并发到原项目
1. `git push [nickname]` 将代码 push 到自己的项目里，nickname就是添加的第二个仓库的名字
2. 自己项目内，点击 pull requests -》 new pull request 将本次修改提交到原项目进行同步。
