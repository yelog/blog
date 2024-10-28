---
title: Hexo主题3-hexo
enlink: 3-hexo
date: 2017-03-07 11:15:50
categories:
- 工具
- hexo
tags:
- 3-hexo
- hexo
---
阮一峰曾言：喜欢写blog的人，会经历三个阶段
>第一阶段，刚接触Blog，觉得很新鲜，试着选择一个免费空间来写。
第二阶段，发现免费空间限制太多，就自己购买域名和空间，搭建独立博客。
第三阶段，觉得独立博客的管理太麻烦，最好在保留控制权的前提下，让别人来管，自己只负责写文章。

有对搭建个人blog有兴趣的朋友，可以翻看我往期文章。

笔者从去年开始通过hexo写blog，使用了yilia主题，但是随着文章数量的上升，检索等操作就显得特别笨重。

在遍寻无果的情况下，就写下了[3-hexo](https://github.com/yelog/hexo-theme-3-hexo)主题。Demo:[http://yelog.org](http://yelog.org)

多图预警 ↓↓↓
## 设计思路
### 整体设计
**三段式设计:**
![三段式设计](https://cdn.jsdelivr.net/gh/yelog/assets/images/Fl2tl1Is5zx-D0DAt03bg0WkWXhO.png)
### 通过分类过滤
![分类过滤文章](https://cdn.jsdelivr.net/gh/yelog/assets/images/FmooXnOPeRPGBts5V5W7CV0AHuIo.gif)
### 通过标题关键字搜索
![文章标题关键字搜索](https://cdn.jsdelivr.net/gh/yelog/assets/images/FkF9lgTJoLdmNlYbTVokSNB3zdS4.gif)
### 通过作者搜索
**若开启了多作者模式，则可以通过输入@，进行作者搜索，如下所示**
![通过作者搜索](https://cdn.jsdelivr.net/gh/yelog/assets/images/FhbFRRPIDuz1pEKH-dr-RWDHVvXn.gif)
### 通过标签搜索
**输入#，就会出现标签提示**
![通过标签搜索](https://cdn.jsdelivr.net/gh/yelog/assets/images/FoJsDnsoLWKo7ECSzcLmzUX_uWgw.gif)
### 评论功能
![测试一下评论](https://cdn.jsdelivr.net/gh/yelog/assets/images/FtDD77YX_xenS-AZQW56qrwrQc4D.gif)
### 打赏功能
![打赏功能](https://cdn.jsdelivr.net/gh/yelog/assets/images/FhlNgOF7ipEIVrrztFdRam3WRikw.gif)
### 文章置顶
![文章置顶](https://cdn.jsdelivr.net/gh/yelog/assets/images/FhQLLqrRCr4yFGl9nDb_9oc4yME-.png)
### 返回头部
![返回头部](https://cdn.jsdelivr.net/gh/yelog/assets/images/FjpVByJViwYEWHHMTeayiQ-FD_qG.gif)

## 使用
### 1.安装
```bash 	
$ git clone https://github.com/yelog/hexo-theme-3-hexo.git themes/3-hexo
```
### 2.配置
1） 修改hexo根目录的`_config.yml`的两处，如下
```xml
theme: 3-hexo
highlight:
  enable: false #关闭hexo渲染高亮，使用主题代码块高亮
```

2） 在hexo 根目录source下添加`avatar.jpg`文件，作为头像

3) 安装字数统计(由于主题使用这个插件，必须安装，否则会报错)
```bash
$ $ npm i --save hexo-wordcount
```
**注意：** 如果没有安装会在 `hexo g` 的时候报错
### 3.更新
```bash
$ cd themes/3-hexo
$ git pull
```
