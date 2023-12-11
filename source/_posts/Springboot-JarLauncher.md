---
title: 'Caused by: java.lang.ClassNotFoundException: org.springframework.boot.loader.JarLauncher'
enlink: 'springboot-jarlauncher'
date: 2023-12-11 16:58:00
categories:
- 后端
tags:
- java
- spring-boot
- docker
---

## 问题现象

今天同事再升级框架后(spring-cloud 2022.0.4 -> 2023.0.0)(spring-boot 3.1.6 -> 3.2.0)

同时因为 spring-boot 的版本问题, 需要将 maven 升级到 3.6.3+


![](https://cdn.jsdelivr.net/gh/yelog/assets/images/202312111837604.png)

升级后构建 jar 包和构建镜像都是正常的, 但是发布到测试环境就报错 `Caused by: java.lang.ClassNotFoundException: org.springframework.boot.loader.JarLauncher`

![](https://cdn.jsdelivr.net/gh/yelog/assets/images/202312111851862.png)

## 问题分析

报错为 JarLauncher 找不到, 检查了 `Jenkins` 中的打包任务, 发现并没有编译报错, 同事直接使用打包任务中产生的 xx.jar, 可以正常运行.

说明在打包 `Docker` 镜像前都没有问题, 这时就想起来我们在打包镜像时, 先解压 xx.jar, 然后直接执行 `org.springframework.boot.loader.JarLauncher`, 所以很可能是升级后, 启动文件 `JarLauncher` 的路径变了.

为了验证我们的猜想, 我们得看一下实际容器内的文件结构, 但是这时容器一直报错导致无法启动, 不能直接通过 `Rancher` 查看文件结构, 我们可以通过文件拷贝的方式来解决, 如下:

```bash
# 下载有问题的镜像, 并且创建容器(不启动)
docker create -it --name dumy 10.188.132.123:5000/lemes-cloud/lemes-gateway:develop-202312111536 bash
# 直接拷贝容器内的我们想要看的目录
docker cp dumy:/data .
```
到本地后, 就可以通过合适的工具查找 `JarLauncher` 文件, 我这里通过 `vim` 来寻找, 如下图:

![](https://cdn.jsdelivr.net/gh/yelog/assets/images/202312111902709.png)

发现比原来多了一层目录 `launch`, 所以问题就发生在这里了.

## 解决方案

我们在打包脚本 `JenkinsCommon.groovy` 中根据当前打包的 JDK 版本来判断使用的启动类路径, 如下:

![](https://cdn.jsdelivr.net/gh/yelog/assets/images/202312111905161.png)

再次打包, 应用正常启动.

## Reference

[JarLauncher](https://docs.spring.io/spring-boot/docs/2.5.15/api/org/springframework/boot/loader/JarLauncher.html)


