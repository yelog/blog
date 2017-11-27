---
title: '每天一个linux命令（4）: mkdir'
permalink: 'linux-command（4）-mkdir'
date: 2016-12-04 09:27:32
categories:
- 运维
tags:
- linux命令
---
　　linux mkdir 命令用来创建指定的名称的目录，要求创建目录的用户在当前目录中具有写权限，并且指定的目录名不能是当前目录中已有的目录。
<!--more -->
### 命令格式
```bash
$ mkdir [选项] 目录...
```
### 命令功能
　　通过 mkdir 命令可以实现在指定位置创建以 DirName(指定的文件名)命名的文件夹或目录。要创建文件夹或目录的用户必须对所创建的文件夹的父文件夹具有写权限。并且，所创建的文件夹(目录)不能与其父目录(即父文件夹)中的文件名重名，即同一个目录下不能有同名的(区分大小写)。
### 命令参数
| 参数 | 说明     |
| :------------- | :------------- |
| -m, --mode=模式       | 设定权限<模式> (类似 chmod)，而不是 rwxrwxrwx 减 umask       |
|  -p, --parents |  可以是一个路径名称。此时若路径中的某些目录尚不存在,加上此选项后,系统将自动建立好那些尚不存在的目录,即一次可以建立多个目录 |
| -v, --verbose  | 每次创建新目录都显示信息 |
| --help |显示此帮助信息并退出|
| --version |  输出版本信息并退出 |
### 命令是实例
**`例一`：创建一个空目录**
```bash
$ mkdir test
```
**`例二`：递归创建多个目录**
```bash
#在当前目录创建一个嵌套文件夹test1/test11
$ mkdir -p test1/test11
```
**`例三`：创建权限为777的目录**
```bash
$ mkdir -m 777 test
```
**`例四`：创建新目录都显示信息**
```bash
$ mkdir -v test
```
**`例五`：一个命令创建项目的目录结构**
```bash
$ mkdir -vp scf/{lib/,bin/,doc/{info,product},logs/{info,product},service/deploy/{info,product}}
mkdir: 已创建目录 “scf”
mkdir: 已创建目录 “scf/lib”
mkdir: 已创建目录 “scf/bin”
mkdir: 已创建目录 “scf/doc”
mkdir: 已创建目录 “scf/doc/info”
mkdir: 已创建目录 “scf/doc/product”
mkdir: 已创建目录 “scf/logs”
mkdir: 已创建目录 “scf/logs/info”
mkdir: 已创建目录 “scf/logs/product”
mkdir: 已创建目录 “scf/service”
mkdir: 已创建目录 “scf/service/deploy”
mkdir: 已创建目录 “scf/service/deploy/info”
mkdir: 已创建目录 “scf/service/deploy/product”
[root@localhost test]# tree scf/
scf/
|-- bin
|-- doc
|   |-- info
|   `-- product
|-- lib
|-- logs
|   |-- info
|   `-- product
`-- service
    `-- deploy
        |-- info
        `-- product
12 directories, 0 files
```
