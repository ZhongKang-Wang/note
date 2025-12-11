虚函数机制：执行期绑定

在C语言中，`数据`和处理数据的操作（`函数`）是分开声明的。

# 第1章 关于对象

## C++对象模型

在C++中，有两种class data members，分别是`static`和`nonstatic`。以及三种class member functions，分别是`static`，`nonstatic`，`virtual`。

```c++
class Point {
    public:
    	Point(float xval);
    	virtual ~Point();
    	float x() const;
    	static int PointCount();
    private:
    	virtual ostream& print(ostream& os) const;
    	float _x;
    	static int _point_count;
}
```

像上述这样一个类会在机器中如何表现呢？

1. 为了支持`virtual functions`，每个类产生一个指向`virtual functions`的指针，放在表格中，这个表格称为`virtual table`（`vtbl`)
2. 每一个类对象各有一个指针，指向同一类的`virtual table`，这个指针被称为`vptr`



![image-20251107094728794](深度探索C++对象模型.assets/image-20251107094728794.png)

class object（类对象）只存储`nonstatic data members`

```plaintext
class object
-----------
nonstatic data members
virtual pointer ->指向相关的virtual table

virtual table 存放的都是virtual functions 的指针
```



virtual 虚拟，也是共享的意思。



## 关键词struct和class

主要区别：struct内成员默认访问权限为public，而class的默认访问权限是private

## 多态

C++以下列方法支持多态：

1. 经由一组隐式的转化操作。例如把一个`derived class`指针转化为一个指向其`public base type`的指针。

```c++
shape* ps = new circle(); //很明显，是基类指针指向子类对象
```

2. 经由`virtual function`机制

```c++
ps->rotate();
```

3. 经由`dynamic_cast`和`typeid`运算符

```c++
if (circle* pc = dynamic_cast<circle*>(ps))...
```



# 第2章 构造函数

## 默认构造

```c++
class Foo {
public:
    int val;
    Foo *pnext; // 内置类型如int,指针默认不初始化
};

void foo_bar() { 
    Foo bar; // 由于没有定义构造函数，编译器会默认生成一个，但是这个构造函数什么也不做，所以编译不会报错，但是运行时未定义行为
    if (bar.val || bar.pnext)
        // do something
}
```

Q:编译器在什么样的情况下才会生成默认构造函数？

仅当用户没有为类显示定义任何构造函数，且类内所有非静态成员都能被默认初始化。

举个反例

```c++
class B {
public:
    B(int) {}; // B不存在默认构造函数
};
class A {
public:
    B b; // 尝试用B的默认构造函数初始化b
};
int main() {
    A a; // 编译错误！
    return 0;
}
```



 下面是4种`nontrivial default constructor`，有用的默认构造

### 带有 Default Constructor 的 Member Class Object

如果一个`class`没有任何`constructor`，但它内含一个`member object`，而后者有`default constructor`，那么这个`class`的`immplicit default constructor`就是有用的(书中用"nontrivial")。

编译器需要为此class合成出一个default constructor，不过这个合成操作只有在constructor真正需要被调用时才会发生。 

```c++
class Foo {
public:
    Foo();
    Foo(int);
};
class Bar {
public:
    Foo foo;
    char* str;
};
void foo_bar() {
    Bar bar; // 成员foo将会在编译器隐式生成的默认构造中初始化
    // 但是成员str将会是未定义的垃圾值，非法访问属于未定义行为
    if (bar.str) {}...
}
```

如果用户显式定义了构造函数，那么编译器会扩张已存在的constructors，使得user code被执行前，先调用必要的default constructors。

按照成员对象在类中的声明顺序初始化。

例如

```c++
class Dopey {public: Dopey(); ...};
class Sneezy {public: Sneezy(int); Sneezy(); ...};
class Bashful {public: Bashful(); ...};
class Snow_Write {
public:
    Dopey dopey;
    Sneezy sneezy;
    Bashful bashful;
    ...
private:
    int mumble;
}
// 如果没有定义默认构造，编译器就会合成一个有用的默认构造，依次调用上述三个类的默认构造函数，而mumble如果程序员不初始化，那么值将是未定义的。
```



### 带有 `Default Constructor` 的 Base Class

如果没有任何构造函数的类，派生自一个带有默认构造函数的基类，那么编译器为派生类隐式生成的默认构造函数也不是无用的。

它将调用基类的默认构造函数。



### 带有 Virtual Function 的 Class



下面两种情况也需要合成一个default constructor(nontrivial)

1. class 声明（或继承）一个virtual function。
2. class派生自一个继承串链，其中有一个或更多的virtual base classes

这个主要是考虑到带虚函数的对象内保存有一个指向虚函数表的指针，编译器需要保证每个带虚函数的类对象其虚函数指针指向正确的地址。



### 带有 Virtual Base Class 的 Class

菱形继承

```c++
class X {public: int i; };
class A: public virtual X {public: int j;};
class B: public virtual X {public: int d;};
class C: public A, public B {public: int k;};
// if
class X;
class A;
class B;
class C;
```

可能的内存布局

 ```plaintext
 ------- // X的起始地址 假设是1000
 int i
 --------
 
 													  virtual table 假设是 2000
 -------  1004 // A的起始地址 假设机器是32位的				------------
 vbptr A // 虚指针放在对象起始位置能让编译器快速定位 -> 1000（X的起始地址）-1004（A的起始地址）=-4
 ------- 1008											-----------
 int j
 -------
 
 													virtual table 假设是 2004
 ------- 1012 B的起始地址  								-----------					
 vbptr B ------------------------------------>    1000 - 1012 = -12
 ------- 1016 									--------------------
 int d
 -------
 
 
 ------- 1020 C
 vbptr A    存的是 2000
 ------
 int j
 -------
 vbptr B    存的是 2004
 -------
 int d
 -------
 int k
 --------
 
 ```



## 拷贝构造

在 C++11 及以后，拷贝构造函数和拷贝赋值运算符的**生成条件完全一致**，核心规则如下：

1. **默认生成的前提**：当类**未显式定义**拷贝构造函数 / 拷贝赋值运算符时，编译器会自动生成，除非满足以下任一 “抑制条件”。
2. **抑制生成的条件**（满足其一则不生成，且被编译器标记为 `delete`）：
    - 类中**显式定义了移动构造函数**（`T(T&&)`）；
    - 类中**显式定义了移动赋值运算符**（`T& operator=(T&&)`）；
    - 类的成员或基类中存在**不可拷贝的成员**（如成员的拷贝构造 / 赋值被 `delete` 或不可访问，例如 `std::unique_ptr`）。



如果一个类没有显式声明拷贝构造，就会有隐式的声明出现。

那如何判断这个隐式声明的拷贝构造是有用的还是无用的呢？

```c++
class Word {
public:
    Word(const char*);
    ~Word() {delete [] str;};
private:
    int cnt;
    char* str;
}; // 在这种情况下，在C++11以后，会生成默认拷贝函数，但是是浅拷贝

class Word {
public:
    Word(const strong&);
    ~Word();
private:
    int cnt;
    String str;
}; // 在这种情况下，C++11以后，由于成员对象有拷贝函数，所以编译器会隐式生成拷贝构造
class String {
public:
    String(const String&);
};

// 可能长这样
inline Word::Word(const Word& wd) {
    str.String::String(wd.str);
    cnt = wd.cnt;
}
```



### 位逐次拷贝

当类满足以下情况时，就**不展现位逐次拷贝语义**，编译器会合成 nontrivial 拷贝构造函数：

1. 当一个类内含一个成员对象，而该对象声明了拷贝构造，编译器合成的拷贝构造就会调用后者的拷贝构造，是有用的。
2. 派生类继承自基类，基类有拷贝构造
3. 类内存在虚函数
4. 派生类继承自虚基类



# 第3章 Data语意学

```c++
class X {};
class Y: public virtual X {};
class Z: public virtual X {};
class A: public Y, public Z {};
```

即便是空的类，也会占用1个字节，核心原因是为了**保证每个对象都有唯一的内存地址**。

在64位环境中，Y和Z是8字节，A是16字节，是因为虚继承引入了一个8字节的虚基类指针

当然，不同的编译器可能显示的结果会不一样。



一个类的静态成员，只有一个实例，存放在程序的代码段中。

若取一个静态成员的地址没回得到一个指向其数据类型的指针。

```c++
&Point3d::chunkSize // 得到的是const int*
```

非静态成员，直接存放在每个类对象中。

```c++
&Point3d::z // 这里得到的是成员变量z在类中的偏移量
&origin.z // 这会得到该成员变量在内存中的真正地址
```



不知道这一整个章节是想告诉我啥，囫囵吞枣，走马观花地看过去了。



# 第4章 Function语意学







