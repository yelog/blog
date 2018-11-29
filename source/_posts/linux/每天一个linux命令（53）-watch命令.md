---
title: '每天一个linux命令（53）: watch'
permalink: 'linux-command（53）-watch'
date: 2017-01-21 10:12:30
categories:
- 运维
tags:
- linux命令
---
　　watch是一个非常实用的命令，基本所有的Linux发行版都带有这个小工具，如同名字一样，watch可以帮你监测一个命令的运行结果，省得你一遍遍的手动运行。在Linux下，watch是周期性的执行下个程序，并全屏显示执行结果。你可以拿他来监测你想要的一切命令的结果变化，比如 tail 一个 log 文件，ls 监测某个文件的大小变化，看你的想象力了！
<!--more -->
### 命令格式
```bash
$ watch[参数][命令]
```
### 命令功能
　　可以将命令的输出结果输出到标准输出设备，多用于周期性执行命令/定时执行命令
### 命令参数
| 参数 | 描述 |
| :- | :- |
| -n或--interval | watch缺省每2秒运行一下程序，可以用-n或-interval来指定间隔的时间 |
| -d或--differences | watch 会高亮显示变化的区域 |
| -d=cumulative | 会把变动过的地方(不管最近的那次有没有变动)都高亮显示出来 |
| -t 或-no-title | 会关闭watch命令在顶部的时间间隔,命令，当前时间的输出 |
| -h, --help | 查看帮助文档 |

### 使用实例
**`例一`：每隔一秒高亮显示网络链接数的变化情况**
```bash
$ watch -n 1 -d netstat -ant
```
>**说明：**
其它操作：
切换终端： Ctrl+x
退出watch：Ctrl+g (deepin系统没效果，只能使用Ctrl+c退出了)

**`例二`：每隔一秒高亮显示http链接数的变化情况**
```bash
# 每隔一秒高亮显示http链接数的变化情况。 后面接的命令若带有管道符，需要加''将命令区域归整。
$ watch -n 1 -d 'pstree|grep http'
```
**`例三`：实时查看模拟攻击客户机建立起来的连接数**
```bash
$ watch 'netstat -an | grep:21 | \ grep<模拟攻击客户机的IP>| wc -l'
```
**`例四`：监测当前目录中 scf' 的文件的变化**
```bash
$ watch -d 'ls -l|grep scf'
```
**`例五`：10秒一次输出系统的平均负载**
```bash
$ watch -n 10 'cat /proc/loadavg'
```
