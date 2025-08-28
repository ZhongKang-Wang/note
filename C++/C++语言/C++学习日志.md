# 20250825

## Dynamic Typing

C++存在一些机制可以在**运行时**处理类型信息，其中 `void*` 和 `std::any` 是常见的两种方式。

`void*`（无类型指针）

- **本质**：`void*` 是一种特殊的指针类型，可以指向任何类型的对象，但它本身不存储类型信息。

- 运行时特性：

  - 它允许在运行时指向不同类型的数据，但编译器不会跟踪它所指向的具体类型。
  - 使用时必须类型强制转换，否则会导致未定义行为。

```c++
int num = 42;
const char* str = "hello";

void* ptr;
ptr = &num;       // 指向int
int* intPtr = static_cast<int*>(ptr);  // 必须手动转换回int*

ptr = str;        // 指向const char*
const char* strPtr = static_cast<const char*>(ptr);  // 手动转换
```

- 缺点：
  - 没有类型安全保证，转换错误在编译期无法检测，在运行时可能崩溃。



`std::any`（C++17 引入）

- **本质**：`std::any` 是一个类型安全的容器，可以存储任意类型的单个值，并且**在内部保存了值的类型信息**。

- 运行时特性：

  - 可以在运行时存储和提取不同类型的值，通过 `type()` 方法获取存储值的类型信息。
  - 提取值时需要显式指定目标类型，若类型不匹配会抛出 `bad_any_cast` 异常（编译期不报错，运行时检查）。

```c++
#include <any>
#include <iostream>

int main() {
    std::any a;
    a = 42;                  // 存储int
    std::cout << std::any_cast<int>(a) << '\n';  // 提取int

    a = 3.14;                // 存储double
    std::cout << std::any_cast<double>(a) << '\n';

    a = "hello";             // 存储const char*
    std::cout << std::any_cast<const char*>(a) << '\n';

    // 若类型不匹配，会抛出std::bad_any_cast异常
    try {
        std::any_cast<int>(a); // std::any_cast 在类型转换失败时会主动抛出一个 std::bad_any_cast 类型的异常对象，catch 块捕获这个对象后，可以通过它的成员方法获取错误信息。
    } catch (const std::bad_any_cast& e) {
        std::cout << "错误：" << e.what() << '\n';
        // std::bad_any_cast 继承自 std::exception，因此包含 what() 成员方法，该方法返回一个描述错误的字符串（如 "bad any_cast"），通过 e.what() 可以获取并输出这个信息。
    }
    return 0;
}
```

- 优点：
  - 类型安全，运行时可以检测类型是否匹配。
  - 自动管理存储值的生命周期（如内存释放）。
- 缺点：
  - 有一定的性能开销（存储类型信息和动态内存分配）。
  - 仍需手动指定提取类型，无法完全自动推断。



## RTTI

运行时类型确认（RTTI)

There are two main mechanisms for RTTI in C++:

- `typeid` operator
- `dynamic_cast` operator



`typeid operator`

`typeid` is an operator that returns a reference to an object of type `std::type_info`, which contains information about the type of the object. The header file `<typeinfo>` should be included to use `typeid`.

`typeid` 运算符返回的是对 **`std::type_info` 类型对象的引用**，这个 `std::type_info` 对象中包含了**操作数的具体类型信息**。

Here is an example:

```c++
#include <iostream>
#include <typeinfo>

class Base { virtual void dummy() {} };
class Derived : public Base { /* ... */ };

int main() {
    Base* base_ptr = new Derived;

    // Using typeid to get the type of the object
    std::cout << "Type: " << typeid(*base_ptr).name() << '\n'; // 7Derived 7指的是后面跟着7个字符

    delete base_ptr;
    return 0;
}
```



`dynamic_cast operator`

```c++
#include <iostream>

class Base { virtual void dummy() {} };
class Derived1 : public Base { /* ... */ };
class Derived2 : public Base { /* ... */ };

int main() {
    Base* base_ptr = new Derived1;

    // Using dynamic_cast to safely downcast the pointer
    Derived1* derived1_ptr = dynamic_cast<Derived1*>(base_ptr);
    if (derived1_ptr) {
        std::cout << "Downcast to Derived1 successful\n";
    }
    else {
        std::cout << "Downcast to Derived1 failed\n";
    }

    Derived2* derived2_ptr = dynamic_cast<Derived2*>(base_ptr);
    if (derived2_ptr) {
        std::cout << "Downcast to Derived2 successful\n";
    }
    else {
        std::cout << "Downcast to Derived2 failed\n";
    }

    delete base_ptr;
    return 0;
}
```

注意：

- 对**指针**使用 `dynamic_cast`：失败时返回 `nullptr`（不抛异常）
- 对**引用**使用 `dynamic_cast`：失败时抛出 `std::bad_cast` 异常





# 20250828

