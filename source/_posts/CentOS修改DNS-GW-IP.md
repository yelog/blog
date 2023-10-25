---
title: CentOS修改DNS/GW/IP
enlink: CentOS-DNS-GW-IP
date: 2017-05-23 09:53:52
categories:
- 运维
tags:
- 运维
- centos
---
## 1.修改DNS
**解决方案一：**
修改网卡的DNS的配置文件
```bash
$ vim /etc/resolv.conf
```
添加以下内容,设置两条dns
```
nameserver 8.8.8.8 #google域名服务器
nameserver 8.8.4.4 #google域名服务器
```
>若未生效，可执行 `chattr +i /etc/resolv.conf` 设置文件属性只有root用户才能修改
然后执行 `service NetworkManager restart `

**解决方案二：**
对接口添加dns信息；编辑/etc/sysconfig/network-scripts/ifcfg-xxx，xxx为你的网卡名，但一般是ifcfg-eth0的，具体的xxx根据你的网卡确定，在最下面添加：
```
DNS1=8.8.8.8   #google dns服务器, 根据实际情况更换
DNS2=8.8.4.4   #google dns服务器, 根据实际情况更换
```
保存后重启网络
```bash
$ service network restart
```

## 2.修改网关
修改网关的配置文件(第3部分也可以设置)
```bash
$ vim /etc/sysconfig/network
```
修改为一下内容
```
NETWORKING=yes(表示系统是否使用网络，一般设置为yes。如果设为no，则不能使用网络，而且很多系统服务程序将无法启动)
HOSTNAME=centos(设置本机的主机名，这里设置的主机名要和/etc/hosts中设置的主机名对应)
GATEWAY=192.168.1.1(设置本机连接的网关的IP地址。例如，网关为10.0.0.2)
```

## 3.修改ip
修改对应的网卡的IP地址的配置文件
```bash
$ vim /etc/sysconfig/network-scripts/ifcfg-eth0
```
修改为一下内容
```
DEVICE=eth0 #描述网卡对应的设备别名，例如ifcfg-eth0的文件中它为eth0
BOOTPROTO=static #设置网卡获得ip地址的方式，可能的选项为static，dhcp或bootp，分别对应静态指定的 ip地址，通过dhcp协议获得的ip地址，通过bootp协议获得的ip地址
BROADCAST=192.168.0.255 #对应的子网广播地址
HWADDR=00:07:E9:05:E8:B4 #对应的网卡物理地址
IPADDR=12.168.1.2 #如果设置网卡获得 ip地址的方式为静态指定，此字段就指定了网卡对应的ip地址
IPV6INIT=no
IPV6_AUTOCONF=no
NETMASK=255.255.255.0 #网卡对应的网络掩码
NETWORK=192.168.1.0 #网卡对应的网络地址
ONBOOT=yes #系统启动时是否设置此网络接口，设置为yes时，系统启动时激活此设备
```
