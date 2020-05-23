---
title: 3-hexo评论设置
permalink: 3-hexo-comment
date: 2020-05-23 22:26:23
categories:
- 工具
- hexo
tags:
- 3-hexo
---

## 前言

目前 `3-hexo` 已经集成了评论系统有 `gitalk` 、`gitment`、 `disqus` 、`来必力`、`utteranc`

## 一、gitalk

gitalk 是一款基于 Github Issue 和 Preact 开发的评论插件 官网: [https://gitalk.github.io/](https://gitalk.github.io/)

### 1. 登录 github ，注册应用

[点击进行注册](https://github.com/settings/applications/new) ，如下

![注册应用](https://i.loli.net/2020/05/23/6BmnUbX5gzPHqk1.png)

注册完后，可得到 `Client ID` 和 `Client Secret`

### 2. 新建存放评论的仓库

因为 `gitalk` 是基于 Github 的 Issue 的，所以需要指定一个仓库，用来承接 gitalk 的评论，我们一般使用 Github Page 来做我们博客的评论，所以，新建仓库名为 `xxx.github.io`，其中 xxx 为你的 Github 用户名

### 3. 配置主题

在主题下 `_config.yml` 中找到如下配置，启用评论，并使用 `gitalk`

```yaml
##########评论设置#############
comment:
  on: true
  type: gitalk
```

在主题下 `_config.yml` 中找到 gitalk 配置，将 第1步 得到的  `Client ID` 和 `Client Secret` 复制到如下位置

```yaml
gitalk:
  githubID:    # 填你的 github 用户名
  repo:  xxx.github.io	 # 承载评论的仓库，一般使用 Github Page 仓库
  ClientID:   # 第1步获得 Client ID
  ClientSecret:  # 第1步获得 Client Secret
  adminUser:     # Github 用户名
  distractionFreeMode: true
  language: zh-CN
  perPage: 10
```



## 二、来必力

### 1. 创建来必力账号，并选择 City 免费版

官网[http://livere.com/](http://livere.com/) ，创建账号，点击上面的安装，选择 City 免费版

![选择 city 免费版](https://i.loli.net/2020/05/23/mLYfjrJ1UgOIpiD.png)

复制获取到的代码中的 `data-uid`

![复制 data-uid](http://yelog-img.test.upcdn.net/447D431A-998C-4327-9463-A51D7CE91CE3.png)

### 2. 主题选择使用来必力评论

在主题下 `_config.yml`  

在找到来必力配置如下，第一步中复制的 `data-uid` 粘贴到下面 `data_uid` 处

```yaml
livere:
  data_uid: xxxxxx
```

找到以下代码， 开启并选择 livere (来必力)

```yaml
##########评论设置#############
comment:
  on: true
  type: livere
```



## 三、utteranc

官网地址：[https://utteranc.es/](https://utteranc.es/)

### 1. 安装 utterances

[点我进行安装](https://github.com/apps/utterances)

### 2. 配置主题

在主题下 `_config.yml` 中找到 `utteranc` 的配置 ，修改 `repo` 为自己的仓库名

```yaml
utteranc:
  repo: xxx/xxx.github.io # 承载评论的仓库，填上自己的仓库
  issue_term: pathname    # Issue 与 博客文章 之间映射关系
  label: utteranc         # 创建的 Issue 添加的标签
  theme: github-light     # 主题，可选主题请查看官方文档 https://utteranc.es/#heading-theme
# 官方文档 https://utteranc.es/
# 使用说明 https://yelog.org//2020/05/23/3-hexo-comment/
```

在主题下 `_config.yml` 中找到如下配置，启用评论，并使用 `utteranc`

```yaml
comment:
  on: true
  type: utteranc
```

