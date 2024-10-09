---
title: Find cause of memory leak on Java
enlink: find-cause-of-memory-leak-on-java
date: 2024-10-09 11:42:33
categories:
- 开发
- java
tags:
- java
- memory-leak
- jvm
- heap-dump
- jprofiler
---

# 一、背景

最近一年多, `job` 经常有如下**告警**, 告警内容如下

尊敬的用户，您关注的监控已触发警报，内容如下，请您关注！

> Dear user, the following monitoring alert you are concerned about is triggered, please pay attention!
> 
> Summary: [MonitoringID: MOC0000000606595] [Tingyun Alert] [LEMES-PCG-Prod_MajorGc]lemes-job-outbound-executor-idg-lssc-prodJVM每分钟Major GC时间 alert triggered
> 
> Notes: [Details:违反规则告警，APM应用实例/lemes-job-outbound-executor-54858fdf5b-dsxrq:0(10.188.138.17)，告警级别:严重,JVM每分钟Major GC时间大于阈值(JVM每分钟Major GC时间:2,490ms>阈值:900ms)] [EventID:85882166655658][TriggerTime: 2024-09-30 15:52:00]
> 
> 由于 job 停一下也没问题, 外加精力在其他任务上, 所以每次都是通过重启来解决问题

# 二、原因分析

国庆前又发了告警邮件, 觉得这个问题优先级可以提到前面了...

首先根据问题出现的频率分析, 大概是每个 `job` 运行几个月以上就开始报上面的告警, 根据不同 `job` 微服务的强度不同, 尤其是 `outbound`, 大概两个月就开始告警了...

所以基本定位问题为内存泄漏, 比如有框架或者开发的代码存在内存没释放的问题, 如IO流、数据库连接等没关闭的问题

这种问题可以直接对当前的微服务内存进行分析(导出内存快照)

1.我们的微服务是运行在 `k8s` 上的, 所以首先通过 `Rancher` 进入出问题的微服务的命令行, 通过如下命令对内存快照进行导出, 因为我们的 /data/logs  目录已经映射到宿主机了, 所以我们可以导出到这个目录

```bash
# 找到当前微服务的进程 id
jps
# 假如是9, 我们将其放到最后
jmap -dump:live,format=b,file=/data/logs/lemes-job-outbound-executor/lemes-job-outbound-executor.hprof 9

```

2.然后我们将导出的 `lemes-job-outbound-executor.hprof` 文件从宿主机上下载到本地电脑上

3.通过 `IDEA` 的 `Profiler` 进行内存分析, 可以在 IDEA→ View → Tool Windows → Profiler

4.然后点击 **Open Snapshot**  , 选择我们刚才下载的文件

![Open Snapshot](https://cdn.jsdelivr.net/gh/yelog/assets/images/202410091148848.png)

5.然后点击右边的 `Biggest Objects → Calculate retained size and biggest objects` 来进行大对象分析

6.发现在 `ThreadLocal` 中的 `ArrayDeque` 的占用非常大, 根据 `referent` 分析, 来自于 `DynamicDataSourceContextHolder` 中的, 并且查看 `elements` 中都是数据源的名字

![Find biggest objects](https://cdn.jsdelivr.net/gh/yelog/assets/images/202410091149828.png)

![DynamicDataSourceContextHolder](https://cdn.jsdelivr.net/gh/yelog/assets/images/202410091153140.png)

7.分析出是关于多数据源框架的问题, 还是要分析出来是使用问题, 还是框架中问题. 我们直接来到上面找到的类 DynamicDataSourceContextHolder, 找到了内存泄漏的变量是存储用于切换数据源的栈, 并且在这个文件中还找到了一句话, **防止内存泄漏，如手动调用了push可调用此方法确保清除**

![biggest objects](https://cdn.jsdelivr.net/gh/yelog/assets/images/202410091153383.png)

![message](https://cdn.jsdelivr.net/gh/yelog/assets/images/202410091154971.png)

8.也就是通过 `DynamicDataSourceContextHolder.push(xxx);` 切换数据源后, 是需要手动调用 `poll()` 方法进行移除, 或者在任务执行结束后调用 `clear()`, 进行清空.

9.查看代码后, 发现 `job` 中有 `DynamicDataSourceContextHolder.push(xxx)` 的操作, 却没有移除的方法, 所以定位到了问题.



# 三、解决问题

根据上一步我们知道了问题出在了没有进行移除操作, 移除操作有两种, 我们去每个执行 push 的地方进行 `poll()` 移除是比较麻烦的, 也不能避免再有同学漏掉 `poll()`, 从而导致问题复现.

所以我打算在 `job` 执行结束后, 统一调用 `DynamicDataSourceContextHolder.clear()` 来进行清空操作, 问了同事当前 `job` 框架是没有统一的开始和结束的地方, 但是所以 `job` 都是实现 `SimpleJob` 的 `execute` 方法来执行的, 所以可以使用切面来统一处理. 代码如下:

```java
package com.lenovo.lemes.job.core.executor.interceptor;
 
import com.baomidou.dynamic.datasource.toolkit.DynamicDataSourceContextHolder;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.stereotype.Component;
 
/**
 * 任务拦截器
 * 用于在任务执行前后做一些操作
 *
 * @author Yujie Yang
 * @date 2024/10/8 10:55
 */
@Aspect
@Component
@EnableAspectJAutoProxy
public class JobInterceptor {
 
    // 定义切入点，匹配实现了 SimpleJob 接口的类的 execute 方法
    // 正则解释: Pointcut 由两部分组成, 第一 execution 指明了切入的方法的全路径规则, 第二部分 target 限制了切入的类必须实现 SimpleJob 接口
    @Pointcut("execution(void com.lenovo.lemes.job..jobhandler..*.execute(org.apache.shardingsphere.elasticjob.api.ShardingContext)) && target(org.apache.shardingsphere.elasticjob.simple.job.SimpleJob)")
    public void executeMethodPointcut() {
    }
 
    @After("executeMethodPointcut()")
    public void afterJob(JoinPoint joinPoint) {
        // 在 execute 方法执行完成后清理数据源上下文
        DynamicDataSourceContextHolder.clear();
    }
 
}
```


