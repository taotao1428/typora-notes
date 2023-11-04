# hadoop入门

Apache Hadoop 是一种开源框架，用于高效存储和处理从 GB 级到 PB 级的大型数据集。利用 Hadoop，您可以将多台计算机组成集群以便更快地并行分析海量数据集，而不是使用一台大型计算机来存储和处理数据。



Hadoop 由四个主要模块组成：

- Hadoop 分布式文件系统 (HDFS)—一个在标准或低端硬件上运行的分布式文件系统。除了更高容错和原生支持大型数据集，HDFS 还提供比传统文件系统更出色的数据吞吐量。

- Yet Another Resource Negotiator (YARN)—管理与监控集群节点和资源使用情况。它会对作业和任务进行安排。

- MapReduce—一个帮助计划对数据运行并行计算的框架。该 Map 任务会提取输入数据，转换成能采用键值对形式对其进行计算的数据集。Reduce 任务会使用 Map 任务的输出来对输出进行汇总，并提供所需的结果。

- Hadoop Common—提供可在所有模块上使用的常见 Java 库。





hadoop核心的组件都与谷歌谷歌的影子

![img](/开源框架/hadoop/.assert/hadoop入门/v2-5ce9db6d9b9b17c005cc8f127ccfe8be_1440w.webp)





## HDFS

HDFS是分布式文件系统，包含NameNode和DataNode两种节点。

**NameNode：**是Master节点（主节点），可以看作是分布式文件系统中的管理者，主要负责管理文件系统的命名空间、集群配置信息和存储块的复制等。NameNode会将文件系统的Meta-data存储在内存中，这些信息主要包括了文件信息、每一个文件对应的文件块的信息和每一个文件块在DataNode的信息等。

**DataNode：**是Slave节点（从节点），是文件存储的基本单元，它将Block存储在本地文件系统中，保存了Block的Meta-data，同时周期性地将所有存在的Block信息发送给NameNode。

**Client：**切分文件；访问HDFS；与NameNode交互，获得文件位置信息；与DataNode交互，读取和写入数据。 



还有一个**Block（块）**的概念：Block是HDFS中的基本读写单元；HDFS中的文件都是被切割为block（块）进行存储的；这些块被复制到多个DataNode中；块的大小（通常为64MB）和复制的块数量在创建文件时由Client决定。

![img](/开源框架/hadoop/.assert/hadoop入门/v2-12bac7206f243ab217e58a23a555da47_1440w.webp)



### 写入流程



![img](/开源框架/hadoop/.assert/hadoop入门/v2-cbf6dfb751bcf61d74726948f3df550c_1440w.webp)



1 用户向Client（客户机）提出请求。例如，需要写入200MB的数据。

2 Client制定计划：将数据按照64MB为块，进行切割；所有的块都保存三份。

3 Client将大文件切分成块（block）。

4 针对第一个块，Client告诉NameNode（主控节点），请帮助我，将64MB的块复制三份。

5 NameNode告诉Client三个DataNode（数据节点）的地址，并且将它们根据到Client的距离，进行了排序。

6 Client把数据和清单发给第一个DataNode。

7 第一个DataNode将数据复制给第二个DataNode。

8 第二个DataNode将数据复制给第三个DataNode。

9 如果某一个块的所有数据都已写入，就会向NameNode反馈已完成。

10 对第二个Block，也进行相同的操作。

11 所有Block都完成后，关闭文件。NameNode会将数据持久化到磁盘上。



### 读取流程

![img](/开源框架/hadoop/.assert/hadoop入门/v2-70f0c5acbc21cfacae10c19981522395_1440w.webp)





1 用户向Client提出读取请求。

2 Client向NameNode请求这个文件的所有信息。

3 NameNode将给Client这个文件的块列表，以及存储各个块的数据节点清单（按照和客户端的距离排序）。

4 Client从距离最近的数据节点下载所需的块。



（注意：以上只是简化的描述，实际过程会更加复杂。）



### GFS论文

中文版（里面缺少论文中的图）：https://blog.csdn.net/xiaotom5/article/details/8142728

英文版：https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/gfs-sosp2003.pdf



1. master节点使用记录操作日志方式，保证操作的原子性和持久性。另外master日志会传输到其他节点，避免单点问题
2. 在向chunk写入数据时，为了保证写入的一致性，会由主chunk控制写入的顺序。主chunk节点会向master节点续约，以保持自己的主chunk地位。
3. GFS一致性松弛一致性介绍以及问题。https://mp.weixin.qq.com/s/ut8Q7vXa5Lm0auNaN2_Emg





## MapReduce

MapReduce其实是一种编程模型。这个模型的核心步骤主要分两部分：**Map（映射）**和**Reduce（归约）**。



当你向MapReduce框架提交一个计算作业时，它会首先把计算作业拆分成若干个**Map任务**，然后分配到不同的节点上去执行，每一个Map任务处理输入数据中的一部分，当Map任务完成后，它会生成一些中间文件，这些中间文件将会作为**Reduce任务**的输入数据。Reduce任务的主要目标就是把前面若干个Map的输出汇总到一起并输出。

![img](/开源框架/hadoop/.assert/hadoop入门/v2-eb95cb2b3b945f38e89758e9f8ecebb6_1440w.webp)

我们来举个例子。

![img](/开源框架/hadoop/.assert/hadoop入门/v2-42b95bf6958ee05771bddfdf0c48ac60_1440w.webp)

### JobTracker**和**TaskTracker

JobTracker用于调度和管理其它的TaskTracker。JobTracker可以运行于集群中任一台计算机上。TaskTracker 负责执行任务，必须运行于 DataNode 上。

<img src="/开源框架/hadoop/.assert/hadoop入门/v2-624d63d33f832cbb64235f23ad22809c_1440w.webp" alt="img" style="zoom:50%;" />



### map和shuffle和reduce流程

https://blog.csdn.net/weixin_45366499/article/details/109030203

![在这里插入图片描述](/开源框架/hadoop/.assert/hadoop入门/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM2NjQ5OQ==,size_16,color_FFFFFF,t_70.jpeg)

1. 每个map只处理部分数据
2. map处理数据时，会先将数据写入到环形内存中，当内存使用率达到80%，会将内存写入到文件中。每个文件会包含不同分区的数据，在写入文件前，会对数据进行排序，保证分区内的数据是排序后的。当map所有的数据处理完成后，可能会存在多个文件。此时会将文件合并成1个文件，该文件会保证分区内的数据已排序，供reduce获取。
3. 如果配置了combiner，在map端，会先试用combiner对数据进行一次reduce，减少map到reduce数据传输。例如如果只需要统计最大值，map端可以先计算最大值，然后将最大值传给reduce。reduce只需要比较不同map的最大值就可以获取总的最大值。
4. reduce不同map获取自己分区的数据，然后进行排序合并，最后使用reduce方法对数据进行统计



## YARN（资源管理框架）

2.0版本中，在HDFS之上，增加了**YARN（资源管理框架）**层。它是一个资源管理模块，为各类应用程序提供资源管理和调度。



![img](/开源框架/hadoop/.assert/hadoop入门/v2-5d528279a099eddac50b03651808d222_1440w.jpeg)

### yarn架构

![img](/开源框架/hadoop/.assert/hadoop入门/v2-b5804eb2bb1b3f2ff091c18bbf915819_1440w.webp)

https://zhuanlan.zhihu.com/p/55108455



#### container

容器(Container)这个东西是 Yarn 对资源做的一层抽象。就像我们平时开发过程中，经常需要对底层一些东西进行封装，只提供给上层一个调用接口一样，Yarn 对资源的管理也是用到了这种思想。



![img](/开源框架/hadoop/.assert/hadoop入门/v2-be002ea685e5b64c2b579d9b9b17d998_1440w.webp)



如上所示，Yarn 将CPU核数，内存这些计算资源都封装成为一个个的容器(Container)。需要注意两点：

- 容器由 NodeManager 启动和管理，并被它所监控。
- 容器被 ResourceManager 进行调度。



#### 组件

再看最上面的图，我们能直观发现的两个主要的组件是 ResourceManager 和 NodeManager ，但其实还有一个 ApplicationMaster 在图中没有直观显示。我们分别来看这三个组件。

**ResourceManager**

我们先来说说上图中最中央的那个 ResourceManager(RM)。从名字上我们就能知道这个组件是负责资源管理的，整个系统有且只有一个 RM ，来负责资源的调度。它也包含了两个主要的组件：定时调用器(Scheduler)以及应用管理器(ApplicationManager)。

定时调度器(Scheduler)：从本质上来说，定时调度器就是一种策略，或者说一种算法。当 Client 提交一个任务的时候，它会根据所需要的资源以及当前集群的资源状况进行分配。注意，它只负责向应用程序分配资源，并不做监控以及应用程序的状态跟踪。

应用管理器(ApplicationManager)：同样，听名字就能大概知道它是干嘛的。应用管理器就是负责管理 Client 用户提交的应用。上面不是说到定时调度器(Scheduler)不对用户提交的程序监控嘛，其实啊，监控应用的工作正是由应用管理器(ApplicationManager)完成的。

**ApplicationMaster**

每当 Client 提交一个 Application 时候，就会新建一个 ApplicationMaster 。由这个 ApplicationMaster 去与 ResourceManager 申请容器资源，获得资源后会将要运行的程序发送到容器上启动，然后进行分布式计算。

**NodeManager**

NodeManager 是 ResourceManager 在每台机器的上代理，负责容器的管理，并监控他们的资源使用情况(cpu，内存，磁盘及网络等)，以及向 ResourceManager/Scheduler 提供这些资源使用报告。



### 提交applicaiton到yarn的流程

<img src="/开源框架/hadoop/.assert/hadoop入门/v2-dfa7e31fd818a9bb29008b953a56be97_1440w.webp" alt="img" style="zoom: 67%;" />

1. 客户端向resourceManager提交一个任务
2. yarn会分配一个container运行applicationMaster。
3. applicationMaster对任务进行拆解，并向resourceManager申请资源，并向resourceManager发送心跳，然后applicationMaster向NodeManager启动容器，运行子任务。
4. 容器中运行的任务会向 ApplicationMaster 发送心跳，汇报自身情况。当程序运行完成后， ApplicationMaster 再向 ResourceManager 注销并释放容器资源





## **Hadoop的生态圈**



经过时间的累积，Hadoop已经从最开始的两三个组件，发展成一个拥有20多个部件的生态系统。



![img](/开源框架/hadoop/.assert/hadoop入门/v2-143af4d32b9d3a9a4fc749cf410882c1_1440w.webp)



在整个Hadoop架构中，计算框架起到承上启下的作用，一方面可以操作HDFS中的数据，另一方面可以被封装，提供Hive、Pig这样的上层组件的调用。



我们简单介绍一下其中几个比较重要的组件。



**HBase**：来源于Google的BigTable；是一个高可靠性、高性能、面向列、可伸缩的分布式数据库。



**Hive**：是一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析。



**Pig**：是一个基于Hadoop的大规模数据分析工具，它提供的SQL-LIKE语言叫Pig Latin，该语言的编译器会把类SQL的数据分析请求转换为一系列经过优化处理的MapReduce运算。



**ZooKeeper**：来源于Google的Chubby；它主要是用来解决分布式应用中经常遇到的一些数据管理问题，简化分布式应用协调及其管理的难度。



**Ambari**：Hadoop管理工具，可以快捷地监控、部署、管理集群。



**Sqoop**：用于在Hadoop与传统的数据库间进行数据的传递。



**Mahout**：一个可扩展的机器学习和数据挖掘库。



再上一张图，可能看得更直观一点：



![img](/开源框架/hadoop/.assert/hadoop入门/v2-35e976c5f4e7266fe8ec1a46f273a675_1440w.webp)