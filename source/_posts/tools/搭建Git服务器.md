---
title: 搭建Git服务器
permalink: set-up-git-server-on-vps
date: 2016-10-22 20:08:04
categories:
- 工具
tags:
- Git
---
最近由于准备在公司的服务器上面搭建静态博客（Hexo），然后需要先搭建一个git服务器作为转接，整个过程看似顺利，十几分钟就搭建完成，不过最后在验证这块卡了两个小时，在此记录下来，供准备搭建git服务器的新手小伙伴们借鉴。
<!--more -->
## 搭建git服务器
通过ssh链接到服务器，开始进行操作
### 第一步
在服务器上安装 git
```bash
$ sudo apt-get install git
```
### 第二步
创建 git 用户，用来运行git服务
```bash
$ sudo adduser git
```
### 第三步
创建证书，免密码登录：
收集所有需要登录的用户的公钥（`id_rsa.pub`）文件，把所有公钥导入到 `/home/git/.ssh/authorized_keys` 文件内，一行一个。
如果个人的git中的公钥已经连接了其他服务器如：github，可以参考 {% post_link computer-mutiple-github-account %}
>**注意：一定要通过下面的命令将该文件其他用户的所有权限移除，否则会出现文章尾部问题**

```bash
$ chmod 600 authorized_keys
```
### 第四步
初始化git仓库
```bash
$ git init --bare test.git
```
git创建一个裸仓库，裸仓库没有工作区，因为服务器上的git仓库纯粹为了共享，所有不能让用户直接登录到服务器上去改工作区，并且服务器的git仓库通常以 `.git` 结尾。然后，修改owner改为git：
```bash
$ sudo chown -R git:git test.git
```
### 第五步
禁用shell登录：
处于安全的考虑，第二步创建的git用户不允许登录shell，这可以通过编辑 `/etc/passwd` 文件完成。
```bash
git:x:1003:1003::/home/git:/bin/bash
```
改为
```bash
git:x:1003:1003::/home/git:/usr/bin/git-shell
```
这样，git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git用户指定的git-shell每次一登录就自动退出。
### 第六步
克隆远程仓库：
现在，可以通过git clone命令克隆远程仓库了，在各自的电脑上运行：
```bash
$ git clone git@server:/home/git/test.git
```
如果服务器的ssh端口不是默认的22的话，比如说6789，可以这样写：
```bash
$ git clone ssh://git@server:6789/home/git/test.git
```
## 问题来了
本来根据文档，根据广大猿友的经验，我的搭建之路已经完成了，然后最后一步出现了问题。每次跟服务器进行交互(clone,pull,push)，都让我输入git的密码，也就是说，我配置的ssh没有生效。然后就开始到处找原因，重新生成rsa，提升authorized_keys权限，重新创建服务器git账户，重新。。。。。

翻遍了 Stack Overflow 和 segmentfault ,两个小时过去了，问题仍然没有进展，这么简单的东西，问题到底出在哪里。

就在心灰意冷，准备放弃的时候，不知道是哪里来的灵感，准备把 authorized_keys 文件的其他用户的权限删掉，然后就能用了，后就能用了，就能用了，能用了，用了，了～～～～，命令如下，不想多说话，我想静静。
```bash
$ chmod 600 authorized_keys
```
