---
title: Hexo置顶及排序问题
enlink: hexo-top-sort
date: 2017-02-24 15:50:38
categories:
- 工具
- hexo
tags:
- hexo
---
近期在写3-hexo主题时，发现文章（`site.posts`）排序按照.md文件的创建时间排序，而没有按照文章中的date排序。

这就导致了一个问题，我重装了一次电脑，.md文件通过git备份了，还原回来的时候，md的创建时间都是一样的，所以文章列表就按照文章标题排序了

随后就想起了以前使用yilia主题时，设置过置顶文章。所以做了排序，顺便做了置顶的功能。

>**[@牵猪的松鼠](http://s.amlove.cn/)根据这篇文章写了一个npm插件 [hexo-generator-topindex](https://www.npmjs.com/package/hexo-generator-topindex)
安装插件命令： `npm install hexo-generator-topindex --save`
如果安装插件，可跳过第一部分 [#修改hexo的js代码](#修改hexo的js代码)，直接看第二部分 [#设置置顶](#设置置顶)**

## 修改hexo的js代码

直接上操作，修改`node_modules/hexo-generator-index/lib/generator.js`
```javascript
'use strict';
var pagination = require('hexo-pagination');
module.exports = function(locals){
  var config = this.config;
  var posts = locals.posts;
    posts.data = posts.data.sort(function(a, b) {
        if(a.top && b.top) { // 两篇文章top都有定义
            if(a.top == b.top) return b.date - a.date; // 若top值一样则按照文章日期降序排
            else return b.top - a.top; // 否则按照top值降序排
        }
        else if(a.top && !b.top) { // 以下是只有一篇文章top有定义，那么将有top的排在前面（这里用异或操作居然不行233）
            return -1;
        }
        else if(!a.top && b.top) {
            return 1;
        }
        else return b.date - a.date; // 都没定义按照文章日期降序排
    });
  var paginationDir = config.pagination_dir || 'page';
  return pagination('', posts, {
    perPage: config.index_generator.per_page,
    layout: ['index', 'archive'],
    format: paginationDir + '/%d/',
    data: {
      __index: true
    }
  });
};
```

## 设置置顶
给需要置顶的文章加入top参数，如下
```xml
---
title: 每天一个linux命令
date: 2017-01-23 11:41:48
top: 1
categories:
- 运维
tags:
- linux命令
---
```
如果存在多个置顶文章，top后的参数越大，越靠前。

## 2020-05-20 更新

3-hexo 主题已经内置排序算法，无需上面下载插件或修改源码，可以直接使用，具体可看 {% post_link  3-hexo-instruction %} 中的排序相关内容

## References

　　Netcan 的 [解决Hexo置顶问题](http://www.netcan666.com/2015/11/22/%E8%A7%A3%E5%86%B3Hexo%E7%BD%AE%E9%A1%B6%E9%97%AE%E9%A2%98/)
