---
title: windows环境下使用hexo搭建blog平台
enlink: windows-hexo
date: 2017-03-17 14:02:55
categories:
- 工具
- hexo
tags:
- hexo
---
之前已经出过几期搭建 `hexo` 的文章，但是有不少朋友私信说系统是windows的，希望出一期windows环境下的hexo搭建文章。

为此，确保可用性，笔者在linux（笔者的系统环境）环境下安装了windows虚拟机重新演练了一边，确保没有什么漏洞，才写下了此文。

本文会非常详细，以确保没有计算机背景的小白也可以轻松上手
## 环境搭建
### 安装git
1).下载：从官网下载windows版本的git,地址在下方。
[https://git-scm.com/download/win](https://git-scm.com/download/win)
2).安装：双击安装，一直点击下一步即可

### 安装node.js
1).下载：从官网下载windows版本的node.js安装包（.msi后缀），地址下方
[https://nodejs.org/zh-cn/download/](https://nodejs.org/zh-cn/download/)
2).安装：双击安装，一直点击下一步即可

## 安装hexo
在任意目录如桌面，点击鼠标右键，选择Git Bash Here这一项，打开git bash命令框（前提是git安装成功），如下图
![鼠标右键](https://cdn.jsdelivr.net/gh/yelog/assets/images/FraxkCebAzx-dlPGsUmgcLZRR_Lj.png)
在打开的命令窗内输入下面的命令进行安装
```bash
npm install hexo-cli -g
```
安装过后，输入 `hexo -v`，出现下面信息，则表示安装成功
```bash
$ hexo -v
hexo-cli: 1.0.2
os: Windows_NT 6.1.7601 win32 x64
http_parser: 2.7.0
node: 6.10.0
v8: 5.1.281.93
uv: 1.9.1
zlib: 1.2.8
ares: 1.10.1-DEV
icu: 58.2
modules: 48
openssl: 1.0.2k
```
## 初始化blog
进入准备创建blog的目录，同样点击鼠标右键，打开git bash命令框，
执行一下命令进行初始化
```bash
$ hexo init myblog
```
就会自动创建一个名字为myblog目录，这时本地blog就已经创建好了。
进入blog目录，启动 blog
```bash
$ cd myblog
$ hexo server
```
在浏览器输入 `127.0.0.1:4000`就可以访问到刚刚创建好的blog了。
## 换皮肤
如果觉的自带的皮肤太难看。可以根据以下步奏更换皮肤
在[官网](https://hexo.io/themes/) 可以查看各种各样的皮肤，挑选自己喜欢的皮肤

这里以 `3-hexo` 这款皮肤为例（这款皮肤是笔者写的，效果可查看 [yelog.org](http://yelog.org/)）
1）进入皮肤的 `github` 官网，如[3-hexo](https://github.com/yelog/hexo-theme-3-hexo)的网址
找到 `clone or download` ,复制它的url：https://github.com/yelog/hexo-theme-3-hexo.git

2）进入 `myblog` 目录，打开 `git bash` 命令框，执行以下命令将皮肤下载到themes目录下
```bash
$ git clone https://github.com/yelog/hexo-theme-3-hexo.git themes/3-hexo
```

3) 修改 `myblog/_config.yml` 中的 `theme: landscape` 为 `theme: 3-hexo`

> 如果使用 `3-hexo` 主题的话，还需要注意两点
①因为主题使用了自己的高亮效果，还需要修改 `highlight enable: true` 的 `true` 改为 `false`。
②由于主题启用了文章字数统计功能，需要安装一个插件，在 `myblog` 目录下，打开 `git bash` 命令框，执行 `npm i --save hexo-wordcount` 即可

4) 重新渲染，启动服务器
```bash
$ hexo clean && hexo g && hexo s
```

5) 打开浏览器查看效果，换肤成功

## 如何写文章
文章在 `myblog/source/_posts/` 下，以markdown格式写成，笔者推荐使用`atom`作为写作工具。
可以通过 `hexo new 文章名` 来创建一篇文章，当然也可以直接在 `_posts` 目录下直接新建.md文件。
执行命令 仍是在 `myblog` 目录下，打开 `git bash` 命令框。以下是常用命令，其他可以查阅官网。
```bash
# 创建一个标题为“git教程”的文章
$ hexo new "git教程"

# 清除所有渲染的页面
$ hexo clean

# 将markdown渲染成页面
$ hexo g

# 启动hexo
$ hexo s
```

## 发布到网上
如果想要在github上搭建blog，或者在自己的购买的服务器上搭建blog
可以查看笔者的往期文章

今天的教程就到这里，有什么问题可以在评论区交流。
