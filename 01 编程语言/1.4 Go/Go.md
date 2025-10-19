# 第1章 入门

`Go`是编译型语言。

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, 世界")
}
```

使用`go`命令配合子命令`run`对以`.go`为后缀的源文件进行编译、链接，然后生成可执行文件

```cmd
go run helloworld.go
```

如果该程序不是一次性的实验，我们可以通过`build`子命令生成名为`helloworld`的二进制程序

```cmd
go build helloworld.go
```



Go代码是使用包来组织的，包类似于库或者模块，一个包包含一个或多个.go源文件。



每一个源文件的开始都使用`package`声明，指明这个文件属于哪个包。

后面跟着它导入的其他包的列表。



`fmt`包中的函数用于**格式化输出和扫描输入**

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



`var`关键字声明了两个`string`类型的变量

变量可以声明时初始化。

如果没有明确初始化，则隐式初始化为该类型的空值，数字为0，字符串为空。



`os`包提供一些函数和变量，和操作系统打交道。

命令行参数以`os`包中`Args`名字的变量供程序访问，在`os`包外面，使用`os.Args`这个名字。

`os.Args`的数据类型是`[]string`字符串切片。

切片（`slice`）是 Go 语言中动态长度的序列容器，其特性与 C++ 中的 `vector<string>` 非常相似

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

```go
// 输出url获取的内容
package main

import (
	"fmt"      // 用于格式化输出和扫描输入
	"io"       // 处理各种i/o操作
	"net/http" // 用于实现 HTTP 客户端与服务端功能
	"os"
)

func main() {
	for _, url := range os.Args[1:] {
		resp, err := http.Get(url) //  resp是 HTTP 响应的结构体指针
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch: %v\n", err)
			os.Exit(1)
		}
		b, err := io.ReadAll(resp.Body)
		resp.Body.Close()
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch: reading %s: %v\n", url, err)
			os.Exit(1)
		}
		fmt.Printf("%s", b)
	}
}
```



并发获取多个`url`

```go
// 并发获取url并报告它们的时间和大小
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"time"
)

// main是主线程，go开辟的是子线程
func main() {
	start := time.Now()
	ch := make(chan string) // 内置函数make会默认初始化，创建一个字符串类型的通道
	for _, url := range os.Args[1:] {
		go fetch(url, ch) // 启动一个goroutine(轻量级线程)，运行fetch
	}
	for range os.Args[1:] {
		fmt.Println(<-ch) // 从通道ch接收
	}
	fmt.Printf("%.2fs elasped\n", time.Since(start).Seconds())
}

func fetch(url string, ch chan<- string) { // 浅拷贝
	start := time.Now()
	resp, err := http.Get(url)
	if err != nil {
		ch <- fmt.Sprint(err) // 发送操作 浅拷贝
		return
	}
	nbytes, err := io.Copy(io.Discard, resp.Body) // io.Discard，执行写操作，但是内部不做任何存储和输出处理
	resp.Body.Close()
	if err != nil {
		ch <- fmt.Sprintf("while reading %s: %v", url, err)
		return
	}
	secs := time.Since(start).Seconds()
	ch <- fmt.Sprintf("%.2fs %7d %s", secs, nbytes, url)
}
```

# 第2章 程序结构

---

## 2.1 名称



![image-20251007094012750](Go.assets/image-20251007094012750.png)



如果一个实体在函数中声明，则只在函数局部有效。

如果声明在函数外，它将对包里面的所有源文件可见。

实体的第一个字母的大小写决定其可见性是否跨包，大写字母开头表示是导出的，它对于包外是可见和可访问的，如`fmt`包中的`Printf`。



包名总是小写字母

Go程序员使用驼峰式风格

## 2.2 声明

Go程序存储在一个或多个以.go为后缀的文件里。

每个文件以`package`声明开头，表明文件属于哪个包。

`package`声明后面跟的是`import`声明。

然后是包级别的类型、变量、常量、函数的声明。（包级别的声明对于同一个包的所有源文件都可见）



## 2.3 变量

```go
// var 声明创建一个具体类型的变量，以下是通用形式
var name type(类型) = expression(表达式)
```

注：

1. 类型和表达式可以省略一个，但是不能都省略；
2. 若表达式省略，则变量的初始值将由**零值机制**保证，初始化为对应于类型的零值

```go
// example
var i, j, k int // int int int
var b, f, s = true, 2.3, "four" // bool float64 string
```

另，包级别的初始化在main之前进行，局部变量的初始化和声明在函数执行期间进行，很好理解。



### 短变量声明

自动推导类型

```go
// 语法
name := expression // name的类型将由expression决定
// example
f, err := os.Open(infile) // 短变量声明至少声明1个新变量
// 只有变量已经存在的情况下，短声明的行为才和赋值操作一样
```

### 指针

指针类型的零值是`nil`，表示未指向任何有效的内存地址



在Go中，返回局部变量的地址是安全的。



### new函数

创建变量的又一种方式。

```go
p := new(int) // p的类型是*int，指向未命名的int变量
```

new是一个内置的，预声明的函数，不是关键字



### 变量的生命周期



Q:垃圾回收器如何知道一个变量是否应该被回收？

每一个包级别的变量，以及每一个当前执行函数的局部变量，可以作为追溯该变量的路径的源头，通过指针和其他方式的引用可以找到该变量。

如果变量的路径不存在，那么变量变得不可访问，就应该垃圾回收了。



编译器可以选择使用堆或者栈上的空间来分配。

```go
var p = f()

func f() *int {
    v := 1
    return &v // p指向v，这时称变量逃逸出函数，编译器会自动将其分配在堆上，由垃圾回收器负责管理
}
```



## 2.5 类型声明

`type`声明定义一个新的命名类型，它和某个已有类型使用同样的底层类型。

```go
// 语法
type name underlying-type
// example
type Celsius float64
```

注：对于每个类型T，都有一个对应的类型转换操作T(x)将值x转换成类型T

## 2.6 包和文件

如果导入一个没有被引用的包，会触发错误。

包的初始化从初始化包级别的变量开始，变量又按照声明的顺序，在依赖已解析完毕的情况下，根据依赖的顺序进行初始化。



Go 中的 `init` 函数会在**包被导入时自动执行**，用于做一些初始化工作。

任何文件都可以包含任意数量的声明如下的函数

```go
func init { /* ... */ }
```

该函数不能被调用和被引用，它是普通函数。

当程序启动时，`init`函数按照它们生命的顺序自动执行。

# 第3章 基本数据

----

Go的数据类型分为四大类

- 基础类型（basic type)：数字、字符串（`string`）、布尔型
- 聚合类型（aggregate type)：数组、结构体
- 引用类型(reference type)：指针、$slice$、$map$、函数、通道
- 接口类型(interface type)



## 3.1 整数

Go支持有符号数和无符号数,如`int8`，`int16`，`int32`，`int64`，`uint8`。。。

`rune`类型是`int32`类型的同义词

`byte`类型是`unit8`类型的同义词



在Go中，尽管int天然就是32位，但是若要将int值当作int32使用，必须显示转换，反之亦然。



## 3.2 浮点数

`float32`和`float64`

`math`包中的`MaxFloat32`和`MaxFloat64`分别给出了浮点值的极限。

十进制下，前者的有效数字大约是6位，后者的大约是15位。



`math.IsNaN`函数判断其参数是否是非数值。

`math.NaN`函数返回非数值



## 3.3 复数

`complex64`和`complex128`

```go
var x complex128 = complex(1, 2) // 1 + 2i
```

`real`函数提取复数实部

`imag`函数提取复数虚部



## 3.4 布尔值

`true`和`false`

布尔值无法隐式转换成数值，反之也不行。



## 3.5 字符串

在`Go`语言中，字符串底层是只读的字节数组。切片操作不会复制原字符串的内容，而是会和原字符串共享同一块底层数组。该切片只记录起点和长度，因此操作十分高效。

但是，字符串的拼接操作会创建新的字符串。由于字符串的不可变性，拼接操作会重新计算字符串的长度，并为其分配新的内存。



与`c/cpp`不同，go中的字符串不以`\0`结尾。



字符串是**不可变**的字节序列。

这里的不可变意味着字符串内部的数据不允许更改。

内置的`len`函数返回字符串的字节数。例如

```go
s[0] = 'L' // 编译错误，s[0]无法赋值
```



子串生成操作`s[i: j]`，左闭右开区间，**和原串共用同一段底层内存**。

```go
s := "hello world"
sub := s[0:5] // sub是hello 与 s 共用底层内存
```



`Unicode`的理念是尽可能涵盖多的字符



其中，`utf-32`固定4字节编码，导致了不必要的存储空间消耗。

而`utf-8`兼容了ascii码，而其他常用的文书字符的编码只是2-3个字节。

Go的源文件总是以`utf-8`编码。



### 操作字符串常用的包

- `bytes`

用于操作字节类型的切片（字节类型的切片：动态长度的字节序列）

字节类型的切片，也即字节slice([]byte类型)，本身不存储数据，但包含3个部分：1）指向底层数组的指针；2）切片长度；3）切片容量



字符串和字节`slice`可以相互转换。

```go
s := "abc"
b := []byte(s) // 拷贝s含有的字节，为新的字节数组分配内存，编译器可能产生优化
s2 := string(b) // 也会拷贝
```



`bytes`包为处理`slice`提供了`Buffer`类型。

`Buffer`起初为空，动态扩容。





- `strings`

提供了许多函数，用于搜索，替换，比较，修整，切分与连接字符串



为了避免转换和不必要的内存分配，`bytes`包和`strings`包有许多函数

```go
// strings 包
func Contains(s, substr string) bool
func Count(s, sep string) int
func Fields(s string) []string
func HasPrefix(s, prefix string) bool
func Index(s, sep string) int
func Join(a, []string, sep string) string

// bytes包里面的对应函数
func Contains(b, subslice []byte) bool
func Count(s, sep []byte) int
func Fields(s []byte) [][]byte
func HasPrefix(s, prefix []byte) bool
func Index(s, sep []byte) int
func Join(a, [][]byte, sep []byte) []byte
```



- `strconv`

将布尔值、整数、浮点数转换成与之对应的字符串形式



整数转换为字符串

```go
x := 123
y := fmt.Sprintf("%d", x) // 第1种做法 不加换行
fmt.Println(y, strconv.Itoa(x)) // 123
```



将字符串类型的整数转换为整型

```go
x, err := strconv.Atoi("123") // x 是整型
y, err := strconv.ParseInt("123", 10, 64) // 十进制 最长为64位
```

- `unicode`



## 3.6 常量

在编译期计算表达式的值。

```go
const pi = 3.14159 // 在go语言中，类型推导机制使得即便没有提前声明，赋值运算符也可自行按照初始值推导其类型
// 类似地，还可以声明一系列常量
const (
	e = 2.718
    pi = 3.14159
)
```

常量生成器`iota`，默认从0开始递增。

### 无类型常量

go的常量自有其特殊之处。

虽然常量可以是任意数据类型，但是许多常量并不从属于某一具体类型。编译器讲这些从属类型待定的常量表示成某些值，这不仅可以使其具有更高的精度，而且还可以写进更多表达式中而无需转换类型（自动适配）。

如，一般的字面量默认是无类型常量。



只有常量才可以使无类型的。

```go
va f float64 = 3 + 0i // 无类型复数 -> float64
```



在变量声明中，如果没有显示指定类型，无类型常量会隐式转换成该变量的默认类型



# 第4章 复合数据类型

数组、`slice`、`map`和结构体。



Q：如何使用这些数据类型的结构化数据编码为`JSON`数据，从`JSON`数据转换为结构化数据，以及从模版生成`HTML`页面？

## 4.1 数组

```go
// 声明一个数组
var a [3]int // 索引从0到数组长度减1

// 默认情况下，一个新数组的元素初始值为元素类型的零值

var q [3]int{1, 2, 3} // 使用数组字面量初始化一个数组

var q [...]int{1, 2, 3} // ...表示数组的长度由初始化数组的元素个数决定

// 还可以这样，给出索引和索引对应的值，其余默认初始化
r := [...]int{99: -1} // 100个元素，最后1个元素值是1，其余被初始化为0
```

注意：与c/cpp一样的是，数组的长度必须在编译期确定。



如果一个数组的元素类型是可比较的，那么这个数组也是可比较的

```go
a := [2]int{1, 2}
b := [...]int{1, 2}
fmt.Println(a == b) // true
```



在go语言中，调用函数时，传入的参数是拷贝的副本，也即值传递。

Go中只有值传递。当传递数组的指针给函数时，也会产生拷贝。不过拷贝的指针仍然指向原始数组。



## 4.2 `slice`

`slice`表示一个拥有相同类型元素的可变长度的序列。

`slice`是一种**轻量级**的数据结构，它有三个属性：

1. 指针。指向底层数组的第一个可访问的元素。该元素不一定是数组的第一个元素。
2. 长度。`slice`中的元素个数。
3. 容量。

内置函数`len`和`cap`用来返回`slice`的长度和容量。



因为`slice`包含了指向数组元素的指针，所以将一个`slice`传递给函数时，可以修改底层数组。



和数组不同的是，`slice`无法做比较。

标准库仅对字节`slice`（[]byte)提供了函数`bytes.Equal`来比较两个字节。



Q：在Go语言中，为什么数组可以做比较，但是`slice`不可以？



数组长度固定，编译期可知，比较时可以逐元素对比，因此可以比较；

而`slice`是动态的，包含指针，长度和容量三个属性。是比较指针还是底层数组的元素，语义不确定。



`slice`类型的零值是`nil`，允许它和`nil`做比较。

值为`nil`的`slice`长度和容量都是0，没有对应的底层数组。



内置函数`make`可以创建一个具有指定元素类型、长度和容量的`slice`。其中，容量参数可以省略。在这种情况下，`slice`的长度和容量相等。

```go
make([]T, len)
make([]T, len, cap)
```

### append函数

将元素追加到`slice`后面。

`slice`可能是下面这种聚合类型。

```go
type IntSlice struct {
    ptr *int
    len, cap int
}
```

内置的`append`函数可以给`slice`添加多个元素

```go
var x []int
x = append(x, 1, 2, 3)
```

## 4.3 `map`

散列表。

键值对元素的无序集合。

键的值是唯一的。



在Go语言中，`map`是散列表的引用。`map`的类型是`map[k]v`，其中k和v是字典的键和值对应的数据类型。



键的类型可以用`==`进行比较，进而检测一个键是否存在。

```go
// 内置函数make可以用来创建一个map
ages := make(map[string]int)

// 还可以这样
map[string]int{}
```

可以使用内置函数`delete`从字典中根据键移除元素。及时键不在map中，操作也是安全的。

`map`中元素的迭代顺序也是不固定的，不同的实现方式会使用不同的散列算法。



如果需要按照某种顺序遍历map中的元素，可以显式地给键排序。

```go
import "sort" //Strings

var names []string 
for name := range ages { // ages是散列表
    names = append(names, name)
}

sort.Strings(names)

for _, name := range names {
    fmt.Printf("%s\t%d\n", name, ages[name])
}
```



`map`类型的零值是`nil`



访问不存在的键时，会返回该值类型的零值



## 4.4 结构体

