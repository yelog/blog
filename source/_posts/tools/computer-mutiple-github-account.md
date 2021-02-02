---
title: 一个客户端设置多个github账号
enlink: computer-mutiple-github-account
date: 2016-09-30 16:42:48
categories:
- 工具
- git
tags: [Git,GitHub]
---
最近想要使用自己的GitHub搭建Hexo博客，同时还要使用工作的GitHub开发项目，所以在网上找寻了一些文章，在此将自己的搭建过程记录一下。
<!--more -->
## 前期工作

两个GitHub账号（假设两个账号为one,two）
取消Git全局设置

```bash
$ git config --global --unset user.name
$ git config --global --unset user.email
```
## SSH配置
生成`id_rsa`私钥，`id_rsa.pub`公钥。one可以直接回车，默认生成 id_rsa 和 id_rsa.pub 。

```bash
$ ssh-keygen -t rsa -C "one@xx.com"
```
添加two会出现提示输入文件名，输入与默认配置不一样的文件名，如：id_rsa_two。

```bash
$ cd ~/.ssh
$ ssh-keygen -t rsa -C "two@126.com"  #  之后会提示输入文件名
```
GitHub添加公钥 id_rsa.pub 、 id_rsa_two.pub，分别登陆one,two的账号，在 Account Settings 的 SSH Keys 里，点 `Add SSH Keys` ，将公钥(.pub文件)中的内容粘贴到 Key 中，并输入 Title。
添加 ssh Key

```bash
$ ssh-add ~/.ssh/id_rsa
$ ssh-add ~/.ssh/id_rsa_two
```
可以在添加前使用下面命令删除所有的 key

```bash
$ ssh-add -D
```
最后可以通过下面命令，查看 key 的设置

```bash
$ ssh-add -l
```
## 修改ssh config文件
```bash
$ cd ~/.ssh/
$ touch config
```
打开 .ssh 文件夹下的 `config` 文件，进行配置

```yml
#  default
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa

#  two
Host two.github.com  #  前缀名可以任意设置
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_two
```
- 这里必须采用这样的方式设置，否则 push 时会出现以下错误:

>ERROR: Permission to two/two.github.com.git denied to one.

简单分析下原因，我们可以发现 ssh 客户端是通过类似:
```bash
git@github.com:one/one.github.com.git
```

这样的 Git 地址中的 User 和 Host 来识别使用哪个本地私钥的。
很明显，如果 User 和 Host 始终为 git 和 github.com，那么就只能使用一个私钥。
所以需要上面的方式配置，每个账号使用了自己的 Host，每个 Host 的域名做 CNAME 解析到 github.com，这样 ssh 在连接时就可以区别不同的账号了。
```bash
$ ssh -T git@github.com        #  测试one ssh连接
# Hi ***! You've successfully authenticated, but GitHub does not provide shell access.
$ ssh -T git@two.github.com    #  测试two ssh连接
# Hi ***! You've successfully authenticated, but GitHub does not provide shell access.
```
但是这样还没有完，下面还有关联的设置。
## 在Git项目中配置账号关联
可以用 `git init` 或者 `git clone` 创建本地项目
分别在one和two的git项目目录下，使用下面的命令设置名字和邮箱

```bash
$ git config user.name "__name__"            #  __name__ 例如 one
$ git config user.email "__email__"          #  __email__ 例如 one@126.com
```
注意：由于我不知道Hexo怎样配置 局部的config，所以，我将two的config使用全局，而工作目录配置局部。

```bash
$ git config --global user.name "__name__"            #  __name__ 例如 two
$ git config --global user.email "__email__"          #  __email__ 例如 two@126.com
```
查看git项目的配置

```bash
$ git config --list
```
查看 one 的 remote.origin.url=git@github.com:one/one.github.com.git
查看 two 的 remote.origin.url=git@github.com:two/two.github.com.git
由于 one 使用的是默认的 Host ，所以不需要修改，但是 two 使用的是 two.github.com ，则需要进行修改
```bash
$ git remote rm origin
$ git remote add origin git@two.github.com:two/two.github.com.git
```
我在Hexo中的配置（使用two账号）

```yml
deploy:
    type: git
    repo: git@two.github.com:two/two.github.io.git
    branch: master
```
## 上传更改
上面所有的设置无误后，可以修改代码，然后上传了。

```bash
$ git add -A
$ git commit -m "your comments"
$ git push
```
如果遇到warning

>warning: push.default is unset; its implicit value is changing in Git 2.0 from ‘matching’ to ‘simple’. To squelch this messageand maintain the current behavior after the default changes, use…

推荐使用

```bash
$ git config --global push.default simple
```
