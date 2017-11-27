---
title: 搭建dubbo+zookeeper平台
permalink: 搭建dubbo+zookeeper平台
date: 2017-09-25 16:29:07
categories: 后端
tags:
- dubbo
- zookeeper
---
## 前言
本文将介绍在SpringMVC+Spring+Mybatis项目中添加 `dubbo` 作为 `rpc` 服务。

文末有项目代码地址。

## 一.搭建zookeeper
使用 docker 一句话创建：
```bash
docker run -dit --name zookeeper --hostname zookeeper-host -v /data:/data -p 2181:2181 jplock/zookeeper:latest
```
## 二.安装zkui（非必须）
这个项目为 zookeeper 提供一个 web 的管理界面。当然我们也可以直接在zookeeper中使用命令查看，所以此步骤可以忽略

在开始前需要安装 Java 环境、Maven 环境。

1. 到 [zkui](https://github.com/DeemOpen/zkui) 的项目中下载代码。
```bash
git clone https://github.com/DeemOpen/zkui.git
```
2. 执行 `mvn clean install` 生成jar文件。
3. 将config.cfg复制到上一步生成的jar文件所在目录，然后修改配置文件中的zookeeper地址。
4. 执行 `nohup java -jar zkui-2.0-SNAPSHOT-jar-with-dependencies.jar &`
5. 测试 `http://localhost:9090`，如果能看到如下页面，表示安装成功。

![登录页面](http://oncj6b2vl.bkt.clouddn.com/Fherw3peRgh-grmGz6qkNri5J1aG.png)
![首页](http://oncj6b2vl.bkt.clouddn.com/FvEVMOzSZBP4N4-Q14noQT_VsKF6.png)

## 三.使用dubbo
1. 在原来 SpringMVC+Spring+Mybatis 项目中，除了原来 spring 相关依赖外，还需要加入以下依赖
```xml
<dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>dubbo</artifactId>
        <version>2.5.5</version>
</dependency>
<dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.4.9</version>
</dependency>
<dependency>
        <groupId>com.101tec</groupId>
        <artifactId>zkclient</artifactId>
        <version>0.2</version>
</dependency>
```
2. 定义服务接口
```java
public interface IPersonService {
    List<Person> listAll();
    Person getById(Integer id);
    Integer delById(Person person);
    Integer updatePerson(Person person);
}
```
3. 定义服务实现类
```java
@Service
public class PersonService implements IPersonService {

    @Autowired
    PersonMapper personMapper;

    public List<Person> listAll() {
        return personMapper.findAll();
    }

    public Person getById(Integer id) {
        return personMapper.findOneById(id);
    }

    public Integer delById(Person person) {
        return personMapper.del(person);
    }

    public Integer updatePerson(Person person) {
        return personMapper.update(person);
    }
}
```
4. 配置生产者，注册服务信息
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
	http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!--定义了提供方应用信息，用于计算依赖关系；-->
    <dubbo:application name="demotest-provider" />

    <!-- 使用 zookeeper 注册中心暴露服务地址 -->
    <dubbo:registry address="zookeeper://192.168.0.86:2181"/>

    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880"/>

    <!-- 和本地bean一样实现服务 -->
    <bean id="personService" class="com.ssm.service.PersonService"/>

    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="com.ssm.iservice.IPersonService" ref="personService"/>
</beans>
```
5. 配置消费者，订阅服务
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
	http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="demo-consumer"/>

    <!-- 使用 zookeeper 注册中心暴露发现服务地址 -->
    <dubbo:registry address="zookeeper://192.168.0.86:2181"/>

    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="personService" check="false" interface="com.ssm.iservice.IPersonService"/>
</beans>
```
6. 调用远程服务
配置完成后，我们就可以像使用本地 bean 一样，使用 rpc 的 service；
```java
@Controller
public class IndexController {

    @Autowired
    IPersonService personService;

    @RequestMapping("/index.html")
    public String index(Model model) {
        RpcContext.getContext().setAttachment("index", "1");//测试ThreadLocal
        List<Person> list = personService.listAll();
        model.addAttribute("command",list);
        return "index";
    }
}
```

## 最后
至此，单机运行的 rpc 服务已搭建完成。

代码传送文 [ssm](https://github.com/yelog/ssm)
