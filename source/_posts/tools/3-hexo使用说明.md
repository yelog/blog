---
title: 3-hexo使用说明
permalink: 3-hexo-instruction
date: 2017-03-23 15:13:47
top: 2
categories:
- 工具
- hexo
tags:
- hexo
- 3-hexo
---
>下面如果没有特殊说明， `_config.yml` 都指主题配置文件，即 `3-hexo` 目录下

## 一、初始化博客下 _config.yml

### 1.1 国际化

```yaml
language: zh-CN #支持 zh-CN、en
```

### 1.2 关掉 hexo 自带的代码高亮

主题内置了主题高亮，所以需要禁用 hexo 自带的高亮

```yaml
highlight:
	enable: false
```



## 二、功能相关
### 2.1 自定义首页

可查看这篇文章： {% post_link 3-hexo-homepage  %}

### 2.3 blog快捷键
可查看这篇文章： {% post_link 3-hexo-shortcuts %}

### 2.4 多作者模式
可查看这篇文章： {% post_link 3-hexo-multiple-author %}

### 2.5 开启`关于`页面
1. 在 `hexo` 根目录执行以下，创建 `关于` 页面
```bash
hexo new page "about"
```
2. 位置： `source/aoubt/index.md` ，根据需要进行编辑。
3. 在主题中开启显示：修改主题根目录 `_config.yml` 中的 `about` 的 `on` 为 `true`，如下所示
```yml
menu:
  about:  # '关于' 按钮
    on: true # 是否显示
    url: /about  # 跳转链接
    type: 1 # 跳转类型 1：站内异步跳转 2：当前页面跳转 3：打开新的tab页
```

### 2.6 添加音乐插件
{% post_link  3-hexo-add-music %}

### 2.7 配置评论系统
{% post_link  3-hexo-comment %}

## 三、样式设置
### 3.1 代码高亮
首先要关闭hexo根目录下`_config.yml`中的高亮设置：
```yaml
highlight:
  enable: false
```
配置主题下`_config.yml`中的高亮设置：
可以根据提示，配置喜欢的高亮主题
```yaml
highlight:
  on: true # true开启代码高亮
  lineNum: true # true显示行号
  theme: darcula
# 代码高亮主题,效果可以查看 https://highlightjs.org/static/demo/
# 支持主题：
# sublime : 参考sublime的高亮主题
# darcula : 参考idea中的darcula的主题
# atom-dark : 参考Atom的dark主题
# atom-light : 参考Atom的light主题
# github : 参考GitHub版的高亮主题
# github-gist : GitHub-Gist主题
# brown-paper : 牛皮纸效果
# gruvbox-light : gruvbox的light主题
# gruvbox-dark ： gruvbox的dark主题
# rainbow :
# railscasts :
# sunburst :
# kimbie-dark :
# kimbie-light :
# school-book : 纸张效果
```
### 3.2 MathJax数学公式
修改 `_config.yml`
```yaml
# MathJax 数学公式支持
mathjax:
  on: true #是否启用
  per_page: false # 若只渲染单个页面，此选项设为false，页面内加入 mathjax: true
```
考虑到页面的加载速度，支持渲染单个页面。
设置 `per_page: false` ,在需要渲染的页面内 加入 `mathjax: true`

> **`注意: `**
由于hexo的MarkDown渲染器与MathJax有冲突，可能会造成矩阵等使用不正常。所以在使用之前需要修改两个地方
编辑 `node_modules\marked\lib\marked.js` 脚本

1. 将451行 ，这一步取消了对 `\\,\{,\}` 的转义(escape)
```js
escape: /^\\([\\`*{}\[\]()# +\-.!_>])/,
改为
escape: /^\\([`*\[\]()# +\-.!_>])/,
```
2. 将459行，这一步取消了对斜体标记 `_` 的转义
```js
em: /^\b_((?:[^_]|__)+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
改为
em:/^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```

### 3.3 表格样式
目前提供了3中样式，修改 `_config.yml`
```yaml
table: green_title
# table 的样式
# 为空时类似github的table样式
# green 绿色样式
# green_title 头部为青色的table样式
```
### 3.4 文章列表的hover样式
鼠标移入的背景色和文字颜色变动，设置 `_config.yml`
```yaml
#文章列表 鼠标移上去的样式, 为空时使用默认效果
article_list:
  hover:
    background: '#e2e0e0'  # 背景色:提供几种：'#c1bfc1'  '#fbf4a8'
    color:     # 文字颜色 提供几种：'#ffffff'
# 注意：由于颜色如果包含#，使用单引号 ' 引起来
```
### 3.5 开启字数统计
1. 开启此功能需先安装插件，在 hexo根目录 执行 `npm i hexo-wordcount --save`
2. 修改 `_config.yml`

```yaml
word_count: true
```
### 3.6 更换头像
两种方式：
1. 替换 `source/img/avatar.jpg` 图片。
2. 修改 `_config.yml` 中头像的配置记录

```yaml
# 你的头像url
avatar: /img/avatar.jpg
favicon: /img/avatar.jpg
```

### 3.7 设置链接图标

> 如果需要自定义图标可以看这篇文章 {% post_link 3-hexo-add-icon %}

如下，如果没有连接，则不展示图标。
```yaml
#链接图标，链接为空则不显示
link:
  rss: /atom.xml
  github: https://github.com/yelog
  facebook: https://www.facebook.com/faker.tops
  twitter:
  linkedin:
  instagram:
  reddit: https://www.reddit.com/user/yelog/
  weibo: http://weibo.com/u/2307534817
  email: jaytp@qq.com
```

## 四、排序及置顶

### 4.1 分类排序

默认按照首字母正序排序，由于中文在 `nodejs` 环境下不能按照拼音首字母排序，所以添加了自定义排序方式，在主题下 `_config.yml` 中找到如下配置，`category.sort`则是定义分类顺序的。

**规则：**在 `sort`中定义的 `category` 比 没有在 `sort` 中定义的更靠前

```yaml
# 文章分类设置
category:
  num: true # 分类显示文章数
  sub: true # 开启多级分类
  sort:
    - 读书
    - 大前端
    - 后端
    - 数据库
    - 工具
    - 运维
```



### 4.2 文章排序

> 2020-05-20 更新：无需安装插件或修改源码，主题以内置排序算法

文章列表默认按照创建时间（如下文章内定义的`date`）倒序。

使用 `top` 将会置顶文章，多个置顶文章时，`top` 定义的值越大，越靠前。

```yml
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

## 五、关于写文章
### 5.1 如何写
每篇文章最好写上文集和标签，方便筛选和查看。
一般推荐一篇文章设置一个文集，一个或多个标签
`categories`:文集，为左侧列表
`tags`:标签，通过#来筛选
例如 本篇文章的设置
```yaml
---
title: 3-hexo使用说明
date: 2017-03-23 15:13:47
categories:
- 工具
tags:
- hexo
- 3-hexo
---
```
### 5.2 写作
1.设置模板，blog根目录 `scaffolds/post.md`
加入categories,tags等，这样以后通过 `hexo new` 生成的模板就不用写这两个单词了。
当然，你也可以写入任何你每个文章中都会有的部分。

```yaml
---
title: {{ title }}
date: {{ date }}
categories:
tags:
---
```

## 六、技巧
### 6.1 快捷命令
其实就通过alias，触发一些命令的集合
在 `~/.bashrc` 文件中添加

```bash
alias hs='hexo clean && hexo g && hexo s'  #启动本地服务
alias hd='hexo clean && hexo g && hexo d'  #部署博客
```
甚至你也可以加入备份文章的命令，可以自由发挥。

### 6.3 博客备份（快捷命令升级版）
为了保证我们写的文章不丢失、快速迁移博客，都需要备份我们的blog。
1. 博客根目录，执行 `git init` 创建 git 仓库。
2. 在 github（或其他托管平台、自建远程仓库等） 创建仓库并和本地仓库建立联系。
3. 在 `~/.bashrc` 文件中添加
```bash
alias hs='hexo clean && hexo g && hexo s'
alias hd='hexo clean && hexo g && hexo d && git add . && git commit -m "update" && git push -f'
```

这样，我们在执行 `hd` 进行部署时，就一同将博客进行备份了
