---
title: docker备份恢复之save与export
permalink: docker-save-export
date: 2017-09-18 22:38:52
categories:
- 运维
tags:
- docker
---
## docker save
### 导出
`docker save` 命令用于持久化 **镜像**，先获得镜像名称，再执行保存：
```bash
# 通过此命令查出要持久化的镜像名称
$ docker images
# 持久化镜像名为 image_name 的镜像，
$ docker save image_name -o ~/save.tar
```
> **注意：** 如果镜像是在远程仓库，执行保存镜像的时候可能会报 `Cowardly refusing to save to a terminal. Use the -o flag or redirect.` 的错，可以通过 `docker save image_name > image_name.tar` 将镜像从远程仓库持久化到本地。

### 导入
```bash
# 导入 save.tar
$ docker load < ~/save.tar
# 查看镜像
$ docker images images
```

## docker export
### 导出
`docker export` 命令用于持久化 **容器**，先获取容器ID，再执行保存。
```bash
# 通过此命令查出要持久化的容器ID
$ docker ps -a
# 持久化容器id为 container_id 的容器
$ docker export container_id > ~/export.tar
```
### 导入
```bash
# 从 export.tar 导入镜像
$ cat ~/export.tar | docker import - my-images:latest
# 查看镜像
$ sudo docker images
```

## 不同
通过 `sudo docker images --tree` 可以查看到镜像的所有层，就会发现， `docker export` 丢失了所有的历史，而`docker save` 则会保存所有历史。
