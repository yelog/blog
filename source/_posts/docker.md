---
title: Docker 常用命令
enlink: docker-commands
date: 2024-08-30 10:43:21
categories:
- 运维
tags:
- docker
- commands
---


# Commands
```bash
# 根据镜像启动容器
# -d 后台运行 -p 端口映射 -v 挂载目录 -e 环境变量 --name 容器名 --restart 重启策略
docker run --name some-nginx -v /some/content:/usr/share/nginx/html:ro -d nginx
# 查看正在运行的容器列表, -a 包含已经停止的容器
docker ps [-a]
# 查看指定容器的日志, --tail 50 最后50调， -f 持续查看
docker logs --tail 50 -f <container-id> 
# 命令行进行指定容器
docker exec -it <container-id> /bin/sh
# 启动容器
docker start <container-id>
# 停止容器
docker stop <container-id>
# 重启容器
docker restart <container-id>
# 删除容器
docker rm <container-id>
# 拉取镜像
docker pull nginx
# 查看本地仓库有哪些镜像
docker images
# 查询 docker 的磁盘使用情况
docker system df
# 每个容器的磁盘占用情况
docker system df -v
# 从容器中拷贝到宿主机
docker cp <container-id>:<path> . # 将 nginx 镜像保存到 nginx.tar 文件
docker save -o nginx.tar nginx
# 还原镜像
docker load -i nginx.tar
# 清除所有不在使用的资源
docker system prune -af
# 查看镜像的构建历史
docker history nginx
# 查看镜像的详细信息
docker inspect nginx
```
# 常用操作

## docker install

```bash
# 安装依赖
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# 设置 yum 源
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 查看 docker 版本
sudo yum list docker-ce --showduplicates | sort -r
# 选取一个版本进行安装
sudo yum install -y docker-ce-20.10.9
# 启动 docker 并设置开机启动
sudo systemctl enable --now docker
```

## docker uninstall

```bash
# 卸载
sudo yum remove -y docker docker-ce docker-common docker-selinux docker-engine
```

## docker 设置私服和存储地址
```bash
sudo vi /etc/docker/daemon.json
```
```json
{
  "registry-mirrors": [],
  "insecure-registries": [
    "10.188.132.44:5000",
    "10.188.132.123:5000",
    "10.176.2.207:5000"
  ],
  "data-root":"/data/docker/system",
  "debug": true,
  "experimental": false,
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "1",
    "labels": "production_status",
    "env": "os,customer"
  }
}
```

```bash
sudo systemctl restart docker
# 直接生效 daemon.json
sudo kill -SIGHUP $(pidof dockerd)
```

> 其他方法

```bash
# 创建一个挂在镜像和容器的地方，最好选存储空间大的位置
mkdir -p /data/docker/system
#如果下面位置找不到 可以通过 sudo find / -name 'docker.service' -type f 寻找 docker.service 位置
sudo vim /usr/lib/systemd/system/docker.service
# 在 ExecStart 最后添加 私服和镜像存储位置 --insecure-registry 10.176.66.20:5000 --graph /data/docker/system
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry 10.176.66.20:5000 --graph /data/docker/system
# 重载配置
sudo systemctl daemon-reload
# 重启服务
sudo systemctl restart docker
```

## 授权当前用户 docker 的执行权限
```bash
# 将当前用户添加到 docker 组内
sudo gpasswd -a $USER docker
# 刷新 docker 组配置
newgrp docker
# 重启 docker 服务
sudo systemctl restart docker
```

## docker-compose

1.自带插件

```bash
sudo yum install -y docker-compose-plugin
```

2.下载

```bash
# 下载安装docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.17.2/docker-compose-linux-x86_64" -o /usr/bin/docker-compose
# 设置执行权限
sudo chmod 777 /usr/bin/docker-compose
```



## 查看容器文件（原始启动失败的情况下）
```bash
docker run -it 10.176.2.207:5000/lemes-cloud/lemes-gateway:pgsql-master-202306181324 /bin/bash
```

docker create -it --name dumy 10.188.132.123:5000/lemes-cloud/lemes-gateway:develop-202312111536 bash
docker cp dumy:/data .

## 从镜像想宿主机复制文件
```bash
# 创建容器, 不启动
docker create -it --name dumy 10.188.132.123:5000/library/mysql2postgresql-jdbc-agent:1.0.0 bash
# 从容器内复制文件出来
docker cp dumy:/tmp/mysql2postgresql-jdbc-agent-1.0.0.jar .
# 移除容器
docker rm dumy
```

## 打包镜像并还原镜像
```bash
# 打包镜像
docker save -o lemes-web.tar 10.176.2.207:5000/lemes-cloud/lemes-web:pgsql-master-202306272212
# 还原镜像
docker load -i lemes-web.tar
```

## 修改启动容器的端口映射 modify port mapping of running container

假设我们有已经启动的容器 `my-nginx`, 目前的端口映射为 `80:80`, 现在需要修改为 `8080:80`

1. 首先通过 `docker ps | grep nginx` 查看容器的 `CONTAINER ID`
2. 找到 `docker` 的磁盘存储地址, 默认为 `/var/lib/docker/containers`, 如果修改了 `daemon.json` 中的 `data-root` 则需要修改对应的地址
3. 在 `containers` 下载找到对应的 `CONTAINER ID` 的文件夹, 编辑其下的 `hostconfig.json` 文件
4. 找到 `PortBindings` 下的 `80/tcp` 对应的 `HostPort` 目前应该是 `80`, 修改为 `8080`
5. 重启 `docker` 服务, `systemctl restart docker`

> 注意: 如果需要修改容器内的端口, 比如要修改为 `80:8080`
> 那么除了修改 `config.v2.json` 中的 `PortBindings` 的 `key` 从 `80/tcp` 到 `8080/tcp` 外
> 还需要修改 `config.v2.json` 中的 `ExposedPorts` 中的 `80/tcp` 为 `8080/tcp`

# Issues

## getting the final child's pid from pipe caused: EOF: unknown

```bash
sudo sysctl -w user.max_user_namespaces=150000
```

## failed to start daemon: Devices cgroup isn't mounted

start docker deamon faild.
```bash
$ sudo systemctl start docker
Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.`

$ journalctl -xe
-- Subject: Unit docker.socket has finished start-up
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- Unit docker.socket has finished starting up.
--
-- The start-up result is done.
May 19 10:26:34 szxlpidgapp06 systemd[1]: Starting Docker Application Container Engine...
-- Subject: Unit docker.service has begun start-up
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- Unit docker.service has begun starting up.
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.580459394+08:00" level=info msg="Starting up"
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.580654556+08:00" level=debug msg="Listener created for HTTP on fd ()"
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.580999123+08:00" level=debug msg="Golang's threads limit set to 922770"
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.582149991+08:00" level=info msg="parsed scheme: \"unix\"" module=grpc
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.582178725+08:00" level=info msg="scheme \"unix\" not registered, fallback to default scheme" module=grpc
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.582285273+08:00" level=info msg="ccResolverWrapper: sending update to cc: {[{unix:///run/containerd/containerd.sock 0  <nil>}] <nil>}" module=grpc
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.582321352+08:00" level=info msg="ClientConn switching balancer to \"pick_first\"" module=grpc
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.583565030+08:00" level=info msg="parsed scheme: \"unix\"" module=grpc
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.583585521+08:00" level=info msg="scheme \"unix\" not registered, fallback to default scheme" module=grpc
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.583602813+08:00" level=info msg="ccResolverWrapper: sending update to cc: {[{unix:///run/containerd/containerd.sock 0  <nil>}] <nil>}" module=grpc
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.583610828+08:00" level=info msg="ClientConn switching balancer to \"pick_first\"" module=grpc
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.584849300+08:00" level=debug msg="Using default logging driver json-file"
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.584898470+08:00" level=debug msg="processing event stream" module=libcontainerd namespace=plugins.moby
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.584902148+08:00" level=debug msg="[graphdriver] priority list: [btrfs zfs overlay2 aufs overlay devicemapper vfs]"
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.594035041+08:00" level=debug msg="backingFs=xfs, projectQuotaSupported=false, indexOff=\"index=off,\"" storage-driver=overlay2
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.594052376+08:00" level=info msg="[graphdriver] using prior storage driver: overlay2"
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.594062706+08:00" level=debug msg="Initialized graph driver overlay2"
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.594715135+08:00" level=warning msg="Your kernel does not support cgroup memory limit"
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.594726229+08:00" level=warning msg="Unable to find cpu cgroup in mounts"
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.594731880+08:00" level=warning msg="Unable to find blkio cgroup in mounts"
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.594737414+08:00" level=warning msg="Unable to find cpuset cgroup in mounts"
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.594741722+08:00" level=warning msg="mountpoint for pids not found"
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: time="2022-05-19T10:26:34.594868843+08:00" level=debug msg="Cleaning up old mountid : start."
May 19 10:26:34 szxlpidgapp06 dockerd[33200]: failed to start daemon: Devices cgroup isn't mounted
May 19 10:26:34 szxlpidgapp06 systemd[1]: docker.service: main process exited, code=exited, status=1/FAILURE
May 19 10:26:34 szxlpidgapp06 systemd[1]: Failed to start Docker Application Container Engine.
-- Subject: Unit docker.service has failed
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- Unit docker.service has failed.
--
-- The result is failed.
May 19 10:26:34 szxlpidgapp06 systemd[1]: Unit docker.service entered failed state.
May 19 10:26:34 szxlpidgapp06 systemd[1]: docker.service failed.
```

**failed to start daemon: Devices cgroup isn't mounted** is very important message.

We check docker config by `https://raw.githubusercontent.com/moby/moby/master/contrib/check-config.sh`

```bash
warning: /proc/config.gz does not exist, searching other paths for kernel config ...
info: reading kernel config from /boot/config-3.10.0-1160.62.1.el7.x86_64 ...

Generally Necessary:
- cgroup hierarchy: nonexistent??
    (see https://github.com/tianon/cgroupfs-mount)
- CONFIG_NAMESPACES: enabled
- CONFIG_NET_NS: enabled
- CONFIG_PID_NS: enabled
- CONFIG_IPC_NS: enabled
- CONFIG_UTS_NS: enabled
- CONFIG_CGROUPS: enabled
- CONFIG_CGROUP_CPUACCT: enabled
- CONFIG_CGROUP_DEVICE: enabled
- CONFIG_CGROUP_FREEZER: enabled
- CONFIG_CGROUP_SCHED: enabled
- CONFIG_CPUSETS: enabled
- CONFIG_MEMCG: enabled
- CONFIG_KEYS: enabled
- CONFIG_VETH: enabled (as module)
- CONFIG_BRIDGE: enabled (as module)
- CONFIG_BRIDGE_NETFILTER: enabled (as module)
- CONFIG_IP_NF_FILTER: enabled (as module)
- CONFIG_IP_NF_TARGET_MASQUERADE: enabled (as module)
- CONFIG_NETFILTER_XT_MATCH_ADDRTYPE: enabled (as module)
- CONFIG_NETFILTER_XT_MATCH_CONNTRACK: enabled (as module)
- CONFIG_NETFILTER_XT_MATCH_IPVS: enabled (as module)
- CONFIG_NETFILTER_XT_MARK: enabled (as module)
- CONFIG_IP_NF_NAT: enabled (as module)
- CONFIG_NF_NAT: enabled (as module)
- CONFIG_POSIX_MQUEUE: enabled
- CONFIG_DEVPTS_MULTIPLE_INSTANCES: enabled
- CONFIG_NF_NAT_IPV4: enabled (as module)
- CONFIG_NF_NAT_NEEDED: enabled

```

**cgroup hierarchy: nonexistent??**, so we mounted cgroup by following script

> reference url: https://github.com/tianon/cgroupfs-mount/blob/master/cgroupfs-mount

```bash
#!/bin/sh
# Copyright 2011 Canonical, Inc
#           2014 Tianon Gravi
# Author: Serge Hallyn <serge.hallyn@canonical.com>
#         Tianon Gravi <tianon@debian.org>
set -e

# for simplicity this script provides no flexibility

# if cgroup is mounted by fstab, don't run
# don't get too smart - bail on any uncommented entry with 'cgroup' in it
if grep -v '^#' /etc/fstab | grep -q cgroup; then
 echo 'cgroups mounted from fstab, not mounting /sys/fs/cgroup'
 exit 0
fi

# kernel provides cgroups?
if [ ! -e /proc/cgroups ]; then
 exit 0
fi

# if we don't even have the directory we need, something else must be wrong
if [ ! -d /sys/fs/cgroup ]; then
 exit 0
fi

# mount /sys/fs/cgroup if not already done
if ! mountpoint -q /sys/fs/cgroup; then
 mount -t tmpfs -o uid=0,gid=0,mode=0755 cgroup /sys/fs/cgroup
fi

cd /sys/fs/cgroup

# get/mount list of enabled cgroup controllers
for sys in $(awk '!/^#/ { if ($4 == 1) print $1 }' /proc/cgroups); do
 mkdir -p $sys
 if ! mountpoint -q $sys; then
  if ! mount -n -t cgroup -o $sys cgroup $sys; then
   rmdir $sys || true
  fi
 fi
done

# example /proc/cgroups:
#  #subsys_name hierarchy num_cgroups enabled
#  cpuset 2 3 1
#  cpu 3 3 1
#  cpuacct 4 3 1
#  memory 5 3 0
#  devices 6 3 1
#  freezer 7 3 1
#  blkio 8 3 1

# enable cgroups memory hierarchy, like systemd does (and lxc/docker desires)
# https://github.com/systemd/systemd/blob/v245/src/core/cgroup.c#L2983
# https://bugs.debian.org/940713
if [ -e /sys/fs/cgroup/memory/memory.use_hierarchy ]; then
 echo 1 > /sys/fs/cgroup/memory/memory.use_hierarchy
fi

exit 0
```

## systemctl start docker 卡住

通过 `systemctl status docker` 查看状态, 发现并没有关闭成功

我们可以通过如下脚本, 进行强制关闭

```bash
while true; do
    kill -9 $(pidof containerd) $(pidof dockerd)
    if [[ ! "$?" = "0" ]]; then
        break
    fi
done
```

然后再次启动 `docker` 即可

# Link

1. [docker centos rpm](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)

