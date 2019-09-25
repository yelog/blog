---
title: Hexo创建404页面
permalink: hexo-create-404-page
date: 2017-02-25 15:18:39
categories:
- 工具
- hexo
tags:
- hexo
---
对于github page来说，只要在根目录又404.html，当页面找不到时，就会被转发到/404.html页面，所以我们只要更改这个页面，就可以实现自定义404页面了。

但是我们通常会需要与本主题相符的404页面。那我们就需要以下操作

### 新建404页面
1. 进入 Hexo 所在文件夹，输入 `hexo new page 404` ;
2. 打开刚新建的页面文件，默认在 Hexo 文件夹根目录下 /source/404/index.md；
3. 在顶部插入一行，写上 `permalink: /404`，这表示指定该页固定链接为 `http://"主页"/404.html`

```xml
---
title: 404
permalink: /404
date: 2016-09-27 11:31:01
---
---
## 页面未找到！
```

 ### 效果
 > <http://yelog.org/举个404例子>

 ![404](http://img.saodiyang.com/FjSPGVPAu_7d0aMPqErpI1HN_985.png)
