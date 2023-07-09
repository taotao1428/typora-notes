# linux进程资源控制-cgroup

Linux CGroup全称Linux Control Group， 是Linux内核的一个功能，用来限制，控制与分离一个进程组群的资源（如CPU、内存、磁盘输入输出等）。防止进程间不利的资源抢占。

​    一般在较大的公司中会经常用到，例如多个数据库跑在一个物理机上，一般按照需要对不同的数据库进行资源的分配和限制，避免单个库的消耗增加影响其他数据库的性能。



## 概念

**控制族群(cgroup)** - 关联一组task和一组subsystem的配置参数。一个task对应一个进程, cgroup是资源分片的最小单位。
**子系统(subsystem)** - 资源管理器，一个subsystem对应一项资源的管理，如 cpu, cpuset, memory等
**层级(hierarchy)** - 关联一个到多个subsystem和一组树形结构的cgroup. 和cgroup不同，hierarchy包含的是可管理的subsystem而非具体参数
**任务(task)**- 每个cgroup都会有一个task列表文件tasks，一个task就对应一个进程。



相互关系：

> 1）.每次在系统中创建新层级时，该系统中的所有任务都是那个层级的默认 cgroup（我们称之为 root cgroup ，此cgroup在创建层级时自动创建，后面在该层级中创建的cgroup都是此cgroup的后代）的初始成员。
> 2）.一个子系统最多只能附加到一个层级。
> 3）.一个层级可以附加多个子系统
> 4）.一个任务可以是多个cgroup的成员，但是这些cgroup必须在不同的层级。
> 5）.系统中的进程（任务）创建子进程（任务）时，该子任务自动成为其父进程所在 cgroup 的成员。然后可根据需要将该子任务移动到不同的 cgroup 中，但开始时它总是继承其父任务的cgroup。



![img](/linux/.assert/cgroup/2021100622534598.png)



## **1.1、subsystem**

centos支持的subsystem如下

1）blkio 这个子系统为块设备设定输入/输出限制，比如物理设备（磁盘，固态硬盘，USB 等等）。
2）cpu 这个子系统使用调度程序提供对 CPU 的 cgroup 任务访问。
3）cpuacct 这个子系统自动生成 cgroup 中任务所使用的 CPU 报告。
4）cpuset 这个子系统为 cgroup 中的任务分配独立 CPU（在多核系统）和内存节点。
5）devices 这个子系统可允许或者拒绝 cgroup 中的任务访问设备。
6）freezer 这个子系统挂起或者恢复 cgroup 中的任务。
7）memory 这个子系统设定 cgroup 中任务使用的内存限制，并自动生成由那些任务使用的内存资源报告。
8）net_cls 这个子系统使用等级识别符（classid）标记网络数据包，可允许 Linux 流量控制程序（tc）识别从具体 cgroup 中生成的数据包。
9）ns 名称空间子系统。



##  1.2、**hierarchy** 



​    `hierarchy`包含的是可管理的`subsystem`

​    创建`hierarchy`

        mount -t cgroup -o cpuset cgroup_t  /cg_t1（关联cpuset 这个subsystem）
        mount -t cgroup -o memory cgroup_t  /cg_t2（关联memory 这个subsystem）


使用`mount -t cgroup`可以查看当前cgroup挂载的路径

<img src="/linux/.assert/cgroup/image-20230101184523108.png" alt="image-20230101184523108" style="zoom:50%;" />



每个hierarchy会有以下文件

<img src="/linux/.assert/cgroup/image-20230101184753906.png" alt="image-20230101184753906" style="zoom:50%;" />

## **1.3、cgroup**

cgroup管理的是具体的subsystem的参数

如果需要创建cgroup，在hierarchy的根目录下或者其他cgroup目录下直接创建目录就可以

创建出来的目录会自动集成父目录的subsystem，自动创建相关文件

例如，我在如`/sys/fs/cgroup/cpu_cg/`下面创建`sub_cpu_cg`下会自动生成下面文件



<img src="/linux/.assert/cgroup/image-20230101211300040.png" alt="image-20230101211300040" style="zoom:50%;" />

<img src="/linux/.assert/cgroup/image-20230101211324880.png" alt="image-20230101211324880" style="zoom:50%;" />



**准则**

​    1.一个hierarchy可以有多个 subsystem (mount 的时候hierarchy可以attach多个subsystem)

​    2.一个已经被挂载的 subsystem 只能被再次挂载在一个空的 hierarchy 上 (已经mount一个subsystem的hierarchy不能挂载一个已经被其它hierarchy挂载的subsystem)

​    3.每个task只能在一同个hierarchy的唯一一个cgroup里(不能在同一个hierarchy下有超过一个cgroup的tasks里同时有这个进程的pid)

​    4.子进程在被fork出时自动继承父进程所在cgroup，但是fork之后就可以按需调整到其他cgroup



## 不同资源

https://www.cnblogs.com/zhrx/p/16388175.html

### cpu资源

<img src="/linux/.assert/cgroup/image-20230101215343723.png" alt="image-20230101215343723" style="zoom:50%;" />

1. cpu.cfs_burst_us：



## 资源限制

1. 配置cgroup时，下级的cgroup资源不能大于父级cgroup资源
2. 当前cgroup的进程与下级cgroup的进程的资源不能大于当前cgroup配置的资源



## 实验



创建busy.sh 里面是一个死循环

```shell
a=1
while (( 1 ))
do
	let a++
done
```

执行脚本，进程的cpu为99.7%

```shell
nohup sh busy.sh &
```

<img src="/linux/.assert/cgroup/image-20230101213243925.png" alt="image-20230101213243925" style="zoom:50%;" />

创建cpu的hierarchy

```shell
cd /sys/fs/cgroup/
mkdir cpu_cg
mount -t cgroup -o cpu cpu_cg /sys/fs/cgroup/cpu_cg/
```



创建子cgroup sub_sub_cpu_cg

```
cd /sys/fs/cgroup/cpu_cg/
mkdir sub_sub_cpu_cg
```



指定子cgroup的cpu只有20%

```
cd /sys/fs/cgroup/cpu_cg/sub_sub_cpu_cg
echo 20000 > cpu.cfs_quota_us
```



将busy.sh进程加入子cgroup。进程占用cpu变成19.9%

```
echo 85394 > /sys/fs/cgroup/cpu_cg/sub_cpu_cg/tasks
```

<img src="/linux/.assert/cgroup/image-20230101213425960.png" alt="image-20230101213425960" style="zoom:50%;" />















/sys/fs/cgroup/cpu_cg/sub_cpu_cg/sub_sub_cpu_cg/tasks



mount -t cgroup -o cpuset cgroup_t /cg_t1

mount -t cgroup

umount -t cgroup /cg_t1



mount -t cgroup -o cpu cgroup_t /cg_t1





/sys/fs/cgroup/cpu_cg/sub_cpu_cg/

echo /sys/fs/cgroup/cpu_cg/sub_cpu_cg/tasks







<img src="/linux/.assert/cgroup/image-20230101184024892.png" alt="image-20230101184024892" style="zoom:50%;" />



将刚才的busy.sh进程kill掉

<img src="/linux/.assert/cgroup/image-20230101214223356.png" alt="image-20230101214223356" style="zoom:50%;" />

在sub_cpu_cg下再创建一个子cgroup sub_sub_cpu_cg

```shell
cd /sys/fs/cgroup/cpu_cg/sub_cpu_cg/
mkdir sub_sub_cpu_cg
```

<img src="/linux/.assert/cgroup/image-20230101214358324.png" alt="image-20230101214358324" style="zoom:50%;" />



创建两个busy.sh的进程

<img src="/linux/.assert/cgroup/image-20230101214733938.png" alt="image-20230101214733938" style="zoom:50%;" />



并将两个busy.sh进程加入cgroup sub_sub_cpu_cg。进程占用的资源分别变成了10%，跟sub_cpu_cg配置的一样

<img src="/linux/.assert/cgroup/image-20230101211009806.png" alt="image-20230101211009806" style="zoom:50%;" />

<img src="/linux/.assert/cgroup/image-20230101210949546.png" alt="image-20230101210949546" style="zoom:50%;" />

<img src="/linux/.assert/cgroup/image-20230101210733221.png" alt="image-20230101210733221" style="zoom:50%;" />



将sub_sub_cpu_cg的cpu资源配置为10%。进程占用的资源分别变成5%

```shell
echo 10000 > /sys/fs/cgroup/cpu_cg/sub_cpu_cg/sub_sub_cpu_cg/cpu.cfs_quota_us
```

<img src="/linux/.assert/cgroup/image-20230101210923333.png" alt="image-20230101210923333" style="zoom:50%;" />

<img src="/linux/.assert/cgroup/image-20230101210900104.png" alt="image-20230101210900104" style="zoom:50%;" />



sub_sub_cpu_cg配置的cpu资源不能大于sub_cpu_cg

<img src="/linux/.assert/cgroup/image-20230101211137241.png" alt="image-20230101211137241" style="zoom:50%;" />



