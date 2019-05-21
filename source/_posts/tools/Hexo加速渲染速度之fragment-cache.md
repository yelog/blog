---
title: Hexo加速渲染速度之fragment_cache
permalink: hexo-fragment_cache
date: 2017-09-21 19:34:37
categories:
- 工具
- hexo
tags:
- hexo
- fragment_cache
---
## 前文
从开发 `3-hexo` 主题到现在已过去 9 个月时间了，累计在博客中写 132 篇文章了。

现在发现了严重的问题，`hexo generate` 渲染的速度越来越慢，现在132篇左右，每次渲染时间到达了 50+ s，相当不爽。

今日抽时间，查看了官方api，看到了 `fragment_cache` 局部缓存这个东西，解决了渲染速度的问题。

## 使用
### 官方文档
局部缓存。它储存局部内容，下次使用时就能直接使用缓存。
```javascript
<%- fragment_cache(id, fn); %>
```
### 替换简单文本区域
a. 我们可以将所有页面都一样的区域，如下所示，缓存下来。当下一篇文章在渲染到这个位置时，将不再渲染，直接拿缓存数据。
```js
<%- fragment_cache('header', function(){
    return partial('<head></head>');
}) %>
```


b. 文章模块也可以使用，原来公共引用部分（没有和当前文章耦合的内容）使用下面的方式：
```js
<%- partial('_partial/header'); %>
```
改进为以下代码：
```js
<%- fragment_cache('header', function(){
    return partial('_partial/header');
}) %>
```

## 最后
这个语法只适用于所有页面都相同，不随文章内容变化的部分。

作者在 `3-hexo` 中加入了此语法，渲染132篇文章的速度已从 50+s 到现在 3s 左右了。
