# java远程调试



## 增加java运行参数

在java命令参数中，添加下面参数。

```shell
# 如果想程序先运行再远程连接调试，可以把suspend修改为n
-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5555
```



## idea调试

使用idea打开源码工程，在idea添加Remote JVM debug运行命令即可，点击运行即可

![image-20230909234610777](/java虚拟机/.assert/java远程调试/image-20230909234610777.png)