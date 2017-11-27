---
title: '每天一个linux命令（3）: pwd'
permalink: 'linux-command（3）-pwd'
date: 2016-12-03 09:15:52
categories:
- 运维
tags:
- linux命令
---
　　Linux中用 pwd 命令来查看”当前工作目录“的完整路径。 简单得说，每当你在终端进行操作时，你都会有一个当前工作目录。
　　在不太确定当前位置时，就会使用pwd来判定当前目录在文件系统内的确切位置。
<!--more -->
### 命令格式
```bash
$ pwd [选项]
```
### 命令功能
　　查看”当前工作目录“的完整路径
### 常用参数
一般情况下不带任何参数
如果目录是链接时：
格式：pwd -P 显示出实际路径，而非使用链接（link） 的路径
### 实用实例
**`例一`：用 pwd 命令查看当前工作目录的完整路径**
```bash
$ pwd
/home/faker
```
**`例二`：目录连接链接时，pwd -P  显示出实际路径，而非使用连接（link）路径；pwd显示的是连接路径**
```bash
#目录为链接时，输出链接路径
$ pwd -L
#目录为链接时，输出物理路径
$ pwd -P
/home/faker
```
**`例三`：当前目录被删除了，而pwd命令仍然显示那个目录**
```bash
$ cd /opt/soft
$ rm ../soft -rf
$ pwd
/opt/soft
$ /bin/pwd
/bin/pwd: couldnt find directory entry in “..” with matching i-node
/home/faker
```
