---
title: 不蒜子适配pjax
permalink: busuanzi-pjax
date: 2017-03-09 20:15:51
categories:
- 大前端
tags:
- hexo
- jQuery
---
## 不蒜子一般配置
加入脚本
```html
<script async src="//dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
<!--pv方式 -->
<span id="busuanzi_container_site_pv">
    本站总访问量<span id="busuanzi_value_site_pv"></span>次
</span>
<!--uv方式 -->
<span id="busuanzi_container_site_uv">
  本站访客数<span id="busuanzi_value_site_uv"></span>人次
</span>
<!--pv方式 -->
<span id="busuanzi_container_page_pv">
  本文总阅读量<span id="busuanzi_value_page_pv"></span>次
</span>
```
只安装脚本，不安装标签代码，即可实现只记数，不显示。

## 适配pjax
最近开发3-hexo主题，由于主题使用的pjax，异步加载页面时不蒜子会出现加载不到多说js的问题。
在pjax：end加载下面js代替标签即可
```JavaScript
$.getScript("//dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js");
```
