---
title: 为Hexo添加RSS和Sitemap
enlink: Hexo-RSS-Sitemap
date: 2017-03-14 09:44:29
categories:
- 工具
- hexo
tags:
- hexo
---
## 添加RSS
使用`RSS`是为自己的blog提供订阅功能。
### 1.用`npm`安装插件
```bash
$ npm install hexo-generator-feed --save
```
### 2.配置根目录_config.yml
```yml
# Extensions
## Plugins: http://hexo.io/plugins/
#RSS订阅
plugin:
- hexo-generator-feed
#Feed Atom
feed:
type: atom
path: atom.xml
limit: 20
```
### 3.验证配置是否成功
执行 `hexo g`，查看一下public目录下，如果有 `atom.xml` 文件，则表明配置成功。
### 4.显示RSS图标
这里以3-hexo主题为例，给rss添加链接`/atom.xml`修改`/themes/3-hexo/_config.yml`
```xml
link:
  rss: /atom.xml
```
### 5.效果
**链接图标：**
![图标](https://cdn.jsdelivr.net/gh/yelog/assets/images/FlmC3WWi9jzgVRdSKJ2_li5UHVsr.png)
**链接地址效果**
![效果](https://cdn.jsdelivr.net/gh/yelog/assets/images/FuTy1C-xSgdTTOZch_UH1355NAs9.png)

## 添加Sitemap
Sitemap，网站地图，是网站优化中重要的一环，无论是对于访问者还是对于搜索引擎。
### 1.用`npm`安装插件
```bash
$ npm install hexo-generator-sitemap --save
```
### 2.配置根目录_config.yml
```xml
plugin:
- hexo-generator-feed
- hexo-generator-sitemap
```
### 3.验证配置是否成功
执行 `hexo g`，查看一下public目录下，如果有 `sitemap.xml` 文件，则表明配置成功。

### 4.效果
访问 /sitemap.xml 就能看到生成的站点地图了
![站点地图效果](https://cdn.jsdelivr.net/gh/yelog/assets/images/FtFLZ2CSfYa_IKLS-a9ymvKvaztp.png)
