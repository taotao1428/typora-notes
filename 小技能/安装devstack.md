# 安装devstack

## 操作系统

openEuler-22.03-LTS-everything-aarch64-dvd



## vpn



文件/Users/hewutao/Downloads/clash-linux-armv8

拷贝到用户目录

```
# 下载clash文件
# https://github.com/Dreamacro/clash/releases/tag/v1.16.0

mv clash-linux-armv8 clash
chmod u+x clash

./clash

# 等待下载文件，启动成功

cd .config/clash
# 通过订阅目录下载config.yaml
wget -O config.yaml https://subscribe.s7p7.top/link/v4pAAdNwCrMdJVKV?clash=3

wget -O config.yaml https://sub.gt-key.com/app/clash/51168/C8me97dLANiw

wget -O config.yaml https://sub.gt-key.com/app/clash/51168/C8me97dLANiw

cd ~
nohup ./clash > /tmp/clash.log 2>&1 &

nohup ./clash-linux-amd64-v1.14.0 > /tmp/clash.log 2>&1 &

wget -O config.yaml https://sub.gt-key.com/app/clash/51168/C8me97dLANiw

nohup ./clash-linux-amd64-v1.14.0 > /tmp/clash.log 2>&1 &

```



## 配置全局代理

```
vim /etc/bashrc

export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890

# 插入
export http_proxy='http://127.0.0.1:7890'
export https_proxy='http://127.0.0.1:7890'
```



## 问题

### 已存在pip

<img src="/小技能/.assert/安装devstack/image-20220425213625972.png" alt="image-20220425213625972" style="zoom:50%;" />





卸载已安装的pip

```
yum remove python3-pip
```



### 不存在Python.h



<img src="/小技能/.assert/安装devstack/image-20220425213959054.png" alt="image-20220425213959054" style="zoom:50%;" />



```
yum install python3-devel.aarch64
```



### 缺少apxs



<img src="/小技能/.assert/安装devstack/image-20220425214250853.png" alt="image-20220425214250853" style="zoom:50%;" />



```
yum -y install httpd-devel
```



### 缺少service文件

<img src="/小技能/.assert/安装devstack/image-20220425215514080.png" alt="image-20220425215514080" style="zoom:50%;" />







<img src="/小技能/.assert/安装devstack/image-20220425223959173.png" alt="image-20220425223959173" style="zoom:50%;" />





### 安装低版本python

```
dnf install -y python37
```



### 修改python3版本号后，yum工作不正常

```
vim /usr/bin/yum
```

<img src="/小技能/.assert/安装devstack/image-20220428221012019.png" alt="image-20220428221012019" style="zoom:50%;" />



### git ssl报错

<img src="/小技能/.assert/安装devstack/image-20220430110336846.png" alt="image-20220430110336846" style="zoom:50%;" />

```
git config --global http.sslVerify false
```



### 缺少pip模块



```
wget --no-check-certificate  https://bootstrap.pypa.io/get-pip.py

python3 get-pip.py
```





### 其他问题

```
yum install open-iscsi.aarch64
yum install libvirt.aarch64
```



cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DSYSCONFDIR=/etc -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_EXTRA_CHARSETS=all -DWITH_BOOST=/usr/local/boost -DDOWNLOAD_BOOST=1

```
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DSYSCONFDIR=/etc -DDEFAULT_CHARSET=utf8  -DDEFAULT_COLLATION=utf8_general_ci  -DWITH_EXTRA_CHARSETS=all
————————————————
版权声明：本文为CSDN博主「爱雪君」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_33291624/article/details/114350481
```



wget https://github.com/thkukuk/rpcsvc-proto/releases/download/v1.4.1/rpcsvc-proto-1.4.1.tar.xz
xz -d rpcsvc-proto-1.4.1.tar.xz
tar -xvf rpcsvc-proto-1.4.1.tar
cd rpcsvc-proto-1.4.1 
./configure
make
make install