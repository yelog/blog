---
title: JDK 垃圾回收介绍
enlink: jdk-gc
date: 2024-08-30 11:12:15
categories:
- 后端
tags:
- java
- gc
- jdk
---

## 推荐配置
### 容器内
```bash
-XX:+UseContainerSupport -XX:InitialRAMPercentage=75.0 -XX:MaxRAMPercentage=75.0 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/admin/nas/dump-${POD_IP}-$(date '+%s').hprof
```

## G1
https://juejin.cn/post/6856222574155104270
https://juejin.cn/post/7007343142328352804
https://blog.csdn.net/jiguansheng/article/details/105406343
https://tech.meituan.com/2016/09/23/g1.html

### 概念

#### 新生代

新生代又叫年轻代，大多数对象在新生代中被创建，很多对象的生命周期很短。每次新生代的垃圾回收（又称Young GC、Minor GC、YGC）后只有少量对象存活，所以使用复制算法，只需少量的复制操作成本就可以完成回收。

新生代内又分三个区：一个Eden区，两个Survivor区(S0、S1，又称From Survivor、To Survivor)，大部分对象在Eden区中生成。当Eden区满时，还存活的对象将被复制到两个Survivor区（中的一个）。当这个Survivor区满时，此区的存活且不满足晋升到老年代条件的对象将被复制到另外一个Survivor区。对象每经历一次复制，年龄加1，达到晋升年龄阈值后，转移到老年代

#### 老年代

在新生代中经历了N次垃圾回收后仍然存活的对象，就会被放到老年代，该区域中对象存活率高。老年代的垃圾回收通常使用“标记-整理”算法


### YGC

触发条件: 新生代占据整个堆大小的 60%

新生代: Eden Space + Survivor Space

新生代晋升老年代条件
- 对象超过 age 阈值 15
- 附质量过程超过 50, age 最大的放到老年代

-XX:MaxGCPauseMils 默认为200ms
在优先时间内尽量回收垃圾多的区域, 让时间效率最大化

Young GC 每次都会引起全线停顿(Stop-The-World)，暂停所有的应用线程，停顿时间相对老年代GC的造成的停顿，几乎可以忽略不计

### Mixed GC
新生代和老年代进行收集和整理
触发条件: 老年代超过堆 45%


### 压缩算法回收 STW(Stop-The-World)

G1 开辟一块最多 5% 堆空间的内存用于标记压缩的数据交换, 过程产生 STW, STW 200ms内最多回收 10% 垃圾最多的区域, 回收后检查老年代是否低于 45%, 未达标继续再来一次, 最多 8 次, 8次未达标 Serial Old GC(Full GC)

### Other

used = resident + swapped pages

## jstat

[doc](https://docs.oracle.com/en/java/javase/14/docs/specs/man/jstat.html)

### -gc
| head | description                                                |
|------|------------------------------------------------------------|
| S0C  | Current survivor space 0 capacity (KB).                    |
| S1C  | Current survivor space 1 capacity (KB).                    |
| S0U  | Survivor space 0 utilization (KB).                         |
| S1U  | Survivor space 1 utilization (KB).                         |
| EC   | Current eden space capacity (KB).                          |
| EU   | Eden space utilization (KB).                               |
| OC   | Current old space capacity (KB).                           |
| OU   | Old space utilization (KB).                                |
| MC   | Metaspace Committed Size (KB).                             |
| MU   | Metaspace utilization (KB).                                |
| CCSC | Compressed class committed size (KB).                      |
| CCSU | Compressed class space used (KB).                          |
| YGC  | Number of young generation garbage collection (GC) events. |
| YGCT | Young generation garbage collection time.                  |
| FGC  | Number of full GC events.                                  |
| FGCT | Full garbage collection time.                              |
| GCT  | Total garbage collection time.                             |

### jstat -gcutil 10

S0: Survivor 0区的空间使用率 Survivor space 0 utilization as a percentage of the space's current capacity.

S1: Survivor 1区的空间使用率 Survivor space 1 utilization as a percentage of the space's current capacity.

E: Eden区的空间使用率 Eden space utilization as a percentage of the space's current capacity.

O: 老年代的空间使用率 Old space utilization as a percentage of the space's current capacity.

M: 元数据的空间使用率 Metaspace utilization as a percentage of the space's current capacity.

CCS: 类指针压缩空间使用率 Compressed class space utilization as a percentage.

YGC: 新生代GC次数 Number of young generation GC events.

YGCT: 新生代GC总时长（从应用程序启动到采样时年轻代中gc所用时间 单位：s）
	  Young generation garbage collection time.

FGC: Full GC次数 Number of full GC events.

FGCT: Full GC总时长（从应用程序启动到采样时old代(全gc)gc所用时间 单位：s）
	  Full garbage collection time.

GCT: 总共的GC时长 （从应用程序启动到采样时gc用的总时间 单位：s）Total garbage collection time.


## 查询当前使用的是什么垃圾回收器

### 查看是否通过 JVM 参数指定了虚拟机类型
```bash
ps -ef | grep webservice
```

### 查询 JDK 默认虚拟机类型
```bash
java -XX:+PrintCommandLineFlags -version
```

## others
```bash
# 查看堆空间占用类， 前20条
jmap -histo PID | head -n20
```
bash-4.2# jmap -histo 9 | head -n20
 num     #instances         #bytes  class name (module)
-------------------------------------------------------
   1:        544456     3280814896  [C (java.base@11.0.18)
   2:       3076195     1819165352  [B (java.base@11.0.18)
   3:       1029042      626773184  [I (java.base@11.0.18)
   4:       6287573      201202336  java.lang.ClassValue$Entry (java.base@11.0.18)
   5:       4575213      183008520  java.util.WeakHashMap$Entry (java.base@11.0.18)
   6:       2214270      141713280  java.util.concurrent.ConcurrentHashMap (java.base@11.0.18)
   7:       1085202      130240080  [Ljava.lang.Object; (java.base@11.0.18)
   8:       1492883      130059632  [Ljava.util.WeakHashMap$Entry; (java.base@11.0.18)
   9:       3101147      124045880  java.lang.ref.SoftReference (java.base@11.0.18)
  10:       3068059      122722360  java.lang.invoke.BoundMethodHandle$Species_LL (java.base@11.0.18)
  11:       4341767      104202408  java.lang.ClassValue$Version (java.base@11.0.18)
  12:       1506502       84364112  jdk.nashorn.internal.runtime.ScriptFunction (jdk.scripting.nashorn@11.0.18)
  13:       2004817       80192680  java.util.TreeMap$Entry (java.base@11.0.18)
  14:       2316002       74112064  java.util.HashMap$Node (java.base@11.0.18)
  15:       2282867       73051744  jdk.nashorn.internal.runtime.PropertyHashMap$Element (jdk.scripting.nashorn@11.0.18)
  16:       4341767       69468272  java.lang.ClassValue$Identity (java.base@11.0.18)
  17:       1435938       68925024  java.util.WeakHashMap (java.base@11.0.18)
  18:       1657484       66299360  jdk.nashorn.internal.runtime.CompiledFunction (jdk.scripting.nashorn@11.0.18)

[C is a char[]
[B is a byte[]
[I is a int[]
[S is a short[]
[[I is a int[][]

```bash
# 查看堆空间存活, 注意, 会触发 full gc
jmap -histo:live PID | head -n20
```

