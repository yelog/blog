---
title: 如何在linux中搭建ftp服务
permalink: linux-ftp
date: 2017-03-06 08:47:48
categories:
- 运维
tags:
- ftp
---
## 什么是 FTP
`FTP` 是文件传输协议File Transfer Protocol的缩写。顾名思义，FTP用于计算机之间通过网络进行文件传输。你可以通过FTP在计算机账户间进行文件传输，也可以在账户和桌面计算机之间传输文件，或者访问在线软件归档。但是，需要注意的是多数的FTP站点的使用率非常高，可能需要多次重连才能连接上。

FTP地址和HTTP地址（即网页地址）非常相似，只是FTP地址使用 `ftp://前缀而不是http://`

## FTP 服务器是什么
通常，拥有FTP地址的计算机是专用于接收FTP连接请求的。一台专用于接收FTP连接请求的计算机即为FTP服务器或者FTP站点。

现在，我们来开始一个特别的冒险，我们将会搭建一个FTP服务用于和家人、朋友进行文件共享。在本教程，我们将以vsftpd作为ftp服务。

VSFTPD是一个自称为最安全的FTP服务端软件。事实上VSFTPD的前两个字母表示“非常安全的very secure”。该软件的构建绕开了FTP协议的漏洞。

尽管如此，你应该知道还有更安全的方法进行文件管理和传输，如：SFTP（使用OpenSSH）。FTP协议对于共享非敏感数据是非常有用和可靠的。

## 安装 VSFTP
```bash
#使用 rpm 安装
$ dnf -y install vsftpd
#使用 deb 安装
$ sudo apt-get install vsftpd
#在 Arch 中安装
$ sudo pacman -S vsftpd
```

## 配置 FTP 服务
多数的VSFTPD配置项都在/etc/vsftpd.conf配置文件中。这个文件本身已经有非常良好的文档说明了，因此，在本节中，我只强调一些你可能进行修改的重要选项。使用man页面查看所有可用的选项和基本的 文档说明：
```bash
$ man vsftpd.conf
```
根据文件系统层级标准，FTP共享文件默认位于/srv/ftp目录中。
**允许上传：**
为了允许ftp用户可以修改文件系统的内容，如上传文件等，“write_enable”标志必须设置为 YES
```xml
write_enable=YES
```
**允许本地（系统）用户登录：**
为了允许文件/etc/passwd中记录的用户可以登录ftp服务，“local_enable”标记必须设置为YES。
```xml
local_enable=YES
```
**匿名用户登录**
下面配置内容控制匿名用户是否允许登录：
```xml
# 允许匿名用户登录
anonymous_enable=YES
# 匿名登录不需要密码（可选）
no_anon_password=YES
# 匿名登录的最大传输速率，Bytes/second（可选）
anon_max_rate=30000
# 匿名登录的目录（可选）
anon_root=/example/directory/
```
**根目录限制（Chroot Jail）**
（ LCTT 译注：chroot jail是类unix系统中的一种安全机制，用于修改进程运行的根目录环境，限制该线程不能感知到其根目录树以外的其他目录结构和文件的存在。详情参看[chroot jail](https://zh.wikipedia.org/wiki/Chroot)）

有时我们需要设置根目录（chroot）环境来禁止用户离开他们的家（home）目录。在配置文件中增加/修改下面配置开启根目录限制（Chroot Jail）:
```xml
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list
```
“chroot_list_file”变量指定根目录限制所包含的文件/目录（ LCTT 译注：即用户只能访问这些文件/目录）

最后你必须重启ftp服务，在命令行中输入以下命令：
```bash
$ sudo systemctl restart vsftpd
```
到此为止，你的ftp服务已经搭建完成并且启动了。
