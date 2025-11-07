在C语言中，`数据`和处理数据的操作（`函数`）是分开声明的。

# 第1章 关于对象

## C++对象模型

在C++中，有两种class data members，分别是static和nonstatic。以及三种class member functions，分别是static，nonstatic，virtual。

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

1. 为了支持`virtual functions`，每个类产生一个指向`virtual functions`的指针，放在表格中，这个表格称为`virtual table`（vtbl)
2. 每一个类对象各有一个指针，指向相关的`virtual table`，这个指针被称为`vptr`



![image-20251107094728794](深度探索C++对象模型.assets/image-20251107094728794.png)

class object（类对象）只存储nonstatic data members

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



# 第2章 构造函数

## 默认构造

```c++
class Foo {
    public:
    	int val;
    	Foo *pnext;
};

void foo_bar() { 
    Foo bar; // 由于没有定义构造函数，编译器会默认生成一个，但是这个构造函数什么也不做，所以。。。
    if (bar.val || bar.pnext)
        // do something
}
```

 下面是4种`nontrivial default constructor`有用的默认构造

### 带有 Default Constructor 的 Member Class Object

如果一个class没有任何constructor，但它内含一个member object，而后者有default constructor，那么这个class的immplicit default constructor就是有用的(书中用"nontrivial")。



编译器需要为此class合成出一个default constructor，不过这个合成操作只有在constructor真正需要被调用时才会发生。 



如果用户显式定义了构造函数，那么编译器会扩张已存在的constructors，使得user code被执行前，先调用必要的default constructors。

按照成员对象在类中的声明顺序初始化。



### 带有 Default Consturctor 的 Base Class

如果没有任何构造函数的类，派生自一个带有默认构造函数的基类，那么编译器为派生类隐式生成的默认构造函数也不是无用的。

它将调用基类的默认构造函数。



### 带有 Virtual Function 的 Class



下面两种情况也需要合成一个default constructor(nontrivial)

　　1) class 声明（或继承）一个virtual function。

　　2）class派生自一个继承串链，其中有一个或更多的virtual base classes

这个主要是考虑到带虚函数的对象内保存有一个指向虚函数表的指针，编译器需要保证每个带虚函数的类对象其虚函数指针指向正确的地址。



## 拷贝构造

