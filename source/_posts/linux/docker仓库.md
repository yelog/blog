---
title: docker仓库
enlink: docker-registry
date: 2017-05-23 21:15:52
categories:
- 运维
tags:
- docker
- docker仓库
---
## Docker Hub
目前 Docker 官方维护了一个公共仓库 [Docker Hub](https://hub.docker.com/)，其中已经包括了超过 15,000 的镜像。大部分需求，都可以通过在 Docker Hub 中直接下载镜像来实现。
### 登录
可以通过执行 docker login 命令来输入用户名、密码和邮箱来完成注册和登录。 注册成功后，本地用户目录的 .dockercfg 中将保存用户的认证信息。
### 基本操作
用户无需登录即可通过 docker search 命令来查找官方仓库中的镜像，并利用 docker pull 命令来将它下载到本地。

例如以 centos 为关键词进行搜索：
```bash
$ sudo docker search centos
NAME                                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                                          The official build of CentOS.                   465       [OK]
tianon/centos                                   CentOS 5 and 6, created using rinse instea...   28
blalor/centos                                   Bare-bones base CentOS 6.5 image                6                    [OK]
saltstack/centos-6-minimal                                                                      6                    [OK]
tutum/centos-6.4                                DEPRECATED. Use tutum/centos:6.4 instead. ...   5                    [OK]
...
```
可以看到返回了很多包含关键字的镜像，其中包括镜像名字、描述、星级（表示该镜像的受欢迎程度）、是否官方创建、是否自动创建。 官方的镜像说明是官方项目组创建和维护的，automated 资源允许用户验证镜像的来源和内容。

根据是否是官方提供，可将镜像资源分为两类。 一种是类似 centos 这样的基础镜像，被称为基础或根镜像。这些基础镜像是由 Docker 公司创建、验证、支持、提供。这样的镜像往往使用单个单词作为名字。 还有一种类型，比如 tianon/centos 镜像，它是由 Docker 的用户创建并维护的，往往带有用户名称前缀。可以通过前缀 user_name/ 来指定使用某个用户提供的镜像，比如 tianon 用户。

另外，在查找的时候通过 -s N 参数可以指定仅显示评价为 N 星以上的镜像。

下载官方 centos 镜像到本地。
```bash
$ sudo docker pull centos
Pulling repository centos
0b443ba03958: Download complete
539c0211cd76: Download complete
511136ea3c5a: Download complete
7064731afe90: Download complete
```
用户也可以在登录后通过 docker push 命令来将镜像推送到 Docker Hub。

## 私有仓库
### 安装 docker-registry
#### 容器运行
在安装了 Docker 后，可以通过获取官方 registry 镜像来运行。
```bash
$ sudo docker run -d -p 5000:5000 registry
```
这将使用官方的 registry 镜像来启动本地的私有仓库。 用户可以通过指定参数来配置私有仓库位置，例如配置镜像存储到 Amazon S3 服务。
```
$ sudo docker run \
         -e SETTINGS_FLAVOR=s3 \
         -e AWS_BUCKET=acme-docker \
         -e STORAGE_PATH=/registry \
         -e AWS_KEY=AKIAHSHB43HS3J92MXZ \
         -e AWS_SECRET=xdDowwlK7TJajV1Y7EoOZrmuPEJlHYcNP2k4j49T \
         -e SEARCH_BACKEND=sqlalchemy \
         -p 5000:5000 \
         registry
```
此外，还可以指定本地路径（如 /home/user/registry-conf ）下的配置文件。
```bash
$ sudo docker run -d -p 5000:5000 -v /home/user/registry-conf:/registry-conf -e DOCKER_REGISTRY_CONFIG=/registry-conf/config.yml registry
```
默认情况下，仓库会被创建在容器的 /tmp/registry 下。可以通过 -v 参数来将镜像文件存放在本地的指定路径。 例如下面的例子将上传的镜像放到 /opt/data/registry 目录。
```bash
$ sudo docker run -d -p 5000:5000 -v /opt/data/registry:/tmp/registry registry
```
#### 本地安装
对于 Ubuntu 或 CentOS 等发行版，可以直接通过源安装。
1.Ubuntu
```bash
$ sudo apt-get install -y build-essential python-dev libevent-dev python-pip liblzma-dev
$ sudo pip install docker-registry
```
2.CentOS
```bash
$ sudo yum install -y python-devel libevent-devel python-pip gcc xz-devel
$ sudo python-pip install docker-registry
```
3.源码安装
```bash
$ sudo apt-get install build-essential python-dev libevent-dev python-pip libssl-dev liblzma-dev libffi-dev
$ git clone https://github.com/docker/docker-registry.git
$ cd docker-registry
$ sudo python setup.py install
```
然后修改配置文件，主要修改 dev 模板段的 storage_path 到本地的存储仓库的路径。
```bash
$ cp config/config_sample.yml config/config.yml
```
之后启动 Web 服务。
```bash
$ sudo gunicorn -c contrib/gunicorn.py docker_registry.wsgi:application
```
或者
```bash
$ sudo gunicorn --access-logfile - --error-logfile - -k gevent -b 0.0.0.0:5000 -w 4 --max-requests 100 docker_registry.wsgi:application
```
此时使用 curl 访问本地的 5000 端口，看到输出 docker-registry 的版本信息说明运行成功。

>*注：config/config_sample.yml 文件是示例配置文件。*

### 在私有仓库上传、下载、搜索镜像
创建好私有仓库之后，就可以使用 docker tag 来标记一个镜像，然后推送它到仓库，别的机器上就可以下载下来了。例如私有仓库地址为 192.168.7.26:5000。

先在本机查看已有的镜像。
```bash
$ sudo docker images
REPOSITORY                        TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                            latest              ba5877dc9bec        6 weeks ago         192.7 MB
ubuntu                            14.04               ba5877dc9bec        6 weeks ago         192.7 MB
```
使用docker tag 将 ba58 这个镜像标记为 192.168.7.26:5000/test（格式为 docker tag IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]）。
```bash
$ sudo docker tag ba58 192.168.7.26:5000/test
root ~ # docker images
REPOSITORY                        TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                            14.04               ba5877dc9bec        6 weeks ago         192.7 MB
ubuntu                            latest              ba5877dc9bec        6 weeks ago         192.7 MB
192.168.7.26:5000/test            latest              ba5877dc9bec        6 weeks ago         192.7 MB
```
使用 docker push 上传标记的镜像。
```bash
$ sudo docker push 192.168.7.26:5000/test
The push refers to a repository [192.168.7.26:5000/test] (len: 1)
Sending image list
Pushing repository 192.168.7.26:5000/test (1 tags)
Image 511136ea3c5a already pushed, skipping
Image 9bad880da3d2 already pushed, skipping
Image 25f11f5fb0cb already pushed, skipping
Image ebc34468f71d already pushed, skipping
Image 2318d26665ef already pushed, skipping
Image ba5877dc9bec already pushed, skipping
Pushing tag for rev [ba5877dc9bec] on {http://192.168.7.26:5000/v1/repositories/test/tags/latest}
```

用 curl 查看仓库中的镜像。
```bash
$ curl http://192.168.7.26:5000/v1/search
{"num_results": 7, "query": "", "results": [{"description": "", "name": "library/miaxis_j2ee"}, {"description": "", "name": "library/tomcat"}, {"description": "", "name": "library/ubuntu"}, {"description": "", "name": "library/ubuntu_office"}, {"description": "", "name": "library/desktop_ubu"}, {"description": "", "name": "dockerfile/ubuntu"}, {"description": "", "name": "library/test"}]}
```
这里可以看到 {"description": "", "name": "library/test"}，表明镜像已经被成功上传了。

现在可以到另外一台机器去下载这个镜像。
```bash
$ sudo docker pull 192.168.7.26:5000/test
Pulling repository 192.168.7.26:5000/test
ba5877dc9bec: Download complete
511136ea3c5a: Download complete
9bad880da3d2: Download complete
25f11f5fb0cb: Download complete
ebc34468f71d: Download complete
2318d26665ef: Download complete
$ sudo docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
192.168.7.26:5000/test             latest              ba5877dc9bec        6 weeks ago         192.7 MB
```
可以使用 [这个脚本](https://github.com/yeasy/docker_practice/raw/master/_local/push_images.sh) 批量上传本地的镜像到注册服务器中，默认是本地注册服务器 127.0.0.1:5000。例如：

```bash
$ wget https://github.com/yeasy/docker_practice/raw/master/_local/push_images.sh; sudo chmod a+x push_images.sh
$ ./push_images.sh ubuntu:latest centos:centos7
The registry server is 127.0.0.1
Uploading ubuntu:latest...
The push refers to a repository [127.0.0.1:5000/ubuntu] (len: 1)
Sending image list
Pushing repository 127.0.0.1:5000/ubuntu (1 tags)
Image 511136ea3c5a already pushed, skipping
Image bfb8b5a2ad34 already pushed, skipping
Image c1f3bdbd8355 already pushed, skipping
Image 897578f527ae already pushed, skipping
Image 9387bcc9826e already pushed, skipping
Image 809ed259f845 already pushed, skipping
Image 96864a7d2df3 already pushed, skipping
Pushing tag for rev [96864a7d2df3] on {http://127.0.0.1:5000/v1/repositories/ubuntu/tags/latest}
Untagged: 127.0.0.1:5000/ubuntu:latest
Done
Uploading centos:centos7...
The push refers to a repository [127.0.0.1:5000/centos] (len: 1)
Sending image list
Pushing repository 127.0.0.1:5000/centos (1 tags)
Image 511136ea3c5a already pushed, skipping
34e94e67e63a: Image successfully pushed
70214e5d0a90: Image successfully pushed
Pushing tag for rev [70214e5d0a90] on {http://127.0.0.1:5000/v1/repositories/centos/tags/centos7}
Untagged: 127.0.0.1:5000/centos:centos7
Done
```

## 仓库配置文件
Docker 的 Registry 利用配置文件提供了一些仓库的模板（flavor），用户可以直接使用它们来进行开发或生产部署。
### 模板
在 config_sample.yml 文件中，可以看到一些现成的模板段：

- common：基础配置
- local：存储数据到本地文件系统
- s3：存储数据到 AWS S3 中
- dev：使用 local 模板的基本配置
- test：单元测试使用
- prod：生产环境配置（基本上跟s3配置类似）
- gcs：存储数据到 Google 的云存储
- swift：存储数据到 OpenStack Swift 服务
- glance：存储数据到 OpenStack Glance 服务，本地文件系统为后备
- glance-swift：存储数据到 OpenStack Glance 服务，Swift 为后备
- elliptics：存储数据到 Elliptics key/value 存储

用户也可以添加自定义的模版段。

默认情况下使用的模板是 dev，要使用某个模板作为默认值，可以添加 SETTINGS_FLAVOR 到环境变量中，例如
```
export SETTINGS_FLAVOR=dev
```
另外，配置文件中支持从环境变量中加载值，语法格式为 `_env:VARIABLENAME[:DEFAULT]`。
### 示例配置
```
common:
    loglevel: info
    search_backend: "_env:SEARCH_BACKEND:"
    sqlalchemy_index_database:
        "_env:SQLALCHEMY_INDEX_DATABASE:sqlite:////tmp/docker-registry.db"

prod:
    loglevel: warn
    storage: s3
    s3_access_key: _env:AWS_S3_ACCESS_KEY
    s3_secret_key: _env:AWS_S3_SECRET_KEY
    s3_bucket: _env:AWS_S3_BUCKET
    boto_bucket: _env:AWS_S3_BUCKET
    storage_path: /srv/docker
    smtp_host: localhost
    from_addr: docker@myself.com
    to_addr: my@myself.com

dev:
    loglevel: debug
    storage: local
    storage_path: /home/myself/docker

test:
    storage: local
    storage_path: /tmp/tmpdockertmp
```
