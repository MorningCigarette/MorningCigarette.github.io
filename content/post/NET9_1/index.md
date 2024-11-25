---
title: ".Net9新特性"
description: 
slug: "NET9_1"
date: 2024-11-25
categories:
    - C#
---

## 隐式索引访问

现在对象初始值设定项表达式中允许隐式“从末尾开始”索引运算符[^]。

我们来看一个例子，首先创建了一个ImplicitIndex类并且包含一个Numbers属性，该属性是一个长度为 5 的整数数组。

现在我们可以在初始化类ImplicitIndex时初始化属性Numbers，并使用“从末尾开始”索引运算符来填充数组值。

```
/// <summary>
/// 隐式索引访问
/// </summary>
private static void TestImplicitIndex()
{
    var implicitIndex = new ImplicitIndex()
    {
        Numbers = { [^1] = 5, [^2] = 4, [^3] = 3, [^4] = 2, [^5] = 1 }
    };
    foreach (var item in implicitIndex.Numbers)
    {
        Console.WriteLine(item);
    }
}
public class ImplicitIndex
{
    public int[] Numbers { get; set; } = new int[5];
}
```

## 锁对象

本次更新引入新的锁类型System.Threading.Lock，用于实现互斥。在之前的版本中通常通过object类型进行加锁，而现在有了专门的Lock类型用来加锁。

新的Lock类型会使得代码更干净、更安全、更高效。

在新的锁定机制中EnterScope替换了Monitor底层实现。同时它遵循Dispose模式返回ref struct，因此可以与using语句结合使用

```
/// <summary>
/// 锁对象
/// </summary>
public class LockExample()
{
    private readonly Lock _lock = new Lock();

    public void Print()
    {
        lock (_lock)
        {
            Console.WriteLine("Lock");
        }
    }
}
```

## 生成UUID v7

我们经常在实体中使用Guid作为主键，并且通过Guid.NewGuid()可以很方便的生成一个新的Guid，而此方法生成的Guid是依据UUID第四个版本规范生成的。

当前已经可以通过Guid.CreateVersion7()方法创建UUID第七个版本，这个版本UUID主要功能就是包含了时间戳，数据结构如下：

| 48位时间戳 | 12位随机 | 62位随机 |

这也意味着v7版本的UUID可以按时间排序了，在数据库中使用起来更方便，同时Guid.CreateVersion7()方法还有一个重载方法接收DateTimeOffset类型时间戳，用来通过指定时间创建UUID。

```
var guid_v4 = Guid.NewGuid();
var guid_v7 = Guid.CreateVersion7();
var guid_v7_time = Guid.CreateVersion7(TimeProvider.System.GetLocalNow());
```

## Linq新方法 CountBy 和 AggregateBy

引入了新的方法 CountBy 和 AggregateBy后，可以在不经过GroupBy 分配中间分组的情况下快速完成复杂的聚合操作，同时方法命名也非常直观，可以大大提升工作效率。

``````
public class Student
{
    public string Name { get; set; }
    public int Age { get; set; }
}
public void CountByExample()
{
    var students = new List<Student>
    {
        new Student { Name = "小明", Age = 10 },
        new Student { Name = "小红", Age = 12 },
        new Student { Name = "小华", Age = 10 },
        new Student { Name = "小亮", Age = 11 }
    };
    //统计不同年龄有多少人，两个版本实现
    //.NET 9 之前
    var group = students.GroupBy(x => x.Age);
    foreach (var item in group)
    {
        Console.WriteLine($"年龄为：{item.Key}，有：{item.Count()} 人。");
    }
    //.NET 9
    foreach (var student in students.CountBy(c => c.Age))
    {
        Console.WriteLine($"年龄为：{student.Key}，有：{student.Value} 人。");
    }
}

``````

```
public class Student
{
    public string Name { get; set; }
    public string Grade { get; set; }
    public int Age { get; set; }        
}
public void AggregateByExample()
{
    var students = new List<Student>
    {
        new Student { Name = "小明", Grade = "一班", Age = 10 },
        new Student { Name = "小红", Grade = "二班", Age = 12 },
        new Student { Name = "小华", Grade = "一班", Age = 10 },
        new Student { Name = "小亮", Grade = "二班", Age = 11 }
    };
    //统计每个班级各自学生总年龄，两个版本实现
    //.NET 9 之前
    var old = students
       .GroupBy(stu => stu.Grade)
       .ToDictionary(group => group.Key, group => group.Sum(stu => stu.Age))
       .AsEnumerable();
    foreach (var item in old)
    {
        Console.WriteLine($"班级：{item.Key}，总年龄：{item.Value} 。");
    }
    //.NET 9
    foreach (var group in students.AggregateBy(c => c.Grade, 0, (acc, stu) => acc + stu.Age))
    {
        Console.WriteLine($"班级：{group.Key}，总年龄：{group.Value} 。");
    }
}

```

