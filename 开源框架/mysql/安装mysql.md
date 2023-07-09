# 安装mysql





```
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DSYSCONFDIR=/etc -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_EXTRA_CHARSETS=all -DWITH_BOOST=/usr/local/boost
————————————————
版权声明：本文为CSDN博主「你的kd大哥」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_44242973/article/details/120292307
```



## 安装依赖

```
yum -y install gcc gcc-c++ ncurses-devel openssl openssl-devel bison libtirpc-devel
```



### 安装

#### 下载rpcsvc-proto

https://github.com/thkukuk/rpcsvc-proto/tags



```
tar -zxf <文件名>

cd <文件夹>

./configure

make && make install



wget https://github.com/thkukuk/rpcsvc-proto/releases/download/v1.4.1/rpcsvc-proto-1.4.1.tar.xz
xz -d rpcsvc-proto-1.4.1.tar.xz
tar -xvf rpcsvc-proto-1.4.1.tar
cd rpcsvc-proto-1.4.1
./configure
make
make install
```





### 下载软件源码

```
https://github.com/mysql/mysql-server/tags

wget https://github.com/mysql/mysql-server/archive/refs/tags/mysql-5.7.38.zip
unzip mysql-5.7.38.zip

cd mysql-server-mysql-5.7.38/




```



## cmake

```shell
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DSYSCONFDIR=/etc -DDEFAULT_CHARSET=utf8mb4 -DDEFAULT_COLLATION=utf8mb4_general_ci -DWITH_EXTRA_CHARSETS=all -DWITH_BOOST=/usr/local/boost -DDOWNLOAD_BOOST=1

# 没有安装boost添加参数：-DDOWNLOAD_BOOST=1

# -DCMAKE_INSTALL_PREFIX：指定将 MySQL 数据库程序安装到某目录下，如目录/usr/local/mysql。
# -DSYSCONFDIR：指定初始化配置文件目录。
# -DDEFAULT_CHARSET：指定默认使用的字符集编码，如 utf8。
# -DDEFAULT_COLLATION：指定默认使用的字符集校对规则，utf8_general_ci 是适 用于 UTF-8 字符集的通用规则。
# -DWITH_EXTRA_CHARSETS：指定额外支持的其他字符集编码。
# -DWITH_BOOST：指定 boost 库的位置，MySQL5.7 版本编译安装时必须添加这个参 数。
```



mysql5.7支持openssl1.1



```
使用openssl version查看版本是否超过为3.以上，如果是，需要卸载后重新装1.1版本

yum remove openssl openssl-devel

yum install openssl1.1-devel openssl1.1
```





### make

```
make && make install
```



### 创建mysql用户



```
useradd -M -s /sbin/nologin mysql
```







## 编辑配置文件

配置文件为/etc/my.cnf。

```shell
vim /etc/my.cnf
# 插入
[client]
socket=/usr/local/mysql/data/mysql.sock
[mysqld]
socket=/usr/local/mysql/data/mysql.sock
# 服务器开启此sock监听
bind-address=0.0.0.0
# 监听本机上的所有网卡接口
skip-name-resolve
# 跳过域名解析
port=3306
basedir=/usr/local/mysql
# 设置 mysql 的安装目录
datadir=/usr/local/mysql/data
# 设置 mysql 数据库的数据的存放目录 
max_connections=2048
# 允许最大连接数 
character-set-server=utf8
# 服务端使用的字符集默认为 utf8 
default-storage-engine=INNODB
# 创建新表时将使用的默认存储引擎 
lower_case_table_names=1
# 表名存储在磁盘是小写的，比较的时候不区分大小写，可选值有（0|1|2）
max_allowed_packet=16M
# 限制server接受的数据包大小
```







## 初始化

,jdVtuP:B3-k

efJLmhvw6x>f

```shell
/usr/local/mysql/bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
# 注意最后的乱码（密码）

# 初始密码：fJ9&j.NDvjJ2
```

![image-20220502154214052](/开源框架/mysql/.assert/安装mysql/image-20220502154214052.png)



## 配置service



```shell
vim /lib/systemd/system/mysqld.service
[Unit]
Description=mysqld
After=network.target
[Service]
Type=forking
ExecStart=/usr/local/mysql/support-files/mysql.server start
ExecReload=/usr/local/mysql/support-files/mysql.server restart
ExecStop=/usr/local/mysql/support-files/mysql.server stop
PrivateTmp=true
[Install]
WantedBy=multi-user.target

```



## 配置mysql

```
ln -s  /usr/local/mysql/bin/mysql  /usr/local/bin/mysql
```



## 连接

使用初始密码登录，然后需要修改密码

```
mysql -uroot -pfJ9&j.NDvjJ2 -S /usr/local/mysql/data/mysql.sock
```



```
set password=password('新密码')


create user 'hewutao'@'%' IDENTIFIED BY '******' ;

grant all on *.* to 'hewutao'@'%';
```









```shell
### 首先编译cmake 编译过程视电脑情况而定 有的编译很快
[root@node2 src]# cd cmake-3.16.2
[root@node2 cmake-3.16.2]# ./configure
[root@node2 cmake-3.16.2]# echo $?  ### 如果不为0 请检查环境 如果为0 请继续执行下一步
0
[root@node2 cmake-3.16.2]# gmake && gmake install
[root@node2 cmake-3.16.2]# echo $? ###说明Gmake安装成功
0
[root@node2 cmake-3.16.2]# cd .. 
[root@node2 src]# mv boost_1_59_0/ /usr/local/boost 

### 创建一个mysql的用户
[root@node2 src]#  useradd -M -s /sbin/nologin mysql ### 禁止他登陆

###  最重要的一步来了
[root@node2 src]# cd /usr/src/mysql-5.7.28 
[root@node2 mysql-5.7.28]#  cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DSYSCONFDIR=/etc -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_EXTRA_CHARSETS=all -DWITH_BOOST=/usr/local/boost   ### 这一步时间也挺长的
# -DCMAKE_INSTALL_PREFIX：指定将 MySQL 数据库程序安装到某目录下，如目录/usr/local/mysql。
# -DSYSCONFDIR：指定初始化配置文件目录。
# -DDEFAULT_CHARSET：指定默认使用的字符集编码，如 utf8。
# -DDEFAULT_COLLATION：指定默认使用的字符集校对规则，utf8_general_ci 是适 用于 UTF-8 字符集的通用规则。
# -DWITH_EXTRA_CHARSETS：指定额外支持的其他字符集编码。
# -DWITH_BOOST：指定 boost 库的位置，MySQL5.7 版本编译安装时必须添加这个参 数。
[root@node2 mysql-5.7.28]#  echo $? 
0
[root@node2 mysql-5.7.28]# make && make install  这一步编译大概二十分钟 我是在我本地的虚拟机上装的
[root@node2 mysql-5.7.28]#  echo $? 编译完成的每一步都要检查一下因为mysql编译的时间较长  所以我们还是要谨慎一点
0
vim /etc/my.cnf    
[client]
socket=/usr/local/mysql/data/mysql.sock
# 客户端通过此sock文件连接服务器
[mysqld]
socket=/usr/local/mysql/data/mysql.sock
# 服务器开启此sock监听
bind-address=0.0.0.0
# 监听本机上的所有网卡接口
skip-name-resolve
# 跳过域名解析
port=3306
basedir=/usr/local/mysql
# 设置 mysql 的安装目录
datadir=/usr/local/mysql/data
# 设置 mysql 数据库的数据的存放目录 
max_connections=2048
# 允许最大连接数 
character-set-server=utf8
# 服务端使用的字符集默认为 utf8 
default-storage-engine=INNODB
# 创建新表时将使用的默认存储引擎 
lower_case_table_names=1
# 表名存储在磁盘是小写的，比较的时候不区分大小写，可选值有（0|1|2）
max_allowed_packet=16M
# 限制server接受的数据包大小

```

