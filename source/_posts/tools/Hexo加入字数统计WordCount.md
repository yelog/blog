---
title: Hexo加入字数统计WordCount
permalink: Hexo-WordCount
date: 2017-03-09 16:57:10
categories:
- 工具
- hexo
tags:
- hexo
---
只需要安装一个插件 WordCount

## 安装
```bash
$ npm i hexo-wordcount --save
```
## 使用
单篇文章字数
```html
<%=wordcount(post.content) %>
```
所有文章的总字数
```html
<%=totalcount(site) %>
```
## 日志
2017年3月9日，给3-hexo添加字数统计功能
