---
title: "C#泛型基础"
description: 
slug: "GenericType-Base"
date: 2025-02-21
categories:
    - C#
---

C# 泛型是一种强大的编程特性，允许开发者编写可重用、类型安全且高效的代码。通过泛型，可以在定义类、接口、方法或委托时使用**类型参数**，实际使用时再指定具体类型。以下是关于 C# 泛型的详细介绍：

* * *

### **一、泛型的核心优势**

1.  **类型安全**  
    泛型在编译时检查类型，避免运行时类型转换错误（如 `object` 的装箱拆箱操作）。
2.  **代码复用**  
    同一套逻辑可适用于多种数据类型，无需为不同类型重复编写代码。
3.  **性能优化**  
    避免值类型的装箱拆箱操作，提升性能（例如使用 `List<int>` 而非 `ArrayList`）。

* * *

### **二、泛型的基本用法**

#### 1\. **泛型类（Generic Class）**

```csharp
public class GenericRepository<T>
{
    private List<T> _items = new List<T>();

    public void Add(T item) => _items.Add(item);
    public T Get(int index) => _items[index];
}

// 使用示例
var intRepo = new GenericRepository<int>();
intRepo.Add(42);

var stringRepo = new GenericRepository<string>();
stringRepo.Add("Hello");
```

#### 2\. **泛型接口（Generic Interface）**

```csharp
public interface IComparer<T>
{
    int Compare(T x, T y);
}

public class IntComparer : IComparer<int>
{
    public int Compare(int x, int y) => x.CompareTo(y);
}
```

#### 3\. **泛型方法（Generic Method）**

```csharp
public static void Swap<T>(ref T a, ref T b)
{
    T temp = a;
    a = b;
    b = temp;
}

// 使用示例
int x = 1, y = 2;
Swap(ref x, ref y); // 编译器自动推断类型为 int
```

#### 4\. **泛型委托（Generic Delegate）**

```csharp
public delegate T Processor<T>(T input);

// 使用示例
Processor<int> square = n => n * n;
int result = square(5); // 25
```

* * *

### **三、类型约束（Type Constraints）**

通过 `where` 关键字限制泛型参数的类型范围，增强类型安全性。

| 约束类型         | 示例                       | 说明                            |
| ---------------- | -------------------------- | ------------------------------- |
| **值类型**       | `where T : struct`         | T 必须是值类型（如 `int`）      |
| **引用类型**     | `where T : class`          | T 必须是引用类型（如 `string`） |
| **无参构造函数** | `where T : new()`          | T 必须有默认构造函数            |
| **基类约束**     | `where T : SomeBaseClass`  | T 必须继承自指定基类            |
| **接口约束**     | `where T : ISomeInterface` | T 必须实现指定接口              |
| **组合约束**     | `where T : class, new()`   | 多条件约束                      |

**示例**：

```csharp
public class Factory<T> where T : IAnimal, new()
{
    public T Create() => new T();
}
```

* * *

### **四、协变与逆变（Covariance & Contravariance）**

允许在泛型类型中更灵活地处理继承关系，通过 `out`（协变）和 `in`（逆变）关键字实现。

#### 1\. **协变（Covariance）**

- 用于返回值类型（`out T`）。
- 允许将派生类型作为泛型参数传递给接受基类型的接口。

```csharp
IEnumerable<Cat> cats = new List<Cat>();
IEnumerable<Animal> animals = cats; // 合法，因为 IEnumerable<out T>
```

#### 2\. **逆变（Contravariance）**

- 用于参数类型（`in T`）。
- 允许将基类型作为泛型参数传递给接受派生类型的接口。

```csharp
Action<Animal> animalAction = a => a.Eat();
Action<Cat> catAction = animalAction; // 合法，因为 Action<in T>
catAction(new Cat()); // 执行 animalAction
```

* * *

### **五、泛型中的默认值**

使用 `default(T)` 获取类型的默认值：

- 引用类型：`null`
- 值类型：0、`false` 等。

```csharp
public T GetDefault<T>() => default(T);
```

* * *

### **六、泛型与集合**

.NET 提供了丰富的泛型集合类，如：

- `List<T>`：动态数组
- `Dictionary<TKey, TValue>`：键值对
- `Queue<T>`：先进先出队列
- `Stack<T>`：后进先出栈

**对比非泛型集合（如 `ArrayList`）**：

- 泛型集合无需类型转换，直接操作具体类型。
- 避免因错误类型导致的运行时异常。

* * *

### **七、高级主题**

1.  **反射与泛型**  
    通过反射动态创建泛型类型或方法：
    
    ```csharp
    Type openType = typeof(List<>);
    Type closedType = openType.MakeGenericType(typeof(int));
    List<int> intList = (List<int>)Activator.CreateInstance(closedType);
    ```
    
2.  **泛型类型推断**  
    编译器根据参数自动推断类型：
    
    ```csharp
    var list = new List<int> { 1, 2, 3 };
    var first = list.First(); // 推断返回类型为 int
    ```
    
3.  **静态成员行为**  
    每个封闭泛型类型（如 `MyClass<int>` 和 `MyClass<string>`）拥有独立的静态成员。
    

* * *

### **八、使用场景**

- 需要处理多种数据类型的通用算法（如排序、搜索）。
- 构建可复用的库或框架（如 ORM 中的泛型 Repository）。
- 实现设计模式（工厂模式、策略模式等）。

* * *

### **总结**

C# 泛型通过类型参数化显著提升了代码的灵活性和安全性，是编写高效、可维护代码的重要工具。合理使用泛型约束、协变/逆变等特性，可以进一步优化设计，减少重复代码。
