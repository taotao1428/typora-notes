# gcc

https://www.runoob.com/w3cnote/gcc-parameter-detail.html

## gcc默认搜索路径

```
1.For C:

    gcc -xc -E -v -

2.For C++:

    gcc -xc++ -E -v -
```





## 常用命令



```
gcc -o output file1.c file2.c 编译成可执行文件

g++ -o dylibtest  dylibtest.cpp

gcc 与 g++ 分别是 gnu 的 c & c++ 编译器 gcc/g++ 在执行编译工作的时候，总共需要4步：

1、预处理,生成 .i 的文件[预处理器cpp]
2、将预处理后的文件转换成汇编语言, 生成文件 .s [编译器egcs]
3、有汇编变为目标代码(机器代码)生成 .o 的文件[汇编器as]
4、连接目标代码, 生成可执行程序 [链接器ld]


1. -c 只编译成机器码，不链接成执行程序
2. -o <filename> 指定目标文件名称，默认为a.out
3. -I<dir> 指定头文件的查找路径，会优先查找该路径，再查找系统路径
4. -nostdinc,-nostdin C++ 不在系统默认的路径中找头文件，与-Idir配合使用


编译动态库
g++ -Wall -shared -o libservice.so service.cpp

使用动态库
g++ -Wall -lservice main.cpp

编译目标文件
g++ -Wall -c -o service.o service.cpp

将目标文件制作成静态库
ar -rcs libMyMath.a add.o sub.o div1.o

```



## 头文件和cpp文件的理解

1. 头文件只是一个申明，不提供方法的具体实现
2. cpp文件提供具体的实现。实现方法时和引用方法时，使用的头文件可以不同
3. 动态库编译时，不会打包到可执行文件中，在执行动态加载。



头文件搜索路径

1. cpp文件当前目录
2. g++参数`-I<dir>`
3. 系统默认路径



动态库的搜索方式

1. 动态库的搜索路径可以使用`-L`执行
2. 动态库可以使用`-l`指定。如果执行的名字为`ssl`，搜索的动态库名为`libssl.so`



静态库

1. 静态库其实是由编译的目标文件组成
2. 静态库也可以使用-l加载，但需要执行全名



## rpath

*运行时顺序举例说明*：

```
-Wl,-rpath=/home/hello/lib*
```

*表示将/home/hello/lib目录作为程序运行时**第一个寻找库文件的目录**，程序寻找顺序是：/home/hello/lib-->/usr/lib-->/usr/local/lib。*





补充缺少的动态库方法

1. 将动态库拷贝到默认库文件所在文件夹中/usr/lib,/usr/lib64,usr/local/lib,usr/local/lib64中

<img src="/其他语言/C/.assert/gcc/image-20221231220159358.png" alt="image-20221231220159358" style="zoom:50%;" />



2. 使用LD_LIBRARY_PATH变量指定动态库所在路径

<img src="/其他语言/C/.assert/gcc/image-20221231220945356.png" alt="image-20221231220945356" style="zoom:50%;" />



## 查看链接的动态库



```
ldd java
```



![img](/其他语言/C/.assert/gcc/1620.png)





