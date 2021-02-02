---
title: Hexo+Git服务器搭建blog
enlink: hexo-git-server-blog
date: 2016-10-23 09:24:03
categories:
- 工具
- hexo
tags:
- hexo
- Git
---
博主最近在服务器上搭建Hexo发布平台，感觉整个搭建过程和搭建思想蛮有意思，在此记录一下，供猿友参考
Hexo 是一个快速，简单，功能强大，主题社区特别庞大的开源blog框架-》[官网](https://hexo.io/)
本次搭建是通过在服务器上搭建Git服务器来实现一键发布blog
<!--more -->
## 搭建思路
![图解](http://img.saodiyang.com/FjUDxVksbmykVH1wvQJc4h57wMrQ.jpg)
1. 客户端就是自己的电脑,可以把hexo的静态资源目录当成一个git仓库.
2. 首先配置好远程git仓库,通过 hexo d 将静态网站资源push到远程git仓库
3. git仓库接收到push处理完成后,自动触发post-receive这个钩子.
4. 执行钩子内容,进入到 /var/www/blog 目录(也是一个git仓库),拉取刚才hexo推送到git服务端的静态网站资源.
5. 配置nginx,将80端口映射到 /var/www/blog 目录.
6. 就可以直接通过ip访问到静态blog了

## 搭建过程
### 环境准备
#### 在服务器上安装git并创建git远程仓库 如 `blog.git`
搭建过程移步 {% post_link tools/set-up-git-server-on-vps %}
#### 在 `_config.yml` 中配置git服务器
```xml
deploy:
    type: git
    repo: git@server:/home/git/blog.git
    branch: master
```
如果ssh端口不是默认的22的话，如下配置,8080改为自己服务器上ssh端口
```xml
deploy:
    type: git
    repo: ssh://git@server:8080/home/git/blog.git
    branch: master
```
#### 配置nginx
现在已经可以使用 `hexo d` 将hexo中的生成的静态资源发送到远程服务器中，接下来我们要配置nginx来配置静态web。
安装过程可以自行Google，在此只说明nginx如何配置静态web
首先创建一个目录作为存放web资源（hexo生成的）的目录，如： `/var/www/blog`
```bash
cd /var/www
#创建blog目录，并克隆blog.git仓库的内容
git clone /home/git/blog.git blog
```
找到 `nginx.conf` 添加以下信息
```xml
server {
    listen 80;
    charset utf-8;
    root   /var/www/blog;
    index  index.htm index.html index.jsp;
}
```
```bash
#重启并加载配置文件
$ nginx -s reload
```
#### 配置git服务器hooks
这个钩子的作用是，当git服务器接受客户端push完成更新，执行此文件内容
```bash
#创建并编辑post-receive
$ vim blog.git/hooks/post-receive
```
内容如下
```bash
#!/bin/sh
unset GIT_DIR #还原环境变量，否则会拉不到代码
cd /var/www/blog
git pull origin master #拉取最新代码
```
#### 测试效果
在本地的hexo下执行 `hexo d`
查看 /var/www/blog文件夹内的内容也发生变化
