---
title: '每天一个linux命令（41）: netstat'
permalink: 'linux-command（41）-netstat'
date: 2017-01-10 09:54:16
categories:
- 运维
tags:
- linux命令
---
　　netstat命令用于显示与IP、TCP、UDP和ICMP协议相关的统计数据，一般用于检验本机各端口的网络连接情况。netstat是在内核中访问网络及相关信息的程序，它能提供TCP连接，TCP和UDP监听，进程内存管理的相关报告。
<!--more -->
　　如果你的计算机有时候接收到的数据报导致出错数据或故障，你不必感到奇怪，TCP/IP可以容许这些类型的错误，并能够自动重发数据报。但如果累计的出错情况数目占到所接收的IP数据报相当大的百分比，或者它的数目正迅速增加，那么你就应该使用netstat查一查为什么会出现这些情况了。
### 命令格式
```bash
$ netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]
```
### 命令功能
　　netstat用于显示与IP、TCP、UDP和ICMP协议相关的统计数据，一般用于检验本机各端口的网络连接情况。
### 命令参数
| 命令 | 描述     |
| :------------- | :------------- |
| -a或–all | 显示所有连线中的Socket |
| -A<网络类型>或–<网络类型> | 列出该网络类型连线中的相关地址 |
| -c或–continuous | 持续列出网络状态 |
| -C或–cache | 显示路由器配置的快取信息 |
| -e或–extend | 显示网络其他相关信息 |
| -F或–fib | 显示FIB |
| -g或–groups | 显示多重广播功能群组组员名单 |
| -h或–help | 在线帮助 |
| -i或–interfaces | 显示网络界面信息表单 |
| -l或–listening | 显示监控中的服务器的Socket |
| -M或–masquerade | 显示伪装的网络连线 |
| -n或–numeric | 直接使用IP地址，而不通过域名服务器 |
| -N或–netlink或–symbolic | 显示网络硬件外围设备的符号连接名称 |
| -o或–timers | 显示计时器 |
| -p或–programs | 显示正在使用Socket的程序识别码和程序名称 |
| -r或–route | 显示Routing Table |
| -s或–statistice | 显示网络工作信息统计表 |
| -t或–tcp | 显示TCP传输协议的连线状况 |
| -u或–udp | 显示UDP传输协议的连线状况 |
| -v或–verbose | 显示指令执行过程 |
| -V或–version | 显示版本信息 |
| -w或–raw | 显示RAW传输协议的连线状况 |
| -x或–unix | 此参数的效果和指定”-A unix”参数相同 |
| –ip或–inet | 此参数的效果和指定”-A inet”参数相同 |

### 使用实例
**`例一`：无参数使用**
```bash
$ netstat
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0    268 192.168.120.204:ssh         10.2.0.68:62420             ESTABLISHED
udp        0      0 192.168.120.204:4371        10.58.119.119:domain        ESTABLISHED
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node Path
unix  2      [ ]         DGRAM                    1491   @/org/kernel/udev/udevd
unix  4      [ ]         DGRAM                    7337   /dev/log
unix  2      [ ]         DGRAM                    708823
unix  2      [ ]         DGRAM                    7539   
unix  3      [ ]         STREAM     CONNECTED     7287   
unix  3      [ ]         STREAM     CONNECTED     7286
```
>**说明：**
　　从整体上看，netstat的输出结果可以分为两个部分：
　　一个是Active Internet connections，称为有源TCP连接，其中"Recv-Q"和"Send-Q"指的是接收队列和发送队列。这些数字一般都应该是0。如果不是则表示软件包正在队列中堆积。这种情况只能在非常少的情况见到。
　　另一个是Active UNIX domain sockets，称为有源Unix域套接口(和网络套接字一样，但是只能用于本机通信，性能可以提高一倍)。
　　Proto显示连接使用的协议,RefCnt表示连接到本套接口上的进程号,Types显示套接口的类型,State显示套接口当前的状态,Path表示连接到套接口的其它进程使用的路径名。
**套接口类型：**
　　-t ：TCP
　　-u ：UDP
　　-raw ：RAW类型
　　--unix ：UNIX域类型
　　--ax25 ：AX25类型
　　--ipx ：ipx类型
　　--netrom ：netrom类型
**状态说明：**
　　LISTEN：侦听来自远方的TCP端口的连接请求
　　SYN-SENT：再发送连接请求后等待匹配的连接请求（如果有大量这样的状态包，检查是否中招了）
　　SYN-RECEIVED：再收到和发送一个连接请求后等待对方对连接请求的确认（如有大量此状态，估计被flood攻击了）
　　ESTABLISHED：代表一个打开的连接
　　FIN-WAIT-1：等待远程TCP连接中断请求，或先前的连接中断请求的确认
　　FIN-WAIT-2：从远程TCP等待连接中断请求
　　CLOSE-WAIT：等待从本地用户发来的连接中断请求
　　CLOSING：等待远程TCP对连接中断的确认
　　LAST-ACK：等待原来的发向远程TCP的连接中断请求的确认（不是什么好东西，此项出现，检查是否被攻击）
　　TIME-WAIT：等待足够的时间以确保远程TCP接收到连接中断请求的确认
　　CLOSED：没有任何连接状态

**`例二`：列出所有端口**
```bash
$ netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 localhost:smux              *:*                         LISTEN      
tcp        0      0 *:svn                       *:*                         LISTEN      
tcp        0      0 *:ssh                       *:*                         LISTEN      
tcp        0    284 192.168.120.204:ssh         10.2.0.68:62420             ESTABLISHED
udp        0      0 localhost:syslog            *:*                                     
udp        0      0 *:snmp                      *:*                                     
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node Path
unix  2      [ ACC ]     STREAM     LISTENING     708833 /tmp/ssh-yKnDB15725/agent.15725
unix  2      [ ACC ]     STREAM     LISTENING     7296   /var/run/audispd_events
unix  2      [ ]         DGRAM                    1491   @/org/kernel/udev/udevd
unix  4      [ ]         DGRAM                    7337   /dev/log
unix  2      [ ]         DGRAM                    708823
unix  2      [ ]         DGRAM                    7539   
unix  3      [ ]         STREAM     CONNECTED     7287   
unix  3      [ ]         STREAM     CONNECTED     7286   
```
>**说明：**
　　显示一个所有的有效连接信息列表，包括已建立的连接（ESTABLISHED），也包括监听连接请（LISTENING）的那些连接

**`例三`：显示当前UDP连接状况**
```bash
$ netstat -nu
```
**`例四`：显示UDP端口号的使用情况**
```bash
$ netstat -apu
```
**`例五`：显示UDP端口号的使用情况**
```bash
$ netstat -i
```
**`例六`：显示组播组的关系**
```bash
$ netstat -g
```
**`例七`：显示网络统计信息**
```bash
$ netstat -s
```
>**说明：**
　　按照各个协议分别显示其统计数据。如果我们的应用程序（如Web浏览器）运行速度比较慢，或者不能显示Web页之类的数据，那么我们就可以用本选项来查看一下所显示的信息。我们需要仔细查看统计数据的各行，找到出错的关键字，进而确定问题所在。

**`例八`：显示监听的套接口**
```bash
$ netstat -l
```
**`例九`：显示所有已建立的有效连接**
```bash
$ netstat -n
```
**`例十`：显示关于以太网的统计数据**
```bash
$ netstat -e
```
>**说明：**
　　用于显示关于以太网的统计数据。它列出的项目包括传送的数据报的总字节数、错误数、删除数、数据报的数量和广播的数量。这些统计数据既有发送的数据报数量，也有接收的数据报数量。这个选项可以用来统计一些基本的网络流量）

**`例十一`：显示关于路由表的信息**
```bash
$ netstat -r
```
**`例十二`：列出所有 tcp 端口**
```bash
$ netstat -at
```
**`例十三`：统计机器中网络连接各个状态个数**
```bash
$ netstat -a | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```
**`例十四`：把状态全都取出来后使用uniq -c统计后再进行排序**
```bash
$ netstat -nat |awk '{print $6}'|sort|uniq -c
```
**`例十五`：查看连接某服务端口最多的的IP地址**
```bash
$ netstat -nat | grep "192.168.120.20:16067" |awk '{print $5}'|awk -F: '{print $4}'|sort|uniq -c|sort -nr|head -20
```
**`例十六`：找出程序运行的端口**
```bash
$ netstat -ap | grep ssh
```
**`例十七`：在 netstat 输出中显示 PID 和进程名称**
```bash
$ netstat -pt
```
>**说明：**
　　netstat -p 可以与其它开关一起使用，就可以添加 “PID/进程名称” 到 netstat 输出中，这样 debugging 的时候可以很方便的发现特定端口运行的程序

**`例十八`：查看所有端口使用情况**
```bash
$ netstat -tln
```

**`例十九`：找出运行在指定端口的进程**
```bash
$ netstat -anpt | grep ':4000'
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:4000            0.0.0.0:*               LISTEN      5362/hexo           
tcp        0      0 127.0.0.1:4000          127.0.0.1:45884         ESTABLISHED 5362/hexo           
tcp        0      0 127.0.0.1:4000          127.0.0.1:45886         ESTABLISHED 5362/hexo  
```
>**说明：**
　　运行在端口4000的进程id为5362，再通过ps命令就可以找到具体的应用程序了
