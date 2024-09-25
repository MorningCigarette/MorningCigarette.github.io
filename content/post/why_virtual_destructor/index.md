---
title: "Why Virtual Destructor"
description: 
slug: "why_virtual_destructor"
date: 2024-09-25
categories:
    - C++
---

# C++中基类的析构函数为什么要用virtual虚析构函数
大家知道，析构函数是为了在对象不被使用之后释放它的资源，虚函数是为了实现多态。那么把析构函数声明为vitual有什么作用呢？ 直接的讲，C++中基类采用virtual虚析构函数是为了防止内存泄漏。具体地说，如果派生类中申请了内存空间，并在其析构函数中对这些内存空间进行释放。假设基类中采用的是非虚析构函数，当删除**基类指针指向的派生类对象**时就不会触发动态绑定，因而只会调用基类的析构函数，而不会调用派生类的析构函数。那么在这种情况下，派生类中申请的空间就得不到释放从而产生内存泄漏。所以，为了防止这种情况的发生，C++中基类的析构函数应采用virtual虚析构函数。

请看下面的代码：
```
#include <iostream>
using namespace std;

class Base
{
public:
    Base() {}; // Base的构造函数
    ~Base()    // Base的析构函数
    {
        cout << "Output from the destructor of class Base!" << endl;
    };
    //   virtual  ~Base()    // Base的析构函数
    // {
    //     cout << "Output from the destructor of class Base!" << endl;
    // };
    virtual void DoSomething()
    {
        cout << "Do something in class Base!" << endl;
    };
};

class Derived : public Base
{
public:
    Derived() {}; // Derived的构造函数
    ~Derived()    // Derived的析构函数
    {
        cout << "Output from the destructor of class Derived!" << endl;
    };
    void DoSomething()
    {
        cout << "Do something in class Derived!" << endl;
    };
};

int main()
{
    Derived *pTest1 = new Derived(); // Derived类的指针
    pTest1->DoSomething();
    delete pTest1;
    cout << endl;
    Base *pTest2 = new Derived(); // Base类的指针
    pTest2->DoSomething();
    delete pTest2;
    return 0;
}
```
先看程序输出结果：

```
Do something in class Derived!
Output from the destructor of class Derived!
Output from the destructor of class Base!

Do something in class Derived!
Output from the destructor of class Base!
```

可以正常释放pTest1的资源，而没有正常释放pTest2的资源，因为从结果看Derived类的析构函数并没有被调用。通常情况下类的析构函数里面都是释放内存资源，而析构函数不被调用的话就会造成内存泄漏。原因是指针pTest2是Base类型的指针，释放pTest2时只进行Base类的析构函数。

在代码~Base析构函数前面加上virtual关键字后的运行结果如下：
```
Do something in class Derived!
Output from the destructor of class Derived!
Output from the destructor of class Base!

Do something in class Derived!
Output from the destructor of class Derived!
Output from the destructor of class Base!
```

此时释放指针pTest2时，由于Base的析构函数是virtual的，就会先找到并执行Derived类的析构函数，然后再执行Base类的析构函数，资源正常释放，避免了内存泄漏。 

因此，**只有当一个类被用来作为基类的时候，才会把析构函数写成虚函数。**
