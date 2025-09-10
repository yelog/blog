---
title: Centos7 常用命令
enlink: centos7-commands
date: 2024-08-30 10:46:15
categories:
- 运维
tags:
- linux
- centos7
- yum
---

# Software installation

## yum

```bash
# 就是把服务器的包信息下载到本地电脑缓存起来，makecache建立一个缓存，以后用install时就在缓存中搜索，提高了速度。
yum makecache
# 不用上网检索就能查找软件信息
yum -C search git
# 清理缓存
yum clean all
# 添加 Extra Packages for Enterprise Linux 源，安装后就可以在 /etc/yum.repos.d/ 看到 epel 源信息
yum install -y epel-release
# 接下来以 ansible 这个软件为例
yum install ansible     # 安装
yum reinstall ansible   # 重新安装
yum upgrade ansible     # 升级
yum info ansible        # 查看软件信息
yum remove ansible      # 删除
yum update              # 升级所有包同时也升级软件和系统内核(慎用
yum upgrade             # 升级所有包，但不升级软件和系统内核
yum list ansible        # 查看是否安装
yum list all            # 列出所有软件
yum list installed      # 列出所有安装的软件
yum list available      # 列出所有可以安装的软件
yum search ansible      # 搜索软件信息
yum whatprovides rm     # yum源中查找包含rm的软件包
yum check-update        # 查看可更新的软件列表
rpm -ql ansible | more  # 查看 ansible 的安装位置


# 换源
## 备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
## 下载新的配置文件
### CentOS 6
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo
### CentOS 7
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
### CentOS 8
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
## 生成缓存
yum makecache
```

## ansible

```bash
# 修改要管理的机器
vim /etc/ansible/hosts
[webservers]
192.168.1.100
192.168.1.101
```

## man

```bash
# 在 .bashrc 中放入，可以高亮man手册
function man()
{
    env \
    LESS_TERMCAP_mb=$(printf "\e[1;31m") \
    LESS_TERMCAP_md=$(printf "\e[1;31m") \
    LESS_TERMCAP_me=$(printf "\e[0m") \
    LESS_TERMCAP_se=$(printf "\e[0m") \
    LESS_TERMCAP_so=$(printf "\e[1;44;33m") \
    LESS_TERMCAP_ue=$(printf "\e[0m") \
    LESS_TERMCAP_us=$(printf "\e[1;32m") \
    man "$@"
}
```

## zsh/on-my-zsh

```bash
# 安装 zsh git
yum install -y zsh git
# 设置默认shell为 zsh
chsh -s /bin/zsh
# 安装 on-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
# 复制配置
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc

# 手动安装 zsh http://zsh.sourceforge.net/Arc/source.html
yum -y install gcc perl-ExtUtils-MakeMaker ncurses-devel
# 编译安装
tar xvf zsh-5.8.tar.xz
cd zsh-5.8
./configure
make && make install
# 将zsh加入/etc/shells
vim /etc/shells # 添加：/usr/local/bin/zsh
```

## git

```bash
sudo yum install -y https://packages.endpointdev.com/rhel/7/os/x86_64/endpoint-repo.x86_64.rpm
sudo yum install -y git
```

## neovim

```bash
# Download source code
git clone https://github.com/neovim/neovim.git
# install cmake and dependency
sudo yum install -y cmake gcc-c++ libtool unzip
# compile with cmake
make CMAKE_BUILD_TYPE=Release
# install
sudo make install
# fix error: Failed to load python3 host
pip3 install --upgrade --force-reinstall neovim
```

## neofetch

```bash
dnf copr enable -y konimex/neofetch
dnf install -y neofetch
```

## rainbarf

```bash
# Download source code
git clone https://github.com/creaktive/rainbarf.git
# install dependency
yum install -y perl-Module-Build perl-Test-Simple
# install
perl Build.PL
./Build test
./Build install
```

## node

Mange node using [nvm](https://github.com/nvm-sh/nvm)
```bash
# installation
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash
# set path，put following content into ~/.bashrc
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
# install latest node
nvm install node
```

## docker

## docker install

```bash
# 安装依赖
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# 设置 yum 源
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 查看 docker 版本
sudo yum list docker-ce --showduplicates | sort -r
# 选取一个版本进行安装
sudo yum install -y docker-ce-28.2.2
# 启动 docker 并设置开机启动
sudo systemctl enable --now docker
```

## docker install offline

去 [docker 官网](https://download.docker.com/linux/static/stable/x86_64/) 下载对应的 docker 离线包, 并上传到服务器上

```bash
# 解压 docker 离线包
tar -xzvf Docker\ 20.10.24.tgz
# 将解压后的文件移动到 /usr/bin 目录下
sudo cp docker/* /usr/bin/
```

通过 `sudo vim /usr/lib/systemd/system/containerd.service` 创建 `containerd.service` 文件, 内容如下

```bash
# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target dbus.service

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
```


```bash
# 启动并设置开机启动
sudo systemctl enable --now containerd
# 查看状态
sudo systemctl status containerd
```

通过 `sudo vim /usr/lib/systemd/system/docker.service` 创建 `docker.service` 文件, 内容如下


```yaml
[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.com
After=network.target docker.socket
[Service]
Type=notify
EnvironmentFile=-/run/flannel/docker
WorkingDirectory=/usr/local/bin
ExecStart=/usr/bin/dockerd \
                -H tcp://0.0.0.0:4243 \
                -H unix:///var/run/docker.sock \
                --selinux-enabled=false
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

```bash
# 启动并设置开机启动
sudo systemctl enable --now docker
# 检查状态
sudo systemctl status docker
```


## docker uninstall

```bash
# 卸载
sudo yum remove -y docker docker-ce docker-common docker-selinux docker-engine
```

## docker-compose

[[docker#docker-compose]]

## rancher

```bash
sudo docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  --name=rancher-2.5 \
  10.188.132.123:5000/rancher/rancher:v2.5.12


docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  --name=rancher \
  rancher/rancher:v2.5.17

docker restart lemes-rancher-2.5
docker stop lemes-rancher-2.5
docker start lemes-rancher-2.5

docker run -d --restart=unless-stopped \
  -p 9080:80 -p 8443:443 \
  --privileged \
  --name=lemes-rancher-2.5-prod \
  10.188.132.123:5000/rancher/rancher:v2.5.12

# 生产
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  --name=lemes-rancher-2.5-prod \
  10.188.132.44:5000/rancher/rancher:v2.5.12
docker run -d --restart=unless-stopped \
  -p 9080:80 -p 8443:443 \
  --privileged \
  --name=lemes-rancher-2.5-prod \
  10.188.132.44:5000/rancher/rancher:v2.5.12

docker run -d --restart=unless-stopped \
  -p 9080:80 -p 9443:443 \
  --privileged \
  --name=lemes-rancher-2.5-prod \
  rancher/rancher:v2.5.12

docker run -d --restart=unless-stopped \
  -p 8080:80 -p 8083:443 \
  --privileged \
  --name=lemes-rancher-2.5-prod \
  10.176.2.207:5000/rancher/rancher:v2.5.12
```

## Harbor

> 前提: 需要先安装 docker & docker-compose

复制最新的包的链接: https://github.com/goharbor/harbor/releases
```bash
wget https://github.com/goharbor/harbor/releases/download/v2.3.1/harbor-offline-installer-v2.3.1.tgz
tar -zxf harbor-offline-installer-v2.3.1.tgz -C /data/docker/harbor
sudo chown -R lemes:lemes /data/docker/harbor
cd /data/docker/harbor/harbor
cp harbor.yml.tmpl harbor.yml
vi harbor.yml
sudo su root
export PATH=$PATH:/usr/local/bin
./install.sh
```

```yaml
# Configuration file of Harbor

# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname: 10.176.2.207

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 5000

# https related config
#https:
#  # https port for harbor, default is 443
#  port: 443
#  # The path of cert and key files for nginx
#  certificate: /your/certificate/path
#  private_key: /your/private/key/path

# # Uncomment following will enable tls communication between all harbor components
# internal_tls:
#   # set enabled to true means internal tls is enabled
#   enabled: true
#   # put your cert and key files on dir
#   dir: /etc/harbor/tls/internal

# Uncomment external_url if you want to enable external proxy
# And when it enabled the hostname will no longer used
# external_url: https://reg.mydomain.com:8433

# The initial password of Harbor admin
# It only works in first time to install harbor
# Remember Change the admin password from UI after launching Harbor.
harbor_admin_password: Lenovo2021

# Harbor DB configuration
database:
  # The password for the root user of Harbor DB. Change this before any production use.
  password: root123
  # The maximum number of connections in the idle connection pool. If it <=0, no idle connections are retained.
  max_idle_conns: 100
  # The maximum number of open connections to the database. If it <= 0, then there is no limit on the number of open connections.
  # Note: the default number of connections is 1024 for postgres of harbor.
  max_open_conns: 900

# The default data volume
data_volume: /data/docker/harbor

# Harbor Storage settings by default is using /data dir on local filesystem
# Uncomment storage_service setting If you want to using external storage
# storage_service:
#   # ca_bundle is the path to the custom root ca certificate, which will be injected into the truststore
#   # of registry's and chart repository's containers.  This is usually needed when the user hosts a internal storage with self signed certificate.
#   ca_bundle:

#   # storage backend, default is filesystem, options include filesystem, azure, gcs, s3, swift and oss
#   # for more info about this configuration please refer https://docs.docker.com/registry/configuration/
#   filesystem:
#     maxthreads: 100
#   # set disable to true when you want to disable registry redirect
#   redirect:
#     disabled: false

# Trivy configuration
#
# Trivy DB contains vulnerability information from NVD, Red Hat, and many other upstream vulnerability databases.
# It is downloaded by Trivy from the GitHub release page https://github.com/aquasecurity/trivy-db/releases and cached
# in the local file system. In addition, the database contains the update timestamp so Trivy can detect whether it
# should download a newer version from the Internet or use the cached one. Currently, the database is updated every
# 12 hours and published as a new release to GitHub.
trivy:
  # ignoreUnfixed The flag to display only fixed vulnerabilities
  ignore_unfixed: false
  # skipUpdate The flag to enable or disable Trivy DB downloads from GitHub
  #
  # You might want to enable this flag in test or CI/CD environments to avoid GitHub rate limiting issues.
  # If the flag is enabled you have to download the `trivy-offline.tar.gz` archive manually, extract `trivy.db` and
  # `metadata.json` files and mount them in the `/home/scanner/.cache/trivy/db` path.
  skip_update: false
  #
  # The offline_scan option prevents Trivy from sending API requests to identify dependencies.
  # Scanning JAR files and pom.xml may require Internet access for better detection, but this option tries to avoid it.
  # For example, the offline mode will not try to resolve transitive dependencies in pom.xml when the dependency doesn't
  # exist in the local repositories. It means a number of detected vulnerabilities might be fewer in offline mode.
  # It would work if all the dependencies are in local.
  # This option doesn’t affect DB download. You need to specify "skip-update" as well as "offline-scan" in an air-gapped environment.
  offline_scan: false
  #
  # insecure The flag to skip verifying registry certificate
  insecure: false
  # github_token The GitHub access token to download Trivy DB
  #
  # Anonymous downloads from GitHub are subject to the limit of 60 requests per hour. Normally such rate limit is enough
  # for production operations. If, for any reason, it's not enough, you could increase the rate limit to 5000
  # requests per hour by specifying the GitHub access token. For more details on GitHub rate limiting please consult
  # https://developer.github.com/v3/#rate-limiting
  #
  # You can create a GitHub token by following the instructions in
  # https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line
  #
  # github_token: xxx

jobservice:
  # Maximum number of job workers in job service
  max_job_workers: 10

notification:
  # Maximum retry count for webhook job
  webhook_job_max_retry: 10

chart:
  # Change the value of absolute_url to enabled can enable absolute url in chart
  absolute_url: disabled

# Log configurations
log:
  # options are debug, info, warning, error, fatal
  level: info
  # configs for logs in local storage
  local:
    # Log files are rotated log_rotate_count times before being removed. If count is 0, old versions are removed rather than rotated.
    rotate_count: 50
    # Log files are rotated only if they grow bigger than log_rotate_size bytes. If size is followed by k, the size is assumed to be in kilobytes.
    # If the M is used, the size is in megabytes, and if G is used, the size is in gigabytes. So size 100, size 100k, size 100M and size 100G
    # are all valid.
    rotate_size: 200M
    # The directory on your host that store log
    location: /data/docker/harbor/log

  # Uncomment following lines to enable external syslog endpoint.
  # external_endpoint:
  #   # protocol used to transmit log to external endpoint, options is tcp or udp
  #   protocol: tcp
  #   # The host of external endpoint
  #   host: localhost
  #   # Port of external endpoint
  #   port: 5140

#This attribute is for migrator to detect the version of the .cfg file, DO NOT MODIFY!
_version: 2.6.0

# Uncomment external_database if using external database.
# external_database:
#   harbor:
#     host: harbor_db_host
#     port: harbor_db_port
#     db_name: harbor_db_name
#     username: harbor_db_username
#     password: harbor_db_password
#     ssl_mode: disable
#     max_idle_conns: 2
#     max_open_conns: 0
#   notary_signer:
#     host: notary_signer_db_host
#     port: notary_signer_db_port
#     db_name: notary_signer_db_name
#     username: notary_signer_db_username
#     password: notary_signer_db_password
#     ssl_mode: disable
#   notary_server:
#     host: notary_server_db_host
#     port: notary_server_db_port
#     db_name: notary_server_db_name
#     username: notary_server_db_username
#     password: notary_server_db_password
#     ssl_mode: disable

# Uncomment external_redis if using external Redis server
# external_redis:
#   # support redis, redis+sentinel
#   # host for redis: <host_redis>:<port_redis>
#   # host for redis+sentinel:
#   #  <host_sentinel1>:<port_sentinel1>,<host_sentinel2>:<port_sentinel2>,<host_sentinel3>:<port_sentinel3>
#   host: redis:6379
#   password: 
#   # sentinel_master_set must be set to support redis+sentinel
#   #sentinel_master_set:
#   # db_index 0 is for core, it's unchangeable
#   registry_db_index: 1
#   jobservice_db_index: 2
#   chartmuseum_db_index: 3
#   trivy_db_index: 5
#   idle_timeout_seconds: 30

# Uncomment uaa for trusting the certificate of uaa instance that is hosted via self-signed cert.
# uaa:
#   ca_file: /path/to/ca

# Global proxy
# Config http proxy for components, e.g. http://my.proxy.com:3128
# Components doesn't need to connect to each others via http proxy.
# Remove component from `components` array if want disable proxy
# for it. If you want use proxy for replication, MUST enable proxy
# for core and jobservice, and set `http_proxy` and `https_proxy`.
# Add domain to the `no_proxy` field, when you want disable proxy
# for some special registry.
proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - trivy

# metric:
#   enabled: false
#   port: 9090
#   path: /metrics

# Trace related config
# only can enable one trace provider(jaeger or otel) at the same time,
# and when using jaeger as provider, can only enable it with agent mode or collector mode.
# if using jaeger collector mode, uncomment endpoint and uncomment username, password if needed
# if using jaeger agetn mode uncomment agent_host and agent_port
# trace:
#   enabled: true
#   # set sample_rate to 1 if you wanna sampling 100% of trace data; set 0.5 if you wanna sampling 50% of trace data, and so forth
#   sample_rate: 1
#   # # namespace used to differenciate different harbor services
#   # namespace:
#   # # attributes is a key value dict contains user defined attributes used to initialize trace provider
#   # attributes:
#   #   application: harbor
#   # # jaeger should be 1.26 or newer.
#   # jaeger:
#   #   endpoint: http://hostname:14268/api/traces
#   #   username:
#   #   password:
#   #   agent_host: hostname
#   #   # export trace data by jaeger.thrift in compact mode
#   #   agent_port: 6831
#   # otel:
#   #   endpoint: hostname:4318
#   #   url_path: /v1/traces
#   #   compression: false
#   #   insecure: true
#   #   timeout: 10s

# enable purge _upload directories
upload_purging:
  enabled: true
  # remove files in _upload directories which exist for a period of time, default is one week.
  age: 168h
  # the interval of the purge operations
  interval: 24h
  dryrun: false

# cache layer configurations
# If this feature enabled, harbor will cache the resource
# `project/project_metadata/repository/artifact/manifest` in the redis
# which can especially help to improve the performance of high concurrent
# manifest pulling.
# NOTICE
# If you are deploying Harbor in HA mode, make sure that all the harbor
# instances have the same behaviour, all with caching enabled or disabled,
# otherwise it can lead to potential data inconsistency.
cache:
  # not enabled by default
  enabled: false
  # keep cache for one day by default
  expire_hours: 24
```
```bash
# 阻止 vim 样式穿透
```


## nexus

```bash
# create dir of nexus
sudo mkdir /data/nexus-data && sudo chown -R 200 /data/nexus-data
docker run -d -p 8081:8081 --name nexus -v /data/nexus-data:/nexus-data 10.188.132.123:5000/library/sonatype/nexus3:3.63.0
```

## jenkins

修改启动用户，默认 anonymous 在 jenkins 脚本中没有权限创建文件

```bash
sudo vi /etc/sysconfig/jenkins
# 找到如下内容，修改后面的用户为有权限的用户
JENKINS_USER="lemes"
# 重启 jenkins
service jenkins restart
```

## jenkins-docker

```bash
docker run -d --name jenkins -p 9080:8080 10.188.132.44:5000/library/jenkins/jenkins:2.426.2-lts-jdk17
sudo mkdir -p /data/jenkins_home
sudo chown -R 1000:1000 /data/jenkins_home

docker run -d --name jenkins -p 9080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /data/jenkins_home:/var/jenkins_home \
  10.188.132.44:5000/library/jenkins/jenkins:2.426.3-lts-jdk17-dind

docker run -d --name jenkins -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /data/jenkins_home:/var/jenkins_home \
  10.188.132.44:5000/library/jenkins/jenkins:2.426.3-lts-jdk17-dind

docker run -d --name jenkins -p 9080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  10.188.132.44:5000/library/jenkins/jenkins:2.426.3-lts-jdk17-dind

docker run -d --name jenkins -p 9080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  10.188.132.44:5000/library/jenkins/jenkins:2.426.3-lts-jdk17-dind

docker run -d --name jenkins -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  10.188.132.44:5000/library/jenkins/jenkins:2.426.3-lts-jdk17-dind-plugin

docker run -d --name jenkins -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /data/jenkins_home:/var/jenkins_home \
  jenkins:2.426.3-lts-jdk17-dind

docker run \
  --rm \
  -u root \
  -p 8080:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  jenkinsci/blueocean
```
- jenkins plugin




## nginx

```bash
sudo yum install -y epel-release
sudo yum -y install nginx # 安装 nginx
sudo yum remove nginx  # 卸载 nginx
```

## keepalived

```bash
# 安装 keepalived
sudo yum install -y keepalived
```


## python

Download latest installation package from [https://www.python.org/downloads/source/](https://www.python.org/downloads/source/)
```bash
# 安装依赖&编译工具
yum -y groupinstall "Development tools"
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
yum install -y libffi-devel zlib1g-dev
yum install zlib* -y

# 下载安装包
wget https://www.python.org/ftp/python/3.9.1/Python-3.9.1.tgz

# 解压
tar -xvf Python-3.9.1.tgz
# 创建编译目录
mkdir /usr/local/python3

# 编译
./configure --prefix=/usr/local/python3 --enable-optimizations --with-ssl 
make && make install

# 创建软连接
ln -s /usr/local/python3/bin/python3 /usr/local/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/local/bin/pip3

# 验证
python3 -V
pip3 -V
```

## gcc8

```bash
sudo yum install centos-release-scl devtoolset-8-gcc* -y
# 激活生效（临时）
scl enable devtoolset-8 bash
gcc -v
```

## jdk

```bash
# 创建 jdk 存放目录
mkdir -p /data/software/jdk
cd /data/software/jdk
# 下载 jdk 包
wget https://corretto.aws/downloads/latest/amazon-corretto-8-x64-linux-jdk.tar.gz
# 解压
tar -zxvf amazon-corretto-8-x64-linux-jdk.tar.gz
# 设置 JAVA_HOME 和 PATH
vi /etc/profile
export JAVA_HOME=/data/software/jdk/amazon-corretto-8.322.06.2-linux-x64
export PATH=${JAVA_HOME}/bin:${PATH}
# 生效
source /etc/profile
```

```bash
sudo rpm --import https://yum.corretto.aws/corretto.key 
 sudo curl -L -o /etc/yum.repos.d/corretto.repo https://yum.corretto.aws/corretto.repo
sudo yum install -y java-11-amazon-corretto-devel
```


## maven

```bash
# 下载 maven 包 https://dlcdn.apache.org/
mkdir -p /data/software/maven
cd /data/software/maven
wget https://dlcdn.apache.org/maven/maven-3/3.9.1/binaries/apache-maven-3.9.1-bin.tar.gz --no-check-certificate
# 解压
tar -zxvf apache-maven-3.9.1-bin.tar.gz
# 设置环境变量
sudo vi /etc/profile
MAVEN_HOME=/data/software/maven/apache-maven-3.9.1
export PATH=${MAVEN_HOME}/bin:${PATH}
# 生效
source /etc/profile
```


## redis

```bash
# https://redis.io/download/
cd /usr/local/
curl -LO https://codeload.github.com/redis/redis/tar.gz/refs/tags/7.0.5
tar -zxvf redis-7.0.5.tar.gz
cd redis-7.0.5/
```

## ntp

```bash
yum install ntp ntpdate
systemctl start ntpd
systemctl enable ntpd
```

## iptables

```bash
sudo yum install -y iptables-services

sudo systemctl start iptables

sudo yum remove -y iptables-services
```

# System setting

## ssh no password

```bash
# 客户端
## 生成公私钥对
ssh-keygen -t rsa -C "yelog@mail.com"
## 复制下面下面打印出来的公钥
cat ~/.ssh/id_rsa.pub

# 将公钥上传到服务器
ssh-copy-id -i ～/.ssh/id_rsa.pub root@xx.xx.xx.xx

# 手动将密钥上传到服务器
## 创建 authorized_keys（存在则忽略）
touch ~/.ssh/authorized_keys
## 设置权限
chmod 700 -R ~/.ssh
## 追加到文件内
echo "公钥" >> ~/.ssh/authorized_keys
```

## open file limit

```bash
# 获取当前系统设置的文件数
ulimit -n

# 软件限制
ulimit  -Sn

# 硬件限制
ulimit  -Hn

# 临时生效
ulimit -SHn 10000

# 永久生效
sudo vim /etc/security/limits.conf
* soft nofile 9000000
* hard nofile 9000000

# 查看当前进程打开了多少句柄数
lsof -n|awk '{print $2}'|sort|uniq -c|sort -nr|more

sudo vi /etc/sysctl.conf
# 添加
fs.file-max = 9000000
fs.inotify.max_user_instances = 1000000
fs.inotify.max_user_watches = 1000000
# 生效
sudo sysctl -p
```

## firewalld

```bash
# 启动 firewalld
sudo systemctl start firewalld
# 查看 firewalld 状态
sudo systemctl status firewalld
# 关闭 firewalld
sudo systemctl stop firewalld
# 重新加载配置
sudo firewall-cmd --reload
# 允许端口(tcp)范围进行访问
sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="0.0.0.0/0" port port="1-9329" protocol="tcp" accept' --permanent
# 允许端口(udp)范围进行访问
sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="0.0.0.0/0" port port="1-9329" protocol="udp" accept' --permanent

# 添加访问端口 永久生效
sudo firewall-cmd --zone=public --add-port=9332/tcp --permanent


```

## disk

```bash
# 挂载磁盘 /dev/sda3 到/data目录， 重启失效
# 需要提前 创建 /data 目录
mount /dev/sda3 /data
```
永久生效 `vi /etc/fstab`, 添加如下内容
```bash
/dev/sda3 /data ext4 defaults 0 0
```

## Cpu&Memory

```bash
# 查询物理个数
grep 'physical id' /proc/cpuinfo | sort -u | wc -l

# 查看 CPU 物理核心数量
grep 'core id' /proc/cpuinfo | sort -u | wc -l

# 查看 CPU 逻辑核心数量(一般说几C几G, 说的是逻辑核心)
grep 'processor' /proc/cpuinfo | sort -u | wc -l

```

# command

## network


### DNS

[[CentOS修改DNS-GW-IP# 1.修改DNS]]

### gateway

[[CentOS修改DNS-GW-IP# 2.修改网关]]

### IP

[[CentOS修改DNS-GW-IP# 3.修改IP]]

```bash
# 监控 eth1 网卡的上下行网络
watch -d ifstat eth1
```

## files

#### search and delete file

```bash
# 查找并删除当前文件夹下（包括子目录） 的所有以 .bak 结尾的文件
find . -name *.bak -type f -exec rm -rf {} \;
# 查找并删除当前文件夹（包括子目录） 的所有 .settings 目录，并执行删除命令
find . -name '.settings' -type d -exec rm -rf {} \;
```

## ctrl-w delete word

add the following lines to my .bashrc
```bash
stty werase undef
bind '\C-w:unix-filename-rubout'
```

## enable vim on cli

```bash
set -o vi
```

