# java虚拟机



## 垃圾回收器的评价指标

1. 吞吐量。吞吐量 = 用户程序运行时间/总运行时间。总运行时间=用户程序运行时间+垃圾回收时间
2. 暂停时间。执行垃圾回收时，用户程序暂停的时间
3. 内存占用。java堆占用的时间



## 垃圾回收器

主流的垃圾回收器有以下几种

1. 串行垃圾回收器：Serial，SerialOld
2. 并行垃圾回收器：ParNew，ParallelScavenge，ParallelOld
3. 并发垃圾回收器：CMS，G1

![4](/java虚拟机/.assert/java虚拟机参数/172f888c00412b4c~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)



- 新生代收集器： Serial、 ParNew、Parallel Scavenge；
- 老年代收集器： Serial 0ld、 Parallel 0ld、 CMS；
- 整堆收集器： G1；

![5](/java虚拟机/.assert/java虚拟机参数/172f88913bd8914c~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

### 垃圾回收器搭配

![6](/java虚拟机/.assert/java虚拟机参数/172f88972faa533c~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

搭配的组合

1. **SerialGC+SerialOld**
2. SerialGC+CMS（java1.8中已经废弃，jdk1.9完成移除）
3. **ParNew+CMS+SerialOld**。其中serialOld做为CMS备用的垃圾回收器
4. ParNew+Serial Old（java1.8中已经废弃，jdk1.9完成移除）
5. Parallel Scavenge+SerialOldGC（jdk14中已经弃用）
6. **Parallel Scavenge+Parallel Old**
7. G1



### 垃圾回收器介绍

#### Serial和Serial Old

Serial和SerialOld是串行垃圾回收器

##### 回收策略

年轻代采用复制，老年代使用标记整理算法

![9](/java虚拟机/.assert/java虚拟机参数/172f88af4c12170b~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)



##### 用处

1. 在client模式下，作为默认的垃圾回收器
2. 在server模式下，作为CMS的补充



##### 参数

使用-XX:+UseSerialGC启动Serial垃圾回收器



#### ParNew

是一款针对年轻代的并行垃圾回收器

##### 回收策略

除并行外，其他回收特性与Serial一致

![10](/java虚拟机/.assert/java虚拟机参数/172f88b5da16f393~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

##### 用处

1. 当下服务端通常为多cpu的场景，ParNew回收器比Serial更高效，GC的时间更短



##### 参数

-XX:ParallelGCThreads：执行垃圾回收的线程数，默认开启与cpu个数相同的线程

#### Parallel Scavenge/Parallel Old（吞吐量优先）

Parallel回收器也是采用并行设计，但是其以高吞吐量优先。这是它与ParNew一个重要区别。在一些侧重计算的程序中，需要更快的完成计算，此时吞吐量就非常重要



##### 回收策略

1. 年轻代使用复制算法，老年代使用标记整理算法
2. 回收时都是采用多线程并发

![11](/java虚拟机/.assert/java虚拟机参数/172f88bd8272c80d~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)



##### 用处

1. 适用于后台计算的场景，不使用侧重交互的场景



##### 参数

1. 使用-XX:+UseParallelGC或者-XX:+UseParallelOldGC启用Parallel垃圾回收器
2. -XX:ParallelGCThreads=8执行垃圾回收线程个数。默认在cpu个数小于等于8时，回收线程数等于cpu个数；在cpu个数大于8时，回收线程数=3+(5*cpu个数/8)
3. -XX:MaxGCPauseMillis=10 设置垃圾回收器最大的停顿时间，设置后，回收器会尽量将停顿时间控制在该时间内。
4. -XX:GCTimeRatio=99 设置GC时间的比例。GC时间比例=1/(1+N)，取值范围为0到100，默认值为99，也就是GC时间为1%。
5. -XX:+UseAdaptiveSizePolicy 是否使用自适应策略。在这种模式下，年轻代的大小、Eden和Survivor的比例、晋升老年 代的对象年龄等参数会被自动调整，已达到在堆大小、吞吐量和停顿时间之间的平衡点。在手动调优比较困难的场合，可以直接使用这种自适应的方式，仅指 定虚拟机的最大堆、目标的吞吐量（GCTimeRatio）和停顿时间（MaxGCPauseMills），让虚拟机自己完成调优工作



#### CMS

jdk1.5推出的真正意义上并发的垃圾回收器，可以让用户线程和gc线程同时运行



#### 回收策略

CMS仅回收老年代，采用标记清理的方式。整个回收分为4个阶段

1. 初始标记。标记GCroot，该操作是单线程并STW，但是由于gcroot数量相对较少，该操作的停止时间比较短。
2. 并发标记。通过GCroot并发标记其他对象，该操作是多线程，不会停止STW，因为要扫描所有对象，因此该操作时间较长。
3. 重新标记。因为在并发标记时，用户线程会运行，可能会导致部分对象的变化，因此需要重新标记对象。该操作是多线程并SWT。该操作比初始标记的时间长，但是比并发标记的时间要短
4. 并发清除。通过标记结果，清理垃圾对象。该操作是单线程，不会停止STW。



由于在垃圾回收阶段，用户线程还在运行，因此CMS并不会等内存使用完才开始垃圾回收，而是在内存使用达到一定阈值是操作垃圾回收。

另外，如果在CMS垃圾回收阶段，剩余的内存不能维持用户线程运行，将会出现Concurrent Mode Failure错误，然后使用SerialOld对老年代进行回收



##### CMS的优缺点

1. 由于采用标记清理的方式，会造成内存碎片，可能会导致分配大对象时，提前触发FullGC
2. 垃圾回收线程与用户线程同时运行时，垃圾回收线程会占用用户资源
3. 由于垃圾回收线程与用户线程同时运行，在并发标记过程中产生的垃圾没办法在本次垃圾回收中清理，产生浮动垃圾。这些垃圾只能在下次回收时才能回收，占用堆空间。



##### CMS优点

1. GC时，暂停时间短，适用侧重时延的场景



##### 参数

1. -XX:+UseConcMarkSweepGc。使用CMS垃圾回收器回收老年代，使用ParNew回收新生代
2. -XX:+CMSInitiatingOccupanyFraction。触发FullGC的内存占比。jdk5默认是68，jdk6及以后是92
3. -XX:+UseCMSCompactAtFullCollection。在垃圾回收后，对内存进行整理，减少内存碎片，但是会造成停顿的时间变长
4. -XX:CMSFullGCsBeforeCompaction=4。在多少次垃圾回收后，对内存进行整理
5. -XX:ParallelCMSThreads=5。CMS垃圾回收线程数。默认是（ParallelGCThreads+3） /4，ParallelGCThreads为新生代垃圾回收线程数



#### G1



### 垃圾回收器基本原理

#### 标记

1. 可达性分析法
2. 引用计数法



#### 清理算法

1. 复制
2. 标记清理
3. 标记整理



#### gcroot

1).虚拟机栈中引用的对象
    比如：各个线程被调用的方法中使用到的参数，局部变量等
2).本地方法栈内JNI(通常所说的本地方法)引用的对象
3).方法区中类静态属性引用的对象
  	比如：字符串常量池(String Table)里的引用
4).所有被同步锁synchronized持有的对象
5).java虚拟机内部的引用
	比如：基本数据类型对应的Class对象，一些常驻的异常对象(如：NullPointException、OutOfMemoryError)，系统类加载器
6).反映java虚拟机内部情况的JMXBean，JVMTI中注册的回调、本地代码缓存等



## jvm参数

### 内存相关参数

| 参数                        | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| -Xmx（-XX:MaxHeapSize）     | 堆最大空间                                                   |
| -Xms（-XX:InitialHeapSize） | 堆初始空间                                                   |
| -Xmn（-XX:MaxNewSize）      | 堆新生代最大空间                                             |
| -XX:NewSize                 | 堆新生代初始化大小                                           |
| -XX:SurvivorRatio           | 新生代空间中，Eden区与Survivor区的比例。其中Survivor有两个   |
| -XX:NewRatio                | 老年代与新生代的内存比例。如果设置了-Xmn，-XX:NewRatio会失效 |
| -XX:OldSize                 | 老年代初始大小                                               |
| -XX:PermSize                | 永久代初始大小                                               |
| -XX:MaxPermSize             | 永久代最大空间                                               |

```
-Xmx10240m -Xms10240m -Xmn5120m -XX:SurvivorRatio=3
```



![img](https://images2015.cnblogs.com/blog/285763/201611/285763-20161118115316810-1826109116.png)





/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.332.b09-1.fc35.aarch64



### 回收器参数

#### 参数列表

| 参数                    | 说明                                                         | 组合                          |
| ----------------------- | ------------------------------------------------------------ | ----------------------------- |
| -XX:+UseSerialGC        | 表示新生代使用Serial，老年代使用Serial Old                   | SerialGC+SerialOld            |
| -XX:+UseParallelGC      | 表示新生代使用Parallel Scavenge，同时会隐式激活参数-XX:+UseParallelOldGC | Parallel Scavenge+SerialOldGC |
| -XX:+UseParallelOldGC   | 表示老年代使用Parallel Old，同时会隐式激活参数-XX:+UseParallelGC | Parallel Scavenge+SerialOldGC |
| -XX:+UseConcMarkSweepGC | 表示老年代使用CMS，默认年轻代为ParNew                        | ParNew+CMS+SerialOld          |
| -XX:+UseParNewGC        | 表示年轻代使用ParNew，默认老年代为Serial Old                 | ParNew+Serial Old             |
| -XX:+UseG1GC            | 表示使用G1                                                   | G1                            |



-XX:+UseSerialGC参数不能与-XX:+UseParallelGC和-XX:+UseParallelOldGC一起使用



搭配的组合

1. SerialGC+CMS（java1.8中已经废弃，jdk1.9完成移除）：无法使用这种组合
2. Parallel Scavenge+SerialOldGC（jdk14中已经弃用）：无法使用这种组合



#### 查看默认垃圾回收器

```
java -XX:+PrintCommandLineFlags -version
```

![image-20220612092114813](/java虚拟机/.assert/java虚拟机参数/image-20220612092114813.png)

如果输出的参数中没有指定垃圾回收器的参数，那么默认的垃圾回收器很可能是-XX:+UseSerialGC。





#### 根据垃圾回收日志确定回收器算法

添加下面参数后，可以打印垃圾回收的日志

```
 -XX:+PrintGCDetails -Xloggc:gc-%t.log
```

1. DefNew表示Serial，Tenured表示SerialOld

![image-20220612100859112](/java虚拟机/.assert/java虚拟机参数/image-20220612100859112.png)

2. G1垃圾回收器

<img src="/java虚拟机/.assert/java虚拟机参数/image-20220612101138428.png" alt="image-20220612101138428" style="zoom:50%;" />

3. ParNew表示ParNew，CMS表示CMS

![image-20220612101332121](/java虚拟机/.assert/java虚拟机参数/image-20220612101332121.png)

4. PSYoungGen表示Parallel Scavenge，ParOldGen表示ParallelOld

![image-20220612101526203](/java虚拟机/.assert/java虚拟机参数/image-20220612101526203.png)



### gc日志

运行jvm时，可以将每次gc的信息打印出来，便于分析问题



#### 打印gc日志

```
-XX:PrintGC 打印GC简要信息
-XX:+PrintGCDetails  打印gc详细信息
-Xloggc:gc-%t.log 指定垃圾回收日志输出文件地址
-XX:+PrintGCTimeStamps 打印垃圾回收的时间戳，相对jvm运行时间，不会打印日期，该参数默认开启
-XX:+PrintGCDateStamps 打印垃圾回收的日期时间

日志轮转，不要开启
-XX:+UseGCLogFileRotation 
-XX:NumberOfGCLogFiles=3
-XX:GCLogFileSize=512k
```



```
-XX:+PrintGCDetails -Xloggc:gc-%t.log -XX:+PrintGCDateStamps
```





#### gc日志格式分析



```
2022-06-12T10:42:51.804+0800: 2.097: [Full GC (Allocation Failure) [PSYoungGen: 288K->0K(9216K)] [ParOldGen: 81928K->81928K(81928K)] 82216K->41223K(101376K), [Metaspace: 2977K->2977K(1056768K)], 0.0057102 secs] [Times: user=0.01 sys=0.00, real=0.01 secs]

2022-06-12T10:42:51.804+0800：gc的日期，需开启参数-XX:+PrintGCDateStamps
2.097：gc的时间戳，相对于jvm运行时间
Full GC (Allocation Failure)：垃圾回收原因
[PSYoungGen: 288K->0K(9216K)]：Parallel Scavenge年轻代垃圾回收信息，288K，0K，9216k分别表示年轻代垃圾回收前，回收后和总空间大小
[ParOldGen: 81928K->41223K(92160K)]：Parallel Old老年代垃圾回收信息，81928K，81928K，81928K分别表示老年代垃圾回收前，回收后和总空间大小
82216K->41223K(101376K)：表示整个jvm堆的垃圾回收信息，82216K，41223K，101376K分别表示jvm堆回收前，回收后和总空间大小
[Metaspace: 2977K->2977K(1056768K)]：表示元数据区垃圾回收信息，2977K，2977K，1056768K分别表示回收前，回收后和总空间大小
0.0057102 secs：垃圾回收总耗时
[Times: user=0.01 sys=0.00, real=0.01 secs]：表示垃圾回收时间详情，user表示垃圾回收用户态占用cpu时间，sys表示垃圾回收内核态占用cpu时间，real表示垃圾回收总耗时（STW时间）
```





## outofmemory自动导出java堆



```
-XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=${目录} 堆路径，也可以指定文件名，如果不指定名称默认名称为java_<pid>_<date>_<time>_heapDump.hprof
```













```
-Xms20m -Xmx20m -Xmn2m -XX:SurvivorRatio=2 -XX:+PrintGCDetails -XX:+UseSerialGC

# GC日志输出的文件路径
-Xloggc:F:/jvm/log/gc-%t.log
# 开启日志文件分割
-XX:+UseGCLogFileRotation
# 每个文件上限大小，超过就触发分割
-XX:+UseGCLogFileRotation
# 最多分割几个文件，超过之后从头文件开始写
-XX:+UseGCLogFileRotation


java -XX:+PrintCommandLineFlags -version


 -XX:+UseSerialGC 

有以下几种垃圾回收器： UseSerialGC UseParallelGC UseConcMarkSweepGC UseParNewGC UseParallelOldGC
UseG1GC
```



![img](/java虚拟机/.assert/java虚拟机参数/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ1NDUwODU=,size_16,color_FFFFFF,t_70.png)

![img](/java虚拟机/.assert/java虚拟机参数/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ1NDUwODU=,size_16,color_FFFFFF,t_70-20220610235244281.png)



## 垃圾回收器



### UseParallelGC

![image-20220611120240867](/java虚拟机/.assert/java虚拟机参数/image-20220611120240867.png)



### UseConcMarkSweepGC

![image-20220611120458097](/java虚拟机/.assert/java虚拟机参数/image-20220611120458097.png)



### 都不加

![image-20220611145936980](/java虚拟机/.assert/java虚拟机参数/image-20220611145936980.png)



### UseG1GC

![image-20220611150545279](/java虚拟机/.assert/java虚拟机参数/image-20220611150545279.png)



### UseSerialGC

![image-20220611150836630](/java虚拟机/.assert/java虚拟机参数/image-20220611150836630.png)



java1.8默认垃圾回收器为UseParallelGC，如果机器线cpu个数为1，默认垃圾回收器为UseSerialGC



## 参考文章

https://juejin.cn/post/6844904200678146061



