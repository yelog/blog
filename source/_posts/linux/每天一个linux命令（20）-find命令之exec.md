---
title: '每天一个linux命令（20）: find命令之exec'
permalink: 'linux-command（20）-find命令之exec'
date: 2016-12-20 10:47:32
categories:
- 运维
tags:
- linux命令
---
　　find是我们很常用的一个Linux命令，但是我们一般查找出来的并不仅仅是看看而已，还会有进一步的操作，这个时候exec的作用就显现出来了
<!--more -->
### 命令介绍
　　`-exec`  参数后面跟的是command命令，它的终止是以;为结束标志的，所以这句命令后面的分号是不可缺少的，考虑到各个系统中分号会有不同的意义，所以前面加反斜杠。

　　`{}`  花括号代表前面find查找出来的文件名。

　　使用find时，只要把想要的操作写在一个文件里，就可以用exec来配合find查找，很方便的。在有些操作系统中只允许-exec选项执行诸如ls或ls -l这样的命令。大多数用户使用这一选项是为了查找旧文件并删除它们。建议在真正执行rm命令删除文件之前，最好先用ls命令看一下，确认它们是所要删除的文件。 exec选项后面跟随着所要执行的命令或脚本，然后是一对儿{ }，一个空格和一个\，最后是一个分号。为了使用exec选项，必须要同时使用print选项。如果验证一下find命令，会发现该命令只输出从当前路径起的相对路径及文件名。
### 使用实例
**`例一`：ls -l命令放在find命令的-exec选项中**
```bash
# find命令匹配到了当前目录下的所有普通文件，并在-exec选项中使用ls -l命令将它们列出
$ find . -type f -exec ls -l {} \;
```
**`例二`：在目录中查找更改时间在n日以前的文件并删除它们**
```bash
$ find . -type f -mtime +14 -exec rm {} \;
```
**`例三`：在目录中查找更改时间在n日以前的文件并删除它们，在删除之前先给出提示**
```bash
$ find . -name "*.log" -mtime +5 -ok rm {} \;
```
**`例四`：-exec中使用grep命令**
```bash
$ find /etc -name "passwd*" -exec grep "root" {} \;
```
**`例五`：查找文件移动到指定目录**
```bash
$ find . -name "*.log" -exec mv {} .. \;
```
**`例六`：用exec选项执行cp命令**
```bash
$ find . -name "*.log" -exec cp {} test3 \;
```
