---
title: "C# Invoke"
description: 
slug: "C#Invoke"
date: 2024-09-26
categories:
    - C#
---

在 C# 中，Invoke() 是一个用于调用方法的方法，它能够在运行时动态地调用一个方法。Invoke方法主要用于以下几种场景：

## 委托的Invoke
委托是C#中的一种类型，它表示引用方法的对象。你可以通过委托来调用（或“调用”）它所引用的方法。`Invoke` 方法用于显式地调用委托所引用的方法。

```
delegate void MyDelegate(string message);
class Program{  
static void Main() 
{  
MyDelegate myDelegate = new MyDelegate(Hello);
myDelegate.Invoke("Hello, World!"); // 显式调用  
myDelegate("Hello, World!"); // 隐式调用，效果与上面的Invoke相同 } 
static void Hello(string message)    
{     
Console.WriteLine(message);   }
}
```

在上面的代码中，`myDelegate.Invoke("Hello, World!")` 和 `myDelegate("Hello, World!")` 是等效的。通常，我们更倾向于使用隐式调用（即直接使用委托名和方法参数），因为它更简洁。

  
## 反射的Invoke

  
反射是.NET框架提供的一种功能，它允许程序在运行时检查或修改其类型、成员和属性的行为。使用反射，你可以动态地创建和调用类型、方法、属性等。在这种情况下，`Invoke` 通常用于调用通过反射获取的方法。

```
class Program
{
	static void Main()
	{
		var type = typeof(Program);
		var method = type.GetMethod("Hello");//GetMethod只能获取Public方法
		method.Invoke(null, new object[] { "Hello, Reflection!" });
	}
	public static void Hello(string message)
	{
		Console.WriteLine(message);
	}
}
```

在上面的代码中，我们使用反射来获取`Program`类中的`Hello`方法，并使用`Invoke`来调用它。注意，当使用反射调用静态方法时，第一个参数（即实例对象）通常为`null`。对于实例方法，你需要提供一个有效的实例对象作为第一个参数。

## 跨线程控件Invoke(Windows Forms 和 WPF)

在 Windows Forms 或 WPF 应用程序中，当需要从非 UI 线程更新 UI 控件时，可以使用控件的 Invoke方法。这是因为 UI 控件只能在其所属的 UI 线程上进行操作。如果在其他线程上直接修改控件状态，可能会引发异常或导致不可预测的行为。

* Control.Invoke（Windows Forms） 
* Dispatcher.Invoke（WPF）
同步方法，调用后会阻塞调用线程，直到在 UI 线程上执行完指定委托并返回结果。

## 异步委托调用
* BeginInvoke (Windows Forms 和 WPF)
* Control.BeginInvoke（Windows Forms）
*  Dispatcher.BeginInvoke（WPF）
异步方法，立即返回，不会阻塞调用线程。指定的委托将在 UI 线程上异步执行。


## 事件和回调

在某些情况下，Invoke可能被用作事件处理或回调机制的一部分。例如，在异步编程或多线程环境中，当某个事件发生时，可能需要通过Invoke来调用一个事件处理程序或回调函数。
