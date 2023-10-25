---
title: 多说适配pjax
enlink: duoshuo-pjax
date: 2017-03-09 19:50:45
categories:
- 大前端
tags:
- hexo
- jQuery
---
最近开发3-hexo主题，由于主题使用的pjax，异步加载页面时多说会出现加载不到多说js的问题。

多说加载代码如下：
```JavaScript
//加载多说
function loadComment() {
  duoshuoQuery = {short_name: $(".theme_duoshuo_domain").val()};
  var d = document, s = d.createElement('script');
  s.src = 'https://static.duoshuo.com/embed.js?t='+new Date().getTime();
  s.async = true; s.charset = 'UTF-8';
  (d.head || d.body).appendChild(s);
}
```
当局部加载页面时，就会无法加载多说。
需要编写一个js方法，参考文档：(http://dev.duoshuo.com/docs/50b344447f32d30066000147)
```JavaScript
/**
 * pjax后需要回调函数.加载多说
 */
function pajx_loadDuodsuo(){
  if(typeof duoshuoQuery =="undefined"){
    loadComment();
  } else {
    var dus=$(".ds-thread");
    if($(dus).length==1){
      var el = document.createElement('div');
      el.setAttribute('data-thread-key',$(dus).attr("data-thread-key"));//必选参数
      el.setAttribute('data-url',$(dus).attr("data-url"));
      DUOSHUO.EmbedThread(el);
      $(dus).html(el);
    }
  }
}
```
在pjax:end中调用此方法即可。
