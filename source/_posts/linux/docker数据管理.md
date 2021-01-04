---
title: docker数据管理
enlink: docker-data-manager
date: 2017-05-23 21:43:06
categories:
- 运维
tags:
- docker
---
## 数据卷

数据卷是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

- 数据卷可以在容器之间共享和重用
- 对数据卷的修改会立马生效
- 对数据卷的更新，不会影响镜像
- 数据卷默认会一直存在，即使容器被删除

>注意：数据卷的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会隐藏掉，能显示看的是挂载的数据卷。

### 创建一个数据卷
在用 docker run 命令的时候，使用 -v 标记来创建一个数据卷并挂载到容器里。在一次 run 中多次使用可以挂载多个数据卷。

下面创建一个名为 web 的容器，并加载一个数据卷到容器的 /webapp 目录。
```bash
$ sudo docker run -d -P --name web -v /webapp training/webapp python app.py
```
>注意：也可以在 Dockerfile 中使用 VOLUME 来添加一个或者多个新的卷到由该镜像创建的任意容器。

### 删除数据卷
数据卷是被设计用来持久化数据的，它的生命周期独立于容器，Docker不会在容器被删除后自动删除数据卷，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的数据卷。如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用 docker rm -v 这个命令。无主的数据卷可能会占据很多空间，要清理会很麻烦。Docker官方正在试图解决这个问题，相关工作的进度可以查看这个[PR](https://github.com/docker/docker/pull/8484)。

### 挂载一个主机目录作为数据卷
使用 -v 标记也可以指定挂载一个本地主机的目录到容器中去。
```bash
$ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py
```
上面的命令加载主机的 /src/webapp 目录到容器的 /opt/webapp 目录。这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。本地目录的路径必须是绝对路径，如果目录不存在 Docker 会自动为你创建它。

>注意：Dockerfile 中不支持这种用法，这是因为 Dockerfile 是为了移植和分享用的。然而，不同操作系统的路径格式不一样，所以目前还不能支持。

Docker 挂载数据卷的默认权限是读写，用户也可以通过 :ro 指定为只读。
```bash
$ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp:ro
training/webapp python app.py
```
加了 :ro 之后，就挂载为只读了。

### 查看数据卷的具体信息
在主机里使用以下命令可以查看指定容器的信息
```bash
$ docker inspect web
```
在输出的内容中找到其中和数据卷相关的部分，可以看到所有的数据卷都是创建在主机的/var/lib/docker/volumes/下面的
```bash
"Volumes": {
    "/webapp": "/var/lib/docker/volumes/fac362...80535"
},
"VolumesRW": {
    "/webapp": true
}
...
```
注：从Docker 1.8.0起，数据卷配置在"Mounts"Key下面，可以看到所有的数据卷都是创建在主机的/mnt/sda1/var/lib/docker/volumes/....下面了。
```json
"Mounts": [
            {
                "Name": "b53ebd40054dae599faf7c9666acfe205c3e922fc3e8bc3f2fd178ed788f1c29",
                "Source": "/mnt/sda1/var/lib/docker/volumes/b53ebd40054dae599faf7c9666acfe205c3e922fc3e8bc3f2fd178ed788f1c29/_data",
                "Destination": "/webapp",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ]
...
```
### 挂载一个本地主机文件作为数据卷
-v 标记也可以从主机挂载单个文件到容器中
```bash
$ sudo docker run --rm -it -v ~/.bash_history:/.bash_history ubuntu /bin/bash
```
这样就可以记录在容器输入过的命令了。

>注意：如果直接挂载一个文件，很多文件编辑工具，包括 vi 或者 `sed --in-place`，可能会造成文件 inode 的改变，从 Docker 1.1 .0起，这会导致报错误信息。所以最简单的办法就直接挂载文件的父目录。

## 数据卷容器
如果你有一些持续更新的数据需要在容器之间共享，最好创建数据卷容器。

数据卷容器，其实就是一个正常的容器，专门用来提供数据卷供其它容器挂载的。

首先，创建一个名为 dbdata 的数据卷容器：
```bash
$ sudo docker run -d -v /dbdata --name dbdata training/postgres echo Data-only container for postgres
```
然后，在其他容器中使用 --volumes-from 来挂载 dbdata 容器中的数据卷。
```bash
$ sudo docker run -d --volumes-from dbdata --name db1 training/postgres
$ sudo docker run -d --volumes-from dbdata --name db2 training/postgres
```
可以使用超过一个的 `--volumes-from` 参数来指定从多个容器挂载不同的数据卷。 也可以从其他已经挂载了数据卷的容器来级联挂载数据卷。
```bash
$ sudo docker run -d --name db3 --volumes-from db1 training/postgres
```
>注意：使用 --volumes-from 参数所挂载数据卷的容器自己并不需要保持在运行状态。

如果删除了挂载的容器（包括 dbdata、db1 和 db2），数据卷并不会被自动删除。如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时使用 docker rm -v 命令来指定同时删除关联的容器。 这可以让用户在容器之间升级和移动数据卷。具体的操作将在下一节中进行讲解。

## 利用数据卷容器来备份、恢复、迁移数据卷
可以利用数据卷对其中的数据进行进行备份、恢复和迁移。
### 备份
首先使用 --volumes-from 标记来创建一个加载 dbdata 容器卷的容器，并从主机挂载当前目录到容器的 /backup 目录。命令如下：
```bash
$ sudo docker run --volumes-from dbdata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```
容器启动后，使用了 tar 命令来将 dbdata 卷备份为容器中 /backup/backup.tar 文件，也就是主机当前目录下的名为 backup.tar 的文件。
### 恢复
如果要恢复数据到一个容器，首先创建一个带有空数据卷的容器 dbdata2。
```bash
$ sudo docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
```
然后创建另一个容器，挂载 dbdata2 容器卷中的数据卷，并使用 untar 解压备份文件到挂载的容器卷中。
```bash
$ sudo docker run --volumes-from dbdata2 -v $(pwd):/backup busybox tar xvf
/backup/backup.tar
```
为了查看/验证恢复的数据，可以再启动一个容器挂载同样的容器卷来查看
```bash
$ sudo docker run --volumes-from dbdata2 busybox /bin/ls /dbdata
```
