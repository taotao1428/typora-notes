# make

make用于方便编译c语言的工具



## 使用

https://zhuanlan.zhihu.com/p/92010728



## Makefile文件



### 命令的基本形式

```
target: dependencies
	command

1. target为一个自定义的目标，可以通过make target运行目标的command
2. dependencies为输入文件或者依赖文件
3. 表示该target需要运行的命令



特殊的target
https://www.cnblogs.com/idorax/p/9306528.html
.PHONY: clean
用于指明clean不是一个文件，不用判断clean是不是最新的，总是执行target clean
```



make会根据target和dependencies的修改时间做对比，如果target比较新，就不会再重新执行target命令