---
title: '每天一个linux命令（21）: find命令之xargs'
permalink: 'linux-command（21）-find命令之xargs'
date: 2016-12-21 11:08:01
categories:
- 运维
tags:
- linux命令
---
　　在使用 find命令的-exec选项处理匹配到的文件时， find命令将所有匹配到的文件一起传递给exec执行。但有些系统对能够传递给exec的命令长度有限制，这样在find命令运行几分钟之后，就会出现溢出错误。错误信息通常是“参数列太长”或“参数列溢出”。这就是xargs命令的用处所在，特别是与find命令一起使用。
<!--more -->

　　find命令把匹配到的文件传递给xargs命令，而xargs命令每次只获取一部分文件而不是全部，不像-exec选项那样。这样它可以先处理最先获取的一部分文件，然后是下一批，并如此继续下去。  

　　在有些系统中，使用-exec选项会为处理每一个匹配到的文件而发起一个相应的进程，并非将匹配到的文件全部作为参数一次执行；这样在有些情况下就会出现进程过多，系统性能下降的问题，因而效率不高； 而使用xargs命令则只有一个进程。另外，在使用xargs命令时，究竟是一次获取所有的参数，还是分批取得参数，以及每一次获取参数的数目都会根据该命令的选项及系统内核中相应的可调参数来确定。
### 使用实例
**`例一`：查找系统中的每一个普通文件，然后使用xargs命令来测试它们分别属于哪类文件**
```bash
$ find . -type f -print | xargs file
```
**`例二`：在整个系统中查找内存信息转储文件(core dump) ，然后把结果保存到/tmp/core.log 文件中**
```bash
$ find / -name "core" -print | xargs echo "" >/tmp/core.log
```
**`例三`：在当前目录下查找所有用户具有读、写和执行权限的文件，并收回相应的写权限**
```bash
$ find . -perm -7 -print | xargs chmod o-w
```
**`例四`：用grep命令在所有的普通文件中搜索hostname这个词**
```bash
$ find . -type f -print | xargs grep "hostname"
```
**`例五`：用grep命令在当前目录下的所有普通文件中搜索hostnames这个词**
```bash
# \用来取消find命令中的*在shell中的特殊含义
$ find . -name \* -type f -print | xargs grep "hostnames"
```
**`例六`：使用xargs执行mv**
```bash
$ find . -name "*.log" | xargs -i mv {} test4
```
**`例七`：find后执行xargs提示xargs: argument line too long解决方法**
```bash
# -l1是一次处理一个；-t是处理之前打印出命令
$ find . -type f -atime +0 -print0 | xargs -0 -l1 -t rm -f
```
**`例八`：使用-i参数默认的前面输出用{}代替，-I参数可以指定其他代替字符，如例子中的[]**
```bash
$  find . -name "file" | xargs -I [] cp [] ..
```
**`例九`：xargs的-p参数的使用**
```bash
# -p参数会提示让你确认是否执行后面的命令,y执行，n不执行
$ find . -name "*.log" | xargs -p -i mv {} ..
```
