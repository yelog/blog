---
title: 3-hexo配置首页
date: 2017-03-13 09:56:07
permalink: 3-hexo-homepage
categories:
- 工具
- hexo
tags:
- 3-hexo
- hexo
---
今日将首页提到md文件中了，方便大家的更改。

首页文件位置 /layout/indexs.md ，既然是md格式，要怎么写大家应该都熟门熟路了，阿杰就不赘述了。

如果需要使用以下信息，可以按照下面的方式使用（**以下内容不限首页使用**）
## 文章数统计/字数统计
加入含有 `class="article_number"`的html标签可显示文章数量。
加入含有 `class="site_word_count"`的html标签可显示站点总字数。
```html
<!-- 我这里是借用了code的样式，所以直接使用code标签。
    自定义样式，可加入style属性设置-->
<code class="article_number"></code>
<code class="site_word_count"></code>
```
**上面代码的效果：**
文章：<code class="article_number"></code>篇；总字数：<code class="site_word_count"></code>字；
## 流量统计
>日志： 2017-03-18改动，由原来的 id 改为现在的 class，可在页面添加多个同类标签

加入含有 `class="site_uv"`的html标签可显示站点访问人次。
加入含有 `class="site_pv"`的html标签可显示站点访问量。
```html
<!-- 我这里是借用了code的样式，所以直接使用code标签。
    自定义样式，可加入style属性设置-->
<code class="site_uv"></code>
<code class="site_pv"></code>
```
**上面代码的效果：**
访问人数：<code class="site_uv"></code>人，访问量：<code class="site_pv"></code>次。
