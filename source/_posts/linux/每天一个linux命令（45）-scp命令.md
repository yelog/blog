---
title: '每天一个linux命令（45）: scp'
permalink: 'linux-command（45）-scp'
date: 2017-01-13 10:53:48
categories:
- 运维
tags:
- linux命令
---
　　scp是secure copy的简写，用于在Linux下进行远程拷贝文件的命令，和它类似的命令有cp，不过cp只是在本机进行拷贝不能跨服务器，而且scp传输是加密的。可能会稍微影响一下速度。当你服务器硬盘变为只读 read only system时，用scp可以帮你把文件移出来。另外，scp还非常不占资源，不会提高多少系统负荷，在这一点上，rsync就远远不及它了。虽然 rsync比scp会快一点，但当小文件众多的情况下，rsync会导致硬盘I/O非常高，而scp基本不影响系统正常使用。
<!--more -->
### 命令格式
```bash
$ scp [参数] [原路径] [目标路径]
```
### 命令功能
　　scp是 secure copy的缩写, scp是linux系统下基于ssh登陆进行安全的远程文件拷贝命令。linux的scp命令可以在linux服务器之间复制文件和目录。
### 命令参数
| 参数 | 描述     |
| :------------- | :------------- |
| -1 | 强制scp命令使用协议ssh1 |
| -2 | 强制scp命令使用协议ssh2 |
| -4 | 强制scp命令只使用IPv4寻址 |
| -6 | 强制scp命令只使用IPv6寻址 |
| -B | 使用批处理模式（传输过程中不询问传输口令或短语） |
| -C | 允许压缩。（将-C标志传递给ssh，从而打开压缩功能） |
| -p | 保留原文件的修改时间，访问时间和访问权限 |
| -q | 不显示传输进度条 |
| -r | 递归复制整个目录 |
| -v | 详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题 |
| -c cipher | 以cipher将数据传输进行加密，这个选项将直接传递给ssh |
| -F ssh_config | 指定一个替代的ssh配置文件，此参数直接传递给ssh |
| -i identity_file | 从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh |
| -l limit | 限定用户所能使用的带宽，以Kbit/s为单位 |
| -o ssh_option | 如果习惯于使用ssh_config(5)中的参数传递方式 |
| -P port | 注意是大写的P, port是指定数据传输用到的端口号 |
| -S program | 指定加密传输时所使用的程序。此程序必须能够理解ssh(1)的选项 |

### 使用实例
**从本地服务器复制到远程服务器：**
```bash
# 指定了用户名，命令执行后需输入密码
$ scp -r img/* root@server:/var/project/img/

# 没有指定用户名，命令执行后需要输入用户名密码
$ scp -r img/* server:/var/project/img/
```
**从远程服务器复制到本地当前目录：**
```bash
$ scp -r server:/var/project/img/* .
```
