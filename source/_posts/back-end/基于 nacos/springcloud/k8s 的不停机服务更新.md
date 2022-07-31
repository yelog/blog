---
title: '基于 nacos/springcloud/k8s 的不停机服务更新[graceful shutdown]'
enlink: 'springboot-graceful-shutdown-based-on-nacos2-and-k8s'
date: 2022-07-27 15:35:39
categories:
- 后端
tags:
- java
- nacos
- springcloud
- k8s
---

## 背景
我们的 SpringCloud 是部署在 k8s 上的, 当通过 k8s 进行滚动升级时, 会有请求 500 的情况, 不利于用户体验, 严重的可能造成数据错误的问题
> k8s 滚动更新策略介绍
假设我们要升级的微服务在环境上为3个副本的集群, 升级应用时, 会先启动1个新版本的副本, 然后下线一个旧版本的副本, 之后再启动1个新版本的副本, 一次类推,直到所有旧副本都替换新副本.

通过链路追踪分析, 报错的原因分别由以下两种情况
1. SpringCloud 中的微服务在升级过程中, 当旧的微服务中还有没有处理完成的请求时, 就开始关闭动作, 造成请求中断
2. 当旧应用执行关闭动作时, 已经开始拒绝请求, 但是 nacos 中的路由并没有及时更新, 造成 gateway/openfeign 在路由时仍会命中正在关闭的应用, 造成请求报错

为了解决这个问题, 我们将利用 springboot 的 graceful shutdown 功能和 nacos 的主动下线功能来解决这个问题. 具体思路如下:

比如当我们执行订单微服务(3个副本)滚动更新时
1. 先启动一个新版本`副本4`
2. 然后准备关闭`副本1`, 在关闭之前先通知 nacos 订单服务的`副本1`下线, 然后由 nacos 通知给其他应用(nacos2.x 是grpc, 所以通知速度比较快), 这样, 订单服务的`副本1`就不会再接收到请求, 然后执行 graceful shutdown(springboot 原生支持, 启用方法可以看后面代码), 所有请求处理完成后关闭应用. 这样就完成了 `副本1` 的关闭
3. 启动新版本`副本5`
4. 再优雅关闭`副本2`(如`副本1`)
5. 然后启动新版本`副本6`
6. 再优雅关闭`副本3`
7. 完成了服务不中断的应用升级

## 实现关键点
为了实现上面背景中提到的思路, 主要从一下几个方面入手

### 创建从 nacos 中下线副本的API
我们通过创建自定义名为 `deregister` 的 `endpoint` 来通知 `nacos` 下线副
```java
import com.alibaba.cloud.nacos.NacosDiscoveryProperties;
import com.alibaba.cloud.nacos.registry.NacosRegistration;
import com.alibaba.cloud.nacos.registry.NacosServiceRegistry;
import lombok.extern.log4j.Log4j2;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.endpoint.annotation.Endpoint;
import org.springframework.boot.actuate.endpoint.annotation.ReadOperation;
import org.springframework.stereotype.Component;

@Component
@Endpoint(id = "deregister")
@Log4j2
public class LemesNacosServiceDeregisterEndpoint {

    @Autowired
    private NacosDiscoveryProperties nacosDiscoveryProperties;

    @Autowired
    private NacosRegistration nacosRegistration;

    @Autowired
    private NacosServiceRegistry nacosServiceRegistry;

    /**
     * 从 nacos 中主动下线，用于 k8s 滚动更新时，提前下线分流流量
     *
     * @param
     * @return com.lenovo.lemes.framework.core.util.ResultData<java.lang.String>
     * @author Yujie Yang
     * @date 4/6/22 2:57 PM
     */
    @ReadOperation
    public String endpoint() {
        String serviceName = nacosDiscoveryProperties.getService();
        String groupName = nacosDiscoveryProperties.getGroup();
        String clusterName = nacosDiscoveryProperties.getClusterName();
        String ip = nacosDiscoveryProperties.getIp();
        int port = nacosDiscoveryProperties.getPort();

        log.info("deregister from nacos, serviceName:{}, groupName:{}, clusterName:{}, ip:{}, port:{}", serviceName, groupName, clusterName, ip, port);

        // 设置服务下线
        nacosServiceRegistry.setStatus(nacosRegistration, "DOWN");

        return "success";
    }

}
```

### 支持 Graceful Shutdown
由于 springboot 原生支持, 我们只需要在 `bootstrap.yaml` 中添加一下配置即可

```yaml
server:
  # 开启优雅下线
  shutdown: graceful

spring:
  lifecycle:
    # 优雅下线超时时间
    timeout-per-shutdown-phase: 5m
# 暴露 shutdown 接口
management:
  endpoint:
    shutdown:
      enabled: true
  endpoints:
    web:
      exposure:
        include: shutdown
```

### K8s 配置
有了上面两个 API, 接下来就配置到 k8s 上
1. terminationGracePeriodSeconds 如果关闭应用的时间超过 10 分钟, 则向容器发送 TERM 信号, 防止应用长时间下线不了
2. preStop 先执行下线操作, 等待30s, 留够通知到其他应用的时间, 然后执行 graceful shutdown 关闭应用


```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lemes-service-common
  labels:
    app: lemes-service-common
spec:
  replicas: 2
  selector:
    matchLabels:
      app: lemes-service-common
#  strategy:
#    type: RollingUpdate
#    rollingUpdate:
##     replicas - maxUnavailable < running num  < replicas + maxSurge
#      maxUnavailable: 1
#      maxSurge: 1
  template:
    metadata:
      labels:
        app: lemes-service-common
    spec:
#      容器重启策略 Never Always OnFailure
#      restartPolicy: Never
#     如果关闭时间超过10分钟， 则向容器发送 TERM 信号
      terminationGracePeriodSeconds: 600
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - podAffinityTerm:
                topologyKey: "kubernetes.io/hostname"
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - lemes-service-common
              weight: 100
#          requiredDuringSchedulingIgnoredDuringExecution:
#            - labelSelector:
#                matchExpressions:
#                  - key: app
#                    operator: In
#                    values:
#                      - lemes-service-common
#              topologyKey: "kubernetes.io/hostname"
      volumes:
        - name: lemes-host-path
          hostPath:
            path: /data/logs
            type: DirectoryOrCreate
        - name: sidecar
          emptyDir: { }
      containers:
        - name: lemes-service-common
          image: 10.176.66.20:5000/lemes-cloud/lemes-service-common-server:v0.1
          imagePullPolicy: Always
          volumeMounts:
            - name: lemes-host-path
              mountPath: /data/logs
            - name: sidecar
              mountPath: /sidecar
          ports:
            - containerPort: 80
          resources:
#           资源通常情况下的占用
            requests:
              memory: '2048Mi'
#           资源占用上限
            limits:
              memory: '4096Mi'
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 80
            initialDelaySeconds: 5
#           探针可以连续失败的次数
            failureThreshold: 10
#           探针超时时间
            timeoutSeconds: 10
#           多久执行一次探针查询
            periodSeconds: 10
          startupProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 80
            failureThreshold: 30
            timeoutSeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 80
            initialDelaySeconds: 5
            timeoutSeconds: 10
            periodSeconds: 10
          lifecycle:
            preStop:
              exec:
#               应用关闭操作：1. 从 nacos 下线，2. 等待30s, 保证 nacos 通知到其他应用 2.触发 springboot 的 graceful shutdown
                command:
                  - sh
                  - -c
                  - curl http://127.0.0.1/actuator/deregister;sleep 30;curl -X POST http://127.0.0.1/actuator/shutdown;
```

