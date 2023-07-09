# openjdk

https://github.com/openjdk/jdk/releases/tag/jdk8-b120





```
bash ./configure  --with-boot-jdk=/opt/homebrew/opt/java11/ --disable-warnings-as-errors   --enable-debug 
```







```
报错：was compiled with optimization - stepping may behave oddly; variables may not be available.

bash ./configure  --with-boot-jdk=/opt/homebrew/opt/java11/ --disable-warnings-as-errors   --with-debug-level=slowdebug
```





## 加载class





## 执行字节码



https://zhuanlan.zhihu.com/p/380420495



src/hotspot/share/interpreter/bytecodes.cpp

如下：





寄存器介绍

https://zhuanlan.zhihu.com/p/565060715?utm_id=0



实现

src/hotspot/cpu/aarch64/templateTable_aarch64.cpp



## 垃圾回收

