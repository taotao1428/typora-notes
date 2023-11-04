# 用户切换su



## sudo文件配置

/etc/sudoers



<img src="/linux/.assert/用户切换su/v2-0ed918b6d0907ae0f4fb258e3ba9a668_1440w.webp" alt="img" style="zoom:50%;" />



执行命令的主机可以配置ip地址



```
1. %开头表示用户组
2. NOPASSWD:表示可以不输入密码执行命令
3. !表示排除某个命令
4. *表示通配所有命令

%minsudev ALL=(ALL) NOPASSWD:ALL

%minsudev ALL=(ALL) NOPASSWD:ALL,!/usr/bin/reboot,/usr/bin/*
```



```
# 要求sudo命令在tty中执行，不能直接通过ssh 执行sudo命令
Defaults requiretty

# 要求可以在tty中禁用echo时，才能执行sudo命令。避免明文打印出密码
Defaults !visiblepw
```



### wheel用户组

wheel用户组是运维用户组，可以运行任何命令。一个用户可以添加多个附件用户组`usermod -G <group> <username>`

```
## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL
```



### 查看sudo记录

- 需启用defaults logfile 配置
- 默认日志文件：/var/log/sudo
  查看系统安全日志的目录：cat /var/log/secure



sudo存在secuer内，还可以进入visudo配置文件最后增加一行
Defaults logfile="/var/log/sudo"
下次sudo的日志就可以在此路径查看





## pam安全配置

/etc/pam.d/su



pam配置：https://www.cnblogs.com/fyly/p/12717747.html



### 1、su命令的安全隐患

- 默认情况下，任何用户都允许使用su命令，有机会反复尝试其他用户（如root）的登录密码，带来安全风险
- 为了加强su命令的使用控制，可以借助于PAM认证模块，只允许极个别用户使用su命令进行切换



> 如果切换root失败，可以尝试修改一次root密码试一试



<img src="/linux/.assert/用户切换su/image-20231015145608562.png" alt="image-20231015145608562" style="zoom:50%;" />



```shell
[root@fedora ~]# cat /etc/pam.d/su
#%PAM-1.0
auth		sufficient	pam_rootok.so
# Uncomment the following line to implicitly trust users in the "wheel" group.
# wheel组的成员su到root不输密码
#auth		sufficient	pam_wheel.so trust use_uid
# Uncomment the following line to require a user to be in the "wheel" group.
# 非wheel成员无法切换到root
#auth		required	pam_wheel.so use_uid
auth		substack	system-auth
auth		include		postlogin
account		sufficient	pam_succeed_if.so uid = 0 use_uid quiet
account		include		system-auth
password	include		system-auth
session		include		system-auth
session		include		postlogin
session		optional	pam_xauth.so
auth           sufficient      pam_wheel.so trust use_uid
```





## login.defs

https://www.linuxidc.com/Linux/2019-05/158732.htm

/etc/login.defs

