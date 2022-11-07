---
title: '[Java]通过 CompletableFuture 实现异步多线程优化请求处理速度'
enlink: '[Java]optimize-request-processing-speed-by-completablefuture'
date: 2022-08-01 20:06:14
categories:
- 后端
tags:
- java
- concurrent
---

### 背景
我们在写后端请求的时候, 可能涉及多次 SQL 执行(或其他操作), 当这些请求相互不关联, 在顺序执行时就浪费了时间, 这些不需要先后顺序的操作可以通过多线程进行同时执行, 来加速整个逻辑的执行速度.

既然有了目标和大致思路, 我们首先想到的是 `CompletableFuture` 这个类, 他可以实现多线程和线程阻塞, 这样能够保证等待多个线程执行完成后再继续操作.

### CompletableFuture 是什么

首先我们先了解一下 `CompletableFuture` 是干什么, 接下来我们通过简单的示例来介绍他的作用.

```java
 long startTime = System.currentTimeMillis();
//生成几个任务
List<CompletableFuture<String>> futureList = new ArrayList<>();
futureList.add(CompletableFuture.supplyAsync(()->{
    sleep(4000);
    System.out.println("任务1 完成");
    return "任务1的数据";
}));
futureList.add(CompletableFuture.supplyAsync(()->{
    sleep(2000);
    System.out.println("任务2 完成");
    return "任务2的数据";
}));
futureList.add(CompletableFuture.supplyAsync(()->{
    sleep(3000);
    System.out.println("任务3 完成");
    return "任务3的数据";
}));
//完成任务
CompletableFuture<Void> allTask = CompletableFuture.allOf(futureList.toArray(new CompletableFuture[0]))
        .whenComplete((t, e) -> {
            System.out.println("所有任务都完成了， 返回结果集: "
                    + futureList.stream().map(CompletableFuture::join).collect(Collectors.joining(",")));
        });
// 阻塞主线程
allTask.join();
System.out.println("main end, cost: " + (System.currentTimeMillis() - startTime));
```
执行结果
```bash
任务2 完成
任务3 完成
任务1 完成
所有任务都完成了， 返回结果集: 任务1的数据,任务2的数据,任务3的数据
main end, cost: 4032
```
> **结果分析:** 我们需要执行3个任务, 3个任务同时执行, 互不影响
1. 其中任务2耗时最短,提前打印完成
2. 其次是任务3
3. 最后是执行1完成
4. 当所有任务完成后, 触发 `whenComplete` 方法, 打印任务的返回结果
5. 最后打印总耗时为 4.032s
6. 结论: 多线程执行后, 耗时取决于最耗时的操作, 而单线程是所有操作耗时之和

### 封装工具类
经过上面的测试, 通过 `CompletableFuture` 已经能够实现我们的预想, 为了操作方便, 我们将封装起来, 便于统一管理
```java
package org.yelog.java.usage.concurrent;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.function.Consumer;

/**
 * 执行并发任务
 *
 * @author yangyj13
 * @date 11/7/22 9:49 PM
 */
public class MultiTask<T> {
    private List<CompletableFuture<T>> futureList;

    /**
     * 添加待执行的任务
     *
     * @param completableFuture 任务
     * @return 当前对象
     */
    public MultiTask<T> addTask(CompletableFuture<T> completableFuture) {
        if (futureList == null) {
            futureList = new ArrayList<>();
        }
        futureList.add(completableFuture);
        return this;
    }

    /**
     * 添加待执行的任务
     *
     * @param consumer 任务
     * @return 当前对象
     */
    public MultiTask<T> addTask(Consumer<T> consumer) {
        addTask(CompletableFuture.supplyAsync(() -> {
            consumer.accept(null);
            return null;
        }));
        return this;
    }

    /**
     * 开始执行任务
     *
     * @param callback                当所有任务都完成后触发的回调方法
     * @param waitTaskExecuteComplete 是否阻塞主线程
     */
    private void execute(Consumer<List<T>> callback, Boolean waitTaskExecuteComplete) {
        CompletableFuture<Void> allFuture = CompletableFuture.allOf(futureList.toArray(new CompletableFuture[0]))
                .whenComplete((t, e) -> {
                    if (callback != null) {
                        List<T> objectList = new ArrayList<>();
                        futureList.forEach((future) -> {
                            objectList.add(future.join());
                        });
                        callback.accept(objectList);
                    }
                });
        if (callback != null || waitTaskExecuteComplete == null || waitTaskExecuteComplete) {
            allFuture.join();
        }
    }

    /**
     * 开始执行任务
     * 等待所有任务完成（阻塞主线程）
     */
    public void execute() {
        execute(null, true);
    }

    /**
     * 开始执行任务
     *
     * @param waitTaskExecuteComplete 是否阻塞主线程
     */
    public void execute(Boolean waitTaskExecuteComplete) {
        execute(null, waitTaskExecuteComplete);
    }

    /**
     * 开始执行任务
     *
     * @param callback 当所有任务都完成后触发的回调方法
     */
    public void execute(Consumer<List<T>> callback) {
        execute(callback, true);
    }
}
```
那么上一步我们测试的流程转换成工具类后如下
```java
long startTime = System.currentTimeMillis();
MultiTask<String> multiTask = new MultiTask<>();
multiTask.addTask(t -> {
    sleep(1000);
    System.out.println("任务1 完成");
}).addTask(t -> {
    sleep(3000);
    System.out.println("任务2 完成");
}).addTask(CompletableFuture.supplyAsync(()->{
    sleep(2000);
    System.out.println("任务3 完成");
    return "任务3的数据";
})).execute(resultList->{
    System.out.println("all complete: " + resultList);
});
System.out.println("main end, cost: " + (System.currentTimeMillis() - startTime));

```

### 应用到实际的效果
执行两次数据库的操作如下
```java

```

<++>
