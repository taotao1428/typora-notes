# IPV6网络协议



## ipv6类型

![img](/linux/.assert/ipv6网络协议/v2-9243a091cde383f162698772f24ee661_r.jpg)

### 单播地址

**特殊地址前缀** ： ::/128 或者::1/128

链路本地地址前缀： FE80::/10（每张ipv6的网卡都会有一个链路本地地址）用于邻居发现协议和无状态自动配置进程中链路本地上节点之间的通信。使用链路本地地址作为源或目的地址的数据包不会被转发到其他链路上。使用链路本地前缀FE80::/10(1111 1110 10)和IEEE EUI-64格式的接口标识符（EUI-64可来源于EUI-48）可在以太接口对其进行自动配置

**唯一本地地址前缀**： FC00::/7

**站点本地前缀**： FEC0::/10

**兼容地址前缀**： 0:0:0:0:0:0::/96或者0:0:0:0:0:FFFF::/96







### 组播地址

众所周知的组播：FF00::/8（如果不知道目标ip时，使用该组播地址）

被请求节点组播：FF02::1:FF00:/104（如果知道目标ip时，使用该组播地址，NDP协议通过ip查找mac地址就是使用该类ip）



**组播地址格式**

IPv6组播地址格式如图：

![img](/linux/.assert/ipv6网络协议/v2-ebf5437ca2fd00d779b9b58500f15fbe_1440w.png)

**前8bit**为固定的1111 1111。即FF

**flags：**占位4bit，这个字段: 表明组播地址是“永久性”（由IANA分配的一个地址）的，还是“临时性”(用户自己创建的临时多播组)的。一般情况下，高3bit位总是为0，剩下的低1bit位为0时表示“永久性”，为1表示“临时性”。

**scops：**占位4bit，表示组播的范围。

0:预留

1.节点本地范围

2.链路本地范围 例如：**FE02::1**

5.站点本地范围

8.组织本地范围

E:全球范围

F：预留

![img](/linux/.assert/ipv6网络协议/v2-b0cc1bd45bffc4f5b0d102d18580f4ec_1440w.png)

**group**：1表示节点，2表示路由器。





**节点请求地址**

![img](/linux/.assert/ipv6网络协议/v2-fdd157e994a4ac481508a3754a5c33aa_1440w.png)

生成方式为：前104bit为固定的FF02::1:FF，后面24bit是将IPv6地址的最后24bit直接拷贝。

比如，我们在接口处配置IPv6地址（2001::1/64），其对应的被请求节点组播地址为FF02::1::FF00:1。



### 任播地址

（与全球单播地址同空间）

1. 任播地址只能分配给路由器，不能分配给节点





## NDP协议

https://blog.csdn.net/qq_33162707/article/details/124625008

地址解析过程中使用了两种ICMPv6报文：邻居请求报文NS（Neighbor Solicitation）和邻居通告报文NA（Neighbor Advertisement）。

NS报文：Type字段值为135，Code字段值为0，在地址解析中的作用类似于IPv4中的ARP请求报文。
NA报文：Type字段值为136，Code字段值为0，在地址解析中的作用类似于IPv4中的ARP应答报文。



**流程解析：**

① R1要去R2ping包，但是不知道对方地址（即R1想要知道R2的MAC地址），所以R1会发送NS邻居请求报文（源为R1的IPv6地址），目的地址是（R2的被请求节点组播地址）（想请求2001:2设备的MAC地址，即需要解析的目标是R2的IPv6地址），同时需要指出的是在NS报文的Options字段中还携带了一个R1的MAC地址

② 当R2收到了NS报文后，就会回应NA报文，其中源地址为R2的IPv6地址，目的地址是R1的IPv6d地址（使用NS报文中的R1的MAC地址进行单播），R2的MAC地址被放在Options字段中，这样就完成了一个地址解析的过程



### 邻居请求（Neighbor solicition）NS

- Type=135 ,code=0
- Target Address 是需要解析的IPv6地址，因此该处不准出现组播地址。
- Option 中携带了一个自己源的MAC地址



![img](/linux/.assert/ipv6网络协议/55a195ce0928478799055cc76a9cd63e.png)

![img](/linux/.assert/ipv6网络协议/f4e5bbb7bc3e44c39f2a60205fc5d649.png)



### 邻居通告（Neighbor Advertisement）NA

![img](/linux/.assert/ipv6网络协议/85096b70ab654503ba30ece5c3f78be6.png)

1. Type=136，Code=0
2. R标志（Router flag）表示发送者是否为路由器，如果1则表示是；
3. S标志（Solicited flag）表示发送邻居通告是否是响应某个邻居请求，如果1则表示是（0 例如路由器或PC重启后主动发送RA邻居通告，类似免费arp）；
4. O标志（Overide flag）表示邻居通告中的消息是否覆盖已有的条目信息，如果1则表示可以覆盖，如果是0则表示不可覆盖；
5. Target Address表示所携带的链路层地址对应的IPv6地址。
6. Options 携带了自己作为源的MAC地址

![img](/linux/.assert/ipv6网络协议/eb0612ff56c84d9b8cf2cd0e90eb75b9.png)



### 路由发现（地址解析使用的是NS与NA，路由发现使用RA与RS）
路由器发现功能用来发现与本地链路相连的设备，并获取与地址自动配置相关的前缀和其他配置参数。

在IPv6中，IPv6地址可以支持无状态的自动配置，即主机通过机制获取网络前缀信息，然后主机自己生成地址的接口标识部分。路由器发现功能是IPv6地址自动配置功能的基础，主要通过以下两种报文实现：

1. 路由器通告RA（Router Advertisement）报文：每台设备为了让二层网络上的主机和设备知道自己的存在，定时都会组播发送RA报文，RA报文中会带有网络前缀信息，及其他一些标志位信息。RA报文的Type字段值为134。
2. 路由器请求RS（Router Solicitation）报文：很多情况下主机接入网络后希望尽快获取网络前缀进行通信，此时主机可以立刻发送RS报文，网络上的设备将回应RA报文。RS报文的Tpye字段值为133。



### 地址自动配置

IPv4使用DHCP实现自动配置，包括IP地址，缺省网关等信息，简化了网络管理。IPv6地址增长为128位，且终端节点多，对于自动配置的要求更为迫切，除保留了DHCP作为有状态自动配置外，还增加了无状态自动配置。无状态自动配置即自动生成链路本地地址，主机根据RA报文的前缀信息，自动配置全球单播地址等，并获得其他相关信息。

IPv6主机无状态自动配置过程：

1. 根据接口标识产生链路本地地址。
2. 发出邻居请求，进行重复地址检测。
3. 如地址冲突，则停止自动配置，需要手工配置。
4. 如不冲突，链路本地地址生效，节点具备本地链路通信能力。
5. 主机会发送RS报文（或接收到设备定期发送的RA报文）。
6. 根据RA报文中的前缀信息和接口标识得到IPv6地址。



### DAD重复地址检测

当节点自动获取本地链接地址后，会发送一个NS报文（解析的地址为自己的地址），如果收到回复，说明ip出现冲突，此时则需要手动设置。



## 网络报文工具scapy



https://scapy.readthedocs.io/en/latest/index.html



### 构造NA保报文广播更新缓存



```python
from scapy.all import *
from scapy.layers.inet6 import IPv6, ICMPv6ND_NA, ICMPv6NDOptDstLLAddr
from scapy.layers.l2 import Ether


src_if = 'bridge100'
src_mac = 'f6:d4:88:86:69:64'
src_ip = 'fdb2:2c26:f4e4::1'

multicast_ip = 'ff02::1'  # 本地链路所有节点组播地址

a = Ether(src=src_mac)
b = IPv6(src=src_ip, dst=multicast_ip)
c = ICMPv6ND_NA(R=0, S=0, O=1, tgt=src_ip)  # R=0表示非路由发出的包；S=0表示该NA报文不是回复某个NS报文，而是主动发出的；O=1表示需要覆盖原有的缓存记录
d = ICMPv6NDOptDstLLAddr(lladdr=src_mac)

data = a / b / c / d

sendp(data, inter=0.2, count=10, iface=src_if)
```



![image-20230730090023537](/linux/.assert/ipv6网络协议/image-20230730090023537.png)

scapy发送报文

![image-20230730090718590](/linux/.assert/ipv6网络协议/image-20230730090718590.png)

scoket发送报文

![image-20230730090527262](/linux/.assert/ipv6网络协议/image-20230730090527262.png)





scapy报文

![image-20230730090853233](/linux/.assert/ipv6网络协议/image-20230730090853233.png)

![image-20230730091607320](/linux/.assert/ipv6网络协议/image-20230730091607320.png)



socket报文

![image-20230730091104796](/linux/.assert/ipv6网络协议/image-20230730091104796.png)



![image-20230730092112893](/linux/.assert/ipv6网络协议/image-20230730092112893.png)



![image-20230730095318047](/linux/.assert/ipv6网络协议/image-20230730095318047.png)



socket报文

![image-20230730111522346](/linux/.assert/ipv6网络协议/image-20230730111522346.png)

scapy报文

![image-20230730111904916](/linux/.assert/ipv6网络协议/image-20230730111904916.png)



![image-20230730135012752](/linux/.assert/ipv6网络协议/image-20230730135012752.png)



![image-20230730134929455](/linux/.assert/ipv6网络协议/image-20230730134929455.png)





s

[![img](/linux/.assert/ipv6网络协议/format,png.png)](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8zNDgyNTU1LWY5NWMyNDcyYzUyM2ViNGEucG5n?x-oss-process=image/format,png)



https://zhuanlan.zhihu.com/p/110407399?utm_id=0

![img](/linux/.assert/ipv6网络协议/v2-1f07ba7f7c43144a6d6e41947c4c31c4_1440w.jpg)



![img](/linux/.assert/ipv6网络协议/v2-70c3dbea90b7dbe96be618bae89b5a19_1440w.jpg)



### 使用socket和struct构造NA广播报文



```python
import array
import ipaddress
import socket
import struct
from abc import abstractmethod


class Packet:
    def get_type(self) -> int:
        return -1

    @abstractmethod
    def build(self, upper_layer) -> bytes:
        pass


class Ether(Packet):
    def __init__(self, src_mac, dst_mac):
        self.src_mac = src_mac
        self.dst_mac = dst_mac
        self.payload: Packet | None = None

    def set_payload(self, payload: Packet):
        self.payload = payload

    def build(self, upper_layer=None) -> bytes:
        if self.payload is None:
            raise Exception("payload is not set yet")

        src_mac_bytes = bytes.fromhex(self.src_mac.replace(":", ""))
        dst_mac_bytes = bytes.fromhex(self.dst_mac.replace(":", ""))
        proto_type = self.payload.get_type()
        payload_bytes = self.payload.build(self)

        return dst_mac_bytes + src_mac_bytes + struct.pack("!H", proto_type) + payload_bytes


class IPv6(Packet):
    def __init__(self, src_ip, dst_ip, traffic_class=0x00, flow_label=0, hop_limit=255):
        self.src_ip = src_ip
        self.dst_ip = dst_ip
        self.traffic_class = traffic_class
        self.flow_label = flow_label
        self.hop_limit = hop_limit

        self.payload: Packet | None = None

    def get_type(self) -> int:
        return 0x86dd # 在Ether报文中IPv6的类型

    def set_payload(self, payload: Packet):
        self.payload = payload

    def build(self, upper_layer) -> bytes:
        if self.payload is None:
            raise Exception("payload is not set yet")

        version = 6
        first_row = ((version & 0xf) << 28) | ((self.traffic_class & 0xff) << 20) | (self.flow_label & 0xfffff)
        payload_bytes = self.payload.build(self)
        next_header = self.payload.get_type()
        src_ip_bytes = ipaddress.IPv6Address(self.src_ip).packed
        dst_ip_bytes = ipaddress.IPv6Address(self.dst_ip).packed

        return (struct.pack("!IHBB", first_row, len(payload_bytes), next_header, self.hop_limit)
                + src_ip_bytes + dst_ip_bytes + payload_bytes)


class ICMPv6(Packet):
    def __init__(self, icmp_type, code):
        self.icmp_type = icmp_type
        self.code = code

    def get_type(self) -> int:
        return 58  # 在IPv6报文中ICMPv6的类型

    @abstractmethod
    def get_payload(self):
        pass

    def build_packet(self, checksum, data):
        return struct.pack("!BBH", self.icmp_type, self.code, checksum) + data

    def build_pseudo_header(self, src_ip, dst_ip, packet_bytes):
        source_ip_bytes = ipaddress.IPv6Address(src_ip).packed
        dst_ip_bytes = ipaddress.IPv6Address(dst_ip).packed
        payload_len = len(packet_bytes)

        return source_ip_bytes + dst_ip_bytes + struct.pack("!II", payload_len, self.get_type())

    def do_calc_checksum(self, packet: bytes) -> int:
        if len(packet) % 2 == 1:
            packet += b"\0"
        s = sum(array.array("H", packet))
        s = (s >> 16) + (s & 0xffff)
        s += s >> 16
        s = ~s

        s = ((s >> 8) & 0xff) | s << 8

        return s & 0xffff

    def calc_checksum(self, src_ip, dst_ip, data) -> int:
        packet_bytes = self.build_packet(0, data)
        pseudo_header = self.build_pseudo_header(src_ip, dst_ip, packet_bytes)
        return self.do_calc_checksum(pseudo_header + packet_bytes)

    def build(self, upper_layer: IPv6) -> bytes:
        payload = self.get_payload()
        checksum = self.calc_checksum(upper_layer.src_ip, upper_layer.dst_ip, payload)
        return self.build_packet(checksum, payload)


class ICMPv6NDOpt:
    def __init__(self, opt_type, data: bytes):
        self.opt_type = opt_type
        self.data = data
        if len(data) % 8 != 6:
            raise Exception("data length invalid")

    def get_opt_type(self):
        return self.opt_type

    def build(self) -> bytes:
        length = (len(self.data) + 2) // 8
        return struct.pack("!BB", self.opt_type, length) + self.data


class ICMPv6NDOptDstLLAddr(ICMPv6NDOpt):
    def __init__(self, mac):
        data = bytes.fromhex(mac.replace(":", ""))
        super().__init__(0x02, data)


class ICMPv6ND_NA(ICMPv6):
    def __init__(self, tgt, R=0, S=1, O=1):
        super().__init__(136, 0)
        self.tgt = tgt
        self.R = R
        self.S = S
        self.O = O
        self.opts: list[ICMPv6NDOpt] = list()

    def add_opt(self, opt: ICMPv6NDOpt):
        self.opts.append(opt)

    def get_payload(self):
        first_row = ((self.R & 1) << 31) | ((self.S & 1) << 30) | ((self.O & 1) << 29)
        data = struct.pack("!I", first_row) + ipaddress.IPv6Address(self.tgt).packed
        for opt in self.opts:
            data += opt.build()
        return data


def create_socket(iface) -> socket.socket:
    sock = socket.socket(socket.AF_PACKET, socket.SOCK_RAW)
    sock.bind((iface, 0))
    return sock


def main():
    src_if = 'enp0s5'
    src_ip = "fdb2:2c26:f4e4:0000:5c2d:7ef9:8f01:a3b2"
    dst_ip = "ff02::1"
    src_mac = "00:1c:42:f4:57:12"
    dst_mac = "33:33:00:00:00:01"

    # 1. 构建na报文
    dst_lladdr_opt = ICMPv6NDOptDstLLAddr(src_mac)

    na = ICMPv6ND_NA(src_ip, S=0)
    na.add_opt(dst_lladdr_opt)

    # 2. 构建ipv6报文
    ipv6 = IPv6(src_ip, dst_ip)
    ipv6.set_payload(na)

    # 3. 构建ether报文
    ether = Ether(src_mac, dst_mac)
    ether.set_payload(ipv6)

    # 4. 获取ether报文数据
    data = ether.build()

    # 5. 常见socket，ether报文发送
    sock = create_socket(src_if)
    sock.send(data)


if __name__ == '__main__':
    main()

```

