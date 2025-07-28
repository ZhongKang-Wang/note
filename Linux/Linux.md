# 20240226



在虚拟机中安装了一个叫ubuntu的操作系统，基于 Linux 内核。然后电脑可以和虚拟机共享文件夹。

路径是`files->mnt->hgfs->Share`



## 计算机基础知识

在高级语言中，C，C++是编译性语言，Python是解释性语言，Java介于两者之间。

- 编译性语言
  - 特点：
    - 源程序编译后即可在平台上运行，运行速度快
    - 针对不同的平台需要使用对应的编译器。编译器可以一次性编译成可在该平台上运行的机器码，从而把高级语言包装成可执行程序的格式。
    - 因此，在其他平台使用，要考虑到移植的问题
- 解释性语言
  - 特点：
    - 每次运行期间都要将源代码解释成机器码并执行，虽然效率比较低，但跨平台性好，书写也更简单
    - 不同的平台只要提供相应的解释器，就可以运行源代码，所以移植比较方便。

linux快捷键

- ctrl+alt+t 开启新终端
- ctrl+alt+回车 虚拟机全屏
- clear 终端清屏
- ctrl+shift+ 加号 终端字体变大
- ctrl+alt 鼠标退出虚拟机控制


# 20250303

linux系统下的代码编辑器：Vim

编译器：GCC



## vim编辑器基本操作



命令模式 `vim hello.c `默认打开，只能复制粘贴

插入模式 `i`(insert缩写)，就可以书写代码了

退出插入模式，按下`Esc`，之后按下`shift`+`:`可以使用以下指令：

w 保存

q 退出

a 附加



-  vim切换命令

  - `a` append进入编辑状态，从当前光标的位置之后插入键盘输入的字符
  - `i` insert...,从当前光标之前的位置开始插入
  - `o`...,open插入一新行，光标移到该新行行首，以后键盘输入的字符插入到光标位置
  - `esc` 进入命令
- 拷贝与粘贴命令

  - u (undo) 撤销上一次操作



# 20250605

## `vimtutor zh` 帮助手册

`：q!` 退出，任何改动都丢弃

`: wq` 保存并退出

`x` 用于删除字符

`i` 用于插入字符

`A` 插入文本

`dw` 将光标停靠在一个单词的开头，可以删除该单词

`d$` 可以从当前光标删除直至行末



## 命令和对象

许多改变文本的命令都由一个==操作符==和一个==动作==构成。

举个例子

```shell
//d是删除操作符
//d motion
//motion可以是w/e/$
```

在动作前输入数字可以让这个动作重复执行好几次，如`d [number] motion`

`dd` 可以删除整行

类似地，使用`2dd`可以删除两行。

`u` 可以撤销最后一次动作

`p` 可以在光标下插入最后一次删除的内容

`r`+`替换字符` 可以将当前光标字符替换掉

`ce` 更改文本直到一个单词的末尾

类似地，使用`c$`可以删除一行后面的所有内容，然后进行插入

`gg` 跳转到文件第一行

`G` 跳转到文件最后一行

`Ctrl`+`g` 显示当前文件位置和文件名称，以及当前光标行号（输入挺溜的行号，再输入`g`可跳转到指定行）

`/`+字符串 光标可以跳转到该字符串所在行（按`n`可以继续向下查找，大写`N`反向查找）

`%` 光标定位在（，输入%可以找到与之匹配的）



这么多命令我咋记得住？？？



## GCC编译器的使用

`gcc hello.c`编译代码，系统会默认在当前目录下生成a.out的文件

`./a.out`也即执行`a.out`文件，输出对应结果

`gcc hello.c -o exec` 编译代码，用户自定义生成的可执行文件的名字

`./exec`执行文件

gcc编译过程

- 预处理生成.i

- 编译生成.s

- 汇编生成.o

- 链接生成可执行文件.out

  

# 20250317

Linux 命令通用格式

```
command [-options] [parameter]
```

- command 命令本身
- -options 可选的选项
- parameter 可选的参数

示例：

```
ls -l /home/itheima //以列表形式，显示/home/itheima内的内容
```





#### ls

```
ls [-a -l -h] [linux命令]
//ls 以平铺形式，列出当前工作目录中的内容
//当前工作目录 linux在启动时默认加载当前登陆用户的home目录

-a //以.开头的，表示linux系统中的隐藏文件/文件夹，只有-a才能看到
-l //以列表(竖向排列)的形式展示内容，展示更多信息
-h //以易于阅读的形式，列出文件大小，如K、M、G，-h必须搭配-l一起使用
```



#### cd

```
cd [linux路径] //change directory
直接执行cd,表示回到用户的Home目录
```



#### pwd 查看当前工作目录

```
pwd //print work directory
```



#### 相对路径 和 绝对路径

```
cd /home/linux/itheima/Desktop //绝对路径：以根目录为起点
cd ./itheima/Desktop //相对路径：以当前目录为起点 eg, itheima/Desktop
//特殊路径符
. //表示当前目录 cd ./Desktop 切换到当前目录下的Desktop目录内
.. //表示上一级目录 cd ../..可以切换到上二级目录
~ //表示home目录
```





# 20250318

## Linux 基础命令

#### mkdir

```
mkdir [-p] linux路径 //Make directory [-p] 是可选的，英文原型是parents
//通过-p选项，可以一次性创建多个层级的目录
```





#### touch 创建文件

```
touch linux路径
```



#### cat 查看文件内容

```
cat linux路径
```



#### more 查看文件内容

```
more linux路径
//在查看的过程中，通过空格翻页
//按q退出
```



不同点：

- cat直接将内容全部显示出来
- more支持翻页，如果文件内容过多，可以一页一页展示





#### cp 复制文件/文件夹

```
cp [-r] 参数1 参数2
//-r 可选，用于复制文件夹使用，表示递归recursive
//参数1，linux路径，表示被复制的文件或文件夹
//参数2，linux路径，表示要复制去的地方
```





#### mv 移动文件/文件夹

```
mv 参数1 参数2
//参数1，linux路径，表示被移动的文件或文件夹
//参数2，linux路径，表示要移动去的地方，如果目标不存在，则创建
```





#### rm 删除文件/文件夹

```
rm [-r -f] 参数1 参数2 参数3 ... 
//-r 用于删除文件夹
//-f 表示强制删除force
//参数1，2...表示要删除的文件或者文件路径，按照空格分隔
```

`rm`支持通配符，用来做模糊匹配

- 符号*表示通配符，即包含任意内容（包括空）
- example
  - test* 匹配任何以test打头的内容
  - *test 匹配任何以test结尾的内容
  - 左一个test右一个, 表示匹配任何包含test的内容



#### 切换到root用户

可以通过su - root 输入密码临时切换到root用户体验superuser

通过输入exit命令，退回普通用户



#### which 查找命令

```
which 要查找的命令
```

可以通过which命令，查看这些命令的二进制可执行程序存放在哪里



#### find 按文件名/文件大小查找文件

```
find 起始路径 -name "被查找文件名"
//被查找文件名也支持*做模糊匹配
find 起始路径 -size + - n[k M G]
//+\-表示大于或小于
//n表示大小数字
//kMG表示大小单位,k表示kb,M表示MB，G表示GB
eg.
	查找小于10kB的文件 find / -size -10k
	查找大于100MB的文件 find / -size +100M
```



#### grep

通过grep命令，从文件中通过关键字过滤文件行 globally search a regular expression and print

```
grep [-n] 关键字 文件路径
//-n 可选 表示在结果中显示匹配的行号
//关键字，必填， 如果关键字包含空格或者其他符号，建议用""包围
//文件路径，表示要过滤内容的文件路径
```





#### wc 做数量统计

```
wc [-c -m -l -w] 文件路径
//-c 统计bytes数量
//-m 统计字符数量
//-l 统计行数
//-w 统计单词数量
```





#### 管道符 |

将管道符左边命令的结果，作为右边命令的输入



#### echo

可以使用echo命令在命令行内输出指定内容

```
echo 输出的内容 //如果输出内容复杂，可以用“”
```



#### 反引号 `

```
echo `pwd` //本想输出当前工作路径，但是pwd被当作普通字符输出，反引号包括的内容将被当作命令执行
```



#### 重定向符 > 和 >>

- ... > 将左侧命令的结果，**==覆盖==**写入到符号右侧指定的文件中
- ... >> 将左侧命令的结果，**==追加==**写入到符号右侧指定的文件中

#### tail

使用tail命令，可以查看文件尾部内容，跟踪文件的最新修改

```
tail [-f -num] linux路径
//linux路径，表示被跟踪的文件路径
//-f，表示持续跟踪
//-num,表示查看尾部多少行，不填默认10行
```





# 20250319

## Vim 编辑器

vi/vim ,即visual interface, 是Linux中最经典的文本编辑器

vim是vi的加强版，兼容vi的所有指令，而且具有shell程序编辑的功能

#### vi/vim编辑器的三种模式

- 命令模式：

  - 如果通过vi/vim编辑器编辑文件

  - ```
    vi 文件路径
    vim 文件路径 //如果文件路径下的文件不存在，则创建新的；若存在，则编辑
    ```

- 命令

  - i 在当前光标位置进入 输入模式
  - a 在当前光标位置之后 进入输入模式
  - I 在当前行的开始
  - A 在当前行的结尾
  - o 在当前光标下一行
  - O 在当前光标上一行
  - k 向上移动光标
  - j 向下
  - h 向左
  - l 向右
  - 0 光标移动到当前行开头
  - $ 光标移动到当前行结尾
  - PgUp 向上翻页
  - / 进入搜索模式
  - n 向下继续搜索
  - N 向上继续搜索
  - dd 删掉光标所在行的内容
  - ndd 删除当前光标向下n行
  - yy 复制当前行
  - nyy 复制当前行和下面n行
  - p 粘贴复制的内容
  - u 撤销修改
  - ctrl+r 反向撤销修改
  - gg 跳到首行
  - dG 从当前行开始，向下全部删除
  - dgg 从当前行开始，向上全部删除
  - d$ 从当前光标开始，删除到本行结尾
  - d0 从当前光标开始，删除到本行开头

- 输入模式：编辑模式，可以对文件内容自由编辑

  - 在输入模式下，按下esc，回到命令模式

- 底线模式：通常用于文件的保存、退出

  - ：wq 保存并退出
  - ：q 仅退出
  - ：q! 强制退出且不保存
  - ：w 仅保存
  - ：set nu 显示行号
  - ：set paste 设置粘贴模式





#### linux的root用户

root用户，超级管理员

切换root账户

```
su [-] [用户名]
//-可选
//用户名，表示要切换的用户，用户名省略时表示切换到root
//切换用户后，通过exit命令退回上一个用户，也可使用快捷键：ctrl+d
```



我们可以使用sudo(superuser do)命令，为普通命令授权，临时以root身份运行

```
sudo 其他命令 //并不是所有的用户都有权利使用sudo，我们需要为普通用户授权
```

为普通用户授权sudo

- 切换到root用户

- ```
  chmod u+w /etc/sudoers //给root用户增加w权限
  vim /etc/sudoers //找到root ALL=ALL ALL下面一行
  //添加
  linux ALL=(ALL)		NOPASSWD:ALL
  //保存并退出
  ```

- 回复sudoers文件原来的读写权限

- ```
  chmod u-w /etc/sudoers
  ```

- 退出root用户

#### 用户、用户组

linux关于权限管控的级别有2个

- 针对用户的权限控制
- 针对用户组的权限控制

用户组管理需要root用户执行

```
groupadd 用户组名 //创建用户组名
groupdel 用户组名 //删除用户组名

useradd [-g -d] 用户名 //创建用户
//-g 指定用户的组，不指定-g，会创建同名组并自动加入
```



# 20250411

动态链接库(dynamically linked library, DLL)

程序运行的时候，这些库才会被链接并加载。



# 20250614

在C语言中，`main`的函数原型可能长这样：

> int main(int argc, char* *argv)

- **`argc` (argument count)**: 表示命令行参数的数量，包括程序名本身。因此，`argc` 至少为 1。
- **`argv` (argument vector)**: 是一个指向字符串数组的指针，其中每个字符串是一个命令行参数。数组的第一个元素（即 `argv[0]`）通常是存放该程序的路径。接下来的元素是传递给程序的命令行参数。





在Linux系统中，文件和目录都有两个属性：`所有者（Owner）`和`组（Group）`。这两个属性用于控制文件和目录的访问权限。

### 账号名（Owner）

- **账号名**指的是文件或目录的所有者，即哪个用户拥有对该文件或目录的完全控制权。
- 所有者可以读取、写入、执行文件，或者更改文件的权限。
- 文件或目录的所有者通常是由创建该文件或目录的用户决定的。

### 组名（Group）

- **组名**指的是文件或目录所属的用户组。用户组是系统中用户的一种集合，通常用于权限管理。
- 文件或目录的组可以包含多个用户，这些用户对该文件或目录有一定的访问权限，但这些权限通常不如所有者权限高。
- 组权限通常用于在多个用户之间共享文件或目录，而不需要将每个用户都设置为所有者。





# 20250615

Linux是一个操作系统的内核。

shell 壳



在Linux系统中，可执行文件没有统一的后缀，系统根据文件属性来区分可执行文件和不可执行文件。

gcc编译器语法

`gcc [选项] 准备编译的文件 [选项] 目标文件`

example

`gcc test.c -o test`

用gcc编译c++

`gcc hello.cpp -lstdc++ -o test`

用g++编译c++更简单

`g++ hello.cpp -o test`







openssh是一款加密通信协议软件。

ssh是一种用于远程访问和文件传输的网络安全协议。



如何让主机和虚拟机进行通信？在windows下编辑好的源程序如何发送到linux?





在Linux操作系统中，用gdb调试程序。

`gcc -g test.c -o test`







# 20250619

C语言把所有设备都当作文件，这点和linux很类似。

| 标准文件 | 文件指针 | 设备     |
| -------- | -------- | -------- |
| 标准输入 | stdin    | 键盘     |
| 标准输出 | stdout   | 屏幕     |
| 标准错误 | stderr   | 您的屏幕 |

以上3个文件会在程序执行时自动打开。

printf()函数原型

`int printf(const char* format, ...);`

printf()默认输出到屏幕，scanf()默认输入为键盘

字符串输入函数fgets()，函数原型

`char* fgets(char* str, int n, FILE* stream);`

- str是指向字符数组的指针
- n表示读取的最大字符数
- stream是文件流，

example

`fgets(str, sizeof(str), stdin);`

从键盘读取字符串



C I/O编程

打开文件

`FILE* fopen(const char* filename, const char* mode);`

- filename是字符串，用来命名文件
- mode是访问模式
  - r/w/a(append)/r+(读写)/w+/a+



关闭文件

`int fclose(FILE* fp);`





Linux系统调用下的文件I/O编程

基于C

打开/创建文件

```c
#include <fcntl.h> //file control
int open(const char* pathname, int flags);
int open(const char* pathname, int flags, mode_t mode);
int creat(const char* pathname, mode_t mode);
```

- pathname 可以是绝对路径或者相对路径
- flags
  - O_CREAT 如果文件不存在，就创建
  - O_TRUNC 如果文件存在，就截断
  - O_APPEND
  - O_RDONLY
  - O_WRONLY
  - O_RDWR
  - 可用`|`运算
- mode只有在创建文件时才使用，用于指定文件的访问权限（使用权限宏需要包含sys/stat.h头文件）



我理解了，只有明确声明`O_CREAT`时，open才会在文件不存在时创建文件，此时第三个参数用于指明这个文件的权限。







linux命令

查询linux IP地址

`ifconfig`

在vscode编辑器上编辑好的文件通过sftp插件可以实现远程传输到linux固定目录中。



SFTP（Secure File Transfer Protocol）是一种安全的文件传输协议。它利用SSH（Secure Shell）提供的安全机制来传输文件。SSH为SFTP提供了加密的通信通道，确保文件传输过程中的数据安全，包括文件内容、文件名等信息都是经过加密的，防止被窃听和篡改。





查看linux上ssh协议是否处于运行状态

`sudo systemctl status ssh`

若没有，启动ssh

`sudo systemctl start ssh`



**`sudo dhclient ens37`**

- 这是通过 DHCP（动态主机配置协议）向路由器请求分配 IP 地址的命令。
- 执行后，`ens37`接口状态变为`UP`，并获取到了`192.168.248.135`这个内网 IP，此时网络才真正连通。





# 20250627

## 文件锁定

当多个用户同时操作文件时，需要给文件上锁。

文件锁

- 建议性锁：只在文件上设置一个锁的标识，可以让其他进程检测到
- 强制性锁：
- 对文件上锁是原子性的

```c++
#include <unistd.h>
#include <fcntl.h> //file control
int fcntl(int fd, int cmd, struct flock* lock); 
```

- fd：文件描述符

- cmd：操作命令

  

# 20250702

## 多进程编程

进程标识符`PID`

每个进程都有一个唯一的进程标识符，在程序中类型为`pid_t`，被定义在`sys/type.h`头文件中，虽然底层是`int`。

```c++
#include <unistd.h>
pid_t getpid(void) // 获取当前进程的pid
```

### 创建进程

#### `fork()`

```c++
#include <unistd.h>
pid_t fork();
```

- 初始状态：父子共享同一片物理地址，父子权限都对这片物理地址都是只读状态
- 写入时：任何一方写入时触发内存复制，复制后各自拥有独立副本（==写时复制==）

example

```c++
#include <iostream>
using namespace std;

#include <unistd.h>
#include <stdio.h>
int main() {
    pid_t fpid;
    int count = 0;
    fpid = fork(); // 向子进程返回0， 向父进程返回子进程的PID，否则向父进程返回-1表示子进程创建失败
    if (fpid < 0) // 创建失败
        cout << "failed to fork." << endl;
    else if (fpid == 0) { // 子进程
        cout << "I'm the child process." << endl;
        cout << "my pid is " << getpid() << endl;
        ++count;
    }
    else { // 父进程
        cout << "I'm the parent process." << endl;
        cout << "my pid is " << getpid() << endl;
        cout << "fpid =" << fpid << endl;
        ++count;
    }
    printf("result: %d\n", count);
    return 0;
}
```

下面是我在linux上的输出

```c++
I'm the parent process. // 父进程先执行
I'm the child process. // 子进程被调度执行（并发性）
my pid is 3110 // 父进程继续执行，这是父进程自己的PID
fpid = 3111 // 这是父进程输出的 子进程的PID
result: 1  // 这是父进程中的count，这个时候应该已经触发了写时复制，
my pid is 3111 // 子进程的PID
result: 1 // 子进程结束
```

#### `exec()`

```c++
#include <unistd.h>
int execl(const char* path, coonst char* arg, ...);
int execlp(const char *file, const char *arg0, ...); 
// execlp不需要写出完整路径，它回到环境变量PATH的路径中去找
```

Q：同样是创建进程，`fork()`和`exec()`区别在哪里?

- fork()创建新的进程，会产生一个新的PID
- 而exec()创建的进程会占据原来的进程，PID不变



example

```c++
#include <unistd.h>
int main() {
    // 第二个参数随便编的，因为执行pwd命令后面啥也不跟
    execl("/bin/pwd", "asdfaf", NULL); // 但是最后一个参数必须用NULL表示参数列表结束
    return 0;
}
```



```c++
execl("/bin/ls", "abds", "-al", "./test", NULL); //还可以这样，将当前进程替换掉
//执行命令ls -al ./test
```



这里写一下我对main中指针的理解。

在main函数中，有这样两种写法。

```c++
int main(int argc, char* argv[])
```

- argc表示参数的数目
- argv则是一个char*数组，数组中存放的都是指向char型的指针。

另一种写法和上述写法等价。

```c++
int main(int argc, char** argv)
```

- argv是一个指向char*的指针

```plaintext
argv (char**) ────┐
                  │
                  ▼
+───────────────────────────+
│ 内存区域（char* 数组）    │
├───────────────────────────┤
│ [0]: 指向 "./a.out" 的指针 │
│ [1]: 指向 "hello" 的指针   │
│ [2]: NULL                 │
+───────────────────────────+
```

#### `system()`

```c++
#include <stdlib.h>
int system(const char* command);
```

通过调用Shell程序来执行传入的命令

特点：

- 源进程和子进程各自进行
- 源进程需要等子进程运行完后再运行



Linux采用的是`基于优先级可抢占式的调度系统`

这个抢占仅限于运行在用户态下的进程。



Linux调度程序采用两种优先级

- 静态优先级：只针对实时进程，范围1-99
  - 针对实时进程，有两种调度策略
    - 先入先出
    - 时间片轮转
- 动态~：应用于普通进程



## 进程的分类

- 前台进程/普通进程：需要和用户交互的进程

- 后台进程

  对于不需要交互的进程，我们希望它在后台默默启动，可以这样做。

  `./test &`

  我们把切换到后台运行的进程称为`job`



- 守护进程：运行在后台的一种特殊进程。它==独立于控制终端==且周期性地执行某种任务或等待处理某些发生的事件。

  - 特点

    - 具有超级用户的权限

    - 父进程是`init`进程

      



# 20250704 

## Linux进程间的通信

### 信号

在软件层次上可以理解为中断。

Linux使用信号的目的：

- 让进程意识到发生了一个特定的事件
- 迫使进程执行信号处理程序



针对信号，进程可以采取如下手段：

- 忽略信号：有两个信号不能被忽略，他们分别是`SIGKILL`和`SIGSTOP`
- 执行相应的处理程序



#### 信号的分类

- 实时/可靠信号

- 非实时/不可靠信号

区别在于：实时信号不支持排队，否则信号可能丢失



```c++
kill [参数] [进程号]
```



子进程退出后会变成`僵尸进程`，占用少量系统资源；直到父进程获取子进程的状态后，`僵尸进程`才会消亡。

- **僵尸进程**：子进程执行完毕后，父进程未调用`wait()`或`waitpid()`获取其退出状态，子进程的 PCB（进程控制块）仍保留在系统中，成为僵尸进程。它已无实际运行代码，仅占少量资源，需父进程处理或父进程退出后由 init/systemd 回收。
- **孤儿进程**：父进程先于子进程退出，子进程失去父进程，此时会被 init 进程（PID=1）或 systemd 收养，成为孤儿进程。它仍能正常运行，退出后由收养它的进程处理退出状态，不会占用额外资源。



# 20250714

感觉这本书讲得一般，读起来很生硬，晦涩难懂。



## 管道

连接读进程和写进程，以实现通信的文件共享。

管道是单向的，先进先出，环形队列，写进程写入管道一端，读进程从管道另一端读取数据，且数据一旦被读走，便不再管道中存在。

在Linux中，管道是一个固定大小的缓冲区。



为了协调通信双方，管道需要提供读写进程间的同步机制

- 管道已满，应该阻塞写进程
- 管道为空，应该阻塞读进程



在Shell中，|表示管道。



IPC Inter process communication



## 消息队列

提供了一种让两种不相关的进程之间通信的方法。





# 20250728

## 多线程基本编程

线程的分离状态

POSIX下的线程要么是可分离的，要么是连接的。

- 可分离的线程运行结束后资源立刻被系统回收
- 可连接的线程在自己退出或者`pthread_exit()`时`不会主动释放资源`，需要在主线程中使用`pthread_join`函数等待线程结束后才会被释放



线程的调度策略

- 先来先服务
- 时间片轮转：支持优先级/不支持





## C++ 多线程编程

| 头文件             | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| atomic             | 声明了atomic和atomic_flag两个类，以及C风格的原子类型和原子操作的函数 |
| thread             | thread类                                                     |
| mutex              | 互斥锁相关                                                   |
| condition_variable | 与条件变量相关的类                                           |
| future             |                                                              |
|                    |                                                              |



