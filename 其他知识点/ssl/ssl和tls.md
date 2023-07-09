# ssl和tls





## 加密套件

https://blog.csdn.net/weixin_43408952/article/details/124715927



![TLS密码套件格式](/其他知识点/ssl/.assert/ssl和tls/20200514155024398.png)



```
    Cipher Suite: TLS_CHACHA20_POLY1305_SHA256 (0x1303)
    Cipher Suite: TLS_AES_256_GCM_SHA384 (0x1302)
    Cipher Suite: TLS_AES_128_GCM_SHA256 (0x1301)
    Cipher Suite: TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca9)
    Cipher Suite: TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xcca8)
    Cipher Suite: TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (0xccaa)
    Cipher Suite: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)
    Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 (0xc02c)
    Cipher Suite: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 (0xc028)
    Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384 (0xc024)
    Cipher Suite: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (0xc014)
    Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA (0xc00a)
    Cipher Suite: TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 (0x009f)
    Cipher Suite: TLS_DHE_RSA_WITH_AES_256_CBC_SHA256 (0x006b)
    Cipher Suite: TLS_DHE_RSA_WITH_AES_256_CBC_SHA (0x0039)
    Cipher Suite: Unknown (0xff85)
    Cipher Suite: TLS_DHE_RSA_WITH_CAMELLIA_256_CBC_SHA256 (0x00c4)
    Cipher Suite: TLS_DHE_RSA_WITH_CAMELLIA_256_CBC_SHA (0x0088)
    Cipher Suite: TLS_GOSTR341001_WITH_28147_CNT_IMIT (0x0081)
    Cipher Suite: TLS_RSA_WITH_AES_256_GCM_SHA384 (0x009d)
    Cipher Suite: TLS_RSA_WITH_AES_256_CBC_SHA256 (0x003d)
    Cipher Suite: TLS_RSA_WITH_AES_256_CBC_SHA (0x0035)
    Cipher Suite: TLS_RSA_WITH_CAMELLIA_256_CBC_SHA256 (0x00c0)
    Cipher Suite: TLS_RSA_WITH_CAMELLIA_256_CBC_SHA (0x0084)
    Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)
    Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 (0xc02b)
    Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 (0xc027)
    Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256 (0xc023)
    Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (0xc013)
    Cipher Suite: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA (0xc009)
    Cipher Suite: TLS_DHE_RSA_WITH_AES_128_GCM_SHA256 (0x009e)
    Cipher Suite: TLS_DHE_RSA_WITH_AES_128_CBC_SHA256 (0x0067)
    Cipher Suite: TLS_DHE_RSA_WITH_AES_128_CBC_SHA (0x0033)
    Cipher Suite: TLS_DHE_RSA_WITH_CAMELLIA_128_CBC_SHA256 (0x00be)
    Cipher Suite: TLS_DHE_RSA_WITH_CAMELLIA_128_CBC_SHA (0x0045)
    Cipher Suite: TLS_RSA_WITH_AES_128_GCM_SHA256 (0x009c)
    Cipher Suite: TLS_RSA_WITH_AES_128_CBC_SHA256 (0x003c)
    Cipher Suite: TLS_RSA_WITH_AES_128_CBC_SHA (0x002f)
    Cipher Suite: TLS_RSA_WITH_CAMELLIA_128_CBC_SHA256 (0x00ba)
    Cipher Suite: TLS_RSA_WITH_CAMELLIA_128_CBC_SHA (0x0041)
    Cipher Suite: TLS_ECDHE_RSA_WITH_RC4_128_SHA (0xc011)
    Cipher Suite: TLS_ECDHE_ECDSA_WITH_RC4_128_SHA (0xc007)
    Cipher Suite: TLS_RSA_WITH_RC4_128_SHA (0x0005)
    Cipher Suite: TLS_RSA_WITH_RC4_128_MD5 (0x0004)
    Cipher Suite: TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA (0xc012)
    Cipher Suite: TLS_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA (0xc008)
    Cipher Suite: TLS_DHE_RSA_WITH_3DES_EDE_CBC_SHA (0x0016)
    Cipher Suite: TLS_RSA_WITH_3DES_EDE_CBC_SHA (0x000a)
    Cipher Suite: TLS_EMPTY_RENEGOTIATION_INFO_SCSV (0x00ff)

```





## 证书

https://blog.csdn.net/zengqiang1/article/details/105538692/

所谓数字证书，是一种用于电脑的身份识别机制。由数字证书颁发机构（CA）对使用私钥创建的签名请求文件做的签名（盖章），表示CA结构对证书持有者的认可。数字证书拥有以下几个优点：

使用数字证书能够提高用户的可信度
数字证书中的公钥，能够与服务端的私钥配对使用，实现数据传输过程中的加密和解密
在证认使用者身份期间，使用者的敏感个人数据并不会被传输至证书持有者的网络系统上
X.509证书包含三个文件：key，csr，crt。



key是服务器上的私钥文件，用于对发送给客户端数据的加密，以及对从客户端接收到数据的解密
csr是证书签名请求文件，用于提交给证书颁发机构（CA）对证书签名
crt是由证书颁发机构（CA）签名后的证书，或者是开发者自签名的证书，包含证书持有人的信息，持有人的公钥，以及签署者的签名等信息
备注：在密码学中，X.509是一个标准，规范了公开秘钥认证、证书吊销列表、授权凭证、凭证路径验证算法等。





### Openssl实践

可以在命令行直接输入密码 https://blog.csdn.net/weixin_44129085/article/details/123023459

#### 生成私钥key

使用openssl生成私钥，说明：生成rsa私钥，des3算法，2048位强度，server.key是秘钥文件名。

```
openssl genrsa -des3 -out server.key 2048
```



<img src="/其他知识点/ssl/.assert/ssl和tsl/image-20220723143054692.png" alt="image-20220723143054692" style="zoom:50%;" />

#### 生成证书请求文件csr

使用私钥创建证书请求文件

```
openssl req -new -key server.key -out server.csr
```



#### 生成自签名的证书

```shell
# 生成证书
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

# 查看证书内容
openssl x509 -in server.crt -noout -text
```



#### 使用CA签名证书



```shell
# 生成ca私钥
openssl genrsa -aes256 -passout pass:1234 -out ca.key 2048

# 生成ca证书
openssl req -new -x509 -days 36500 -key ca.key  -out ca.crt -subj "/C=CN/ST=GuangDong/L=ShenZhen/O=Huawei/OU=MyOnlyTest CA/CN=Only Test" -passin pass:1234

# 使用ca私钥和证书对生成证书
openssl x509 -req -days 36500 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server_sign_by_ca.crt -passin pass:1234
```



#### 验证证书是否有效

```shell
openssl verify -CAfile ca.crt server_sign_by_ca.crt
```

<img src="/其他知识点/ssl/.assert/ssl和tsl/image-20220723151746362.png" alt="image-20220723151746362" style="zoom:50%;" />



#### 生成证书链

通常CA是由多级的，然而通常客户端内只会保存少部分根CA，因此为了保证证书可以被校验，通常需要将证书和其上层的CA证书合并成一个证书链。



```shell
# 合并成证书链，ca证书在前
cat tls-intermca.pem tls-cert.pem > tls-bundle.pem
```



#### 秘钥格式（PKCS1和PKCS8）（数据内部组成格式）

openssl命令默认生成的私钥格式为

**PKCS1：**全名《Public-Key Cryptography Standards (PKCS) #1: RSA Cryptography Specifications》最新版本2.2  *(rfc8017, 有兴趣的同学可以读一下)* ，从名称上可以看出它是针对RSA算法的一个规范。里面包含了RSA加密、解密、签名验签等所有的内容，当然也包含了私钥的格式。PKCS1的1.1版本是1991年发布的。
 **PKCS8：**全名《Public-Key Cryptography Standards (PKCS) #8: Private-Key Information Syntax Specification》最新版本1.2，从名称上可以看出它是一个专门用来存储私钥的文件格式规范。PKCS1的1.2版本是2008年发布的。
 刚好它们两个有重合的部分，都定义了私钥的存储，那他们到底有什么关系呢？我们下面实际验证一下



openssl默认生成rsa私钥格式为PKCS1格式，而netty目前仅支持PKCS8格式的私钥



```shell
# pkcs1转为pkcs8
openssl pkcs8 -topk8 -inform PEM -in pkcs1.pem -outform PEM -nocrypt -out pkcs8.pem

# pkcs8转为pkcs1
openssl rsa  -in demo_pkcs8.pem  -out demo_pkcs1.pem
```





#### 私钥和证书的格式（PEM和DER）（数据对外呈现格式）

https://blog.csdn.net/fengshenyun/article/details/124596279



PEM 与 DER是用于存储、传输密钥和证书的标准格式，两者紧密关联，可以互相转换，下面详细介绍两种格式:

DER：Distinguished Encoding Rules，可分辩编码规则。DER格式文件后缀通常为 “.der” 和 “.cer”，后缀名并不会影响 DER 格式文件的解析。

PEM：Privacy-Enhanced Mail，隐私增强邮件。PEM格式文件后缀通常为".pem"、“.cer”、“.crt”、“.key”，后缀名并不会影响 PEM 格式文件的解析。



DER格式里面的数据是二进制，而PEM是base64编码，并且加上了特定的头部和尾部。下面是PEM格式的私钥文件

```
-----BEGIN RSA PRIVATE KEY-----
base64_decode(DER二进制)
-----END RSA PRIVATE KEY-----
```



```shell
# pem转der
openssl rsa -in rsa_private.pem -outform DER -out rsa_private.der

# der转pem
openssl rsa -inform DER -in rsa_private.der -outform PEM -out rsa_private2.pem

```







### java中秘钥

#### 将证书和私钥生成jks



```shell
openssl pkcs12 -export -name myservercert -in server.crt -inkey server.key -out keystore.p12

keytool -importkeystore -destkeystore mykeystore.jks -srckeystore keystore.p12 -srcstoretype pkcs12 -alias myservercert

keytool -importkeystore -srckeystore mykeystore.jks -destkeystore mykeystore.jks -deststoretype pkcs12
```





```
openssl pkcs8 -topk8 -nocrypt -in server.key -out pkcs8_key.pem

openssl pkcs8 -topk8 -inform PEM -in server.key -outform PEM -nocrypt -out server_pkcs8.key


openssl genrsa -out server.key 2048
openssl pkcs8 -topk8 -inform PEM -in server.key -outform PEM -nocrypt
```



## SSL和TLS协议

SSL（Secure Socket Layer）由Netscape开发，协议有1.0,2.0,3.0版本，目前一般仅使用3.0

TLS（Transport Layer Security）由IETF（Internet Engineering Task Force，Internet工程任务组）制定的一种新的协议。IETF对SSL3.0进行了标准化，并添加了少数机制(但是几乎和SSL3.0无差异)，标准化后的IETF更名为TLS1.0，可以说TLS就是SSL的新版本3.1。 总共有1.0,1.1,1.2，1.3三个版本。当前TSL1.2和TSL1.3使用较多。



<img src="/其他知识点/ssl/.assert/ssl和tls/image-20220724110753898.png" alt="image-20220724110753898" style="zoom:50%;" />



### TSL1.2

![tls1.2](/其他知识点/ssl/.assert/ssl和tls/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MzI0MDU3,size_16,color_FFFFFF,t_70-20220724112040357.png)

<img src="/其他知识点/ssl/.assert/ssl和tls/image-20220724111841953.png" alt="image-20220724111841953" style="zoom:50%;" />

![TLS1.2handshake](/其他知识点/ssl/.assert/ssl和tls/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MzI0MDU3,size_16,color_FFFFFF,t_70.png)



### 秘钥协商算法



#### master_secret计算



在TLS协议中，加密使用的主秘钥为master_secret，它是由pre_master_secret，ClientHello.random和ServerHello.random生成。计算公式如下

```
        master_secret = PRF(pre_master_secret, "master secret",
                            ClientHello.random + ServerHello.random)
                            [0..47];
```

1. PRF：伪随机函数，用于生成master_secret
2. Pre_master_secret：该秘钥有两种方式获取。一种是由客户端和服务端通过DH算法协商得到；另外一种是由客户端随机产生，通过公钥加密，发送给服务端
3. "master secret"：是一个常量。
4. ClientHello.random和ServerHello.random：分别是客户端和服务端的Hello消息中的随机数

> 主秘钥还有一种增强计算方式，可以参考：https://blog.csdn.net/mrpre/article/details/80056618



#### pre_master_secret计算

对于不同的协商算法，计算pre_master_secret的计算方式不同，下面介绍两种典型计算方法

##### 依赖RSA计算

1. 在服务端给客户端发送证书时，客户端验证证书正常

2. 将会随机生成一个pre_master_secret，然后通过服务端的公钥加密，通过ClientKeyExchange消息发送给服务端
3. 服务端接收后，使用私钥解密获取pre_master_secret



<img src="/其他知识点/ssl/.assert/ssl和tls/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5byg5a2f5rWpX2pheQ==,size_20,color_FFFFFF,t_70,g_se,x_16.png" alt="在这里插入图片描述" style="zoom:50%;" />



##### 依赖协商算法ECDHE

1. 服务端选择使用的曲线类型和随机生成自己的私钥，并计算出公钥。然后在ServerKeyExchange消息中发送曲线类型和公钥
2. 客户端接收到曲线类型后，随机生成自己的私钥，然后计算出公钥。然后在ClientKeyExchange消息中发送公钥。通过曲线类型、客户端私钥和服务端公钥可以计算出pre_master_secret
3. 服务端接收到客户端的公钥后，通过曲线类型、服务端私钥和客户端公钥可以计算出pre_master_secret



<img src="/其他知识点/ssl/.assert/ssl和tls/image-20220724155305321.png" alt="image-20220724155305321" style="zoom:50%;" />



<img src="/其他知识点/ssl/.assert/ssl和tls/image-20220724155405527.png" alt="image-20220724155405527" style="zoom:50%;" />

> 交互过程参考：https://blog.csdn.net/m0_50180963/article/details/113061162

<img src="/其他知识点/ssl/.assert/ssl和tls/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5byg5a2f5rWpX2pheQ==,size_20,color_FFFFFF,t_70,g_se,x_16-20220724162421107.png" alt="在这里插入图片描述" style="zoom:50%;" />

##### 基本DH交换流程

<img src="/其他知识点/ssl/.assert/ssl和tls/image-20220724113321797.png" alt="image-20220724113321797" style="zoom:50%;" />







秘钥协商算法：https://www.modb.pro/db/217447

TSL1.2的DH秘钥协商：https://blog.csdn.net/m0_50180963/article/details/113061162

TSL1.2主秘钥的计算方法：https://halfrost.com/https-key-cipher/

TLS协议中PRF和TLS1.3中的HKDF：https://blog.csdn.net/mrpre/article/details/80056618



### TSL1.3

TSL1.3的握手流程相对TSL1.2有较大变化

1. TSL1.3中可以在1个RTT内实现秘钥交换
2. TSL1.3不再支持RSA进行秘钥交换

![tls1.3extension](/其他知识点/ssl/.assert/ssl和tls/format,png.png)

