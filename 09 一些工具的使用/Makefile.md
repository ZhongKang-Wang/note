# 20250825

Q1：什么是`makefile`？`makefile`有什么用？

`Makefile` 是一个程序构建工具，它可以帮助我们自动化编译编写好的源代码或者程序。例如，当我们要用`gcc`编译、链接多个文件时，就要在命令行写很长一串命令，太麻烦了。



# 20250828

```makefile
x.txt: m.txt c.txt
	cat m.txt c.txt > x.txt # 命令要以tab键开头
m.txt: a.txt b.txt
	cat a.txt b.txt > m.txt # 把默认执行的规则放第一条，其他规则的顺序是无关紧要的，因为make执行时自动判断依赖
.PHONY: clean # 此时，clean就不被视为一个文件，而是伪目标（Phony Target）
clean:
	rm -f m.txt # 强制删除中间文件
```

注意：要执行`clean`，就必须使用命令`make clean`

因为我们并没有创建一个名为`clean`的文件，所以，每次运行`make clean`，都会执行这个规则的命令。

但是如果我们手动创建一个`clean`的文件，这个`clean`规则就不会执行，解决办法是在`clean`前面一行添加`.PHONY: clean`



example

```makefile
cd:
	pwd
	cd ..
	pwd
```

当执行`make cd`命令时，`cd ..`不会改变当前目录。原因是针对每条命令，都会创建一个独立的Shell环境，解决办法是写成一行，并在每条命令后加`;`。

为了便于预览，还可以在`;`后面添加`\`。





`make`在默认情况下会打印执行的每一条命令。如果我们不需要打印，就在命令前面加`@`