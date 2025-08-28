# 20250812

## `bash shell`基本命令

`man + 命令` 可以查询命令



先记住两个常用的快捷键。

`shift + ctrl + c` 将所选文本复制到剪贴板中

`shift + ctrl + v` 将剪贴板中的文本粘贴到会话中



`linux`命令

- `ls` 检索命令

`-F` 在目录名后添加/，方便分辨文件和目录

`-a` 显示出所有目录和文件，包括以`.`开头的隐藏文件

`-l `显示文件的详细属性

`-i` 可以查看文件或目录的`inode`（每个对象都是唯一的）

模式匹配

1. `?`代表任意单个字符
2. `*`表示0或者多个字符
3. `[]`表示的是该位置可能出现的字符



## 处理文件

----

```shell
# shell的注释以#打头，看起来和Python没什么区别
ls -l my_s*t

ls f[a-i]ll # 也可以指定范围[a, i]
ls -l f[!a]ll # 还可以用`!`将不需要的内容排除在外
```

- `touch` 创建空文件

```shell
touch test_one
```

- `cp` 复制文件

```shell
cp test_one test_two # source destination
```

`-i`选项将会强制`shell`询问是否要覆盖已有文件。

`-r` 递归复制

按下`tab`键竟然还有望补全`shell`命令。

- `ln` 链接文件
  - 软链接：存储的是原文件的**完整路径字符串**，系统会根据这个路径字符串去定位并访问原文件
  - 硬链接：文件别名，和原文件共享同一份数据

```shell
# 要为一个文件创建软连接，原文件必须存在
ln -s test_file slink_test_file # 如果不加-s则创建的是硬链接
```



- `mv` 可以将文件和目录移动到另一个位置或是对文件进行重命名

```shell
mv fall fzll # 将文件名fall修改为fzll,只影响文件名
```

# 20250813

- `rm` 删除文件

`-i`选项强制`shell`询问是否真的要删除文件

- `mkdir` 创建目录

```shell
mkdir NewDir
ls -ld NewDir # d 表示目录 显示该目录的详细信息

mkdir -p NewDir/SubDir/UnderDir # -p 根据需要创建缺失的父目录
```

- `rmdir` 删除目录

注意：只能删除空目录



## 查看文件内容

----

- `file` 快速查看文件类型

```shell
file .bashrc
# output: .bashrc: ASCII text file命令可以确定文件的编码格式

file Documents
# output: Documents/: directory file命令指出该文件是个目录

file slink_test_file
# output: slink_test_file: symbolic link to test_file file命令可以指出它链接到了哪个文件

```

- `cat` 显示文本内容

```shell
cat -n test_file # 显示内容时加上行号
```

- `more`分页查看
- `tail` 显示文件尾部

```shell
# tail 默认显示文件最后10行
tail -n 2 log_file # 只显示文件的最后两行

# 实时监测系统日志
tail -f log_tile # 在其他进程使用此文件时查看文件内容
```

- `head` 与`tail`类似

## 更多的`bash shell`命令

----

属实是太多了!

## 监测程序

- `ps` 监测进程

```shell
# 默认情况下，ps只显示在当前终端中属于当前用户的进程
ps -ef # -e  显示所有进程 -f 显示进程的详细信息
```

- `top` 实时显示进程信息

- `kill` 向进程发出信号

```shell
kill 3940 # 通过pid向进程发出信号
# kill默认向进程发送TRAM（尽可能停止)信号
```

- `pkill` 使用程序名代替`pid`来终止进程

## 监测磁盘空间

暂时不知你有什么用

- `df`会显示已挂载的文件系统

## 处理数据文件

- `sort`数据排序
- `grep` 数据搜索

```shell
grep [tf] file1 # 从file1中检索出以t或者f开头的数据
```



## 理解`shell`

`shell`不仅仅是`cli(Command-Line Interface(命令行界面))`，更是一种交互式程序。



# 20250814

`linux`内核主要负责

- 系统内存管理
- 软件程序管理
- 硬件设备管理
- 文件系统管理



内核是一段程序代码，在计算机完成自检后，将内核加载到内存。

## 外部命令和内建命令

- 外部命令：外部命令程序每次执行时，会创建一个子进程，这种操作叫做`衍生`。
- 内部命令：作为`shell`的组成部分，无需借助外部程序文件执行。

```shell
# history是一个内建命令，可以查看最近使用过的命令列表
# ！！可以唤回并重用最近一条命令
```

# 20250815

## `linux`环境变量

`bash shell`中有两种环境变量：

- 全局变量：对于`shell`和子`shell`都是可见的
- 局部变量

```shell
printenv # 可以查看全局变量
printenv HOME # 可以显示各环境变量

echo $HOME # 与printenv HOME等价

# 在变量前+$，可以让变量作为其他命令的参数
ls $HOME
```

### 设置用户自定义变量

```shell
my_variable=Hello # 创建仅对该进程可见的局部自定义变量

my_variable="Hello world!" # 如果用于赋值的字符串包含空格，需要用单引号或双引号

# 如果要创建全局变量
export my_variable # 通过export命令以及要导出的变量名实现

# 当然也可以在创建全局变量的同时就为变量赋值。

unset my_variable # 删除环境变量
```

注意：

1. `bash shell`所有环境变量都是大写字母。为了加以区分，自己创建的局部变量最好都用小写字母。
2. `=`两边不要像`C++`一样为了美观带上空格



```shell
linux@linux:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/snap/bin
# PATH环境变量定义了用于查找命令和程序的目录
# 目录之间用:分隔

# 我们可以添加搜索路径
PATH=$PATH:/home/linux/Scripts
```

### 数组变量

```shell
mytest=(zero one two three four)

echo $mytest # 默认显示zero,因为索引从0开始

echo $(mytest[2]) # two

# 要显示整个数组变量，可以使用通配符
echo $(mytest[*])
```

## `linux`文件权限

每一个能访问系统的用户都会被分配一个用户`id`，也叫`uid`。

`root`是系统管理员，`uid`是0。

`linux`组：允许多个用户对系统对象共享一组权限。每个组都有唯一的`GID`和`组名`。

# 20250818

----

## 构建脚本

```shell
#!/bin/bash
# shell不会处理shell脚本中的注释行，但是第一行例外
# 第一行后面的！告诉shell用哪个shell运行脚本
# This script displays the date and who's logged on
date
who # 可以将命令使用;放在一行，也可以放在独立的行
```



`-rw-rw-r-- 1 linux linux 22  8月 15 09:51 test1`

来看这一行啥意思。

- -：文件类型（-表示普通文件，d为目录，l为链接）
- 文件所有者、所属用户组、其他用户：r可读，w可写，-不可执行
- 硬链接数量：1
- 文件所有者的用户名
- 所属组的组名
- 文件大小：22字节
- 文件最后修改时间
- 文件名

### 修改权限

```shell
chmod u+x test1 # u代表用户 g代表组 o代表其他用户 a 表示所有 +增加权限 =设置权限
```



### 显示消息

```shell
#!/bin/bash
echo -n "This scripts displays the date and who's logged on" # -n取消输出内容末尾的换行符
date
```

# 20250819

----

## 用户自定义变量

由字母、数字或下划线组成的字符串，区分大小写，长度不会超过20个字符。

用等号赋值，不能出现空格

```shell
var1=10
```

`shell`脚本以字符串形式存储所有的变量值

```shell
#!/bin/bash
value1=10
value2=$value # value2=10,在赋值语句中使用value1变量的值，必须使用$
```

## 命令替换

1. 反引号
2. $()格式

```shell
#!/bin/bash
testing=$(date) # testing保存的是date的输出
echo "The date and time are: " $testing
```

## 输出重定向

```shell
date > test6 # 重定向运算符创建了test6，并将date命令的输出重定向至该文件
# 追加数据，用>>
```

## 管道

使用`|`将一个命令的输出当做另一个命令的输入



```shell
var1=$[1 + 5] # $是取值符
```

## expr命令

专门用于处理数学表达式，笨。

```shell
expr 1 + 5
expr 5 \* 2 # \是转义字符
# bash shell只支持整数运算
```

为了解决`bash shell`只支持整数运算的问题，`bash`内有计算器`bc`

通过`bc`访问计算器

```shell
bc -q # -q 不显示欢迎信息
var1=10
var1 * 4
40

print val1 # print语句打印变量和数字
10

scale=4 # bc自带内部变量，默认为0，表示保留0位小数
3.44 / 5
.6880
quit
```

在脚本中使用`bc`

```shell
var1=$(echo "scale=4; 3.44 / 5" | bc)
```

一个小练习

```shell
#!/bin/bash
date1="Jan 1, 2020"
date2="May 1, 2020"
time1=$(date -d "$date1" +%s) # -d 指定一个日期字符串 %s指定输出格式为unix时间戳
time2=$(date -d "$date2" +%s)
diff=$[$time2 - $time1]
secondsinday=$(expr 24 \* 60 \* 60) # 计算一天有多少秒
days=$(expr $diff / $secondsinday)

echo "The difference between $date2 and $date1 is $days days"

# 输出结果
# linux@linux:~/test$ chmod u+x ex1.sh
# linux@linux:~/test$ ./ex1.sh
# The difference between May 1, 2020 and Jan 1, 2020 is 121 days
```

# 20250819

----

## 结构化命令

`if command`

`then`

​	`command`

`fi`

`bash shell`会运行`if`之后的命令，如果该命令的退出码为0，即表示命令成功运行，那么位于`then`部分的命令才会被执行。

`fi`语句表示`if then`到此为止

类似地，

`if command`

`then`

​	`commands`

`else`

​	`commands`

`fi`

此外，还有

`if command1`

`then`

​	`commands`

`elif command2`

`then`

​	`commands`

`fi`

### `test`命令

`test condition`

可以用在`if-then`语句中测试不同的条件。如果`test`命令中列出的条件成立，那么`test`命令就会退出并返回状态码0，否则`test`返回非0的状态码。

```shell
#!/bin/bash
# testing the test command
if test # 如果不写test后面的condition，它会返回非0的状态码
then
	echo "No expression returns a True"
else
	echo "No expression returns a False" # 执行else代码块
fi
```

除了`test`命令，还有另一种测试方式

```shell
if [ condition ] # 方括号前后的空格是必须的，真奇怪
then
	commands
fi

# 测试3类条件
-- 数值比较
value1=10
value2=11
if [ $value1 -gt 5 ] # -eq 是否等于 -ge 大于等于 -gt 大于 -le 小于等于 -lt 小于 -ne 不等于
then 
	...
fi

-- 字符串比较
str1=christine
str2=$str1
if [ str1 = str2 ] # != < > -n 长度是否不为0， -z 长度是否为0 在处理类似于 > 重定向运算符时需要加\转义字符，不然shell不知道是在比较大小
then 
	...
fi

-- 文件比较
# -d file file是否存在且是目录
# 类似地， -e 存在？ -f 是否存在且是文件吗 -r 可读？ -s 非空？ -w -x - O 属于当前用户？ -G 默认组与当前用户相同吗？ file -nt file2 file是不是比file2更新 -ot 更旧？
```



# 20250820

`if-then`语句可以使用布尔逻辑

```shell
# [ condition1 ] && [ condition2 ]
```

12章的最后讲述的是

```shell
if (commands) # ()会创建子shell，commands会在子shell中运行，如果运行成功则返回0，否则就是非0值
then 
	commands
# 在if 的条件判断中使用(( $value ** 2 == 100 )), expression可以是任意的数学表达式
# [[ expression ]]可以针对字符串进行模式匹配
```

此外，还有一个`case`命令

```shell
case variable in
pattern1 | pattern2) commands1;;
pattern3) commands2;;
*) default commands;
esac
```

## `for`语句

语法

```shell
for var in list
do 
	commands
done

# example
for test in Alabama Alaska # for循环假定各个值之间以空格分隔，如果某个值有空格，可将其放在双引号内
do
	echo $test
done # 即使在循环结束后，test这个变量依然存在，而且是遍历的最后一个值
```

Q：如何向字符串列表中追加值

```shell
list="A B"
list=$list" C"
```

默认情况下，`bash shell`会将空格、制表符和换行符视为字段分隔符，我们可以在脚本中临时修改`IFS`环境变量的值使其只识别换行符

```shell
IFS=$'\n'
# 在修改IFS之前记得先保存旧的IFS值
old_ifs=$IFS
```



`for`循环仿照C语言

```shell
for (( i=1; i <= 10; ++i ))
```

## `while`命令

```shell
while test command # 0状态码才会判定为真
do 
	other commands
done
# while 允许多个测试命令，但只有最后一个测试命令的退出状态码会决定是否要进入循环
# example
var1=10
while echo $var1
	[$var1 -ge 0]
do
	echo "This is inside the loop."
	var1=$[ $var1 - 1 ]
done
```

## `until`命令

```shell
until test commands # 与while相反，只要退出状态码非0，就一直执行循环
do 
	other commands
done

```

### 循环处理文本数据

`IFS` internal field separator，内部字段分隔符，是一个特殊的环境变量，定义了`bash shell`用作字段分隔符的一系列字符。它会将空格、制表符、换行符看做列表中一个新字段的开始。

如果想修改`IFS`的值，使其只能识别换行符，可以

```shell
IFS=$'\n'
```



# 20250827

一个测试案例

```shell
#!/bin/bash
# check system for popular package managers
# 这个脚本会看看你用的是什么包管理器（比如 yum、dnf、apt）、
# 用了哪些容器技术（比如 docker、podman），
# 然后判断你的系统更偏向 Red Hat 系（比如 RHEL、CentOS、Fedora）
# 还是 Debian 系（比如 Ubuntu、Debian）。
echo "  This script checks your linux system for popular"
echo "package managers and application containers, lists"
echo "what's available, and makes an educated guess on your"
echo "distribution's base distro (Red Hat or Debian)."
# Red Hat Checks
echo
echo "Checking for Rad Hat-based package managers & application containers..."
if (which rpm &> /dev/null) 
# &>：把前面命令的所有输出（包括正常输出和错误输出）都重定向到后面指定的位置。
# 如果只使用> 只能把标准输出重定向到 /dev/null，而标准错误依旧会打印到终端
# /dev/null：Linux 的“黑洞”，写入这里的内容都会被丢弃
# which 返回0 找到了 返回1 没找到
then
    item_rpm=1
    echo "You have the basic rpm utility."
else
    item_rpm=0
fi
if (which dnf &> /dev/null)
then 
    item_dnfyum=1
    echo "You have the dnf package manager."
elif (which yum &> /dev/null)
then
    item_dnfyum=1
    echo "You have the yum package manager."
else
    item_dnfyum=0
fi
if (which flatpak &> /dev/null)
then
    item_flatpak=1
    echo "You have the flatpak application container."
else    
    item_flatpak=0
fi
redhatscore=$[$item_rpm + $item_dnfyum + $item_flatpak]
# Debian Checks
echo 
echo "Checking for Debian-based package managers & application containers..."
case $(which dpkg &> /dev/null; echo $?) in 
0)
    item_dpkg=1
    echo "You have the basic dpkg utility.";;
1)
    item_dpkg=0;;
*)
    echo "error.";;
esac
if (which apt &> /dev/null)
then
    item_aptaptget=1
    echo "You have the apt package manager.";
elif (which apt-get &> /dev/null)
then
    item_aptaptget=1
    echo "You have the apt-get/apt-cache package manager."
else
    item_aptaptget=0
fi
if (which snap &> /dev/null)
then 
    item_snap=1
    echo "You have the snap application container."
else
    item_snap=0
fi
debianscore=$[$item_dpkg + $item_aptaptget + $item_snap]

# Determine Distro
echo
if [ $debianscore -gt $redhatscore ] # if 确实大于则返回0，if判定为真
then
    echo "Most likely your linux distribution is Debian-based."
elif [ $redhatscore -gt $debianscore ]
then    
    echo "Most likely your linux distribution is Red Hat-based."
else
    echo "Unable to determine Linux distribution base."
fi
echo
exit # 默认返回上一条命令的退出码，等价于 exit $?

```

