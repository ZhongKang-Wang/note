# 20250714

`继承`机制使得pointers或references拥有两个不同的类型：静态类型[^1]和动态类型[^2]。

[^1]:静态类型：声明时的类型
[^2]:动态类型：实际所指的对象





## C++ 转型操作符

语法

```c++
static_cast<type>(expression)
```

example

```c++
double result = ((double)firstNumber)/secondNumber; // in the past
double result = static_cast<double>(firstNumber)/secondNumber; // now
```



- const_cast

  常用于将某个对象的常量性去掉



example

```c++
class Widget {...};
class SpecialWidget: public Widget {...};
void update(SpeciatialWidget* psw);
SpeciatialWidget sw; // non-const对象
const SpeciatialWidget& csw = sw; // csw 是一个代表sw的reference，并视之为常对象

update(&csw); // error
// reason：普通指针可能修改对象，但const指针不允许修改
// 正确做法
update(const_cast<SpecialWidget*>(csw));

```

- 常对象只能调用常函数[^1]

[^1]:常函数：成员函数后面+`const`





- dynamic_cast

  专门用于`将多态基类的指针或引用强制转换为派生类的指针或引用`，而且能够检查转换的安全性。对于不安全的指针转换，转换结果返回 NULL 指针。

- `运行时检查安全性`

example

```c++
Widget *pw = new SpeciatialWidget; // 基类指针指向子类对象
update(pw); // error
// 正确做法
update(dynamic_cast<SpecialWidget*>(pw)); // 如果pw真的指向SpeciatialWidget，否则传进去的将是一个null指针

void updateViaRef(SpecialWidget& rsw);
updateViaRef(dynamic_cast<SpecialWidget&>(*pw)); // 否则抛出exception
```

