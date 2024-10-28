---
title: k8s 集群搭建和使用及常见问题处理
enlink: k8s-cluster-and-common-problems
date: 2024-08-30 11:16:06
categories:
- 运维
tags:
- k8s
- docker
- container
---

## 简介
kubernetes: k8s 是谷歌在2014年开源的容器化集群管理系统
- **Pod** Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元
- 网络控制器
- ApiServer: Uniform interface for access by all services
- ControllerManager: Maintaining Copy Expectations

- `kubeadm`: the command to bootstrap the cluster.
- `kubelet`: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
- `kubectl`: the command line util to talk to your cluster.

## 搭建方式
### kubeadmin
[kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)

#### 环境安装
master和nodes 均需要执行一下步骤进行安装
```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

# Set SELinux in permissive mode (effectively disabling it)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# turn off swap
swapoff -a # 临时
vim /etc/fstab # 永久

sudo yum install -y kubelet-1.20.5 kubeadm-1.20.5 kubectl-1.20.5 --disableexcludes=kubernetes
sudo systemctl enable --now kubelet

```
>uninstall command `sudo yum remove -y kubelet kubeadm kubectl`

#### 集群部署
假设 master 所在机器为 10.176.66.58
```bash
# 提前拉去镜像
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers
kubeadm config images pull --image-repository 10.176.66.20/google_containers

# ping 不通 service 和 pod 的dns
sudo kubeadm init \
--apiserver-advertise-address=10.114.130.3 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.20.5 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16
# 使用私服
sudo kubeadm init \
--apiserver-advertise-address=10.114.130.3 \
--image-repository 10.176.66.20/google_containers \
--kubernetes-version v1.20.5 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16

# 在 nodes 上直接上面生成的命令，加入集群，如下
kubeadm join 10.176.66.58:6443 --token 7opg66.gcmdavb2vxiliytp \
    --discovery-token-ca-cert-hash sha256:ecb8d4930ac8489c1196560612afa1736dddf7be25244a50e64c82dca9bb2644

# 使用 kubectl 工具
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
# 查看 node 节点加入情况
kubectl get nodes
# 查看 pod 情况
kubectl get pods -o wide
# 安装 pod 网络插件 CNI
kubectl apply -f  https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

> master 重新生成加入命令: `kubeadm token create --print-join-command` `kubeadm token list`
>
> master/node 退出集群 `kubeadm reset`

### 二进制安装
待更新

## YAML 支持的数据结构
```bash
# 查看所有 pod 和 service
kubectl get pod,svc
# 开启检测, 有变化打印
kubectl get pod -w
# 查看 所有组件
kubectl get pods --all-namespaces -o wide
kubectl get componentstatuses
# 查看某个服务情况
kubectl describe pods -n kube-system coredns-7f89b7bc75-hsjdl
# 查看某个 pod 信息
kubectl describe pod <pod-name>
# 查看 pod 下，某个容器日志
kubectl logs <pod-name> -c <container-name>
# 删除某个 pod
kubectl delete pod <pod-name>
# 删除所有 deployment
kubectl delete deployment --all
# 删除所有 pod
kubectl delete pod --all
# 根据配置文件创建对象
kubectl create -f nginx.yaml
# 根据配置文件删除对象 (猜测是根据 meta 标识的唯一对象进行删除)
kubectl delete -f nginx.yaml
# 更新对象配置
kubectl replace -f nginx.yaml

# 进入容器
kubectl exec -it nacos-2 -- /bin/bash
```

## 常用命令

```bash
# 更新
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
# 查看回滚历史
kubectl rollout history deployment.v1.apps/nginx-deployment
# 回滚到上个版本
kubectl rollout undo deployment.v1.apps/nginx-deployment
# 回滚到指定版本
kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2
# 缩放
kubectl scale deployment.v1.apps/nginx-deployment --replicas=10
# 自动伸缩
kubectl autoscale deployment.v1.apps/nginx-deployment --min=10 --max=15 --cpu-percent=80
# 查看各 pod/container cpu和memory 的占用量
kubectl top pod
kubectl top pod test-huishi-server-6f875487d7-9rzpd
kubectl top pod | grep lemes-service-common
# 查看节点的内存和cpu占用情况
kubectl top nodes
sudo systemctl restart docker
```


## 使用Rancher
### deploy nfs
```bash
sudo yum install -y nfs-utils rpcbind
sudo mkdir -p /data/nfs
sudo sh -c "sudo echo '/data/nfs *(rw,sync,no_root_squash)' >> /etc/exports"
sudo systemctl enable --now nfs
sudo systemctl enable --now rpcbind
```
### 部署 ceph (未验证)
```bash
sudo cat > /etc/yum.repos.d/ceph.repo << EOF
[ceph-norch]
name=ceph-norch
baseurl=https://mirrors.aliyun.com/ceph/rpm-nautilus/el7/noarch/
enabled=1
gpgcheck=0
[ceph-x86_64]
name=ceph-x86_64
baseurl=https://mirrors.aliyun.com/ceph/rpm-nautilus/el7/x86_64/
enabled=1
gpgcheck=0
EOF
sudo yum install ceph-common
```
### 部署 redis
```bash
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 {end}')
kubectl exec -it redis-cluster-0 -- redis-cli cluster nodes
```
### 修改 ingress 端口
```yaml
- --http-port=81
- --https-port=8443
```

### 集成 ceph
#### 安装 ceph
```bash
# 安装 ceph
sudo yum -y install ceph-common
# 创建pool
ceph osd pool create kubernetes 16 16
# 初始化pool
rbd pool init kubernetes
# 创建块文件
rbd create -p kubernetes --image-feature layering rbd.img --size 10G

mkdir -p /data/ceph/sdb
mkdir -p /data/ceph/sdc

# 查看 lv path
sudo vgscan
sudo vgdisplay -v datavg


ceph-deploy osd create --data /dev/datavg/lv_data whulpdpms01
ceph-deploy osd create --data /dev/datavg/lv_data whulpdpms02
ceph-deploy osd create --data /dev/datavg/lv_data whulpdpms03
```



## Problem 问题记录
### Snippet directives are disabled by the Ingress administrator
当应用如下配置时报错题目问题
```yaml
# 暴露服务
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: lemes-gateway-ig
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "600"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "600"
    nginx.ingress.kubernetes.io/proxy-body-size: "600m"
    nginx.ingress.kubernetes.io/configuration-snippet: |
      more_set_headers "Host $host";
      more_set_headers "X-Forwarded-Proto $scheme";
      more_set_headers "X-Forwarded-For $proxy_add_x_forwarded_for";
      more_set_headers "X-Real-IP $remote_addr";
spec:
  rules:
    - http:
        paths:
          - path: /lemes-api(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: lemes-gateway-svc
                port:
                  number: 80
```
原因与解决方案: https://github.com/kubernetes/ingress-nginx/issues/7837
1. 编辑 ingress-nginx
```bash
kubectl edit configmap -n ingress-nginx ingress-nginx-controller
```
2. 如果有如下内容, 删除或修改 false 为 true
```yaml
data:
  allow-snippet-annotations: "false"
```
### ingress 前多层反向代理穿透， 获取 real ip

升级 集群名-> System -> 资源 -> 配置映射 中的 `ingress-nginx-controller`
添加如下键值对
```yaml
compute-full-forwarded-for: true
forwarded-for-header: X-Forwarded-For
use-forwarded-headers:true
```
实时生效


### 容器删不掉
docker stop/kill/rm -f 都不好使
```bash
# 找到进程
$ ps axo stat,ppid,pid,comm | grep -w defunct
Zl   19653 19679 java <defunct>
# 找到父进程
$ ps -f 19679
UID        PID  PPID  C STIME TTY      STAT   TIME CMD
root     19653 19635  0 11:02 ?        Ss     0:00 [docker-startup.]
$ sudo kill -9 19635
$ sudo systemctl restart docker
```

### 强制删除所有 terminating 的 pod
```bash
kubectl get pods | grep Terminating | awk '{print $1}' | xargs -I {} kubectl delete pod {} --force --grace-period=0
kubectl delete pod nginx-ingress-controller-lbftg --force --grace-period=0
```

### too many open files
```bash
sudo vi /etc/sysctl.conf
# 添加
fs.file-max=9000000
fs.inotify.max_user_instances = 1000000
fs.inotify.max_user_watches = 1000000

sudo sysctl -p
sudo systemctl restart docker
```

### 删除挂载卷
```bash
# 查看挂载卷
cat /proc/mounts |grep "docker"
# 显示
/dev/mapper/centos-root /var/lib/docker/overlay xfs rw,seclabel,relatime,attr2,inode64,noquota 0 0 overlay /var/lib/docker/overlay/xxxxxxxxxx
# 取消挂载
umount /var/lib/docker/overlay/xxxxxxxxxxx
# 批量取消挂载
sudo umount `cat /proc/mounts |grep "docker"|awk '{print $2}'`
```

### docker 假死
```bash
[lemes@slt6dhqgxev ~]$ docker ps
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

# journalctl -u docker.service
accept4: too many open files
```
**问题解析**
文件数创建上限

**问题解决**
```bash
sudo vi /etc/sysctl.conf
# 添加
fs.file-max = 9000000
fs.inotify.max_user_instances = 1000000
fs.inotify.max_user_watches = 1000000
# 生效
sudo sysctl -p
# 重启 docker 服务
sudo systemctl restart docker
# 启动所有关闭的容器
docker start $(docker ps -a | awk '{ print $1}' | tail -n +2)
```

### rancher 应用 HOST-PATH 时 使用 subPath / subPathExpr 日志没有写入宿主机

问题讨论：[volume hostpath with subpath](https://github.com/rancher/rancher/issues/14836)

问题原因：
映射到了 kubectl 容器内
`docker exec -it $(docker ps -aq --filter "name=kubelet") /bin/sh`

### k8s 集群内 dns 生效(rancher)
修改了宿主机的 dns 后，需要重启 docker 才能全体生效
```bash
sudo systemctl restart docker
```

### 报错 x509: certificate is not valid for any names, but wanted to match ingress-nginx-controller-admission.ingress-nginx.svc
```bash
kubectl delete -A ValidatingWebhookConfiguration foobar-ingress-nginx-admission
```

### network plugin is not ready: cni config uninitialized
网络框架一直安装不上, 根据 `docker logs -f kubelet` 日志查看, 网络插件安装时报 `diskpress` 被放逐
问题: /data 磁盘空间不足
解决方案: 释放磁盘空间解决

### Pod ephemeral local storage usage exceeds the total limit of containers 4Gi.
现象: pod被驱逐, 报错如题
问题: 使用的临时容量超过了节点限制(在此节点上)

2023-08-11 21:22 再次出现这个问题

经[查询资料](https://access.redhat.com/solutions/4367311)， 可能是由于没有限制容器日志造成的

南方厂工厂的 `sudo vi /etc/docker/daemon.json` 配置确实如下
```json
{
  "registry-mirrors": [],
  "insecure-registries": [
    "10.176.66.20:5000",
    "10.188.132.44:5000",
    "10.188.132.123:5000",
    "10.176.2.207:5000"
  ],
  "data-root":"/data/docker/system",
  "debug": true,
  "experimental": false,
}
```

改为如下, 限制每个容器只能保留10m的日志
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

2023-08-25 16:41 再次出现问题
发现 tingyun 使用的 `emptyDir` 中， 一直在写入日志，导致占用临时空间

```bash
# 查询log日志总数
sudo find /data/docker/system/containers/ -name "*-json.log" | xargs sudo ls -l | awk '{print $5}' | awk '{sum+=$1}END{print sum}'
sudo find /data/docker/system/containers/ -name "*-json.log" | xargs sudo rm -rf
```

2023-08-31 smt-wh 和 smt-tjsc 都出现了这个问题 `The node was low on resource: ephemeral-storage. Container lemes-service-wh-report was using 1936Ki, which exceeds its request of 0.`

超出了0，就不是有限制， 而是当前磁盘已经达到了 85%，造成了 pod 驱逐


### 导入 lemes-web 出现问题
Error from server (InternalError): error when creating "management-state/tmp/yaml-397511040": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses?timeout=10s": x509: certificate is not valid for any names, but wanted to match ingress-nginx-controller-admission.ingress-nginx.svc

2024-10-28 导入 moss-web 时, 再次遇到:
Internal error occurred: failed calling webhook \"validate.nginx.ingress.kubernetes.io\": Post \"https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1beta1/ingresses?timeout=10s\": x509: certificate is valid for localhost, rancher.cattle-system, not ingress-nginx-controller-admission.ingress-nginx.svc

解决方案: https://github.com/kubernetes/ingress-nginx/issues/5968#issuecomment-782092413
```bash
# Find name of the ingress-nginx-admission resource
kubectl get -A ValidatingWebhookConfiguration
# Delete it
kubectl delete -A ValidatingWebhookConfiguration <name>
# Example:
kubectl delete -A ValidatingWebhookConfiguration foobar-ingress-nginx-admission
```

### node 节点报错 PLEG is not healthy: pleg was last seen active 7m20.510472824s ago; threshold is 3m0s

有个 `issue` 问题很像 [PLEG is not healthy K8 1.20.4/Ubuntu 20.04](https://github.com/rancher/rancher/issues/31793#issuecomment-911143593)

根据 `minchieh-fay` 老哥的回答,是 `runc` 的 `runc-1.0.0-rc93` 这个版本有问题

可以通过 `docker version` 来查看 `runc` 的版本, 确实是 `runc-1.0.0-rc93`, 按照如下方式进行离线升级

1. 到 `runc` 的 [github release](https://github.com/opencontainers/runc/releases/) 找到升级的版本, 我选的是 `1.1.4`, 选择 `runc.amd64` 进行下载
2. 上传到服务器, 执行 `mv runc.amd64 runc && chmod +x runc` 进行重命名和赋予执行权限
3. 备份原来的 `runc` 文件, `mv /usr/bin/runc /usr/bin/runc.bak`
4. 停止 `docker` 服务, `systemctl stop docker`
5. 移动新的 `runc` 文件到 `/usr/bin/` 目录下, `mv runc /usr/bin/runc`
6. 启动 `docker` 服务, `systemctl start docker`
7. 执行 `docker version` 查看 `runc` 版本, 确认升级成功






