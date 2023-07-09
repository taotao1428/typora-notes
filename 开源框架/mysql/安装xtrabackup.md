# xtrabackup





## 安装

https://blog.csdn.net/chenghuikai/article/details/68946617

### 下载源码



### 编译



```
yum install cmake gcc gcc-c++ libaio libaio-devel automake autoconf \
  bison libtool ncurses-devel libgcrypt-devel libev-devel libcurl-devel \
  vim-common

cmake -DBUILD_CONFIG=xtrabackup_release -DWITH_MAN_PAGES=OFF -DIGNORE_AIO_CHECK=ON -DWITH_BOOST=/usr/local/boost
```

