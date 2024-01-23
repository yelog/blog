---
title: 3-hexo添加自定义图标
enlink: 3-hexo-add-icon
date: 2020-12-28 22:00:00
categories:
- 工具
- hexo
tags:
- 3-hexo
- hexo
---
## 一、前言

鉴于许多人问过如何添加自定义图标，这里就详细说明一下，以备后人乘凉。

这篇文章主要讲解是从 [iconfont](https://www.iconfont.cn/) 添加图标。

## 二、添加彩色图标

### 2.1 登录并添加图标

访问 [iconfont](https://www.iconfont.cn/)，点击如下图位置登录，可以使用 `Github` 账号登录。

![iconfont 登录](https://cdn.jsdelivr.net/gh/yelog/assets/images/picgo_qiniu20201228230707.png)

登录成功后，搜索合适的图标，然后点击添加到购物车，如下图所示。

![](https://cdn.jsdelivr.net/gh/yelog/assets/images/picgo_qiniu20201228231118.png)

添加了多个后，可以点击右上角的“购物车”，添加到项目，点击加号创建项目，如下图所示。

![](https://cdn.jsdelivr.net/gh/yelog/assets/images/picgo_qiniu20201228231558.png)

添加完成后回到项目页面，找到自己刚刚创建的项目。

> 如果没有到项目页面，可以点击上面菜单进入：资源管理 -> 我的项目

### 2.2 引入 3-hexo 中

点击下载到本地，解压并复制其中的 `iconfont.js` 到项目 `3-hexo/source/js/` 下，并改名 `custom-iconfont.js`。

![](https://cdn.jsdelivr.net/gh/yelog/assets/images/picgo_qiniu20201229003210.png)

在文件 `3-hexo/layout/_partial/meta.ejs` 最后追加下面一行。
```html
<script src="<%=theme.blog_path?theme.blog_path.lastIndexOf("/") === theme.blog_path.length-1?theme.blog_path.slice(0, theme.blog_path.length-1):theme.blog_path:'' %>/js/custom-iconfont.js?v=<%=theme.version%>" ></script>
```

### 2.3 在配置文件中添加生效

修改 `3-hexo/_config.yml` 如下图所示

![](https://cdn.jsdelivr.net/gh/yelog/assets/images/picgo_qiniu20201229001129.png)

完成！
> 图标名如上面的 `gitee` 可以在 网站上修改，如下图所示
![](https://cdn.jsdelivr.net/gh/yelog/assets/images/picgo_qiniu20201229002057.png)

## 三、添加黑白图标
`link.theme=white`
### 3.1 同 2.1
### 3.2 引入 3-hexo 中

点击生成代码，如下图所示。

![](https://cdn.jsdelivr.net/gh/yelog/assets/images/picgo_qiniu20201228231715.png)


复制生成的代码，修改 `font-family` 的值为 `custom-iconfont`，添加到 `3-hexo/source/css/_partial/font.styl` 最后，并写入图标信息，`content` 可以移到图标上进行复制，注意前面斜杠转译和去掉后面的分号。
```css
@font-face {
  font-family: 'custom-iconfont';  /* project id 2298064 */
  src: url('//at.alicdn.com/t/font_2298064_34vkk4c9945.eot');
  src: url('//at.alicdn.com/t/font_2298064_34vkk4c9945.eot?#iefix') format('embedded-opentype'),
  url('//at.alicdn.com/t/font_2298064_34vkk4c9945.woff2') format('woff2'),
  url('//at.alicdn.com/t/font_2298064_34vkk4c9945.woff') format('woff'),
  url('//at.alicdn.com/t/font_2298064_34vkk4c9945.ttf') format('truetype'),
  url('//at.alicdn.com/t/font_2298064_34vkk4c9945.svg#iconfont') format('svg');
}
.icon-gitee:before {
  content: "\e602";
}

.icon-youtubeautored:before {
  content: "\e649";
}
```
### 3.3 在配置文件中添加生效 同2.2

结束！
