# Tomcat



## Tomcat类加载机制

![img](/复习/.assert/tomcat/137084-20180526104342525-959933190.png)



### 问题1：tomcat为什么打破双亲加载结构



tomcat在WebappClassLoader类加载器中，可能会优先自己加载，然后再尝试给父类加载。这与标准的双亲加载方式相反。



加载流程

1. 查看是否加载过
2. 使用java标准classLoader加载，该classLoader为ExtClassLoader
3. 根据类名，判断类是否为tomcat一些基础类。如果是，先使用parent加载再自己加载，如果不是，自己加载再用parent加载
4. 上述3步都没有加载过，抛出ClassNotFound异常



为什么这么做？

1. 为了更好的隔离WebApp，保持WebApp的独立性，尽量不要使用公共的类。



### 问题2：低级类如何获取到高级类

1. 将高级的的类加载器放在线程中，currentContextLoader。spi