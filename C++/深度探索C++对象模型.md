```c++
#define X(p, xval) (p.x) = (xval)
...
x(pt, 0.0);
```

# 20250708

在C语言中，`数据`和处理数据的操作（`函数`）是分开声明的。

C++对象模式

class data members

- static
- nonstatic

class member functions

- static
- nonstatic
- virtual



class object只存储nonstatic data members

```plaintext
class object
-----------
nonstatic data members
virtual pointer ->指向相关类的virtual table

virtual table 存放的都是virtual functions 的指针
```



虚拟继承
