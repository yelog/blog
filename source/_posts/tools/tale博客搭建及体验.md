---
title: tale博客搭建及体验
permalink: tale-build-experience
date: 2017-05-24 11:29:30
categories:
- 工具
- 软件记录
tags:
- tale
---
不久之前在逛blog时，发现了这款tale，今天抽空搭建了一下，将搭建过程写于此。
demo website：https://tale.biezhi.me

## 搭建思路
看了tale作者的(github)[https://github.com/otale] 发现有建好docker，所以果断使用docker搭建tale的环境

### 构建docker镜像
下载[tale-docker](https://github.com/otale/tale-docker)到本地。
```bash
# 下载官方Dockerfile
$ git clone https://github.com/otale/tale-docker.git
# 构建 tale 镜像
$ docker build -t tale:1.0 .
```

### 下载tale博客文件
```bash
# 下载压缩包
$ sudo wget http://7xls9k.dl1.z0.glb.clouddn.com/tale.zip

# 讲解压出来的文件夹移入home目录
$ unzip tale.zip
$ mv tale /home
```

### 构建tale镜像
```bash
docker run -d --privileged --hostname tale --name tale \
-v /etc/localtime:/etc/localtime:ro \
-v /home/tale:/var/tale_home -p 80:9000 \
-m 1024m --memory-swap -1 tale:1.0
```

### 访问
浏览器进入 `127.0.0.1` 即可访问
![管理后台](http://img.saodiyang.com/FtwT7OgspZR3YjpJhH5VSuUmnLjP.png)
![首页](http://img.saodiyang.com/FnFBhuwvQ4atG2F2ahdj5PSFF3eL.png)
![文章](http://img.saodiyang.com/Fg1891hUmEATDbFDBgdBsncb4U1U.png)

## 体验
### 管理后台
1. 文章支持Markdown和富文本。
2. 文章/评论/友链/标签管理/主题，设置简单，一目了然
3. 支持插件扩展

### 博客
1. 主题简洁（当然支持切换主题）
2. 使用 `instantclick` ，页面切换流畅
3. 评论系统，简洁易用
4. 搜索只支持文章标题

### 整体
1. 管理简单方便
2. 使用docker后，迁移数据也方便
3. 主题还不是很多
4. 对于常年使用静态blog，手动渲染/发布，使用这个之后还有点小清新。

## 最后
tale整体不错，值得入手。

不过目前没有笔者喜欢的主题（当然默认主题也不错），暂时不打算更换blog，笔者也打算过一段时间开发一个tale的主题，然后正式迁入tale。
