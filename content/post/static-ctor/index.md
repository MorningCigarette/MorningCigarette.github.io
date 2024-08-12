---
title: "静态构造函数"
description: 
slug: "static-ctor"
date: 2024-08-12
categories:
    - C#

---


在C#中，静态构造函数（也称为类型构造函数）是一种特殊的构造函数，用于初始化静态成员或执行只需要一次的操作。静态构造函数的主要特点包括：

1. **无需显式调用**：静态构造函数在第一次访问类型的静态成员或实例成员之前，由运行时自动调用。

2. **不能有访问修饰符**：静态构造函数不能有`public`、`private`等访问修饰符。它只能由类型定义来确定。

3. **无参数**：静态构造函数不能有参数。

4. **每个类型只能有一个静态构造函数**。

5. **不能通过对象实例来调用**：静态构造函数只能通过类型本身触发，而不能通过类型的实例来触发。

下面是一个简单的示例，展示了如何使用静态构造函数：

```csharp
using System;

class MyClass
{
    // 静态字段
    public static int StaticField;

    // 静态构造函数
    static MyClass()
    {
        Console.WriteLine("静态构造函数被调用");
        StaticField = 42;
    }

    // 实例构造函数
    public MyClass()
    {
        Console.WriteLine("实例构造函数被调用");
    }

    // 静态方法
    public static void StaticMethod()
    {
        Console.WriteLine("静态方法被调用");
    }
}

class Program
{
    static void Main()
    {
        // 访问静态字段，触发静态构造函数
        Console.WriteLine(MyClass.StaticField);

        // 创建实例，静态构造函数不会再次被调用
        MyClass myClass = new MyClass();

        // 调用静态方法
        MyClass.StaticMethod();
    }
}
```

输出：
```
静态构造函数被调用
42
实例构造函数被调用
静态方法被调用
```

在这个示例中，`MyClass`类定义了一个静态字段`StaticField`和一个静态构造函数。在`Main`方法中，第一次访问`StaticField`时，静态构造函数被调用并初始化了`StaticField`。随后的实例创建和静态方法调用不会再次触发静态构造函数。

**静态构造函数通常用于执行类型级别的初始化任务，例如设置静态字段的默认值、配置类型级别的资源等。它保证这些初始化逻辑在类型首次使用时恰当地执行。**
