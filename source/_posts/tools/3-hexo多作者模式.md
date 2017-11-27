---
title: 3-hexo多作者模式
permalink: 3-hexo-multiple-author
date: 2017-02-28 10:55:31
categories:
- 工具
tags:
- hexo
- 3-hexo
---
尽管hexo是为个人blog而生的工具，但是有时也可能会有多作者需求，比如他人投稿等等，为此笔者在写3-hexo主题时,顺便添加了此功能 。

## 1.修改配置文件
修改 `3-hexo/_config.yml`，开启多作者模式，并添加blog中出现的作者，为搜索提供数据
```xml
author:
  on: true #true：开启多作者模式
  authors:
    author1: yelog #添加两个作者yelog、小马哥
    author2: 小马哥
```

## 2.修改文章头部信息
添加 `author: yelog` ，表示这篇文章的作者为yelog
```xml
---
title: reading-list
date: 2017-01-31 15:29:32
author: yelog
top: 2
categories:
- 读书
tags:
- reading
---
```
**效果：**
![](http://oncj6b2vl.bkt.clouddn.com/Fjq0M7pBzl6fsnC3ivpqMsdLdXc0.png)

## 搜索某个作者的所有文章
在搜索栏中输入`@小马哥`就可以显示出所有小马哥的文章。
如果你在_config.xml中配置了作者名，就可以出现`提示`,具体看第一部分
**效果如下：**
![](http://oncj6b2vl.bkt.clouddn.com/Fm2PK5W9Rd6ojYq055zZoWcbioAn.gif)
