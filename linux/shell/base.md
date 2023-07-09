# 基础

http://c.biancheng.net/view/2804.html

## 基础语法







### 定义变量和引用变量

```shell
name=hwt

echo "hello $name"
```



> shell中除了关键字（function，if，case等）和声明变量语句不能使用变量替换，其他都可以使用变量替换



### 定义方法和调用方法

```shell
# 定义方法
function hello() {
	echo "hello world!"
}

# 调用方法
hello


# 定义有入参的方法
function hello2() {
	local name=$1
	echo "hello $name"
}

# 调用方法
hello2 hwt
```



### 控制符

#### if

```shell
name=hwt
if [ $name = hwt ]; then
	echo "name is hwt"
fi


if [ $name = hwt ]; then
	echo "name is hwt"
elif [ $name = tao ]; then
	echo "name is tao"
else
	echo "unknow name!"
fi

```



`[]`是一个命令，支持==，!=判断两个字符串是否相等。里面不能使用> < = 之类的操作符，需要使用-gt,-lt,-eq等字符替代。

```shell
if [ $name == 'hwt' -a  $age -gt  27 ]; then
    echo "is me"
else
    echo "is not me"
fi
```



`test`是一个命令， 它与`[]`基本是等价

```shell
if test $name == 'hwt' -a  $age -gt  27 ; then
    echo "is me"
else
    echo "is not me"
fi
```



`[[]]`是一个内建的关键字，它的功能比`[]`要丰富。可以使用>,<,=等操作符（不要使用<>比较数字，它是按照字符串比较，需要使用-lt,gt），还可以在内部直接使用||,&&等逻辑操作符，不能使用-a和-o逻辑操作符，但是支持-gt等比较操作符。还支持使用`=~`进行正在表达式匹配。

```shell
if [[ $1 == 'hwt' &&  $2 >  27 ]] ; then
    echo "is me"
else
    echo "is not me"
fi

if [[ $1 =~ ^hwt+$ ]]; then
    echo "is my name"
else
    echo "is not my name"
fi
```



`(( ))`主要用于数学计算

```shell
if (( $1 > 20 )); then
    echo "right"
else
    echo "wrong"
fi
```





`=~`正则表达式不支持\s,\S,\w,\W特殊字符，需要使用下面的字符替代

[:alnum:]	表示所有字母数字的集合，[a-z A-Z 0-9]
[:alpha:]	表示所有字母的集合，[a-z A-Z]
[:digit:]	表示所有数字，[0-9]
[:lower:]	表示所有的小写字母的集合，小写[a-z]
[:upper:]	表示所有大写字母的集合，大写[A-Z]
[:space:]	表示空格
[:blank:]	表示所有空格或者制表键（tab键）的集合
[:punct:]	表示所有的标点字符
[:cntrl:]	表示所有的控制字符
[:print:]	表示所有的非控制字符
[:graph:]	表示所有可视，可打印的字符（不包含空格）
[:xdigit:]	表示所有十六进制的数字的集合，[0-9 a-f A-F]



```shell
# 特殊字符需要放在[]中使用，不支持单独使用
if [[ $1 =~ ^hwt[[:space:]]+$ ]]; then
    echo "is my name"
else
    echo "is not my name"
fi
```



特殊的符号

-n str 如果字符串不为空，则为真

-z str 如果字符串为空，则为真

e filename 如果 filename存在，则为真

-d filename 如果 filename为目录，则为真 

-f filename 如果 filename为常规文件，则为真

-L filename 如果 filename为符号链接，则为真

-r filename 如果 filename可读，则为真 

-w filename 如果 filename可写，则为真 

-x filename 如果 filename可执行，则为真

-s filename 如果文件长度不为0，则为真

-h filename 如果文件是软链接，则为真

filename1 -nt filename2 如果 filename1比 filename2新，则为真。

filename1 -ot filename2 如果 filename1比 filename2旧，则为真。



```shell
if [ -f filename ]; then
	echo "file existed"
fi
```





if的缩写

```shell
[ -f test.sh ] && echo 'test.sh existed'
```



#### while

其中while中的条件与if相同

```shell
num=$1
i=1
while (( i < num  ))
do
   echo $i
   (( i++ ))
done


i=1

while [[ $i -lt $num  ]]
do
   echo $i
   (( i++ ))
done
```



#### for



c语言风格

```shell
sum=0
for (( i=0; i < $1; i++))
do
	((sum+=i))
done
echo $sum
```



python风格

```shell

for num in 1 100 1000 11
do
	echo $num
done

```



```shell
echo {1..100} # 可以输出 1 2 3 到 100的数
echo $(seq 4 4 100) # seq 2 2 100表示从 2 开始，每次增加 2，到 100 结束。
```

<img src="/linux/shell/.assert/base/image-20220828233353869.png" alt="image-20220828233353869" style="zoom:50%;" />





#### case

除最后一个分支外，其他分支都需要以`;;`结尾。`*)`通常放在最后，用于处理没有匹配到的情况

```shell
#!/bin/bash

printf "Input integer number: "
read num

case $num in
    1)
        echo "Monday"
        ;;
    2)
        echo "Tuesday"
        ;;
    3)
        echo "Wednesday"
        ;;
    4)
        echo "Thursday"
        ;;
    5)
        echo "Friday"
        ;;
    6)
        echo "Saturday"
        ;;
    7)
        echo "Sunday"
        ;;
    *)
        echo "error"
esac
```



每个选项的模式可以是普通字符串，也可以是简单的正则表达式，支持以下格式。每种格式可以混用

| 格式  | 说明                                                         |
| ----- | ------------------------------------------------------------ |
| *     | 表示任意字符串。                                             |
| [abc] | 表示 a、b、c 三个字符中的任意一个。比如，[15ZH] 表示 1、5、Z、H 四个字符中的任意一个。 |
| [m-n] | 表示从 m 到 n 的任意一个字符。比如，[0-9] 表示任意一个数字，[0-9a-zA-Z] 表示字母或数字。 |
| \|    | 表示多重选择，类似逻辑运算中的或运算。比如，abc \| xyz 表示匹配字符串 "abc" 或者 "xyz"。 |



```shell
name=$1

case $name in
    hwt) echo "name is hwt";;
    h[a-c]) echo 'match h[a-c]';;
    w[tao]*) echo 'match w[tao]*';;
    tao|Tao) echo 'match tao|Tao';;
    n*|k[a-c]) echo 'match n*|k[a-c]';;
    *) echo 'other'
esac
```





#### (( ))

进行整数运算，使用$可以获取计算后的值

```shell
echo $((2*3))
echo $((a+b*c)) # 变量可以不加$
echo $(($a+$b*$c))

((b=1+2)) # 可以在变量中直接赋值
echo $b # 3
```



$(( ))可以将其他进制转成十进制数显示出来。用法如下：`echo $((N#xx))`
其中，N为进制，xx为该进制下某个数值，命令执行后可以得到该进制数转成十进制后的值

```shell
echo $((2#110))
```





## 特殊变量



| 变量 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| `$?` | 上一次命令或者方法的返回值                                   |
| `$#` | 参数的长度                                                   |
| `$n` | 获取第n个参数，其中`$0`表示当前的脚本                        |
| `$@` | 表示所有参数。当加双引号时，参数还是独立的参数。如果是参数传递，最好增加双引号 |
| `$*` | 表示所有参数，与`$@`略有差异。当加双引号时，所有参数将会称为一个参数 |
| `$$` | 表示当前的进程id                                             |



`shift`：将会将第一个参数去掉，在处理输入参数时常用



## 字符串

使用单引号的字符串表示里面的字符串不用解析，使用双引号的字符串会解析里面的变量

```
name=hwt

echo 'hello, $name'
echo "hello, $name"
```

<img src="/linux/shell/.assert/base/image-20220828162454937.png" alt="image-20220828162454937" style="zoom:50%;" />

shell在执行时，会将文本中的变量替换成字符串后，再执行。在shell中，任何文本都可以使用文本表示



shell只能识别最外层的引号，其他都是使用空格进行分割。

```shell
function print_args() {
    echo "print args:"
    while (( $# > 0 ))
    do
        echo $1
        shift
    done
}
print_args "hwt" "he wu tao"
args='"hwt" "he wu tao"'
print_args $args
print_args "$args"
```



<img src="/linux/shell/.assert/base/image-20220829234457792.png" alt="image-20220829234457792" style="zoom:50%;" />





## 数组

定义数组

```shell
# 定义数组
name=("hwt"  "tao" "hah")
# 为某个元素赋值
name[0]="HWT"

# 特殊的符号
${names[0]} # 获取0号位的值
${names[@]} # 表示获取所有元素
${#names[@]} # 获取数组的长度
${!names[@]} # 获取数组的索引
```



## 输入输出

### 输入

可以使用read获取变量

```shell
read -p '请输入姓名:' name

echo "姓名: $name"
```

<img src="/linux/shell/.assert/base/image-20220830223419154.png" alt="image-20220830223419154" style="zoom:50%;" />

| 选项         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| -a array     | 把读取的数据赋值给数组 array，从下标 0 开始。                |
| -d delimiter | 用字符串 delimiter 指定读取结束的位置，而不是一个换行符（读取到的数据不包括 delimiter）。 |
| -e           | 在获取用户输入的时候，对功能键进行编码转换，不会直接显式功能键对应的字符。 |
| -n num       | 读取 num 个字符，而不是整行字符。                            |
| -p prompt    | 显示提示信息，提示内容为 prompt。                            |
| -r           | 原样读取（Raw mode），不把反斜杠字符解释为转义字符。         |
| -s           | 静默模式（Silent mode），不会在屏幕上显示输入的字符。当输入密码和其它确认信息的时候，这是很有必要的。 |
| -t seconds   | 设置超时时间，单位为秒。如果用户没有在指定时间内输入完成，那么 read 将会返回一个非 0 的退出状态，表示读取失败。 |
| -u fd        | 使用文件描述符 fd 作为输入源，而不是标准输入，类似于重定向。 |





## 操作





### 字符串操作



#### 单个字符串

1. 匹配：可以使用`[[ =~ ]]`判断是否赋值正则表达式
2. 原生截取：直接使用`${name:4:}` 原生截取： https://blog.csdn.net/JineD/article/details/124196546
3. cut截取：`echo '123456涛涛涛7890' | cut -c 2-7` 23456涛。需要知道起点和终点的位置
4. cut+rev：`echo 1234567890 | cut -c 3- | rev | cut -c 4- | rev`34567。可以不知道终点位置，知道倒数第几就可以
5. tr替换: `echo abc123ABC | tr 'a-z' 'A-Z'`  ABC123ABC
6. tr删除字符：`echo abc123ABC | tr -d 'a-z'` 123ABC
7. tr删除连续的字符，只保留一个：`echo "tr    is    a   shell   order   " | tr -s ' '` tr is a shell order
8. 统计词的个数：`wc -w`
9. grep提取：`echo 'afsdfaf234' | egrep -o '[0-9]+'` 234
10. sed提取（使用替换实现）:`echo 1320238094@qq.com | sed -e 's/\([0-9]\+\)@qq.com/\1/g'`1320238094
11. 去重：`uniq`
12. 排序：`sort`



匹配，截取，替换，过滤，提取，统计



#### 文本操作



### 数组操作

定义，遍历



## 常用命令



tr，awk，grep，wc，expr，expect



### glob模式

https://www.linuxidc.com/Linux/2016-08/134192.htm





### expect

用于通过机器完成人机交互

info expect可以查看完整文档



常用方法

```
expect -f <filename> 执行某个文件
 
expect -c "command" 执行命令
```



```shell
#! /usr/bin/expect

# 设置超时时间
set timeout 30 
spawn ssh hwt@192.168.2.9 # 启动一个进程

expect {
		# 如果超时没有匹配到任何分支
    timeout {
        send_user "\nlogin timeout\n" # 向用户终端发送内容
        exit 1 # 退出expect
    }
    eof { # ssh进程已经退出
        exit 1
    }
    "*yes/no*" {
        send -- "yes\n"; exp_continue
    }
    "*password*" {
        send -- "h1ewutao12#$%\n" # 输入执行文本，\n表示回车
    }
}

expect {
    "*password*"  {
        send_user "\npassword is invalid\n"
        exit 1
    }
    timeout {
        send_user "\nlogin timeout\n"
        exit 1
    }
    eof {
        exit 1
    }

    "*\[hwt*"
}

send -- "date\n"

expect {
    timeout {
        send_user "\nexecute date timeout\n"
        exit 1
    }
    eof {
        exit 1
    }
    "*\[hwt*"
}

send -- "exit\n"

expect eof
```



```
#! /usr/bin/expect

spawn ssh hwt@192.168.2.9

expect {
    timeout {
        send_user "\nlogin timeout\n"
        exit 1
    }
    eof {
        exit 1
    }
    "*yes/no*" {
        send -- "yes\n"; exp_continue
    }
    "*password*" {
        send -- "hewutao12#$%\n"
    }
}

expect {
    "*password*"  {
        send_user "\npassword is invalid\n"
        exit 1
    }
    timeout {
        send_user "\nlogin timeout\n"
        exit 1
    }
    eof {
        exit 1
    }

    "*\[hwt*"
}

interact

wait
```



```shell
expect <<'END'
log_user 0 # 关闭命令执行时的输出，默认是1，表示开启
spawn sh -c {echo hello; exit 42}
expect eof
puts $expect_out(buffer)

lassign [wait] pid spawnid os_error_flag value # 获取进程退出状态

if {$os_error_flag == 0} {
    puts "exit status: $value"
} else {
    puts "errno: $value"
}
END
```



```
hello

exit status: 42
```





## 自动填充

https://blog.csdn.net/Q1302182594/article/details/52344503/

利用complete命令实现自动填充内容



实现功能，为命令tsh补充提示/Users/hewutao/Scripts/login/config目录下的文件



1、创建tsh_complete.sh文件，内容如下

```shell
complete -F hello_complete_func tsh

function hello_complete_func()   
{  
    local cur prev opts  
    COMPREPLY=()  
    cur="${COMP_WORDS[COMP_CWORD]}"  
    prev="${COMP_WORDS[COMP_CWORD-1]}"  
    opts="`ls /Users/hewutao/Scripts/login/config`"                                                                                                                                                                                 

    if [[ ${cur} == * ]] ; then  
        COMPREPLY=( $(compgen -W "${opts}" -- ${cur}) )   
        return 0   
    fi  
}
```



2.~/.zshrc文件末尾，添加source tsh_complete.sh，此时将会自动导入complete



3.执行tsh命令时，按下tab键，将会自动补全

<img src="/linux/shell/.assert/base/image-20230528151020712.png" alt="image-20230528151020712" style="zoom:50%;" />





