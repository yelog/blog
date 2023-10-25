---
title: docker初体验
enlink: docker-first
date: 2017-05-19 16:32:23
categories:
- 运维
tags:
- docker
---
## 安装

### 笔者环境

操作系统：deepin 15.4 Desktop 64Bit

### 安装

```bash
# 官方 的安装脚本
$ curl -sSL https://get.docker.com/ | sh

# 阿里云 的安装脚本
$ curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -

# DaoCloud 的安装脚本
$ curl -sSL https://get.daocloud.io/docker | sh
```
### 获取镜像

[Docker Hub](https://hub.docker.com/explore/) 上有大量的高质量的镜像可以用，这里我们就说一下怎么获取这些镜像并运行。
从 Docker Registry 获取镜像的命令是 docker pull。其命令格式为：
```bash
$ docker pull [选项] [Docker Registry地址]<仓库名>:<标签>
```
具体的选项可以通过 `docker pull --help` 命令看到，这里我们说一下镜像名称的格式。

- Docker Registry地址：地址的格式一般是 `<域名/IP>[:端口号]`。默认地址是 `Docker Hub`。
- 仓库名：如之前所说，这里的仓库名是两段式名称，既 `<用户名>/<软件名>`。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。

```bash
$ sudo docker pull ubuntu
```

### 运行

有了镜像后，我们就可以以这个镜像为基础启动一个容器来运行。以上面的 `ubuntu` 为例，如果我们打算启动里面的 `bash` 并且进行交互式操作的话，可以执行下面的命令。
```bash
$ sudo docker run -it --rm ubuntu
root@0ae011f7b5be:/# cat /etc/os-release  
NAME="Ubuntu"
VERSION="16.04.2 LTS (Xenial Xerus)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 16.04.2 LTS"
VERSION_ID="16.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
VERSION_CODENAME=xenial
UBUNTU_CODENAME=xenial
```
`docker run` 就是运行容器的命令

- `-it`：这是两个参数，一个是 `-i`：交互式操作，一个是 `-t` 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。
- `--rm`：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 `docker rm`。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 --rm 可以避免浪费空间。
- `ubuntu`：这是指用 `ubuntu` 镜像为基础来启动容器。
- `bash`：放在镜像名后的是命令，这里我们希望有个交互式 `Shell`，因此用的是 `bash`。

进入容器后，我们可以在 Shell 下操作，执行任何所需的命令。这里，我们执行了 `cat /etc/os-release`，这是 Linux 常用的查看当前系统版本的命令，从返回的结果可以看到容器内是 `Ubuntu 16.04.2 LTS` 系统。

最后通过 exit 退出了这个容器。
