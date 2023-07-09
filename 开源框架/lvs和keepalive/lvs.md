# lvs



## 作用

https://blog.csdn.net/m0_68850571/article/details/123820727



## 安装



```
yum install ipvsadm # 安装lvs
yum install keepalived # 安装keepalived
```



安装的lvs只支持nat，tunnal，dr三种模式，如果要支持fullnat，需要安装内核补丁包



## 使用



### 配置

配置文件路径：/etc/keepalived/keepalived.conf



详细配置介绍

https://segmentfault.com/a/1190000020288076?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io



### fullnat

官方安装指导

http://kb.linuxvirtualserver.org/wiki/IPVS_FULLNAT_and_SYNPROXY



fullnat教程

https://blog.51cto.com/dellinger/2299662



使用阿里开源的fullnat包

https://ieevee.com/tech/2015/12/09/fullnat-2.html



### 实验

#### 环境准备

**网络**

外网：172.16.1.0/24

内网：172.16.2.0/24



**虚拟机**

虚拟机1：ens161:172.16.1.11/24，ens256:172.16.2.11/24

虚拟机2：ens161:172.16.1.12/24，ens256:172.16.2.12/24

虚拟机3：ens256:172.16.2.13/24

虚拟机4：ens256:172.16.2.14/24



#### lvs nat测试



虚拟机3

```shell
# 安装nginx
yum install nginx
# 启动nginx
systemctl start nginx
# 修改index.html内容为：<h1>Hello Server3</h1>
vim /usr/share/nginx/html/index.html

# 测试http服务正常
curl http://172.16.2.13

# 配置172.16.1.0/24转发到虚拟机1 172.16.2.11
nmcli dev modify ens256 +ipv4.routes "172.16.1.0/24  172.16.2.11 100" 
```



虚拟机4

```shell
# 安装nginx
yum install nginx
# 启动nginx
systemctl start nginx
# 修改index.html内容为：<h1>Hello Server4</h1>
vim /usr/share/nginx/html/index.html

# 测试http服务正常
curl http://172.16.2.14

# 配置172.16.1.0/24转发到虚拟机1 172.16.2.11
nmcli dev modify ens256 +ipv4.routes "172.16.1.0/24  172.16.2.11 100" 
```



虚拟机1

```shell
# 开启网络转发
echo 1 >/proc/sys/net/ipv4/ip_forward 
# 配置转发
ipvsadm -A -t 172.16.1.11:80 -s rr
ipvsadm -a -t 172.16.1.11:80 -r 172.16.2.13:80 -m
ipvsadm -a -t 172.16.1.11:80 -r 172.16.2.14:80 -m
```



虚拟机2

```shell
# 测试转发效果
curl http://172/16.1.11
```



```
[root@fedora ~]# curl http://172.16.1.11
<h1>Hello Server3</h1>
[root@fedora ~]# curl http://172.16.1.11
<h1>Hello Server4</h1>
[root@fedora ~]# curl http://172.16.1.11
<h1>Hello Server3</h1>
[root@fedora ~]# curl http://172.16.1.11
<h1>Hello Server4</h1>
```



#### keepalive测试

lvs测试需要使用两台lvs，并且需要1个内网虚拟ip和一个外网虚拟ip



外网虚拟ip：172.16.1.100

内网虚拟ip：172.16.2.100



<img src="/开源框架/lvs和keepalive/.assert/lvs/image-20220710161349991.png" alt="image-20220710161349991" style="zoom:50%;" />



**虚拟机3和虚拟机4**，需要把172.16.1.0/24路由到内网虚拟ip172.16.2.100



```shell
nmcli dev modify ens256 -ipv4.routes "172.16.1.0/24  172.16.2.11 100" 
nmcli dev modify ens256 +ipv4.routes "172.16.1.0/24  172.16.2.100 100" 
```



**虚拟机1**

```shell
# 删除刚才的配置
ipvsadm -D -t 172.16.1.11:80

# 配置keepalived
vim /etc/keepalived/keepalived.conf

# 启动keepalived
systemctl start keepalived
```



```keepalived
global_defs {
}

# 配置外网虚拟ip
vrrp_instance VI_1 {
		# 主节点为MASTER
    state MASTER
    interface ens161
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        # 配置虚拟ip
        172.16.1.100
    }
}

# 配置内网虚拟ip
vrrp_instance VI_2 {
    state MASTER
    interface ens256
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.16.2.100
    }
}

# 配置虚拟主机，这里会调用ipvsadm配置虚拟主机
virtual_server 172.16.1.100 80 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    # persistence_timeout 50
    protocol TCP

    real_server 172.16.2.13 80 {
        weight 10
        TCP_CHECK {
            connect_timeout 3
            retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
    real_server 172.16.2.14 80 {
        weight 10
        TCP_CHECK {
            connect_timeout 3
            retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
}

```

主节点上挂载了虚拟ip，并且配置了lvs

<img src="/开源框架/lvs和keepalive/.assert/lvs/image-20220710153834249.png" alt="image-20220710153834249" style="zoom:50%;" />



**虚拟机2**



```shell
# 配置keepalived。配置与虚拟机1基本相同，除了state需要从MASTER改成BACKUP
vim /etc/keepalived/keepalived.conf

# 启动keepalived
systemctl start keepalived
```

备节点上没有配置虚拟ip和lvs

<img src="/开源框架/lvs和keepalive/.assert/lvs/image-20220710154004360.png" alt="image-20220710154004360" style="zoom:50%;" />







```shell
# 查看配置是否有问题
keepalived -t
```



**业务节点故障切换**

将虚拟机3的web服务停止，会发现172.16.2.13从lvs中剔除了

<img src="/开源框架/lvs和keepalive/.assert/lvs/image-20220710154531253.png" alt="image-20220710154531253" style="zoom:50%;" />

重新启动虚拟机3的web服务后，又会被添加进去

<img src="/开源框架/lvs和keepalive/.assert/lvs/image-20220710154659199.png" alt="image-20220710154659199" style="zoom:50%;" />



**lvs节点故障切换**

将虚拟机1的keepalived服务停止，会发现虚拟ip和lvs在虚拟机2中出现



<img src="/开源框架/lvs和keepalive/.assert/lvs/image-20220710155036917.png" alt="image-20220710155036917" style="zoom:50%;" />









虚拟机3和虚拟机4的172.16.1.0/24地址路由到





echo 1 >/proc/sys/net/ipv4/ip_forward





```
# ip address add 10.30.20.253/32 dev eth0
# ipvsadm -A -t 10.30.20.253:8181 -s rr
# ipvsadm -a -t 10.30.20.253:8181 -r 10.30.8.64:80 -m
# ipvsadm -a -t 10.30.20.253:8181 -r 10.30.12.71:80 -m
# ipvsadm -L -n


ip address add 10.30.20.253/32 dev eth0
ipvsadm -A -t 192.168.3.75:80 -s rr
ipvsadm -a -t 192.168.3.75:80 -r 192.168.3.73:80 -m
ipvsadm -a -t 192.168.3.75:80 -r 192.168.3.74:80 -m
ipvsadm -L -n

nmcli c add type ethernet con-name ens256 ifname ens256 ipv4.addr 172.16.1.13/24 ipv4.gateway 172.16.1.1 ipv4.method manual

nmcli dev modify ens256 +ipv4.routes "172.16.2.0/24  172.16.1.1 100" 
```



在两个不同的子网下，lvs节点是后端节点的子网的网关



外网：172.16.1.0/24

内网：172.16.2.0/24

外网虚拟ip：172.16.1.100

内网虚拟ip：172.16.2.100



**lvs节点**

1节点

172.16.1.11/24

172.16.2.11/24

```
nmcli c add type ethernet con-name ens161 ifname ens161 ipv4.addr 172.16.1.11/24 ipv4.method manual

nmcli c add type ethernet con-name ens256 ifname ens256 ipv4.addr 172.16.2.11/24 ipv4.method manual


ip address add 172.16.1.100/32 dev 
ipvsadm -A -t 172.16.1.11:80 -s rr
ipvsadm -a -t 172.16.1.11:80 -r 172.16.2.13:80 -m
ipvsadm -a -t 172.16.1.11:80 -r 172.16.2.14:80 -m
ipvsadm -L -n
```





2节点

172.16.1.12/24 

172.16.2.12/24 

```
nmcli c add type ethernet con-name ens161 ifname ens161 ipv4.addr 172.16.1.12/24 ipv4.method manual

nmcli c add type ethernet con-name ens256 ifname ens256 ipv4.addr 172.16.2.12/24 ipv4.method manual
```





**后端节点**

3节点

172.16.2.13/24



```
nmcli dev modify ens256 +ipv4.routes "172.16.1.0/24  172.16.2.11 100" 

nmcli dev modify ens256 -ipv4.routes "172.16.2.0/24  172.16.1.1 100" 
```





4节点

172.16.2.14/24



```
nmcli dev modify ens256 -ipv4.routes "172.16.1.0/24  172.16.2.1 100" 

nmcli dev modify ens256 -ipv4.routes "172.16.1.0/24  172.16.2.11 100" 
```













机器1

路由器

2张nat网卡

内网网卡1：172.16.1.1/24

内网网卡2：172.16.2.1/24



机器2

nat网卡



机器3

nat网卡

内网网卡1：172.16.1.13/24



机器4

nat网卡 

内网网卡1：172.16.2.14/24





内网网段1：172.16.1.0/24

内网网段2：172.16.2.0/24



192.128 + 32 + 8





https://blog.csdn.net/javaldk/article/details/122590998

防火墙配置

route持久化