---
title: '每天一个linux命令（40）: traceroute'
enlink: 'linux-command-40-traceroute'
date: 2017-01-09 10:56:53
categories:
- 运维
tags:
- linux命令
---
　　通过traceroute我们可以知道信息从你的计算机到互联网另一端的主机是走的什么路径。当然每次数据包由某一同样的出发点（source）到达某一同样的目的地(destination)走的路径可能会不一样，但基本上来说大部分时候所走的路由是相同的。linux系统中，我们称之为traceroute,在MS Windows中为tracert。 traceroute通过发送小的数据包到目的设备直到其返回，来测量其需要多长时间。一条路径上的每个设备traceroute要测3次。输出结果中包括每次测试的时间(ms)和设备的名称（如有的话）及其IP地址。
<!--more -->
　　在大多数情况下，我们会在linux主机系统下，直接执行命令行：
　　　　`traceroute hostname`
　　而在Windows系统下是执行tracert的命令：
　　　　`tracert hostname`
### 命令格式
```bash
$ traceroute [参数] [主机]
```
### 命令功能
　　traceroute指令让你追踪网络数据包的路由途径，预设数据包大小是40Bytes，用户可另行设置。
　　具体参数格式：traceroute [-dFlnrvx][-f<存活数值>][-g<网关>...][-i<网络界面>][-m<存活数值>][-p<通信端口>][-s<来源地址>][-t<服务类型>][-w<超时秒数>][主机名称或IP地址][数据包大小]

### 命令参数
| 命令 | 描述     |
| :------------- | :------------- |
| -d | 使用Socket层级的排错功能 |
| -f | 设置第一个检测数据包的存活数值TTL的大小 |
| -F | 设置勿离断位 |
| -g | 设置来源路由网关，最多可设置8个 |
| -i | 使用指定的网络界面送出数据包 |
| -I | 使用ICMP回应取代UDP资料信息 |
| -m | 设置检测数据包的最大存活数值TTL的大小 |
| -n | 直接使用IP地址而非主机名称 |
| -p | 设置UDP传输协议的通信端口 |
| -r | 忽略普通的Routing Table，直接将数据包送到远端主机上 |
| -s | 设置本地主机送出数据包的IP地址 |
| -t | 设置检测数据包的TOS数值 |
| -v | 详细显示指令的执行过程 |
| -w | 设置等待远端主机回报的时间 |
| -x | 开启或关闭数据包的正确性检验 |

### 使用实例
**`例一`：traceroute 用法简单、最常用的用法**
```bash
$ traceroute yelog.github.com
traceroute to yelog.github.com (151.101.192.133), 30 hops max, 60 byte packets
 1  vrouter (192.168.0.1)  0.443 ms  0.565 ms  0.684 ms
 2  112.208.32.1.pldt.net (112.208.32.1)  14.518 ms  22.454 ms  23.080 ms
 3  119.93.255.197 (119.93.255.197)  24.492 ms  25.380 ms  26.328 ms
 4  210.213.131.66.static.pldt.net (210.213.131.66)  29.942 ms 210.213.131.70.static.pldt.net (210.213.131.70)  28.209 ms  28.992 ms
 5  122.2.175.30.static.pldt.net (122.2.175.30)  32.429 ms  32.765 ms 210.213.128.29.static.pldt.net (210.213.128.29)  35.165 ms
 6  210.213.130.162.static.pldt.net (210.213.130.162)  32.147 ms  31.403 ms  32.107 ms
 7  las-b3-link.telia.net (62.115.13.128)  198.546 ms  190.829 ms  191.039 ms
 8  las-b21-link.telia.net (213.155.131.82)  194.301 ms las-b21-link.telia.net (62.115.116.179)  191.927 ms las-b21-link.telia.net (213.155.131.84)  194.433 ms
 9  * * *
10  * * *
```
>**说明：**
　　记录按序列号从1开始，每个纪录就是一跳 ，每跳表示一个网关，我们看到每行有三个时间，单位是 ms，其实就是-q的默认参数。探测数据包向每个网关发送三个数据包后，网关响应后返回的时间；如果您用 traceroute -q 4 www.58.com ，表示向每个网关发送4个数据包。
　　有时我们traceroute 一台主机时，会看到有一些行是以星号表示的。出现这样的情况，可能是防火墙封掉了ICMP的返回信息，所以我们得不到什么相关的数据包返回数据。
　　有时我们在某一网关处延时比较长，有可能是某台网关比较阻塞，也可能是物理设备本身的原因。当然如果某台DNS出现问题时，不能解析主机名、域名时，也会 有延时长的现象；您可以加-n 参数来避免DNS解析，以IP格式输出数据。
　　如果在局域网中的不同网段之间，我们可以通过traceroute 来排查问题所在，是主机的问题还是网关的问题。如果我们通过远程来访问某台服务器遇到问题时，我们用到traceroute 追踪数据包所经过的网关，提交IDC服务商，也有助于解决问题；但目前看来在国内解决这样的问题是比较困难的，就是我们发现问题所在，IDC服务商也不可能帮助我们解决。

**`例二`：跳数设置**
```bash
$ traceroute -m 10 www.baidu.com
```
**`例三`：显示IP地址，不查主机名**
```bash
$ traceroute -n www.baidu.com
```
**`例四`：探测包使用的基本UDP端口设置6888**
```bash
$ traceroute -p 6888 www.baidu.com
```
**`例五`：把探测包的个数设置为值4**
```bash
$ traceroute -q 4 www.baidu.com
```
**`例六`：绕过正常的路由表，直接发送到网络相连的主机**
```bash
$ traceroute -r www.baidu.com
```
**`例七`：把对外发探测包的等待响应时间设置为3秒**
```bash
$ traceroute -w 3 www.baidu.com
```
>**Traceroute的工作原理**
　　Traceroute最简单的基本用法是：traceroute hostname
　　Traceroute程序的设计是利用ICMP及IP header的TTL（Time To Live）栏位（field）。首先，traceroute送出一个TTL是1的IP datagram（其实，每次送出的为3个40字节的包，包括源地址，目的地址和包发出的时间标签）到目的地，当路径上的第一个路由器（router）收到这个datagram时，它将TTL减1。此时，TTL变为0了，所以该路由器会将此datagram丢掉，并送回一个「ICMP time exceeded」消息（包括发IP包的源地址，IP包的所有内容及路由器的IP地址），traceroute 收到这个消息后，便知道这个路由器存在于这个路径上，接着traceroute 再送出另一个TTL是2 的datagram，发现第2 个路由器...... traceroute 每次将送出的datagram的TTL 加1来发现另一个路由器，这个重复的动作一直持续到某个datagram 抵达目的地。当datagram到达目的地后，该主机并不会送回ICMP time exceeded消息，因为它已是目的地了，那么traceroute如何得知目的地到达了呢？
　　Traceroute在送出UDP datagrams到目的地时，它所选择送达的port number 是一个一般应用程序都不会用的号码（30000 以上），所以当此UDP datagram 到达目的地后该主机会送回一个「ICMP port unreachable」的消息，而当traceroute 收到这个消息时，便知道目的地已经到达了。所以traceroute 在Server端也是没有所谓的Daemon 程式。
　　Traceroute提取发 ICMP TTL到期消息设备的IP地址并作域名解析。每次 ，Traceroute都打印出一系列数据,包括所经过的路由设备的域名及 IP地址,三个包每次来回所花时间。

### windows之tracert
>**格式：**
```bash
$ tracert [-d] [-h maximum_hops] [-j host-list] [-w timeout] target_name
```
>**参数说明**
　　tracert [-d] [-h maximum_hops] [-j computer-list] [-w timeout] target_name
　　该诊断实用程序通过向目的地发送具有不同生存时间 (TL) 的 Internet 控制信息协议 (CMP) 回应报文，以确定至目的地的路由。路径上的每个路由器都要在转发该 ICMP 回应报文之前将其 TTL 值至少减 1，因此 TTL 是有效的跳转计数。当报文的 TTL 值减少到 0 时，路由器向源系统发回 ICMP 超时信息。通过发送 TTL 为 1 的第一个回应报文并且在随后的发送中每次将 TTL 值加 1，直到目标响应或达到最大 TTL 值，Tracert 可以确定路由。通过检查中间路由器发发回的 ICMP 超时 (ime Exceeded) 信息，可以确定路由器。注意，有些路由器“安静”地丢弃生存时间 (TLS) 过期的报文并且对 tracert 无效。
>**参数：**
-d 指定不对计算机名解析地址。
-h maximum_hops 指定查找目标的跳转的最大数目。
-jcomputer-list 指定在 computer-list 中松散源路由。
-w timeout 等待由 timeout 对每个应答指定的毫秒数。
target_name 目标计算机的名称。
