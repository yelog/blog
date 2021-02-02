---
title: 完美替代多说-gitment
enlink: gitment
date: 2017-06-26 12:08:13
categories:
- 工具
- hexo
tags:
- hexo
---
自从多说要停止服务时，就开始关注第三方评论系统，现在的评论系统都有这样或那样的问题，见 {% post_link tools/the-third-comment %} 。忽然看到作者 孙士权 的一片文章 [Gitment：使用 GitHub Issues 搭建评论系统](https://imsun.net/posts/gitment-introduction/) 。

立即就将 `gitment` 集成到 `3-hexo` 主题内。本篇文章只讲在 `3-hexo` 内如何使用，如果想自定义，可以参考上面原文。

## 注册 OAuth Application
[点击此处](https://github.com/settings/applications/new) 来注册一个新的 OAuth Application。其他内容可以随意填写，但要确保填入正确的 callback URL（一般是评论页面对应的域名，如 http://yelog.org）。
![作者是这样填的](http://img.saodiyang.com/FsumVHpCC4h5JxRNg0TiKlMf0b1Y.png)

## 使用 gitment 评论系统
修改主题 `_config.yml`
```xml
gitment:
  on: true  # 启用gitment评论系统
  owner: yelog  # 你的github账号
  repo: yelog.github.io  # 评论issue保存的仓库，我选择保存在blog仓库，也可以新建一个仓库
  client_id: d64ceca0d8a4e8b1f5c9   # 上一步注册后生成的client_id
  client_secret: fb17d5f0aba31372f61a03df707bb20a39a73a06 # 上一步注册后生成的client_secret
```

## 部署并初始化
1.发布 hexo
```bash
$ hexo clear && hexo g && hexo d
```
2.打开发布的blog，登录github账号，并点击 `Initialize Comments`。
![初始化本页的评论](http://img.saodiyang.com/FnO9sHY-eFVXwnLplwsP9NRvityH.png)

3.现在其他人就可以进行评论了

## 感受
整体评论系统做的简洁，整体来说是个不错的系统。
