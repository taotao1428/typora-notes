# MAT使用

https://blog.csdn.net/yb_butterfly/article/details/105623976



## 下载

官网下载地址，默认镜像速度比较慢，可以选择其他镜像下载

https://www.eclipse.org/mat

![img](https://img-blog.csdnimg.cn/20200419222733610.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3liX2J1dHRlcmZseQ==,size_16,color_FFFFFF,t_70)



## 使用

MAT打开jvm内存导出文件的界面如下，打开的界面中包含了大对象的信息，包含他们的类型和大小。下面还有其他功能的入口

![image-20211212170143540](/Users/hewutao/Library/Application Support/typora-user-images/image-20211212170143540.png)



### 直方图

点击直方图可以查询class的信息。该界面可以很好的分析内存泄漏和大对象

1. Objects。对象的个数
2. Shallow Heap：对象自身占用的内存
3. Retained Heap：对象自身的Shallow Heap加上引用对象的Retained Heap（引用对象需要排除掉被GC roots直接或间接引用的对象）。

[retained heap介绍](https://blog.csdn.net/a740169405/article/details/53610689)

![image-20211212171120862](/Users/hewutao/Library/Application Support/typora-user-images/image-20211212171120862.png)

可以进一步查看对象在哪些对象中被引用和引用哪些对象

![image-20211212174707669](/Users/hewutao/Library/Application Support/typora-user-images/image-20211212174707669.png)

![image-20211212174953737](/Users/hewutao/Library/Application Support/typora-user-images/image-20211212174953737.png)

## 树形图

树形图可以方便查看Retained Heap大的对象，并且会通过树形结构显示其引用的对象。

![image-20211212181327121](/Users/hewutao/Library/Application Support/typora-user-images/image-20211212181327121.png)

注意：其显示的路径并不是真实类的引用路径。例如：下图中HashMap并不是直接被Thread引用，其实它是被LeakApp对象引用。

![image-20211212181716290](/Users/hewutao/Library/Application Support/typora-user-images/image-20211212181716290.png)



### 查看线程信息

![image-20211212183204276](/Users/hewutao/Library/Application Support/typora-user-images/image-20211212183204276.png)



## OQL

OQL是类似于sql，可以通过语句查询堆中的对象。

https://help.eclipse.org/latest/index.jsp?topic=/org.eclipse.mat.ui.help/welcome.html

<img src="/Users/hewutao/Library/Application Support/typora-user-images/image-20211212182445695.png" alt="image-20211212182445695" style="zoom:50%;" />