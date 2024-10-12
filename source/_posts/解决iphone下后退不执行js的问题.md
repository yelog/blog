---
title: 解决iphone下后退不执行js的问题
enlink: iphone-bf-no-run-js
date: 2017-09-21 15:25:32
categories:
- 大前端
tags:
- js
---

## 直接上解决方法

不论页面是否被缓存，都会触发 `pageshow`，所以后退后需要执行的方法可以都放在下面事件内：
```javascript
window.addEventListener('pageshow', function () {
  console.log('on pageshow')
})

`浏览器缓存行为` 的详细介绍可以参考： {% post_link browser-back-forward-cache %}
