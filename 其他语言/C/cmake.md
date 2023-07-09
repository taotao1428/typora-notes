# cmake



```makefile
# cmake版本
cmake_minimum_required(VERSION 3.19)

# 项目名称，可以使用${PROJECT_NAME}获取项目名称
project(exectest)

# 设置变量，这里是设置c++版本
set(CMAKE_CXX_STANDARD 11)

# 添加第三方库和头文件的搜索路径
set(INC_DIR /usr/local/include/)
set(LINK_DIR /usr/local/lib)

include_directories(${INC_DIR})
link_directories(${LINK_DIR})


add_executable(exectest main.cpp service/userservice.h service/userservice.cpp include/service4.h)

# 在目标文件中链接第三方库
target_link_libraries(exectest service4)
```



```makefile
# 扫描所有源文件，放在DIR_SRCS中
aux_source_directory(. DIR_SRCS)
```



### 编译目标

```makefile
# 编译可执行文件
add_executable(zipapp zipapp.cpp)

# 生成库文件，默认是静态库
add_library(archive archive.cpp zip.cpp lzma.cpp) # 静态库
add_library(archive SHARED archive.cpp zip.cpp lzma.cpp)  # 动态库
add_library(archive STATIC archive.cpp zip.cpp lzma.cpp) # 静态库

```



### 编译的控制参数

```makefile
# 添加第三方库和头文件的搜索路径
set(INC_DIR /usr/local/include/)
set(LINK_DIR /usr/local/lib)
include_directories(${INC_DIR})
link_directories(${LINK_DIR})

# 指定需要链接的库
target_link_libraries(zipapp archive)

# 指定编译时定义的变量
target_compile_definitions(archive INTERFACE USING_ARCHIVE_LIB)

# 指定编译时需要包含的头文件
target_include_directories(add INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/iadd)
```



### 子模块

顶级CMakeLists.txt

```makefile
add_subdirectory(mymath) # 添加子模块，mymath为子模块所在文件夹

add_executable(exectest main.cpp service/userservice.h service/userservice.cpp include/service4.h)
target_link_libraries(exectest mymath) # 链接子模块，mymath为子模块输出库的名字
```



mymath的CMakeLists.txt

```makefile
# 申明子模块输出动态库
add_library(mymath SHARED pingfang.cpp sqrt.cpp)

# 将子模块的头文件暴露给依赖者，依赖者编译时会自动导入
target_include_directories(mymath INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```



注意

1. 如果直接编译，最终的执行文件中，会通过rpath搜索依赖的库，所以编译后的文件，可以直接执行。
2. 使用cmake安装软件时，会将rpath去掉，导致程序无法运行。两种方式解决，1.将依赖的库，安装到系统默认的路径中/usr/local/lib,/usr/lib。2.在安装时，指定rpath。





### 安装

https://cmake.org/cmake/help/latest/command/install.html#command:install

```makefile

# 指定安装时rpath的路径，一定要出现在add_executable之前
set(CMAKE_INSTALL_RPATH "/Users/hewutao/CLionProjects/exectest/install/lib")

add_executable(exectest main.cpp service/userservice.h service/userservice.cpp include/service4.h)

# 安装执行文件
install(TARGETS exectest RUNTIME DESTINATION /Users/hewutao/CLionProjects/exectest/install/bin)

# 安装库文件
install(TARGETS mymath LIBRARY DESTINATION /Users/hewutao/CLionProjects/exectest/install/lib)
```



在执行命令时，也可以设置安装路径和rpath

```
 -DCMAKE_INSTALL_PREFIX=/tmp/fc1 
 -DCMAKE_INSTALL_RPATH=/tmp/fc1/lib
```



rpath介绍

https://blog.csdn.net/zhangzq86/article/details/80718559



### 其他语法

```makefile

# list，添加元素到数组中
set(srcs archive.cpp zip.cpp)
if (LZMA_FOUND)
  list(APPEND srcs lzma.cpp)
endif()


```







### 链接时的Public private interface属性

https://blog.csdn.net/qq_45605535/article/details/117323194



1. public表示操作对自己和其他人都有效
2. private表示操作只对自己有效
3. interface表示操作只对其他人有效



可以使用这些属性的标签

```
target_link_libraries 链接其他库

```







生成静态库

```
add_library(MathFunctions mysqrt.cpp)
```





## 动态库链接

动态库级依赖时，仅可见下一级依赖。例如a.so依赖b.so，b.so依赖c.so，那么a.so只可见b.so中的方法





