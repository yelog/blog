---
title: '每天一个linux命令（44）: rcp'
permalink: 'linux-command（44）-rcp'
date: 2017-01-12 10:36:06
categories:
- 运维
tags:
- linux命令
---
　　rcp代表“remote file copy”（远程文件拷贝）。该命令用于在计算机之间拷贝文件。rcp命令有两种格式。第一种格式用于文件到文件的拷贝；第二种格式用于把文件或目录拷贝到另一个目录中。
<!--more -->
### 命令格式
```bash
$ rcp [参数] [源文件] [目标文件]
```
### 命令功能
　　rcp命令用在远端复制文件或目录，如同时指定两个以上的文件或目录，且最后的目的地是一个已经存在的目录，则它会把前面指定的所有文件或目录复制到该目录中。
### 命令参数
| 命令 | 描述     |
| :------------- | :------------- |
| -r | 递归地把源目录中的所有内容拷贝到目的目录中。要使用这个选项，目的必须是一个目录 |
| -p | 试图保留源文件的修改时间和模式，忽略umask |
| -k | 请求rcp获得在指定区域内的远程主机的Kerberos 许可，而不是获得由krb_relmofhost⑶确定的远程主机区域内的远程主机的Kerberos许可。 |
| -x | 为传送的所有数据打开DES加密。这会影响响应时间和CPU利用率，但是可以提高安全性。如果在文件名中指定的路径不是完整的路径名，那么这个路径被解释为相对远程机上同名用户的主目录。如果没有给出远程用户名，就使用当前用户名。如果远程机上的路径包含特殊shell字符，需要用反斜线（\\）、双引号（”）或单引号（’）括起来，使所有的shell元字符都能被远程地解释。需要说明的是，rcp不提示输入口令，它通过rsh命令来执行拷贝。 |
| directory | 每个文件或目录参数既可以是远程文件名也可以是本地文件名。远程文件名具有如下形式：rname@rhost：path，其中rname是远程用户名，rhost是远程计算机名，path是这个文件的路径。 |

### 使用实例
**使用rcp，需要具备的条件**
　　如果系统中有 /etc/hosts 文件，系统管理员应确保该文件包含要与之进行通信的远程主机的项。
　　/etc/hosts 文件中有一行文字，其中包含每个远程系统的以下信息：
　　　　`internet_address   official_name   alias`
　　例如：
　　　　`9.186.10.***  webserver1.com.58.webserver`
.rhosts 文件
　　.rhosts 文件位于远程系统的主目录下，其中包含本地系统的名称和本地登录名。
　　例如，远程系统的 .rhosts 文件中的项可能是：
　　　　`webserver1 root`
　　其中，webserver1 是本地系统的名称，root 是本地登录名。这样，webserver1 上的 root 即可在包含.rhosts 文件的远程系统中来回复制文件。
**配置过程:**
只对root用户生效
1. 在双方root用户根目录下建立.rhosts文件,并将双方的hostname加进去.在此之前应在双方的 /etc/hosts文件中加入对方的IP和hostname

2. 把rsh服务启动起来,redhat默认是不启动的。
方法：用执行ntsysv命令,在rsh选项前用空格键选中,确定退出。然后执行：
service xinetd restart即可。

3.到/etc/pam.d/目录下,把rsh文件中的auth required /lib/security/pam_securetty.so
一行用“#”注释掉即可。（只有注释掉这一行，才能用root用户登录）

**`例一`：将本地img文件夹内的所有内容 复制到服务器相应的img目录下**
```bash
# -r 递归子目录
$ rcp -r img/* webserver1:/var/project/img/
```
**`例二`：将服务器的img文件夹内的所有内容 复制到本地目录下**
```bash
# -r 递归子目录
$ rcp -r webserver1:/var/project/img/* img/
```
**`例三`：将目录复制到远程系统：要将本地目录及其文件和子目录复制到远程系统**
```bash
# 将本地的img目录复制到服务器的project目录下
$ rcp -r img/ webserver1:/var/project/
```
