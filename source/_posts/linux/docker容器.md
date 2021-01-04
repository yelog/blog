---
title: docker容器
enlink: docker-container
date: 2017-05-23 21:29:38
categories:
- 运维
tags:
- docker
---
## 容器
镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。

## 命令
```bash
# 创建一个名为myubuntu的容器
# -t:分配一个伪终端  -i:让容器的标准输入保持打开
$ docker run --name=myubuntu -t -i ubuntu /bin/bash

# 创建一个名为webserver 的nginx容器，使用卷映射本机/home/faker/myspace/nginx目录到docker目录/usr/share/nginx/html
$ docker run --name=webserver -d -v /home/faker/myspace/nginx:/usr/share/nginx/html -p 80:80 nginx

# 查看容器的输出信息（打印信息，如 echo）
# run的时候，使用-d将会不展示在宿主机上，可通过下面命令查看打印信息
$ docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
$ docker logs [container ID or NAMES]

# 启动容器 myubuntu
$ docker start myubuntu

# 关闭容器 myubuntu
$ docker stop myubuntu

# 查看已启动的容器 -a:查看包括未启动的容器在内的所有容器
$ docker ps [-a]

# 进入容器（Docker自带的命令）
$ docker attach [name]

# 进入容器（通过exec）
$ docker exec -it [name] /bin/bash

# 导出容器快照到本地文件
$ docker export [container id] > ubuntu.tar

# 将容器快照导入为镜像
$ cat ubuntu.tar | docker import - test/ubuntu:v1.0

# 从制定 URL 或者某个目录导入
$ docker import http://example.com/exampleimage.tgz example/imagerepo

# 删除容器 -f:删除正在运行的容器
$ docker [-f] rm myubuntu

# 删除所有已关闭的容器
$ docker rm $(docker ps -a -q)

# 查询各容器资源使用情况
$ docker stats $(docker ps --format={{.Names}})
```
