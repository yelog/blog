---
title: MathJax适配Pjax
enlink: MathJax-pjax
date: 2017-07-05 19:24:10
categories:
- 工具
- hexo
tags:
- mathjax
- pjax
---

hexo 添加 MathJax 的过程网上很多，这里就不细讲，这里贴一张写的不错的文章 [Hexo博客(13)添加MathJax数学公式渲染](http://masikkk.com/article/hexo-13-MathJax/)

由于 `3-hexo` 这个主题使用了 `pjax` ，刷新和第一次加载没有问题，但是点到其他文章，再点回来，渲染就无效了。

这个问题和之前适配多说和高亮时，是同样的问题，只需要在下面配置即可。
```js
$(document).on({
    /*pjax请求回来页面后触发的事件*/
    'pjax:end': function () {
        /*渲染MathJax数学公式*/
        $.getScript('//cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML',function () {
            MathJax.Hub.Typeset();
        });
    }
});
```

这样就解决了pjax的适配问题。
