---
title: '每天一个linux命令（38）: route'
enlink: 'linux-command-38-route'
date: 2017-01-07 10:16:34
categories:
- 运维
tags:
- linux命令
---
　　Linux系统的route命令用于显示和操作IP路由表（show / manipulate the IP routing table）。要实现两个不同的子网之间的通信，需要一台连接两个网络的路由器，或者同时位于两个网络的网关来实现。在Linux系统中，设置路由通常是为了解决以下问题：该Linux系统在一个局域网中，局域网中有一个网关，能够让机器访问Internet，那么就需要将这台机器的IP地址设置为Linux机器的默认路由。要注意的是，直接在命令行下执行route命令来添加路由，不会永久保存，当网卡重启或者机器重启之后，该路由就失效了；可以在/etc/rc.local中添加route命令来保证该路由设置永久有效。
<!--more -->
### 命令格式
```bash
route [-f] [-p] [Command [Destination] [mask Netmask] [Gateway] [metric Metric]] [if Interface]]
```
### 命令功能
　　Route命令是用于操作基于内核ip路由表，它的主要作用是创建一个静态路由让指定一个主机或者一个网络通过一个网络接口，如eth0。当使用"add"或者"del"参数时，路由表被修改，如果没有参数，则显示路由表当前的内容。
### 命令参数
| 命令 | 描述     |
| :------------- | :------------- |
| -c | 显示更多信息 |
| -n | 不解析名字 |
| -v | 显示详细的处理信息 |
| -F | 显示发送信息 |
| -C | 显示路由缓存 |
| -f | 清除所有网关入口的路由表 |
| -p 与 add 命令 | -p 与 add 命令一起使用时使路由具有永久性。 |
| add | 添加一条新路由 |
| del | 删除一条路由 |
| -net | 目标地址是一个网络 |
| -host | 目标地址是一个主机 |
| netmask | 当添加一个网络路由时，需要使用网络掩码 |
| gw | 路由数据包通过网关。注意，你指定的网关必须能够达到 |
| metric | 设置路由跳数 |
| Command | 指定您想运行的命令 (Add/Change/Delete/Print) |
| Destination | 指定该路由的网络目标 |
| mask Netmask | 指定与网络目标相关的网络掩码（也被称作子网掩码） |
| Gateway | 指定网络目标定义的地址集和子网掩码可以到达的前进或下一跃点 IP 地址 |
| metric Metric | 为路由指定一个整数成本值标（从 1 至 9999），当在路由表(与转发的数据包目标地址最匹配)的多个路由中进行选择时可以使用 |
| if Interface | 为可以访问目标的接口指定接口索引。若要获得一个接口列表和它们相应的接口索引，使用 route print 命令的显示功能。可以使用十进制或十六进制值进行接口索引 |

### 使用实例
**`例一`：显示当前路由**
```bash
$ route
$ route -n
```
```bash
[root@localhost ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.120.0   *               255.255.255.0   U     0      0        0 eth0
e192.168.0.0     192.168.120.1   255.255.0.0     UG    0      0        0 eth0
10.0.0.0        192.168.120.1   255.0.0.0       UG    0      0        0 eth0
default         192.168.120.240 0.0.0.0         UG    0      0        0 eth0
[root@localhost ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.120.0   0.0.0.0         255.255.255.0   U     0      0        0 eth0
192.168.0.0     192.168.120.1   255.255.0.0     UG    0      0        0 eth0
10.0.0.0        192.168.120.1   255.0.0.0       UG    0      0        0 eth0
0.0.0.0         192.168.120.240 0.0.0.0         UG    0      0        0 eth0
```
>**说明：**
　　第一行表示主机所在网络的地址为192.168.120.0，若数据传送目标是在本局域网内通信，则可直接通过eth0转发数据包;
　　第四行表示数据传送目的是访问Internet，则由接口eth0，将数据包发送到网关192.168.120.240
　　其中Flags为路由标志，标记当前网络节点的状态。
　　Flags标志说明：
　　U Up表示此路由当前为启动状态
　　H Host，表示此网关为一主机
　　G Gateway，表示此网关为一路由器
　　R Reinstate Route，使用动态路由重新初始化的路由
　　D Dynamically,此路由是动态性地写入
　　M Modified，此路由是由路由守护程序或导向器动态修改
　　! 表示此路由当前为关闭状态
**备注：**
　　route -n (-n 表示不解析名字,列出速度会比route 快)

**`例二`：添加网关/设置网关**
```bash
# 增加一条 到达244.0.0.0的路由
$ route add -net 224.0.0.0 netmask 240.0.0.0 dev eth0
```
**`例三`：屏蔽一条路由**
```bash
# 增加一条屏蔽的路由，目的地址为 224.x.x.x 将被拒绝
$ route add -net 224.0.0.0 netmask 240.0.0.0 reject
```
**`例四`：删除路由记录**
```bash
$ route del -net 224.0.0.0 netmask 240.0.0.0
$ route del -net 224.0.0.0 netmask 240.0.0.0 reject
```
**`例五`：删除和添加设置默认网关**
```bash
$ route del default gw 192.168.120.240
$ route add default gw 192.168.120.240
```
