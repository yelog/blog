---
title: '基于 nacos/灰度发布 实现减少本地启动微服务数量的实践'
enlink: 'reducing-local-springcloud-base-on-nacos-and-gray-release'
date: 2022-08-01 20:06:14
categories:
- 后端
tags:
- java
- nacos
- springcloud
- gray-release
- k8s
---

## 一、背景

后台框架是基于 spring cloud 的微服务体系, 当开发同学在自己电脑上进行开发工作时, 比如开发订单模块, 除了需要启动订单模块外, 还需要启动网关模块、权限校验模块、公共服务模块等依赖模块, 非常消耗开发同学的本地电脑的资源, 也及其浪费时间.

![Spring Cloud](https://img.saodiyang.com/picgo_qiniu20220801233224.png)

## 二、解决方案

### 2.1 目标和关键问题
能不能开发同学本地只需要启动需要开发的模块:订单模块, 其他模块均适用测试环境中正在运行的服务.

既然要实现的目标有了, 我们就开始研究可行性和关键问题
1. 开发环境和测试环境要在同一个 nacos 的 namespace 中, 这样才有可能让开发环境调用到测试环境的服务.
2. 测试环境只能调用测试环境的微服务, 实现和开发环境的服务隔离
3. 开发同学之间的微服务也要实现服务隔离

### 2.2 思路

既要在同一个 namespace 下, 又要能够实现不同人访问不同的副本, 很容易想到可以利用`灰度发布`来实现:
1. 测试环境设置 metadata `lemes-env=product` 来标识测试环境副本, 用于区分开发环境的微服务测测试环境的微服务
2. 开发同学本地启动注册开发环境副本, 都会携带唯一IP, 则我们可以通过IP来区分不同开发同学的副本

假设我们需要开发的 API 的后台服务调用链条如下:

![请求调用](https://img.saodiyang.com/picgo_qiniu20220802004250.png)

我们需要开发的 API 为 `/addMo`, 打算写在 `Order` 这个微服务里面, 并且他会调用 `common` 这个微服务的 `/getDict` 获取一个字典数据, `/getDict` 是现成的, 不需要开发, 如果是之前的情况, 开发本地至少需要启动5个微服务才能进行调试.

![实现效果](https://img.saodiyang.com/picgo_qiniu20220802004109.png)

## 三、具体实现

### 3.1 测试环境设置 metadata

由于测试环境都是通过容器部署的, 那么启动方式就是下面容器中的 `CMD`, 我们在其中加入 `-Dspring.cloud.nacos.discovery.metadata.lemes-env=product`, 用于区分开发环境的微服务测测试环境的微服务

```Dockerfile
# 说明：Dockerfile 过程分为两部分。第一次用来解压 jar 包，并不会在目标镜像内产生 history/layer。第二部分将解压内容分 layer 拷贝到目标镜像内
# 目的：更新镜像时，只需要传输代码部分，依赖没有变动则不更新，节省发包时的网络传输量
# 原理：在第二部分中，每次 copy 就会在目标镜像内产生一层 layer，将依赖和代码分开，
#      绝大部分更新都不会动到依赖，所以只需更新代码几十k左右的代码层即可

FROM 10.176.66.20:5000/library/amazoncorretto:11.0.11  as builder
WORKDIR /build
ARG ARTIFACT_ID
COPY target/${ARTIFACT_ID}.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract && rm app.jar

FROM 10.176.66.20:5000/library/amazoncorretto:11.0.11
LABEL maintainer="yangyj13@lenovo.com"
WORKDIR /data
ARG ARTIFACT_ID
ENV ARTIFACT_ID ${ARTIFACT_ID}

# 依赖
COPY --from=builder /build/dependencies/ ./
COPY --from=builder /build/snapshot-dependencies/ ./
COPY --from=builder /build/spring-boot-loader/ ./
# 应用代码
COPY --from=builder /build/application/ ./

# 容器运行时启动命令
CMD echo "NACOS_ADDR: ${NACOS_ADDR}"; \
    echo "JAVA_OPTS: ${JAVA_OPTS}"; \
    echo "TZ: ${TZ}"; \
    echo "ARTIFACT_ID: ${ARTIFACT_ID}"; \
    # 去除了 server 的应用名
    REAL_APP_NAME=${ARTIFACT_ID//-server/}; \
    echo "REAL_APP_NAME: ${REAL_APP_NAME}"; \
    # 获取当前时间
    now=`date +%F+%T+%Z`; \
    # java 启动命令
    java $JAVA_OPTS \
    -Dtingyun.app_name=${REAL_APP_NAME}-${TINGYUN_SUFFIX} \
    -Dspring.cloud.nacos.discovery.metadata.lemes-env=product \
    -Dspring.cloud.nacos.discovery.metadata.startup-time=${now} \
    -Dspring.cloud.nacos.discovery.server-addr=${NACOS_ADDR} \
    -Dspring.cloud.nacos.discovery.group=${NACOS_GROUP} \
    -Dspring.cloud.nacos.config.namespace=${NACOS_NAMESPACE} \
    -Dspring.cloud.nacos.discovery.namespace=${NACOS_NAMESPACE} \
    -Dspring.cloud.nacos.discovery.ip=${HOST_IP} \
    org.springframework.boot.loader.JarLauncher

```
![set nacos metadata](https://img.saodiyang.com/picgo_qiniu20220802135945.png)

### 3.2 开发前端传递开启智能连接

```js
const devIp = getLocalIP('10.')

module.exports = {
  devServer: {
    proxy: {
      '/lemes-api': {
        target: 'http://10.176.66.58/lemes-api',
        ws: true,
        pathRewrite: {
          '^/lemes-api': '/'
        },
        headers: {
          'dev-ip': devIp,
          'dev-sc': 'true'
        }
      }
    }
  },
}

// 获取本机 IP
function getLocalIP(prefix) {
  const excludeNets = ['docker', 'cni', 'flannel', 'vi', 've']
  const os = require('os')
  const osType = os.type() // 系统类型
  const netInfo = os.networkInterfaces() // 网络信息
  const ipList = []
  if (prefix) {
    for (const netInfoKey in netInfo) {
      if (excludeNets.filter(item => netInfoKey.startsWith(item)).length === 0) {
        for (let i = 0; i < netInfo[netInfoKey].length; i++) {
          const net = netInfo[netInfoKey][i]
          if (net.family === 'IPv4' && net.address.startsWith(prefix)) {
            ipList.push(net.address)
          }
        }
      }
    }
  }
  if (ipList.length === 0) {
    if (osType === 'Windows_NT') {
      for (const dev in netInfo) {
        // win7的网络信息中显示为本地连接，win10显示为以太网
        if (dev === '本地连接' || dev === '以太网') {
          for (let j = 0; j < netInfo[dev].length; j++) {
            if (netInfo[dev][j].family === 'IPv4') {
              ipList.push(netInfo[dev][j].address)
            }
          }
        }
      }
    } else if (osType === 'Linux') {
      ipList.push(netInfo.eth0[0].address)
    } else if (osType === 'Darwin') {
      ipList.push(netInfo.en0[0].address)
    }
  }
  console.log('识别到的网卡信息', JSON.stringify(ipList))
  return ipList.length > 0 ? ipList[0] : ''
}

```

### 3.3 后端灰度处理

不论是 `gateway` 还是 `openfeign` 都是通过 spring 的 `loadbalancer` 进行应用选择的, 那我们通过实现或者继承 `ReactorServiceInstanceLoadBalancer` 来重写选择的过程.

```java
@Log4j2
public class LemesLoadBalancer implements ReactorServiceInstanceLoadBalancer{

    @Autowired
    private NacosDiscoveryProperties nacosDiscoveryProperties;

    final AtomicInteger position;
    // loadbalancer 提供的访问当前服务的名称
    final String serviceId;
    // loadbalancer 提供的访问的服务列表
    ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider;

    public LemesLoadBalancer(ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider, String serviceId) {
        this(serviceInstanceListSupplierProvider, serviceId, new Random().nextInt(1000));
    }

    public LemesLoadBalancer(ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider,
                             String serviceId, int seedPosition) {
        this.serviceId = serviceId;
        this.serviceInstanceListSupplierProvider = serviceInstanceListSupplierProvider;
        this.position = new AtomicInteger(seedPosition);
    }

    @Override
    public Mono<Response<ServiceInstance>> choose(Request request) {
        ServiceInstanceListSupplier supplier = serviceInstanceListSupplierProvider
                .getIfAvailable(NoopServiceInstanceListSupplier::new);

        RequestDataContext context = (RequestDataContext) request.getContext();
        RequestData clientRequest = context.getClientRequest();
        return supplier.get(request).next()
                .map(serviceInstances -> processInstanceResponse(clientRequest,supplier, serviceInstances));
    }
    private Response<ServiceInstance> processInstanceResponse(RequestData clientRequest,ServiceInstanceListSupplier supplier,
                                                              List<ServiceInstance> serviceInstances) {
        Response<ServiceInstance> serviceInstanceResponse = getInstanceResponse(clientRequest,serviceInstances);
        if (supplier instanceof SelectedInstanceCallback && serviceInstanceResponse.hasServer()) {
            ((SelectedInstanceCallback) supplier).selectedServiceInstance(serviceInstanceResponse.getServer());
        }
        return serviceInstanceResponse;
    }

    private Response<ServiceInstance> getInstanceResponse(RequestData clientRequest, List<ServiceInstance> instances) {
        if (instances.isEmpty()) {
            if (log.isWarnEnabled()) {
                log.warn("No servers available for service: " + serviceId);
            }
            return new EmptyResponse();
        }

        int pos = Math.abs(this.position.incrementAndGet());

        // 筛选后的服务列表
        List<ServiceInstance> filteredInstances;
        String devSmartConnect = clientRequest.getHeaders().getFirst(CommonConstants.DEV_SMART_CONNECT);
        if (StrUtil.equals(devSmartConnect, "true")) {
            String devIp = clientRequest.getHeaders().getFirst(CommonConstants.DEV_IP);
            // devIp 为空，为异常情况不处理，返回空实例集合
            if (StrUtil.isBlank(devIp)) {
                log.warn("devIp is NULL,No servers available for service: " + serviceId);
                return new EmptyResponse();
            }
            // 智能连接: 如果本地启动了服务，则优先访问本地服务，如果本地没有启动，则访问测试环境服务
            // 优先调用本地自有服务
            filteredInstances = instances.stream().filter(item -> StrUtil.equals(devIp, item.getHost())).collect(Collectors.toList());
            // 如果本地服务没有开启，则调用生产/测试服务
            if (CollUtil.isEmpty(filteredInstances)) {
                filteredInstances = instances.stream()
                        .filter(item -> StrUtil.equals(CommonConstants.LEMES_ENV_PRODUCT, item.getMetadata().get("lemes-env")))
                        .collect(Collectors.toList());
                // 解决开发环境无法访问 k8s 集群内 ip 的问题
                String oneNacosIp = nacosDiscoveryProperties.getServerAddr().split(",")[0].replaceAll(":[\\s\\S]*", "");
                filteredInstances.forEach(item -> {
                    NacosServiceInstance instance = (NacosServiceInstance) item;
                    // cloud 以 80 端口启动，认为是 k8s 内的应用
                    if (instance.getPort() == 80) {
                        instance.setHost(oneNacosIp);
                        instance.setPort(Integer.parseInt(item.getMetadata().get("port")));
                    }
                });
            }
        } else {
            // 不是智能访问，则只访问一个环境
            // 当前服务 ip
            String currentIp = nacosDiscoveryProperties.getIp();
            String lemesEnv = nacosDiscoveryProperties.getMetadata().get("lemes-env");
            filteredInstances = instances.stream()
                    .filter(item -> StrUtil.equals(lemesEnv, CommonConstants.LEMES_ENV_PRODUCT)
                            // 访问测试环境
                            ? StrUtil.equals(CommonConstants.LEMES_ENV_PRODUCT, item.getMetadata().get("lemes-env"))
                            // 访问开发环境
                            : StrUtil.equals(currentIp, item.getHost()))
                    .collect(Collectors.toList());
        }

        if (filteredInstances.isEmpty()) {
            log.warn("No oneself servers and beta servers available for service: " + serviceId + ", use other instances");
            // 找不到自己注册IP对应的服务和测试服务，则用nacos中其它的服务
            filteredInstances = instances;
        }

        //最终的返回的 serviceInstance
        ServiceInstance instance = filteredInstances.get(pos % filteredInstances.size());

        return new DefaultResponse(instance);
    }
}
```
