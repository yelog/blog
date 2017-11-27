---
title: 搭建Maven私服-Nexus
permalink: build-Maven-Nexus
date: 2017-09-06 23:01:31
categories: 运维
tags:
- maven
- nexus
- java
---
> Maven 私服，可以代理远程仓库和部署自己或第三方构件。本文介绍使用最广泛搭建 Maven 私服的工具： Sonatype Nexus。

## 作者环境
  本次搭建私服是在局域网的一台服务器上，操作系统为 CentOS 。
  需要部署到私服的项目 `soul`
  `ssm` 项目需要引用 `soul`

## 安装Java
  确保服务器已经安装了 java 环境，这个过程不是本文重点，安装过程自行百度。

## 安装Nexus
> 官网 pro 版本的是需要付费的。所以我们使用免费的 OSS 版本，下载地址 (https://www.sonatype.com/download-oss-sonatype)

```bash
# 上传到服务器并解压
$ tar xvf nexus-3.5.1-02-unix.tar.gz
```

### 启动Nexus
```bash
# 启动服务
$ cd /nexus-3.5.1-02/bin/
$ ./nexus start
```

### 验证
打开网址：(http://192.168.0.86:8081/) , ip 为搭建私服的服务器 ip 。
用户名/密码： admin/admin123
出现一下画面，就说明安装成功了。
![nexus](http://oncj6b2vl.bkt.clouddn.com/FkIlJRDdMC2yJFT9huIk9gBuCe15.png)

## 发布soul项目到私服
### 创建仓库
1. 创建yelog-release仓库（*名字自定义*）, type选择 ： release
![①.创建宿主仓库](http://oncj6b2vl.bkt.clouddn.com/Fk5UboZXVf4aegz63N4TDrITMh4V.png)
![②.选择hosted类型](http://oncj6b2vl.bkt.clouddn.com/FjAz8w445FFeVqDDzLLTLitOkZyV.png)
![③.创建release类型仓库](http://oncj6b2vl.bkt.clouddn.com/Ft2u5UPqdAFxldkkwAC2uVBtjDhF.png)
2. 创建yelog-snapshot仓库（*名字自定义*）， type选择 ： snapshot
重复上面 ① 和 ② 步，根据下图选择类型:
![创建snapshot类型仓库](http://oncj6b2vl.bkt.clouddn.com/FgX10ufvTV_YUK2vjrdRN3InKN1i.png)

两个都创建完成后，效果如下：
![创建自定义release和snapshot仓库](http://oncj6b2vl.bkt.clouddn.com/FjcMrajAM3anymLemHA2OoZZUut-.png)

### pom中添加部署配置
url 复制上图中新建的仓库的 copy 按钮，复制url。
```xml
<distributionManagement>
    <repository>
        <id>yelog-release</id>
        <name>Release Repository of yelog</name>
        <url>http://192.168.0.86:8081/repository/yelog-release/</url>
    </repository>
    <snapshotRepository>
        <id>yelog-snapshot</id>
        <name>Snapshot Repository of yelog</name>
        <url>http://192.168.0.86:8081/repository/yelog-snapshot/</url>
    </snapshotRepository>
</distributionManagement>
```
### 在maven的 settings.xml 中配置
这里配置 maven 的账号密码，id 要与 distributionManagement 中的id一致。默认账号/密码：admin/admin123
```xml
<servers>
    <server>
      <id>yelog-realease</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
    <server>
      <id>yelog-snapshot</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
</servers>
```

### 执行maven命令部署项目到私服上
我这里直接使用IDE的插件执行
![部署项目到私服](http://oncj6b2vl.bkt.clouddn.com/Frs7jSIClJvF31AnyAAz5JXL-IAP.png)
部署完成后，可以在 yelog-snapshot 仓库中，查看部署的情况，如下图所示
![查看部署情况](http://oncj6b2vl.bkt.clouddn.com/Fi1YtOKOAK7KMigjAc4S7ZfwLLjl.png)

## 从私服拉去依赖库
> 上一步我们已经将项目 `soul` 部署到私服上了，这一步介绍项目 `ssm` 如何依赖引用 `soul`。
私服中的 maven-central 可以链接远程仓库。这样，当有依赖在私服中找不到后，就可以通过远程仓库自动下载依赖。

### pom 文件中添加如下配置
1. public库成员仓库中添加我们自定义的仓库
![public库设置成员仓库](http://oncj6b2vl.bkt.clouddn.com/FlmsId2u1z_vQ__zlfzoAIytujm3.png)
2. 配置远程仓库为私服地址。
```xml
<repositories>
    <repository>
        <id>public</id>
        <name>public Repository</name>
        <url>http://192.168.0.86:8081/repository/maven-public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
<pluginRepositories>
    <pluginRepository>
        <id>public</id>
        <name>Public Repositories</name>
        <url>http://192.168.0.86:8081/repository/maven-public/</url>
    </pluginRepository>
</pluginRepositories>
```
3. 引入依赖
```xml
<dependency>
    <groupId>org.soul</groupId>
    <artifactId>commons</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```
4. ssm项目就可以引用到soul代码
![在代码中就可以直接引用到部署到私服的soul项目代码](http://oncj6b2vl.bkt.clouddn.com/FvVCRBtMwy5ckbase3E3wM9JUXp6.png)

本文结束。
