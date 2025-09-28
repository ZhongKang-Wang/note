# 第1章 入门

`Go`是编译型语言。

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, 世界")
}
```

使用go命令配合子命令run对以.go为后缀的源文件进行编译、链接，然后生成可执行文件

```cmd
go run helloworld.go
```

如果该程序不是一次性的实验，我们可以通过build子命令生成名为`helloworld`的二进制程序

```cmd
go build helloworld.go
```



Go代码是使用包来组织的，包类似于库或者模块，一个包包含一个或多个.go源文件。

每一个源文件的开始都使用`package`声明，指明这个文件属于哪个包。

后面跟着它导入的其他包的列表。



`fmt`包中的函数用于格式化输出和扫描输入

`Println`是`fmt`中一个基本的输出函数，它输出一个或多个用空格分割的值，结尾自动添加换行符。



名为`main`的包用来定义一个可执行程序。

在`main`包中，函数`main`是特殊的，是程序开始执行的地方。



注意：必须精确地导入需要的包。缺失导入或者存在不需要的包，编译都会失败。



`import`声明必须跟在`package`声明之后。

`import`导入声明后面，是组成程序的函数、变量、常量、类型（以`func`、`var`、`const`、`type`开头）声明。



`gofmt`工具将代码以标准格式重写，go工具的`fmt`子命令使用`gofmt`工具来格式化指定包里的所有文件或者当前文件夹中的文件（默认情况下）。



```go
// echo1 输出其命令行参数
package main

import (
	"fmt"
	"os"
)

func main() {
	var s, sep string
	for i := 1; i < len(os.Args); i++ { // go语言只支持后缀
		s += sep + os.Args[i]
		sep = " "
	}
	fmt.Println(s)
}
```

在一个包声明前，使用注释对其进行描述

`var`关键字声明了两个`string`类型的变量

变量可以声明时初始化。如果没有明确初始化，则隐式初始化为该类型的空值，数字为0，字符串为空。



`os`包提供一些函数和变量，和操作系统打交道。命令行参数以`os`包中`Args`名字的变量供程序访问，在`os`包外面，使用`os.Args`这个名字。

`oa.Args`是一个字符串`slice`。

```go
// echo2 输出其命令行参数
package main

import (
	"fmt"
	"os"
)

func main() {
	s, sep := "", ""
	// go不允许存在无用的临时变量
	for _, arg := range os.Args[1:] { // 每一次迭代，range产生一对值，索引和当前索引对应的元素
		s += sep + arg
		sep = " "
	}
    fmt.Println(s) // 使用%v(内置格式的任何值)格式化参数，追加换行符
}
```



撰写性能评估测试





```go
// dup1 输出标准输入中出现次数大于1的行，前面是次数
package main

import (
	"bufio" // 处理输入和输出
	"fmt"
	"os"
)

func main() {
	counts := make(map[string]int) // 内置的make函数可以新建map,map存储一个键值对集合
	input := bufio.NewScanner(os.Stdin)
	// os.Stdin是os包中的变量，类型是*os.File(文件指针)
	// bufio.NewScanner()创建一个带缓冲的输入处理器
	// 从os.Stdin按行读取数据，默认以换行符分割，返回值是指向 bufio.Scanner 结构体的指针
	for input.Scan() { // 返回bool值
		counts[input.Text()]++  // input.Text()返回当前扫描到的字符串
		// 等价于 line := input.Text()
		// counts[line] = counts[line] + 1
	}
	for line, n := range counts { // range是关键字 每次迭代得到键和值，键的迭代顺序是随机的
		if n > 1 {
			fmt.Printf("%d\t%s\n", n, line) //函数Printf默认不写换行符
		}
	}
}
```





```go
// dup2 打印输入中多次出现的行的个数和文本
// 从stdin 或 指定的文件列表中读取输入
package main

import (
	"bufio" // 处理输入和输出
	"fmt"
	"os"
)

func main() {
	counts := make(map[string]int) // 内置的make函数可以新建map,map存储一个键值对集合
	files := os.Args[1:] // []string（字符串切片)，切片中的每个元素都是字符串
	if len(files) == 0 {
		countLines(os.Stdin, counts)
	} else {
		for _, arg := range files {
			f, err := os.Open(arg) // f是*os.File 类型，即指向文件的指针
			// err是 error 接口类型 
			if err != nil { // nil是Go中的空值标识符，表示没有错误
				fmt.Fprintf(os.Stderr, "dup2: %v\n", err) //输出目标是标准错误流，自动调用err.Error()方法，输出一串字符串
				continue
			}
			countLines(f, counts)
			f.Close()
		}
	}
	for line, n := range counts { // range是关键字 每次迭代得到键和值，键的迭代顺序是随机的
		if n > 1 {
			fmt.Printf("%d\t%s\n", n, line)
		}
	}
}
	// 函数无需提前声明，定义后直接调用，map在go中是引用类型
func countLines(f *os.File, counts map[string]int) {
	input := bufio.NewScanner(f)
	for input.Scan() {
		counts[input.Text()]++
	}
}
```



小练习：获取一个`url`

