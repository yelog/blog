---
title: 3-hexo 添加音乐插件
permalink: 3-hexo-add-music
date: 2019-10-08 10:44:30
categories:
- 工具
- hexo
tags:
- hexo
- 3-hexo
---

## 网易云音乐
### 1. 复制网易云音乐插件代码

前往网易云音乐官网，搜索一个作为背景音乐的歌曲，并进入播放页面，点击 **生成外链播放器**
![生成外链播放器](https://i.loli.net/2019/10/08/RgSUj1i8vXNk5IP.png)

设置好想要显示的样式后，复制 html 代码

![](https://i.loli.net/2019/10/08/rbHRZEoB4mzip75.png)

最好外层在加一个 `div`，如下，可直接将上一步复制的 `iframe` 替换下方里面的 `iframe`
```html
<div style="position:absolute; bottom: 0; right: 0;">
    <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=38592976&auto=1&height=66"></iframe>
</div>
```
### 2. 将插件引入到主题中

将上一步加过 `div` 的代码粘贴到主题下 `layout/_partial/footer.ejs` 的最后面
![效果](https://i.loli.net/2019/10/08/FRJOKxLECcvinmf.png)
### 3. 调整位置

默认给的样式是显示在右下角，可以通过调整上一步粘贴的 `div` 的 `style` 中 `bottom` 和 `right` 来调整位置。
