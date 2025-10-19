# 创建型模式

## 简单工厂模式

专门定义一个类（Simple Factory)，它的职责是创建其他类的对象。

这些子类都继承同一个父类。

举个例子：

父类是 “交通工具”；

子类是 “小汽车类”“卡车类”“摩托车类”（都继承自 “交通工具”）；

工厂类根据条件（比如输入 “小汽车”），创建出 “小汽车类” 的**实例（对象）**（比如一辆具体的小汽车）。





# 行为型模式

## 观察者模式

一个对象发生改变时，自动通知其他对象。被通知的对象就叫做观察者。



```c++
#include <iostream>
#include <list>
using namespace std;

class Observer { // 观察者
public:
     virtual void Update(int) = 0;
    // 当一个类包含纯虚函数时，该类是抽象类
    // 抽象类不能实例化
    // 继承子类必须重载纯虚函数，不然也是抽象类
};

class Subject {// 被观察的目标
public:
     virtual void Attach(Observer *) = 0;
     virtual void Detach(Observer *) = 0;
     virtual void Notify() = 0;
};

class ConcreteObserver : public Observer
{
 
public:
     ConcreteObserver(Subject *pSubject) : m_pSubject(pSubject){}

     void Update(int value)
     {
 
          cout<<"ConcreteObserver get the update. New State:"<<value<<endl;
     }

private:
     Subject *m_pSubject;
};

class ConcreteObserver2 : public Observer
{
 
public:
     ConcreteObserver2(Subject *pSubject) : m_pSubject(pSubject){}

     void Update(int value)
     {
 
          cout<<"ConcreteObserver2 get the update. New State:"<<value<<endl;
     }

private:
     Subject *m_pSubject;
};

class ConcreteSubject : public Subject
{
 
public:
     void Attach(Observer *pObserver);
     void Detach(Observer *pObserver);
     void Notify();

     void SetState(int state) // 类内实现默认内联，编译器一般会采取优化措施，因此也不一定就内联
     {
 
          m_iState = state;
     }

private:
     std::list<Observer *> m_ObserverList;
     int m_iState;
};

void ConcreteSubject::Attach(Observer *pObserver)
{
 
     m_ObserverList.push_back(pObserver);
}

void ConcreteSubject::Detach(Observer *pObserver)
{
 
     m_ObserverList.remove(pObserver);
}

void ConcreteSubject::Notify()
{
 
     std::list<Observer *>::iterator it = m_ObserverList.begin();
     while (it != m_ObserverList.end())
     {
 
          (*it)->Update(m_iState);
          ++it;
     }
}

int main()
{
 
     // Create Subject
     ConcreteSubject *pSubject = new ConcreteSubject();

     // Create Observer
     Observer *pObserver = new ConcreteObserver(pSubject);
     Observer *pObserver2 = new ConcreteObserver2(pSubject);

     // Change the state
     pSubject->SetState(2);

     // Register the observer
     pSubject->Attach(pObserver);
     pSubject->Attach(pObserver2);

     pSubject->Notify();

     // Unregister the observer
     pSubject->Detach(pObserver);

     pSubject->SetState(3);
     pSubject->Notify();

     delete pObserver;
     delete pObserver2;
     delete pSubject;
}
```



1. 被观察者（不管抽象还是具体）得知道观察者才能通知 —— 具体被观察者不用自己 “重新知道”，是靠继承抽象被观察者，直接用了抽象层里 “管理观察者” 的能力（比如`list<Observer*>`和`Attach/Detach`）；
2. 观察者得知道 “具体盯哪个被观察者”—— 不然被观察者通知时，它都分不清 “是不是自己要关注的那个”，没法准确响应。
