---
title: SpringCloud系列之接入SkyWalking进行链路追踪和日志收集
enlink: spring-cloud-skywalking
date: 2021-09-26 18:08:00
categories:
- 后端
tags:
- java
- SpringCloud
- SkyWalking
---

## 前言

前一段时间一直在研究升级公司项目的架构，在不断学习和试错后，最终确定了一套基于 k8s 的高可用架构体系，未来几期会将这套架构体系的架设过程和注意事项以系列文章的形式分享出来，敬请期待！

由于集群和分布式规模的扩大，对微服务链路的监控和日志收集，越来越有必要性，所以在筛选了了一些方案后，发现 SkyWalking 完美符合我们的预期，对链路追踪和日志收集都有不错的实现。

## SkyWalking 简介

SkyWalking 是一款 APM（应用程序监控）系统，转为微服务、云原生、基于容器的架构而设计。主要包含了一下核心功能

1. 对服务、运行实例、API进行指标分析
2. 链路检测，检查缓慢的服务和API
3. 对基础设施（VM、网络、磁盘、数据库）进行监控
4. 对超出阈值的情况进行警报
5. 等等

开源地址：[apache/skywalking](https://github.com/apache/skywalking)

官网：[Apache SkyWalking](https://skywalking.apache.org/)

## SpringCloud 整合 SkyWalking

### 1. 搭建 SkyWalking 服务

在使用 SkyWalking 进行链路追踪和日志收集之前，需要先搭建起一套 SkyWalking 的服务，然后才能通过 agent 将 SpringCloud 的运行状态和日志发送给 SkyWalking 进行解析和展示。

SkyWalking 的搭建方式有很多中，我这里介绍两种 docker-compose（非高可用，快速启动，方便测试、学习） 和 k8s（高可用、生产级别）

#### docker-compose 的方式

docker 和 docker-compose 的安装不是本文的重点，所以有需要可以自行查询。

以下操作会启动三个容器
1. `elasticsearch` 作为 skywalking 的存储，保存链路和日志数据等
2. `oap` 数据接收和分析 Observability Analysis Platform
3. `ui` web端的数据展示

```bash
# 创建配置文件保存的目录
mkdir -p /data/docker/admin/skywalking
# 切换到刚创建的目录
cd /data/docker/admin/skywalking
# 将下面的 docker-compose.yml 文件保存到这个目录
vi docker-compose.yml
# 拉去镜像并启动
docker-compose up -d
# 查看日志
docker-compose logs -f
```

docker-compose.yml
```yaml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.1
    container_name: elasticsearch
    restart: always
    ports:
      - 9200:9200
    healthcheck:
      test: ["CMD-SHELL", "curl --silent --fail localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    environment:
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - TZ=Asia/Shanghai
    ulimits:
      memlock:
        soft: -1
        hard: -1
  oap:
    image: apache/skywalking-oap-server:8.7.0-es7
    container_name: oap
    depends_on:
      - elasticsearch
    links:
      - elasticsearch
    restart: always
    ports:
      - 11800:11800
      - 12800:12800
    healthcheck:
      test: ["CMD-SHELL", "/skywalking/bin/swctl"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    environment:
      TZ: Asia/Shanghai
      SW_STORAGE: elasticsearch7
      SW_STORAGE_ES_CLUSTER_NODES: elasticsearch:9200
  ui:
    image: apache/skywalking-ui:8.7.0
    container_name: ui
    depends_on:
      - oap
    links:
      - oap
    restart: always
    ports:
      - 8088:8080
    environment:
      TZ: Asia/Shanghai
      SW_OAP_ADDRESS: http://oap:12800

```

启动之后浏览器访问 `服务ip:8080` 即可

#### k8s

等待更新。。

### 2. 下载 agent 代理包

点击链接进行下载，[skywalking-apm-8.7](https://archive.apache.org/dist/skywalking/8.7.0/apache-skywalking-apm-8.7.0.tar.gz)

> 其他版本可以看 [apache 归档站](https://archive.apache.org/dist/skywalking/)，找到对应版本的 `.tar.gz` 后缀的包，进行下载

通过命令或者软件进行解压 `tar -zxvf apache-skywalking-apm-8.7.0.tar.gz`

### 3. java 命令使用代码启动 jar 包

springcloud/springboot 一般是通过 `java -jar xxx.jar` 进行启动。我们只需要在其中加上 `-javaagent` 参数即可，如下

其中 **自定义服务名** 可以改为应用名 如 `lemes-auth`，**服务ip** 为第一步搭建的 SkyWalking 服务的ip，**端口11800** 为启动的 oap 这个容器的端口

```bash
java -javaagent:上一步解压目录/agent/skywalking-agent.jar=agent.service_name=自定义服务名,collector.backend_service=服务ip:11800 -jar xx.jar
```

执行命令启动后，访问以下接口，就可以在第一步 `服务ip:8080` 中看到访问的链接和调用链路。

![链路追踪](https://img.saodiyang.com/picgo_qiniu20210926164151.png)
![拓扑图](https://img.saodiyang.com/picgo_qiniu20210926164350.png)

### 4. 开启日志收集

本文主要以 log4j2 来介绍，其他的大同小异，可以网上找教程。SpringCloud 集成 log4j2 不是本文重点，所以请自行 Google。

#### 引入依赖

要开启日志收集，必须要添加依赖，如下

```xml
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-log4j-2.x</artifactId>
    <version>8.7.0</version>
</dependency>
```
#### 修改 log4j2.xml

需要修改 log4j2.xml 主要添加下面两个关键点
- 添加 `%traceId` 来打印 traceid
- 声明 GRPCLogClientAppender

完整内容如下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!-- Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，
     你会看到log4j2内部各种详细输出。可以设置成OFF(关闭) 或 Error(只输出错误信息)。
-->
<!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数-->
<configuration status="WARN" monitorInterval="30">

    <Properties>
        <Property name="log.path">logs/lemes-auth</Property>
        <Property name="logging.lemes.pattern">
            %d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level] [%traceId] [%logger{50}.%M:%L] - %msg%n
        </Property>
    </Properties>

    <Appenders>
        <!-- 输出控制台日志的配置 -->
        <Console name="Console" target="SYSTEM_OUT">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY"/>
            <!-- 输出日志的格式 -->
            <PatternLayout pattern="${logging.lemes.pattern}"/>
        </Console>

        <RollingRandomAccessFile name="debugRollingFile" fileName="${log.path}/debug.log"
                                 filePattern="${log.path}/debug/$${date:yyyy-MM}/debug.%d{yyyy-MM-dd}-%i.log.gz">
            <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout charset="UTF-8" pattern="${logging.lemes.pattern}"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1"/>
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
            <DefaultRolloverStrategy max="30"/>
        </RollingRandomAccessFile>

        <GRPCLogClientAppender name="grpc-log">
            <PatternLayout pattern="${logging.lemes.pattern}"/>
        </GRPCLogClientAppender>
    </Appenders>
    <Loggers>
        <!-- ALL < TRACE < DEBUG < INFO < WARN < ERROR < FATAL < OFF -->
        <Logger name="com.lenovo.lemes" level="debug"/>
        <Logger name="org.apache.kafka" level="warn"/>
        <Root level="info">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="debugRollingFile"/>
            <AppenderRef ref="grpc-log"/>
        </Root>
    </Loggers>
</configuration>

```

#### 启动命令中声明上报日志

在上一步的 agent 中添加上报日志的参数 `plugin.toolkit.log.grpc.reporter.server_host=服务ip,plugin.toolkit.log.grpc.reporter.server_port=11800`

完整如下
```bash
java -javaagent:上一步解压目录/agent/skywalking-agent.jar=agent.service_name=自定义服务名,collector.backend_service=服务ip:11800,plugin.toolkit.log.grpc.reporter.server_host=服务ip,plugin.toolkit.log.grpc.reporter.server_port=11800 -jar xx.jar
```

#### 日志收集效果

这样启动日志中就会打印 traceid , `N/A` 代表的是非请求的日志，有 traceid 的为 api 请求日志

![traceid](https://img.saodiyang.com/picgo_qiniu20210926170409.png)

在 skywalking 中就能看到我们上报的日志

![skywalking 日志上报](https://img.saodiyang.com/picgo_qiniu20210926170953.png)

重点：SkyWalking 可以在链路追踪中查看当前请求的所有日志（不同实例/模块）

![SkyWalking 链路日志](https://img.saodiyang.com/picgo_qiniu20210926171217.png)

![SkyWalking 链路日志](https://img.saodiyang.com/picgo_qiniu20210926171256.png)

### 5. 兼容 spring-cloud-gateway

经过上面的步骤之后，链路已经搭建完成，查看发现了一个问题，gateway 模块的 `traceId` 和 业务模块的 `traceId` 不统一。

![拓扑图](https://img.saodiyang.com/picgo_qiniu20210926164350.png)


这是由于 SkyWalking 对于 `spring-cloud-gateway` 的支持不是默认的，所以需要将 `agent/optional-plugins/apm-spring-cloud-gateway-2.1.x-plugin-8.7.0.jar` 复制到 `agent/plugins` 下，然后重启即可。

![优化过 gateway 的拓扑图](https://img.saodiyang.com/picgo_qiniu20210926180619.png)

## 最后

SkyWalking 上面这两个功能就已经非常强大，能够有效帮助我们优化我们的程序，监控系统的问题，并及时报警。日志收集也解决的在大规模分布式集群下日志查询难的问题。

SkyWalking 还支持 VM、浏览器、k8s等监控，后续如果有实践，将会逐步更新。
