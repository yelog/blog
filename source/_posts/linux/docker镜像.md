---
title: docker镜像
permalink: docker-image
date: 2017-05-19 16:47:33
categories:
- 运维
tags:
- docker
---
## What
Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

因为镜像包含操作系统完整的 root 文件系统，其体积往往是庞大的，因此在 Docker 设计时，就充分利用 Union FS 的技术，将其设计为分层存储的架构。所以严格来说，镜像并非是像一个 ISO 那样的打包文件，镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。

## 安装
```bash
# 官方 的安装脚本
$ curl -sSL https://get.docker.com/ | sh

# 阿里云 的安装脚本
$ curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -

# DaoCloud 的安装脚本
$ curl -sSL https://get.daocloud.io/docker | sh
```

## 镜像
```
# 获取镜像，registry为空默认从Docker Hub上获取
docker pull [选项] [Docker Registry地址]<仓库名>:<标签>

# 交互式运行，退出删除: -i:交互式 ,-t:终端,--rm 退出删除 ,bash 启动bash窗口
$ docker run -it --rm ubuntu:14.04 bash

# 列出已下载的镜像（只显示顶层镜像） -a:显示所有镜像 image_name:指定列出某个镜像
$ docker images [-a] [image_name]

# 只显示虚悬镜像(dangling image) -f:--filter 过滤
$ docker images -f dangling=true

# 过滤从mongo:3.2建立之后的镜像
$ docker images -f since=mongo:3.2

# 通过label过滤
$ docker images -f label=com.example.version=0.1

# 只显示镜像id
$ docker images -q

# 只包含镜像ID和仓库名
$ docker images --format "{{.ID}}: {{.Repository}}"

# 以表格等距显示 有标题行，和默认一样，不过自己定义列
$ docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"

# 删除镜像ID为image_id的镜像
$ docker rmi <image_id>

# 删除虚悬镜像
$ docker rmi $(docker images -q -f dangling=true)

# 将容器保存为镜像
$ docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]

# 将容器保存为镜像
$ docker commit \
    --author "Tao Wang <twang2218@gmail.com>" \
    --message "修改了默认网页" \
    webserver \
    nginx:v2

$ docker history nginx:v2
```
