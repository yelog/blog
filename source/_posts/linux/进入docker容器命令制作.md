---
title: 进入docker容器命令制作
enlink: entering-docker
date: 2017-06-01 17:25:11
categories:
- 运维
tags:
- docker
---
## 通过attach进入容器
```bash
# 进入容器（Docker自带的命令）
$ sudo docker attach [name]
```
通过这命令进入容器后，执行ctrl+d退出容器后发现容器也停止了。
所以可以通过
- 先按，ctrl+p
- 再按，ctrl+q

退出

## 制作进入容器的命令
既然attach退出很麻烦，一不小心容器就down掉了

通过 `docker exec` 进入容器是安全的，但是命令过长

所以我们可以通过下面操作，简化命令

1.创建文件 `/usr/bin/ctn`,内容如下
```
docker exec -it $1 /bin/bash
```
2.检查环境变量有没有配置目录 `/usr/bin`
```
$PATH
bash: /usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games: No such file or directory
```
配置环境变量的方式自行百度

3.完成上面两步即可通过命令 `ctn` 进入容器
```bash
$ ctn [name]
```
>注意：如果是使用非root账号创建的命令，而docker命令是root权限，可能会存在权限问题
可以设置 `chmod 777 /usr/bin/ctn` 设置权限
使用 `sudo ctn [name]` 即可进入容器

4.自动补全docker名
使用上面命令时，docker的名字都是手动输入，很麻烦，而且容易出错。

我们可以借助complete命令，来补全docker信息。

在~/.bashrc(作用于当前用户，如果所有用户，修改/etc/bashrc)文件中添加一行
```bash
# ctn auto complete
complete -W "$(docker ps --format "{{.Names}}")" ctn
```
再执行 `source .bashrc` 使之生效。

这样我们输入 `ctn` 后，按 `Tab` 就会提示或自动补全了。

>`注意：` 由于提示的docker名是 `.bashrc` 生效时的列表，所以如果之后docker列表有变动，需重新执行 `source .bashrc` 使之更新提示列表
