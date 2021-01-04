---
title: CentOS7安装配置匿名访问Samba
enlink: CentOS7-anonymous-Samba
date: 2017-07-03 19:40:14
categories:
- 运维
tags:
- samba
---

## 介绍
>**Samba**，是种用来让UNIX系列的操作系统与微软Windows操作系统的SMB/CIFS（Server Message Block/Common Internet File System）网络协议做链接的自由软件   --wikipedia

本文就以 CentOS7 搭建 Samba 匿名完全访问（读/写）为目标，实现一个局域网内的文件共享平台。

## 1.安装Samba服务
使用 yum 工具进行安装
```bash
$ yum install samba samba-client
```

## 2.检查是否安装成功
```bash
$ rpm -qa | grep samba
```

## 3.防火墙开放端口
在 `/etc/sysconfig/iptables` 中添加配置
```xml
-A INPUT -p tcp -m state --state NEW -m tcp --dport 137 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 138 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 139 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 389 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 445 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 901 -j ACCEPT
```
重启 iptables 服务
```bash
$ service iptables restart
```
设置开机自启动
```bash
$ chkconfig --level 35 smb on
```

## 4.共享配置
Samba Server的验证方式有四种：
* share：匿名访问共享，不需要提供用户名和口令, 安全性能较低。
* user：共享目录只能被授权的用户访问,由Samba Server负责检查账号和密码的正确性。账号和密码要在本Samba Server中建立。
* server：依靠其他Windows Server或Samba Server来验证用户的账号和密码,是一种代理验证。此种安全模式下,系统管理员可以把所有的Windows用户和口令集中到一个Server系统上,使用 Windows Server进行Samba认证, 远程服务器可以自动认证全部用户和口令,如果认证失败,Samba将使用用户级安全模式作为替代的方式。
* domain：域安全级别,使用主域控制器(PDC)来完成认证。

>创建一个匿名共享访问，需要使用share模式，但在CentOS安装的samba4中share 和 server验证方式已被弃用

配置如下：
```xml
[global]
        workgroup = MYGROUP
        server string = Samba Server Version %v
        log file = /var/log/samba/log.%m
        max log size = 50
        security = user
        map to guest = Bad User
        load printers = yes
        cups options = raw
[share]
        comment = share
        path = /home/samba
      	directory mask = 0777
      	create mask = 0777
      	#不可视目录
        #browseable = yes
        guest ok=yes
        writable=yes
```

创建 `/home/samba` 共享目录
```bash
$ mkdir /home/samba
```

重启 smb 服务
```bash
$ service smb restart
```

检查服务是否在运行
```bash
$ pgrep smbd
```


检查配置参数
```bash
$ testparm
Load smb config files from /etc/samba/smb.conf
Processing section "[share]"
Loaded services file OK.
Server role: ROLE_STANDALONE

Press enter to see a dump of your service definitions

# Global parameters
[global]
	server string = Samba Server Version %v
	workgroup = MYGROUP
	log file = /var/log/samba/log.%m
	max log size = 50
	map to guest = Bad User
	security = USER
	idmap config * : backend = tdb
	cups options = raw


[share]
	comment = share
	path = /home/samba
	create mask = 0777
	directory mask = 0777
	guest ok = Yes
	read only = No
```

## 访问
以上就配置完成，如服务器地址为192.168.0.87

windows 系统访问，直接运行 `\\192.168.0.87\share`

linux 系统访问， `smb://192.168.0.87/share`

## 遇到的问题
* linux 系统可以正常读写修改，但 windows 系统只可以读写，直接打开修改时就，就为只读文件了。
**解决办法**：修改 `/etc/samba/smb.conf` ,在 `[share]` 中加入以下内容
```xml
create mask = 0777
```
* 访问部分文件可以正常访问，但部分文件无法访问。
**解决方法**：修改文件访问权限
```bash
$ chmod -R 1777 /home/samba
$ chown nobody:nobody
```

# 参考
* [CentOS7 安装Samba服务](http://www.cnblogs.com/lion382/p/4078931.html)
* [CentOS7 安装配置匿名访问Samba](http://blog.leanote.com/post/dapingxia@163.com/CentOS7-%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE%E5%8C%BF%E5%90%8D%E8%AE%BF%E9%97%AESamba)
