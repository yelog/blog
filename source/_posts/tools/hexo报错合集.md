---
title: hexo报错合集
permalink: hexo-error-collection
date: 2017-03-27 19:40:42
categories:
- 工具
tags:
- hexo
---
## hexo server时报错
### FATAL watch ... ENOSPC
日志：2017-03-27 执行 `hexo server` 后报错。
**如图：**
![watch ENOSPC](http://img.xiangzhangshugongyi.com/FqCfDl6mN_Pb1_iH8fRuC5sz4A6o.png)
**分析问题：**
node.js 中 watch 的文件数是有限制的。
**解决问题：**
```bash
$ echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```
