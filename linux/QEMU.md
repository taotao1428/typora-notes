# QEMU

QEMU提供虚拟机运行的环境。



![img](/linux/.assert/QEMU/2413091-20221130114342930-1977725737.png)



## cpu虚拟化

https://zhuanlan.zhihu.com/p/521162423

访问硬件的权限，ring0到ring3

1. ring0：权限最高，可以直接访问任何硬件。操作系统运行在该模式下
2. Ring3：权限最小，无法直接访问硬件。应用程序运行在该模式下



HostOS运行在ring0，但是GuestOS运行在ring1，但是它并不知道自己运行在ring1，所以在执行特权命令时，会抛出权限错误。有3种解决方法

1. 全虚拟化/`Full-Virtualization`
2. 半虚拟化/`Para-Virtualization`
3. 硬件辅助的全虚拟化



### **基于二进制翻译的全虚拟化**

客户操作系统运行在`ring1`，它在执行特权指令时，会触发CPU异常(`Exception`)，然后`VMM`捕获这个异常，在异常里面做翻译，模拟，最后返回到客户操作系统内，客户操作系统认为自己的特权指令工作正常，继续运行。但是这个性能损耗非常的大，简单的一条指令，本来执行完结束，现在却要通过复杂的异常处理过程。



![img](/linux/.assert/QEMU/v2-322989d28f12497976ffbbfad46692b3_1440w.jpg)

`Guest OS` => `Hardware Exception` => `VMM exception handler function(handle+emulate privileged ins)` => `Guest OS`：



### 半虚拟化

半虚拟化的思想是：修改操作系统内核，替换掉不能虚拟化的指令，通过超级调用(`hypercall`)直接和底层的虚拟化层`Hypervisor`来通讯，`Hypervisor`同时也提供了超级调用接口来满足其他关键内核操作，比如内存管理、中断和时间保持。

这种做法省去了全虚拟化中的捕获和模拟，大大提高了效率。所以像`Xen`这种半虚拟化技术，客户机操作系统都是有一个专门的定制内核版本，和`x86`、`mips`、`arm`这些内核版本等价。这样以来，就不会有捕获异常、翻译、模拟的过程了，性能损耗非常低。这就是`Xen`这种半虚拟化架构的优势。这也是为什么`Xen`只支持虚拟化`Linux`，无法虚拟化`windows`原因，因为微软不改代码。

![img](/linux/.assert/QEMU/v2-804b86c96879573a8ba4505cf56fb13c_1440w.jpeg)



### 硬件辅助的全虚拟化

2005年后，`CPU`厂商`Intel`和`AMD`开始支持虚拟化了。`Intel`引入了`Intel-VT(Virtualization Technology)`技术。 这种`CPU`，有`VMX root operation`和`VMX non-root operation`两种模式，两种模式都支持`ring0~ring3`共4个运行级别。这样，`VMM`可以运行在`VMX root operation`模式下，`Guest OS`运行在`VMX non-root operation`模式下。

![img](/linux/.assert/QEMU/v2-4152826feba5b0056c24b34985ef5546_1440w.jpeg)



## VirtIO

https://blog.csdn.net/qq_16054639/article/details/117067397

virtIO用于实现虚拟机的IO设备，通过零拷贝的特性提升IO性能。通过内存共享实现qemu和guest可以相互访问数据。

![在这里插入图片描述](/linux/.assert/QEMU/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2MDU0NjM5,size_16,color_FFFFFF,t_70.png)

![img](/linux/.assert/QEMU/v2-19ae8fb7f9a603a189b02f57a09d05fd_1440w.jpg)





## dpdk

dpdk（Data Plane Development Kit，数据平面开发套件）是一套开源的高性能开发套件。可以在用户态处理网络数据，提高网络数据转发数据。

dpdk与普通内核处理网络数据的路径对比

![img](/linux/.assert/QEMU/962bd40735fae6cd4aace6084871fc2e43a70f8f.png)



## 卸载态

https://forum.huawei.com/enterprise/zh/thread/580935244993937408

前面提到`内核态`,`用户态`是操作系统的概，`卸载态`是网卡概念，网卡offload机制。当网络速度超过1Gb的时候，这些计算会耗费大量的CPU时间，有数据表明，即便使用千兆全双工网卡，TCP通信也将消耗CPU的80%的使用率（以2.4GHz奔腾4处理器为例），这样留给其他应用程序的时间就很少了，表现出来就是用户可能感觉到很慢。

 为了解决性能问题，就产生了**TOE**技术（TCP offload engine），**将TCP连接过程中的相关计算工作转移到专用硬件上**（比如网卡），从而释放CPU资源。从2012年开始，这项技术开始在普通用户的网卡上应用。

 随着技术的日趋成熟，目前越来越多的网卡设备开始支持offload特性，以便提升网络收发和处理的性能。本文所描述的offload特性，主要是指将原本 在协议栈中进行的IP分片、TCP分段、重组、checksum校验等操作，转移到网卡硬件中进行，降低系统CPU的消耗，提高处理性能。



## 参考：

qemu-kvm创建运行虚拟机代码分析： https://zhuanlan.zhihu.com/p/567198188