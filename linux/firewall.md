# 防火墙firewall-cmd

firewalld是一个可以设置网络访问规则的软件，通过firewall-cmd可以管理过滤规则

firewalld配置文件放在/usr/lib/firewalld和/etc/firewalld。firewalld会先在/etc/firewalld，再在/usr/lib/firewalld找

命令操作：http://t.zoukankan.com/liuhaidon-p-11558676.html

firewalld的rich的优先级：https://www.orcy.net.cn/2245.html

## 概念

为了方便配置规则，firewall中有zone区域，service服务等概念

### zone

表示区域，每个网卡可以配置一个zone，zone里面可以配置一些拦截规则和service，网卡将会被应用zone中的规则和service。

| zone                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| 丢弃区域（Drop Zone）     | 如果使用丢弃区域，任何进入的数据包将被丢弃。这个类似与我们之前使用iptables -j drop。使用丢弃规则意味着将不存在响应。 |
| 阻塞区域（Block Zone）    | 阻塞区域会拒绝进入的网络连接，返回icmp-host-prohibited，只有服务器已经建立的连接会被通过即只允许由该系统初始化的网络连接。 |
| 公共区域（Public Zone）   | 只接受那些被选中的连接，默认只允许 ssh 和 dhcpv6-client。这个 zone 是缺省 zone。 |
| 外部区域（External Zone） | 这个区域相当于路由器的启用伪装（masquerading）选项。只有指定的连接会被接受，即ssh，而其它的连接将被丢弃或者不被接受。 |
| 隔离区域（DMZ Zone）      | 如果想要只允许给部分服务能被外部访问，可以在DMZ区域中定义。它也拥有只通过被选中连接的特性，即ssh。 |
| 工作区域（Work Zone）     | 在这个区域，我们只能定义内部网络。比如私有网络通信才被允许，只允许ssh，ipp-client和 dhcpv6-client。 |
| 家庭区域（Home Zone）     | 这个区域专门用于家庭环境。它同样只允许被选中的连接，即ssh，ipp-client，mdns，samba-client和 dhcpv6-client。 |
| 内部区域（Internal Zone） | 这个区域和工作区域（Work Zone）类似，只有通过被选中的连接，和home区域一样。 |
| 信任区域（Trusted Zone）  | 信任区域允许所有网络通信通过。记住：因为trusted是最被信任的，即使没有设置任何的服务，那么也是被允许的，因为trusted是允许所有连接的。 |



```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
</zone>
```





<img src="/linux/.assert/firewall/image-20220523211103348.png" alt="image-20220523211103348" style="zoom:50%;" />





### service

服务，每个里面可以配置协议和端口。zone配置了该服务，表示允许外部使用指定协议访问端口。service的配置文件在/etc/firewalld/service和/usr/lib/firewalld/service



```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>WWW (HTTP)</short>
  <description>HTTP is the protocol used to serve Web pages. If you plan to make your Web server publicly available, enable this option. This option is not required for viewing pages locally or developing Web pages.</description>
  <port protocol="tcp" port="80"/>
</service>
```



## rich规则优先级

rich规格优先级介绍：https://www.orcy.net.cn/2245.html

rich规则默认deny优先级大于allow。可以使用priority调整rich规则的优先级，priority的范围为-32768到32767之间的任何数字，其中数字越小，优先级越高。此范围足够大，以允许从脚本或其他实体自动生成规则



```shell
firewall-cmd --zone=external --add-rich-rule="rule \ 
priority="-100" \
family="ipv4" \ 
source address="192.168.10.102" \ 
service name="ssh" \ 
log prefix=\"ssh connect from:\" \
level="notice" \
accept"
```





## 操作

命令操作：http://t.zoukankan.com/liuhaidon-p-11558676.html

```shell
firewall-cmd --runtime-to-permanent # 将当前配置保存到文件
firewall-cmd --permanent # 表示修改是永久修改，重启后，依然生效
firewall-cmd --reload # 修改配置文件后，将配置生效
```



### 指定网卡所在zone

```shell
# 获取默认的zone，如果网卡没有指定zone，将属于默认的zone
firewall-cmd --get-default-zone 

# 设置默认的zone
firewall-cmd --set-default-zone public

# 将网卡从zone移除
firewall-cmd --zone=public --remove-interface=ens256

# 将网卡加入zone
firewall-cmd --zone=home --add-interface=ens256
```







**默认zone保存的位置**

/etc/firewalld/firewalld.conf

<img src="/linux/.assert/firewall/image-20220710215543338.png" alt="image-20220710215543338" style="zoom:50%;" />

**网卡所属zone的配置文件**是网卡对应的connection的配置文件

如果直接修改connection的配置文件，需要执行`nmcli d reapply <device>`让zone生效

/etc/NetworkManager/system-connections



<img src="/linux/.assert/firewall/image-20220710214539828.png" alt="image-20220710214539828" style="zoom:50%;" />



### 定义rich规则



```shell
#添加规则。其中source和port是可选的，如果不约束可以不写
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.1.1/24" port protocol="tcp" port="3306" accept"

# 直接开放端口
firewall-cmd --zone=home --add-port=8080/tcp --permanent

#如果修改了文件 reload使生效
firewall-cmd --reload

```



```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="mdns"/>
  <service name="dhcpv6-client"/>
  <!-- 生成的规则 -->
  <port port="8080" protocol="tcp"/>
  <rule family="ipv4">
    <source address="192.168.1.1/24"/>
    <port port="3306" protocol="tcp"/>
    <accept/>
  </rule>
  <forward/>
</zone>
```



