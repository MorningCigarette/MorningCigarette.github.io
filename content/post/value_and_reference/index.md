---
title: "值传递 vs 引用传递"
description: 
slug: "value_and_reference"
date: 2024-09-19
categories:
    - C#
---

不知道你在开发过程中有没有遇到过这样的困惑：这个变量怎么值被改？这个值怎么没变？

今天就来和大家分享可能导致这个问题的根本原因值传递 vs 引用传递。

在此之前我们先回顾两组基本概念：

**值类型 vs 引用类型**

**值类型：** 直接存储数据，数据存储在栈上；

**引用类型：** 存储数据对象的引用，数据实际存储在堆上。

**形参 vs 实参**

**形参：** 即形式参数，表示调用方法时，方法需要你传递的值。方法声明定义了其形参。也就是说在定义方法时，紧跟在方法名后面括号中的参数列表就是形参。

**实参：** 即实际参数，表示调用方法时，你传递给方法形参的值。调用代码在调用过程时提供实参。也就是说在调用方法时，紧跟在方法名后面括号中的参数列表就是实参。

再来回顾一下值类型和引用类型在内存中是怎么存储的呢？

对于值类型变量的值直接存储在栈中，如下图的int a=10，10就直接存在栈空间中，而其栈空间对应的内存地址为0x66666668；对于引用类型变量本身存储的是实例对象的引用，即实例对象在堆中的实际内存地址，因此引用类型变量是存储其实例对象的引用于栈上，如下图中变量Test a在栈中实际存储的是实例对象Test a在堆中的内存地址0x88888880，而栈空间对应的内存地址为0x66666668。

![img](https://s2.loli.net/2024/09/19/uTyWtSUOvsnpimR.jpg)

***栈也是有内存地址的，这一点很重要，无论栈空间上存储的是值还是引用地址，这个栈空间本身也有自己对应的内存地址。***

什么是值传递？什么是引用传递？

**值传递**：如果变量按值传递给方法，则会把变量的副本传递给方法。对于值类型则把***变量的副本***传递给方法，对于引用类型则把***变量的引用的副本***传递给方法。因此被调用方法参数会创建一个新的内存地址用于接收存储变量，因此在方法内部对变量修改并不会影响原来的值。

**引用传递**：如果变量按引用传递给方法，则会把变量的引用传递给方法，对于值类型则把***变量的栈空间地址***传递给方法，对于引用类型则把***变量的引用的栈空间地址***传递给方法。因此被调用方法参数不会创建一个新的内存地址用于接收存储变量，意味着形参与实参共同指向相同的内存地址，因此在方法内部修对变量修改会影响原来的值。

上面的描述可能有点拗口，下面我们在基于值类型、引用类型、值传递、引用传递各种组合进行一个详细说明。

# ***01***、值类型按值传递

当值类型按值传递时，调用者会把值类型变量的副本传递给方法，因此被调用方法参数会创建一个新的内存地址用于接收存储变量，因此当在方法内部对参数进行修改时并不会影响调用者调用处的值类型变量。

传递值类型变量的副本就是相当于在栈上，又复制了一个同样的值，而且内存地址还不一样，所以互不影响。如下图把a赋值给b，则b直接新开辟了一个栈空间，虽然a和b都是10，但是它们在不同的地址空间中，因此如果他们各自被修改了，也互不影响。

![img](https://s2.loli.net/2024/09/19/nCzd7UTwlQ2SGic.jpg)

下面我们写个例子演示一下，这个例子就是定义个变量a并赋值，然后调用一个方法此方法内对传进来的参数a进行加1，具体代码如下：

```
public static void ValueByValueRun()
{
    var a = 10;
    Console.WriteLine($"调用者-调用方法前 a 值：{a}");
    ChangeValueByValue(a);
    Console.WriteLine($"调用者-调用方法后 a 值：{a}");
}
public static void ChangeValueByValue(int a)
{
    Console.WriteLine($"    被调用方法-接收到 a 值：{a}");
    a = a + 1;
    Console.WriteLine($"    被调用方法-修改后 a 值：{a}");
}
```

运行结果如下：

![img](https://s2.loli.net/2024/09/19/eGJWzVbUqPcurxQ.png)

通过代码执行结果可以发现，方法内对变量的修改已经生效，但是不没有影响到调用者调用处的变量值。

# ***02***、引用类型按值传递

当引用类型按值传递时，调用者会把引用类型变量的引用副本传递给方法，因此被调用方法参数会创建一个新的内存地址用于接收存储变量，而对于一个引用类型变量来说其本身存储的就是引用类型实例对象的引用副本，而方法接收到的也是此变量引用的副本，所以调用者参数和被调用方法参数是引用了同一个实例对象的两个引用副本。如下图Test a可以理解为调用者传的实参，Test b可以理解为被调用方法定义的形参，这两个参数都只是指向堆中Test a的引用副本。

![img](https://s2.loli.net/2024/09/19/Cr3czeJAKTaYMqs.jpg)

因此可以得出两个结论：

1、变量a和b都是指向实例对象Test a的引用，所以无论变量a或b，只要有一个更新了实例成员则另一个变量也会同步发生变化。

2、虽然变量a和b都是指向实例对象Test a的引用，但是他们存储在栈上的内存地址却不同，因此如果他们各种重新分配实例也就是new一个新对象，则另一个变量无法感知到还是保持原因状态不变。

我们先用代码说明第一个结论：

```
public static void ChangeReferenceByValueRun()
{
    var a = new Test
    {
        Age = 10
    };
    Console.WriteLine($"调用者-调用方法前 a.Age 值：{a.Age}");
    ChangeReferenceByValue(a);
    Console.WriteLine($"调用者-调用方法后 a.Age 值：{a.Age}");
}
public static void ChangeReferenceByValue(Test a)
{
    Console.WriteLine($"    被调用方法-接收到 a.Age 值：{a.Age}");
    a.Age = a.Age + 1;
    Console.WriteLine($"    被调用方法-修改后 a.Age 值：{a.Age}");
}
```

运行结果如下：

![img](https://s2.loli.net/2024/09/19/EqWmzTQf3pN9SU8.png)

可以看到被调用方法中a实例对象的Age属性发生变化后，调用者中变量也同步发生了变化。

对于第二个结论我们这样论证，在方法中直接对参数new一个新对象，看看原变量是否发生变化，代码如下：

```
public static void NewReferenceByValueRun()
{
    var a = new Test
    {
        Age = 10
    };
    Console.WriteLine($"调用者-调用方法前 a.Age 值：{a.Age}");
    NewReferenceByValue(a);
    Console.WriteLine($"调用者-调用方法后 a.Age 值：{a.Age}");
}
public static void NewReferenceByValue(Test a)
{
    Console.WriteLine($"    被调用方法-接收到 a.Age 值：{a.Age}");
    a = new Test
    {
        Age = 100
    };
    Console.WriteLine($"    被调用方法-new后 a.Age 值：{a.Age}");
}
```

执行结果如下：

![img](https://s2.loli.net/2024/09/19/9aIX7q6pkYNuDAK.png)

可以发现当在方法中对变量执行new操作后，调用者处的变量并没有发生变化。

为什么会这样呢？因为对于引用类型来说，形参和实参是对引用类型的实例对象引用的两个副本，而这两个副本存储在栈上又分别在不同的内存地址空间上，而new主要就是重新分配内存，这就导致形参变量a=new后，栈上形参变量a指向了Test新的实例对象的引用，而实参变量a还是保持原有实例对象引用不变。

如下图所示。

![img](https://s2.loli.net/2024/09/19/48rMkTDIHSw9VEx.jpg)

# ***03***、值类型按引用传递

当值类型按引用传递时，调用者会把值类型变量对应的栈空间地址传递给方法，因此被调用方法参数不会创建一个新的内存地址用于接收存储变量，因此当在方法内部对参数进行修改时并同样会影响调用者调用处的值类型变量。

![img](https://s2.loli.net/2024/09/19/eQ9h5qwCHjO2i3I.jpg)

传递值类型变量对应的栈空间地址就意味着形参与实参共同指向相同的内存地址，所以才导致对形参修改时，实参也会同步发生变化。

我们用一个小例子演示一下：

```
public static void ValueByReferenceRun()
{
    Console.WriteLine($"值类型按引用传递");
    var a = 10;
    Console.WriteLine($"调用者-调用方法前 a 值：{a}");
    ChangeValueByReference(ref a);
    Console.WriteLine($"调用者-调用方法后 a 值：{a}");
}
public static void ChangeValueByReference(ref int a)
{
    Console.WriteLine($"    被调用方法-接收到 a 值：{a}");
    a = a + 1;
    Console.WriteLine($"    被调用方法-修改后 a 值：{a}");
}
```

执行结果如下：

![img](https://s2.loli.net/2024/09/19/shgTxAVcfIXmy8w.png)

可以发现调用者处的值类型变量已经发生改变。

# ***04***、引用类型按引用传递

当引用类型按引用传递时，调用者会把引用类型变量对应的栈空间地址传递给方法，因此被调用方法参数不会创建一个新的内存地址用于接收存储变量，因此当在方法内部对参数进行修改时并同样会影响调用者调用处的引用类型变量。

![img](https://s2.loli.net/2024/09/19/GPE5mTd9z8xkaeL.jpg)

传递引用类型变量对应的栈空间地址就意味着形参与实参共同指向相同的内存地址，因此对形参修改时，实参也会同步发生变化，而且这个里的修改不单单指修改实例成员，还包括new一个新实例对象。

下面我们看一个修改实例成员的例子：

```
public static void ChangeReferenceByReferenceRun()
{
    Console.WriteLine($"引用类型按引用传递 - 修改实例成员");
    var a = new Test
    {
        Age = 10
    };
    Console.WriteLine($"调用者-调用方法前 a.Age 值：{a.Age}");
    ChangeReferenceByReference(ref a);
    Console.WriteLine($"调用者-调用方法后 a.Age 值：{a.Age}");
}
public static void ChangeReferenceByReference(ref Test a)
{
    Console.WriteLine($"    被调用方法-接收到 a.Age 值：{a.Age}");
    a.Age = a.Age + 1;
    Console.WriteLine($"    被调用方法-修改后 a.Age 值：{a.Age}");
}
```

执行结果如下：

![img](https://s2.loli.net/2024/09/19/ytRMKOkSfQXvU8c.png)

再看看new一个新对象的例子：

```
public static void NewReferenceByReferenceRun()
{
    Console.WriteLine($"引用类型按引用传递 - new 新实例");
    var a = new Test
    {
        Age = 10
    };
    Console.WriteLine($"调用者-调用方法前 a.Age 值：{a.Age}");
    NewReferenceByReference(ref a);
    Console.WriteLine($"调用者-调用方法后 a.Age 值：{a.Age}");
}
public static void NewReferenceByReference(ref Test a)
{
    Console.WriteLine($"    被调用方法-接收到 a.Age 值：{a.Age}");
    a = new Test
    {
        Age = 100
    };
    Console.WriteLine($"    被调用方法-new后 a.Age 值：{a.Age}");
}
```

执行结果如下：

![img](https://s2.loli.net/2024/09/19/jIDHTPCSzWfqyAb.png)

另外string是一个特殊的引用类型，string类型变量的按值传递和按引用传递和值类型是一致的，也就是要把string类型当值类型一样看待就行。string类型的特殊性我们后面会单独具体介绍。

在C#中以下修饰符可应用与参数声明，并且会使得参数按引用传递：ref、out、readonly ref、in。对于每个修饰符具体怎么使用就不再这里细说了。
