---
title: pjax用法
permalink: pjax
date: 2017-02-08 20:18:54
categories:
- 大前端
tags:
- jQuery
---
最近在开发一款hexo主题3-hexo，其中使用了pjax大大提高了用户体验和加载速度，在此简单介绍一下pjax的用法[github链接](https://github.com/defunkt/jquery-pjax)
<!--more -->
## pjax是什么
>pjax是一款jQuery插件，使用了ajax和pushState的技术，在保留真正永久链接，网页标题和可用的返回功能的情况下，带来一种快速的浏览体验。 --官方介绍

`用人话说`，就是当跳转过去的网页和当前网页的一部分是一样的，这时可以通过pjax就会从响应页面中取出 `不同的那部分` （需指定），替换掉原来的内容。 如果在服务端判断处理，直接返回 `不同的那部分内容`，这样就可以减少带宽占用，提升加载速度。

这样做的优势：
1. 由于从服务器取回的数据量变少，加载速度将会提升。
2. 并且采用异步刷新页面中的不一样的地方，用户体验也是满满的。
3. 保留了浏览器回退的功能（解决了ajax的尴尬）

好了，开始操作。
## Demo
第一步：引入jQuery和jQuery.pjax
```html
<script src="//cdnjs.cloudflare.com/ajax/libs/jquery/2.2.4/jquery.min.js"></script>
<script src="//cdnjs.cloudflare.com/ajax/libs/jquery.pjax/1.9.6/jquery.pjax.min.js"></script>
```

第二步：将指定的a的链接，转为pjax风格
```javascript
/*将#menu中的a的链接的页面，只取回class=pjax元素中的内容，替换掉当前页面class=pjax元素中的内容*/
$(document).pjax('.#menu a', '.pjax', {fragment:'.pjax', timeout:8000});
```

第三步：如果需要在请求的过程中做一些自定义的事件，可以使用下面的方法
```javascript
$(document).on({
  'pjax:click': function() {
    //点击链接时，需要触发的事件写到这里
  },
  'pjax:start': function() {
    //当开始获取请求时，需要触发的事件写在这里
  },
  'pjax:end': function() {
    //当请求完成后，需要触发的事件写在这里
  }
});
```

结束。

## 详细文档
翻译于官方
### 参数
```javascript
$(document).pjax(selector, [container], options)
```
1. `selector` 触发点击事件的选择器，String类型
2. `container` 一个选择器，为唯一的pjax容器
3. `options` 一个可以包含下面这些选项的对象

### pjax options

| key | default | description |
| :- | :-|:- |
| `timeout` | 650 | ajax超时时间，单位毫秒，超时后将请求整个页面进行刷新 |
| `push` | true | 使用 pushState 添加一个浏览器历史导航条目 |
| `replace` | false | 替换URL，而不添加浏览器历史条目 |
| `maxCacheLength` | 20 | 历史内容 cache 的最大size |
| `version` |  | string ： 当前pjax版本 |
| `scrollTo` | 0 | 垂直位置滚动，为了避免改变滚动条位置 |
| `type` | "GET" | 可以查看[jQuery.ajax()](http://api.jquery.com/jQuery.ajax/) |
| `dataType` | "html" | 可以查看[jQuery.ajax()](http://api.jquery.com/jQuery.ajax/) |
| `container` |  | css选择器，此元素内容将被替换 |
| `url` | link.href | string: ajax 请求的URL |
| `target` | link | eventually the relatedTarget value for [pjax events](#Events) |
| `fragment` |  | 从ajax响应的页面中抽取的‘片段’ |

### Events
除了`pjax:click`和`pjax:clicked`外的所有pjax事件从pjax容器中触发，是不需要点击链接的。
**所有事件的生命周期在通过pjax请求链接的过程中**

| event | cancel | arguments | notes |
| :- | :- | :- | :- |
| `pjax:click` | ✔︎ | `options` | 在一个链接被激活（点击）时触发此事件，可以在此取消阻止pjax |
| `pjax:beforeSend` | ✔︎ | `xhr, options` | 可以设置 XHR 头 |
| `pjax:start` |  | `xhr, options` |  |
| `pjax:send` |  | `xhr, options` |  |
| `pjax:clicked` |  | `options` | 当链接被点击，并且已经开始pjax请求后触发 |
| `pjax:beforeReplace` |  | `contents, options	` | 从服务器已经加载到HTML内容，在替换HTML内容之前触发 |
| `pjax:success` |  | `data, status, xhr, options` | 从服务器已经加载到HTML内容，在替换HTML内容之后触发 |
| `pjax:timeout` | ✔︎ | `xhr, options` | 页面将会在`options.timeout`之后直接发起请求刷新页面，除非取消pjax |
| `pjax:error` | ✔︎ | xhr, textStatus, error, options | ajax 错误，将会请求刷新页面，除非取消pjax|
| `pjax:complete` |  | xhr, textStatus, options | 不管结果是什么，在ajax后，都触发 |
| `pjax:end` |  | `xhr, options` |  |

**生命周期在浏览器返回或前进时触发**

| event | cancel | arguments | notes |
| :- | :- | :- | :- |
| `pjax:popstate` |  |  | 事件方向(前进，后退)属性: "back"/"forward" |
| `pjax:start` |  | `null, options` | 替换内容前 |
| `pjax:beforeReplace` |  | `contents, options` | 从cache中读取内容后，替换html前 |
| `pjax:end` |  | `null, options` | 替换内容后 |
