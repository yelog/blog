---
title: '每天一个linux命令（49）: cal'
permalink: 'linux-command（49）-cal'
date: 2017-01-17 09:38:32
categories:
- 运维
tags:
- linux命令
---
　　cal命令可以用来显示公历（阳历）日历。公历是现在国际通用的历法，又称格列历，通称阳历。“阳历”又名“太阳历”，系以地球绕行太阳一周为一年，为西方各国所通用，故又名“西历”。
<!-- more -->
### 命令格式
```bash
$ cal [参数][月份][年份]
```
### 命令功能
用于查看日历等时间信息，如只有一个参数，则表示年份(1-9999)，如有两个参数，则表示月份和年份

### 命令参数
| 参数 | 描述 |
| :- | :- |
| -1 | 显示一个月的月历 |
| -3 | 显示系统前一个月，当前月，下一个月的月历 |
| -s | 显示星期天为一个星期的第一天，默认的格式 |
| -m | 显示星期一为一个星期的第一天 |
| -j | 显示在当年中的第几天（一年日期按天算，从1月1号算起，默认显示当前月在一年中的天数） |
| -y | 显示当前年份的日历 |

### 使用实例
**`例一`：显示当前月份日历**
```bash
$ cal
```
![日历](http://img.saodiyang.com/Fo959HUHU7DyFEaargV5_4n6nsNi.png)
**`例二`：显示指定月份的日历**
```bash
$ cal 6 2016
```
![2016年6月](http://img.saodiyang.com/Fhjcoswylnxplt5CjFHlfb55va4M.png)
**`例三`：显示2016年的日历**
```bash
$ cal -y 2016
$ cal 2016
```
![2016年日历](http://img.saodiyang.com/Fnloz8dl1VKyVgn32hjCM0lPGkJI.png)
**`例四`：显示自1月1日的天数**
```bash
$ cal -j
```
![本年的第几天](http://img.saodiyang.com/FsQNZEvW5UEJHd3IV2sF19OlOjvs.png)

**`例五`：星期一显示在第一列**
```bash
$ cal -m
```
![本机deepin不支持这个参数，登陆到服务器截了一张图==](http://img.saodiyang.com/Ft9YOB5-L4fauP9TWqQLCpAOYwlY.png)
