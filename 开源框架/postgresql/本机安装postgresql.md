# 本机安装postgesql



官方文章介绍

https://www.postgresql.org/download/linux/redhat/



```shell
# 安装
yum install postgresql-server


# 安装后配置
postgresql-setup --initdb
systemctl enable postgresql.service
systemctl start postgresql.service
```





## 登录

postgresql会自动创建一个postgres用户，切换到该用户下，执行gsql命令即可登录



```
su - postgres
psql

tcpdump -i eth0  -w /tmp/file.cap host 10.211.55.15 and tcp port 5432


tcpdump -i eth0 host 10.211.55.15 and tcp port 5432
```





![image-20231006174341951](/开源框架/postgresql/.assert/本机安装postgresql/image-20231006174341951.png)