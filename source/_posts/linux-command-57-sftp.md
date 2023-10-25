---
title: '每天一个linux命令（57）: sftp'
enlink: 'linux-command-57-sftp'
date: 2017-03-05 16:29:38
categories:
- 运维
tags:
- linux命令
---
sFTP（安全文件传输程序）是一种安全的交互式文件传输程序，其工作方式与 FTP（文件传输协议）类似。 然而，sFTP 比 FTP 更安全；它通过加密 SSH 传输处理所有操作。

它可以配置使用几个有用的 SSH 功能，如公钥认证和压缩。 它连接并登录到指定的远程机器，然后切换到交互式命令模式，在该模式下用户可以执行各种命令。

在本文中，我们将向你展示如何使用 sFTP 上传/下载整个目录（包括其子目录和子文件）。

## How to use
默认情况下，SFTP 协议采用和 SSH 传输协议一样的方式建立到远程服务器的安全连接。虽然，用户验证使用类似于 SSH 默认设置的密码方式，但是，建议创建和使用 SSH 无密码登录，以简化和更安全地连接到远程主机。

要连接到远程 sftp 服务器，如下建立一个安全 SSH 连接并创建 SFTP 会话：
```bash
$ sftp root@server
```
登录到远程主机后，你可以如下运行交互式的 sFTP 命令：
```bash
sftp> ls            #列出服务器文件列表
sftp> lls           #列出本地文件列表
sftp> pwd           #当前服务器上路径
sftp> lpwd          #当前本地路径
sftp> cd img        #切换服务器路径
sftp> lcd img       #切换本地路径
sftp> mkdir img     #在服务器上创建一个目录
sftp> lmkdir img    #在本地创建一个目录
```
## 上传文件
```bash
sftp> put readme.md #上传单个文件
sftp> mput *.xls    #上传多个文件
```

## 下载文件
```bash
sftp> get readme.md #下载单个文件
sftp> mget *.xls    #下载多个文件
```

## 上传文件夹
使用`put -r` .但是远程服务器要提前创建一个相同名称的目录; `-r` 递归复制子目录和子文件
```bash
sftp> mkdir img
sftp> put -r img
```
要保留修改时间、访问时间以及被传输的文件的模式，可使用 `-p` 。
```bash
sftp> put -pr img
```

## 下载文件夹
```bash
sftp> get -r img
```

## 退出
```bash
sftp> bye
或
sftp> exit
或
ctrl + d
```
