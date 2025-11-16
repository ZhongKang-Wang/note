# C++ 多线程系统编程

C++多线程编程面临的问题

1. 在析构一个对象时，如何知道此刻是否有别的线程正在执行该对象的成员函数？
2. 如何保证在执行成员函数期间，对象不会在另一个线程被析构？
3. 在调用某个对象的成员函数之前，如何得知这个对象还活着？它的析构函数会不会碰巧执行到一半？

Q：如何定义线程安全？

一个线程安全的 class 应该满足：

- 多个线程同时访问时，其表现出正确的行为。
- 无论操作系统如何调度这些线程，无论这些线程的执行顺序如何交织。
- 调用端代码无需额外的同步或其他协调动作。



`lock_guard`

在`cpp`标准库中，`lock_guard` 是用于**管理锁的生命周期**的 RAII 工具类，作用是**在构造时自动加锁，析构时自动解锁**，确保临界区的安全。



Q：对象的构造如何做到线程安全？

在构造期间不要泄露`this`指针。

```c++
// 对象构造的正确做法
class Foo: public Observer {
public:
    Foo();
    // 另外定义一个函数，在构造后执行注册工作
    void observe(Observable* s) {
        s->register_(this);
    }
}

Foo* pFoo = new Foo;
Observable* s = getSubject();
pFoo->observe(s); // 这叫做二段式构造
```



C++里可能会出现的内存问题

1. 缓冲区溢出
2. 空悬指针/野指针
3. 重复释放（double delete)
4. 内存泄漏（memory leak)
5. 不配对的 new[]/delete
6. 内存碎片

## 正确使用智能指针

1. 用`std::vector<char>/std::string`或者自己编写的`Buffer class`类来管理缓冲区，该类自动记住缓冲区的长度，并通过成员函数修改缓冲区。
2. 用`shared_ptr/weak_ptr`
3. 用`scoped_ptr`，只在对象析构的时候释放一次
4. 用`scoped_ptr`，对象析构时自动释放内存
5. `new[]`替换成`vector`



`shared_ptr`可能会意外延长对象的生命周期。

另，`shared_ptr`的拷贝开销比拷贝原始指针要高，因此多数情况下作函数参数时用`const reference`方式传递。（const 引用只是取别名，不拷贝）



一个`shared_ptr`的引用计数本身是原子的，线程安全的，但是当多线程要修改引用计数的时候，需要加锁。例如，线程 1 和 2 同时：`sp1 = make_shared<int>(0);`（修改同一个 `sp1`，不安全，需要锁）

