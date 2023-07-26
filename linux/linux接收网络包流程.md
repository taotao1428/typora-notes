# linux接收网络包



1. 向内核注册网卡驱动
2. 网卡接收数据后，将数据拷贝到RingBuffer，然后发送中断请求，内核调用驱动程序处理数据。



一个网卡的数据可以被多个cpu处理





## iptables

参考：https://mp.weixin.qq.com/s/Dgv5BU9YU0tuSMxtzcuiVw

<img src="/linux/.assert/linux接收网络包流程/640-20230723182112969.png" alt="图片" style="zoom: 67%;" />



iptables中总共有4张表还有5条链，我们可以在链上加不同的规则。

五张表：filter表、nat表、mangle表、raw表、security表

五条链：prerouting、input、output、forward、postrouting



```
iptables -t ${表名} -nL
```



<img src="/linux/.assert/linux接收网络包流程/image-20230723194922276.png" alt="image-20230723194922276" style="zoom:50%;" />



### iptables的匹配规则

**什么是匹配规则？**

这个东西并不难理解，他其实就是一种描述。比如说：我想丢弃来自A的流量，体现在iptables语法上就是：`iptables -t filter -A INPUT -s ${A的地址} -j DROP` ，这段话中的`来自A`其实就是匹配规则。

**常见的规则如下：**

源地址：`-s 192.168.1.0/24`

目标地址：`-d 192.168.1.11`

协议：`-p tcp|udp|icmp`

从哪个网卡进来：`-i eth0|lo`

从哪个网卡出去：`-o eth0|lo`

目标端口（必须制定协议）：`-p tcp|udp --dport 8080`

源端口（必须制定协议）：`-p tcp|udp --sport 8080`



```
iptables -t filter -A INPUT -s 10.4.4.11 -j DROP
iptables -t filter -A INPUT -s 10.4.4.12 -j ACCEPT
iptables -t filter -A INPUT -s 10.4.4.11 -j ACCEPT
```



### 文档介绍

```
man iptables
man iptables-extensions
```



### 参数

















## 中断

PCIe设置支持三种中断，INTx中断，MSI中断，MSI-X中断。



MSI中断介绍：https://www.codenong.com/cs106676560/



## 命令



### 查看网卡的pci设备id

`ls -l /sys/class/net/`



<img src="/linux/.assert/linux接收网络包流程/image-20230723171603601.png" alt="image-20230723171603601" style="zoom:50%;" />



### 查看所有pci设备

`lspci`

<img src="/linux/.assert/linux接收网络包流程/image-20230723171724304.png" alt="image-20230723171724304" style="zoom:50%;" />



### 查看所有pci的id

`ll /sys/bus/pci/devices/`

<img src="/linux/.assert/linux接收网络包流程/image-20230723171949738.png" alt="image-20230723171949738" style="zoom:50%;" />



### 查看pci设备的中断

#### int中断

`cat /sys/bus/pci/devices/<pci id>/irq`

<img src="/linux/.assert/linux接收网络包流程/image-20230723173147554.png" alt="image-20230723173147554" style="zoom:50%;" />

#### msi中断

`ll /sys/bus/pci/devices/<pci id>/msi_irqs`

<img src="/linux/.assert/linux接收网络包流程/image-20230723172114191.png" alt="image-20230723172114191" style="zoom:50%;" />

### 查看中断

`cat proc/interrupts`

<img src="/linux/.assert/linux接收网络包流程/image-20230723164904474.png" alt="image-20230723164904474" style="zoom:50%;" />



### 查看中断详细信息

`cd /proc/irq`

每个文件夹表示一个中断

<img src="/linux/.assert/linux接收网络包流程/image-20230723174449349.png" alt="image-20230723174449349" style="zoom:50%;" />

每个文件夹中有下面文件

- **smp_affinity**       irq和cpu之间的亲缘绑定关系；修改smp_affinity文件可以调整中断处理的cpu
- smp_affinity_hint  只读条目，用于用户空间做irq平衡只用；
- spurious          可以获得该irq被处理和未被处理的次数的统计信息；
- handler_name    驱动程序注册该irq时传入的处理程序的名字；

<img src="/linux/.assert/linux接收网络包流程/image-20230723174548432.png" alt="image-20230723174548432" style="zoom:50%;" />



