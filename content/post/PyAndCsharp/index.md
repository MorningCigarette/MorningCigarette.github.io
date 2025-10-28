---
title: "Python 的 self 和 C# 的 this"
description: 
slug: "PyAndCsharp"
date: 2025-10-28
categories:
    - C#
---

------

## 🧩 一、基本作用对比

| 对比点               | Python 中的 `self`                                | C# 中的 `this`                           |
| -------------------- | ------------------------------------------------- | ---------------------------------------- |
| **含义**             | 当前对象的引用（实例本身）                        | 当前对象的引用（实例本身）               |
| **显式性**           | 必须**显式写出**在方法参数中（通常命名为 `self`） | 编译器**隐式提供**，在实例方法中自动可用 |
| **是否关键字**       | 不是关键字，只是**约定俗成**的名字（可改成别的）  | 是**语言关键字**                         |
| **使用场景**         | 类的实例方法第一个参数必须传入 `self`             | 类的实例方法可直接使用 `this`            |
| **静态方法是否可用** | 不可用，除非手动传实例                            | 不可用，只能用于实例方法                 |

------

## 🧠 二、示例对比

### 🐍 Python

```python
class Person:
    def __init__(self, name):
        self.name = name  # self 指当前实例

    def greet(self):
        print(f"Hello, I'm {self.name}")
```

使用：

```python
p = Person("Alice")
p.greet()   # 等价于 Person.greet(p)
```

> ✅ 注意：调用 `p.greet()` 时，Python **自动传入** `self=p`。

------

### 💻 C#

```csharp
class Person
{
    private string name;

    public Person(string name)
    {
        this.name = name; // this 指当前实例
    }

    public void Greet()
    {
        Console.WriteLine($"Hello, I'm {this.name}");
    }
}
```

使用：

```csharp
var p = new Person("Alice");
p.Greet();   // 编译器自动绑定 this = p
```

> ✅ 在 C# 中不需要显式传参，`this` 是隐式存在的。

------

## ⚙️ 三、调用机制差异

| 对比点         | Python                                             | C#                                       |
| -------------- | -------------------------------------------------- | ---------------------------------------- |
| 方法本质       | 实例方法其实是**普通函数**，通过绑定对象成为“方法” | 实例方法是**类成员**，编译器自动关联实例 |
| 调用本质       | `p.greet()` → `Person.greet(p)`                    | `p.Greet()` → 编译器自动传递 `this`      |
| 函数与方法界限 | 较模糊（函数本身独立于类存在）                     | 明确区分（方法绑定在类型上）             |

------

## 🧩 四、可用场景差异

| 场景             | Python (self)                     | C# (this)                              |
| ---------------- | --------------------------------- | -------------------------------------- |
| **引用实例成员** | `self.name`                       | `this.name`（也可省略）                |
| **区分同名变量** | `self.name = name`（手动）        | `this.name = name`（常用）             |
| **链式调用**     | `return self` 可实现              | `return this` 可实现                   |
| **静态方法**     | `@staticmethod` 不带 `self`       | `static` 方法不能用 `this`             |
| **类方法**       | `@classmethod` 用 `cls`（类本身） | 没有类似的机制（但可用泛型或反射实现） |

------

## 💡 五、总结一句话

> **`self` 是 Python 显式传递的实例引用；
>  `this` 是 C# 编译器隐式提供的实例引用。**

