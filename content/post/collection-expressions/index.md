---
title: "C#12 中的集合表达式"
description: 
slug: "collection-expressions"
date: 2024-11-14

---

# C#12 中的集合表达式

## [](#经典集合初始化器)经典集合初始化器

自 `C# 3.0` 起，我们就有了 “集合初始化器” 。它们采用 `{}` 模式来初始化任何实现了 `Add()` 方法的 `IEnumerable` 实现。例如，这会创建一个新的 `List<string>` 并用 5 个值初始化它：

```csharp
var values = new List<string> { "1", "2", "3", "4", "5" };
```



幕后，编译器生成的代码看起来像这样：

```csharp
List<string> values = new List<string>();
values.Add("1");
values.Add("2");
values.Add("3");
values.Add("4");
values.Add("5");
```


`C#` 中的数组初始化有其特别之处，你可以用比其他集合更多的方法来初始化它们，尽管看起来与标准集合初始化器相似：

```csharp
var values1 = new[] { "1", "2", "3", "4" };
var values2 = new string[4] { "1", "2", "3", "4" };
string[] values3 = { "1", "2", "3", "4" };
```



这些工作方式与集合初始化器不同；这里没有调用 `Add()` 方法。相反，编译器生成的是如果你手动做这一切时同样的初始化代码：

```csharp
string[] array = new string[4];
array[0] = "1";
array[1] = "2";
array[2] = "3";
array[3] = "4";
```



我们已经讨论了通用集合（如 `List` ）和数组的集合初始化器，还有在 `Span` 世界中变得更为有用的 `stackalloc` 表达式，因为不需要使用 `unsafe` 代码：

```csharp
Span<int> array = stackalloc []{ 1, 2, 3, 4 };
```



幕后，编译器将这个 `stackalloc` 初始化器转化为一些类似这样的 `unsafe` 代码：

```csharp
unsafe {
    byte* num = stackalloc byte[16];
    *(int*)num = 1;
    *(int*)(num + 4) = 2;
    *(int*)(num + (nint)2 * (nint)4) = 3;
    *(int*)(num + (nint)3 * (nint)4) = 4;
    Span<int> array = new Span<int>(num, 4);
}
```



这段代码在栈上创建了一个数组，然后通过指针运算遍历每个元素，设置每个值。

## [](#使用集合表达式统一语法)使用集合表达式统一语法

我们已经看到至少有三种不同的场景下我们在初始化集合：

- 数组
- 类似 `List` 的集合
- 使用 `stackalloc` 的 `ReadOnlySpan`

每种情况都需要稍微不同的语法，例如：

```csharp
int[] array = new[] { 1, 2, 3, 4 };
List<int> list = new() { 1, 2, 3, 4 };
HashSet<int> hashset = new() { 1, 2, 3, 4 };
ReadOnlySpan<int> span = stackalloc [] { 1, 2, 3, 4 };
```



这有点混乱和烦人。`C#12` 中引入的**集合表达式**提供了一种简化、统一的语法，适用于所有这些不同的集合类型。例如：

```csharp
int[] array = [1, 2, 3, 4]
List<int> list = [1, 2, 3, 4];
HashSet<int> hashset = [1, 2, 3, 4];
ReadOnlySpan<int> span = [1, 2, 3, 4];
```



集合表达式在所有集合类型上的一致性是一个真正的优势，但这并不是唯一的优点。集合表达式相比集合初始化器可以提供性能优势（我们将在后续文章中探讨），以及额外的功能。

## [](#推断接口类型)推断接口类型

想象一下，你想要创建一个集合，但你只关心它实现了 `IEnumerable` 。你需要自己决定使用哪个后端类型：

```csharp
IEnumerable<int> list1 = new List<int> { 1, 2, 3, 4 };
IEnumerable<int> list2 = new HashSet<int> { 1, 2, 3, 4 };
IEnumerable<int> list3 = new int[] { 1, 2, 3, 4 };
```



那么应该选择哪一个呢？如果所有你需要做的就是枚举列表，那理论上选择哪种类型都不应该重要，对吧？那么正确的选项是什么？

使用集合表达式，你可以将此决策委托给编译器。你可以不指定后端类型，而由编译器来决定。而且，集合表达式的额外优点是更加简洁：

```csharp
IEnumerable<int> ienumerable = [1, 2, 3, 4];
IList<int> ilist = [1, 2, 3, 4];
IReadOnlyCollection<int> icollection = [1, 2, 3, 4];
```



幕后，编译器完全透明地创建了一个实现所需接口的集合，所以你无需考虑这一点。

值得注意的是，虽然编译器会自动为接口集合选择一个具体的类型，但你需要指定*某种*类型。例如，你不能使用 `var` ：

```csharp
var values = [1, 2, 3, 4]; // ❌ 编译错误，CS9176：集合表达式没有目标类型
Sum(values);
Sum([1, 2, 3, 4]); // ✅ 这是可行的
int Sum(IEnumerable<int> v) => v.Sum();
```



问题在于，根据 C#编译器的工作方式，它无法推断 `values` 的类型应为 `IEnumerable<int>` ，因此抛出了错误。未来版本的 C#可能会改变这种情况，但可能的解决方案是始终选择 `int[]` ，这并不一定是最优的，所以不要抱太大希望。

## [](#针对-readonlyspan-的高效自动-stackalloc)针对 `ReadOnlySpan` 的高效自动 `stackalloc`

对于 `ReadOnlySpan` 和 `Span` 实例来说，如果你只使用集合初始化器，情况也类似。如果你只需要在 `Span` 或 `ReadOnlySpan` 中存储一些数据，那么使用集合初始化器就需要决定将数据放在哪里，然后再从中获取 `Span`：

```csharp
Span<int> spans2 = stackalloc[] { 1, 2, 3, 4 }; // 栈上分配数组
Span<int> spans3 = new[] { 1, 2, 3, 4 }; // 在堆上分配
Span<string> spans4 = new[] { "1", "2", "3", "4" }; // 不能直接使用 stackalloc 分配字符串数组
```



虽然这不是一个很大的决策，通常只有两个合理的选择，但还是需要额外考虑的事情。此外，你不能直接使用 `stackalloc` 为 `string[]` 分配内存，除非通过一些 `InlineArray` 的技巧 。

使用集合表达式，你可以再次将此决策委托给编译器，它会做出正确的处理。

```csharp
ReadOnlySpan<int> readonlyspans = [1, 2, 3, 4];
Span<string> spans = ["1", "2", "3", "4"];
```



在系列的后续部分，你会发现这些特定的集合表达式案例被**高度优化**！

## [](#集合表达式使重构更简单)集合表达式使重构更简单

到目前为止，我展示的所有例子都是将集合表达式赋值给变量，但实际上你也可以直接将集合表达式作为方法参数使用，比如：

```csharp
using System.Linq;
using System.Collections.Generic;

// 创建一个接受IEnumerable<int>的方法
int Sum(IEnumerable<int> values) => values.Sum();

// 使用集合表达式调用方法
Sum([1, 2, 3, 4]);
```



这种模式的一个好处是，如果我改变了 `Sum()` 的签名，我不需要改变调用站点。相比之下，如果你使用集合初始化器：

```csharp
// 如果方法接受数组...
int Sum1(int[] values) => values.Sum();
Sum1(new [] { 1, 2, 3, 4 }); // 必须使用数组语法

// 如果方法接受IEnumerable...
int Sum2(IEnumerable<int> values) => values.Sum();
Sum2(new List<int> { 1, 2, 3, 4 }); // 必须使用显式类型，如List或类似

// 如果方法接受ReadOnlySpan...
int Sum3(ReadOnlySpan<int> values) {
    var total = 0;
    foreach (var value in values) {
        total += value;
    }
    return total;
}
Sum3(new []{ 1, 2, 3, 4 }); // 必须选择标准数组或者使用stackalloc分配的数组
```



如果使用集合表达式，那么我们可以使用完全相同的语法来调用所有三个 `Sum()` 实现：

`Sum1([ 1, 2, 3, 4 ]); Sum2([ 1, 2, 3, 4 ]); Sum3([ 1, 2, 3, 4 ]);`

并且，编译器会使用最有效的实现方式来创建所需类型的集合。

这看起来可能是一件小事，而且在某种程度上确实是这样，但正是所有这些小小的便利特性使得集合表达式成为一个非常整洁的功能！整体而言，正是这些微小的方便性让集合表达式的使用变得极为顺手和高效。

## [](#空集合)空集合

在 `C#` 中，集合表达式的一个特性是编译器明确识别空集合的语法。这意味着你不再需要写像下面这样的代码：

```csharp
var empty = new int[]{}; // 通常你不应该这样做...
var empty = Array.Empty<int>(); // ...相反，更倾向于使用这行代码!
```



现在，你可以使用 `[]` 来生成一个适当的空集合版本，例如：

```csharp
int[] empty = [];
```



集合表达式相比于显式初始化有两大主要优势：

- 编译器可以选择创建空集合的最有效方式，比如选择 `Array.Empty()`（或其等价方法）。
- 对于所有集合类型，你可以使用一致的语法。

以下示例展示了多种集合类型，以及如何使用 `[]` 来创建它们的空版本。每行代码后的注释显示了编译器为特定类型生成的代码：

```csharp
int[] array = []; // Array.Empty()
HashSet<int> hashset = []; // new HashSet<int>()
List<int> list = []; // new List<int>()
IEnumerable<int> ienumerable = []; // Array.Empty()
ICollection<int> icollection = []; // new List<int>()
IList<int> ilist = []; // new List<int>()
IReadOnlyCollection<int> readonlycollection = []; // Array.Empty()
IReadOnlyList<int> readonlyList = []; // Array.Empty()
Span<int> span = []; // default(Span<int>)
ReadOnlySpan<int> readonlyspan = []; // default(ReadOnlySpan<int>)
ImmutableArray<int> immutablearray = []; // ImmutableArray.CreateRange(Array.Empty())
ImmutableList<int> immutablelist = []; // ImmutableList.Create(default(ReadOnlySpan<int>));
```



如你所见，编译器尽可能地高效；如果类型是可变的，如 `HashSet` 或 `List`，那么它只能创建一个新的实例；但如果可以使用不分配内存的版本，如 `Array.Empty()`，那么它就会采用这种方式！请注意，在最后一个示例中，`ImmutableArray` 和 `ImmutableList` 使用了不同的构造函数来创建空集合，因为它们是不可变集合。

## [](#使用展开运算符（spread-element）从其他集合构建集合)使用展开运算符（spread element）从其他集合构建集合

我们已经了解了集合表达式的两个好处：

- 一致的语法
- 高效的编译器生成实现

集合表达式的另一个重要特性是 *展开运算符* (`..`)。这使你能够更容易地从其他集合实例构建新的集合。

举个具体的例子，假设你有两个 `IEnumerable` 集合，并且你想将它们作为数组连接起来。使用 LINQ，这是相当容易做到的，因为它提供了专门用于此类操作的扩展方法：

```csharp
int[] ConcatAsArray(IEnumerable<int> first, IEnumerable<int> second)
{
    return first.Concat(second).ToArray();
}
```



很好，但是如果现在你需要处理的是 `ReadOnlySpan` 而不是 `IEnumerable` 呢？不幸的是，正如我们之前讨论的那样，`ReadOnlySpan` 不实现 `IEnumerable` 接口，所以我们可能需要这样做：

```csharp
int[] ConcatAsArray(ReadOnlySpan<int> first, ReadOnlySpan<int> second)
{
    var list = new List<int>(first.Length + second.Length);
    list.AddRange(first);
    list.AddRange(second);
    return list.ToArray();
}
```



这虽然不是 *糟糕*，但仍然让人感到恼火，因为对于每种不同的集合类型都得考虑这些细节。而使用集合表达式和展开运算符，我们得到了一个简洁的快捷方式，它可以用于所有受支持的集合类型。上述两种重载都可以用同样的方式实现：

```csharp
int[] ConcatAsArray(IEnumerable<int> first, IEnumerable<int> second) => [..first, ..second];
int[] ConcatAsArray(ReadOnlySpan<int> first, ReadOnlySpan<int> second) => [..first, ..second];
```



而且，由于集合表达式的一致性，如果你更改 `ConcatAsArray()` 方法的参数 *或者* 返回类型，你完全不需要修改集合表达式本身，它依旧可以正常工作！

`..` 运算符意味着 “写出集合中的所有值” ，因此再举一个例子：

```csharp
int[] array = [1, 2, 3, 4];
IEnumerable<int> oddValues = array.Where(int.IsOddInteger); // 1, 3
int[] evenValues = [..array.Where(int.IsEvenInteger)];      // 2, 4
int[] allValues = [..oddValues, ..evenValues];             // 1, 3, 2, 4
```



上述代码多次使用了展开运算符，但在每种情况下，它都意味着 “写出集合的所有元素” 。所以在最后一步中，`allValues` 包含了 `oddValues` 中的所有元素，后面跟着 `evenValues` 中的所有值。

你也可以在集合表达式中混合单一值和展开集合，例如：

```csharp
int[] arr = [1, 2, 3, 4];
int[] myValues = [0, ..arr, 5, 6]; // 0, 1, 2, 3, 4, 5, 6
```



最终结果就像是遍历了 `arr` 并添加了每个值一样。

注意，展开运算符 `..` 与 *范围运算符*（如 `1..3` 或 `2..^` 中的 `..`）是 *不同* 的，后者用于对数组进行 *索引*。然而，你可以结合使用它们，先用 *范围* 来选择元素子集，然后将它们 *展开* 到集合表达式中：

```csharp
int[] primes = [1, 2, 3, 5, 7, 9, 11];
int[] some = [0, ..primes[1..^1]]; // 0, 2, 3, 5, 7, 9
```



这段代码使用 `1..^1` 范围运算符取 `primes` 数组的第1个至N-1个元素（即 `2, 3, 5, 7, 9`），然后在集合表达式中使用展开 `..`。

集合表达式为创建集合添加了一种很好的对称性（在重构从一种集合类型到另一种时特别有用），并且通过展开运算符使集合的组合变得更加简单。但集合表达式不仅仅是关于语法。重要的是，集合表达式为编译器提供了优化其生成代码的选项。

在下一篇中，我们将探讨更多类型的集合，如 `T[]` 和 `ReadOnlySpan<T>`，看看它们在使用集合表达式时是如何被高度优化的。

[英文原文](https://www.koudingke.cn/go?link=https%3a%2f%2fandrewlock.net%2fbehind-the-scenes-of-collection-expressions-part-1-introducing-collection-expressions-in-csharp12%2f)
