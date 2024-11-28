---
title: "C# 12 中的新增功能"
description: 
slug: "what_new_in_csharp12"
date: 2024-11-28
categories:
    - C#
---

# C# 12 中的新增功能

## 主构造函数

现在可以在任何 `class` 和 `struct` 中创建主构造函数。 主构造函数不再局限于 `record` 类型。 主构造函数参数都在类的整个主体的范围内。 为了确保显式分配所有主构造函数参数，所有显式声明的构造函数都必须使用 `this()` 语法调用主构造函数。 将主构造函数添加到 `class` 可防止编译器声明隐式无参数构造函数。 在 `struct` 中，隐式无参数构造函数初始化所有字段，包括 0 位模式的主构造函数参数。

编译器仅在 `record` 类型（`record class` 或 `record struct` 类型）中为主构造函数参数生成公共属性。 对于主构造函数参数，非记录类和结构可能并不总是需要此行为。

有关主构造函数的详细信息，请参阅[探索主构造函数](https://learn.microsoft.com/zh-cn/dotnet/csharp/whats-new/tutorials/primary-constructors)的教程和[实例构造函数](https://learn.microsoft.com/zh-cn/dotnet/csharp/whats-new/../programming-guide/classes-and-structs/instance-constructors#primary-constructors)的相关文章。

[](#collection-expressions)

## 集合表达式

集合表达式引入了新的 terse 语法来创建常见的集合值。 可以使用展开运算符 `..` 将其他集合内联到这些值中。

可以创建多个类似集合的类型，而无需使用外部 BCL 支持。 这些类型包括：

- 数组类型，例如 `int[]`。
- [System.Span&lt;T&gt;](https://learn.microsoft.com/zh-cn/dotnet/api/system.span-1) 和 [System.ReadOnlySpan&lt;T&gt;](https://learn.microsoft.com/zh-cn/dotnet/api/system.readonlyspan-1)。
- 支持集合初始值设定项的类型，例如 [System.Collections.Generic.List&lt;T&gt;](https://learn.microsoft.com/zh-cn/dotnet/api/system.collections.generic.list-1)。

以下示例演示了集合表达式的使用：

C#

```
// Create an array:
int[] a = [1, 2, 3, 4, 5, 6, 7, 8];

// Create a list:
List<string> b = ["one", "two", "three"];

// Create a span
Span<char> c  = ['a', 'b', 'c', 'd', 'e', 'f', 'h', 'i'];

// Create a jagged 2D array:
int[][] twoD = [[1, 2, 3], [4, 5, 6], [7, 8, 9]];

// Create a jagged 2D array from variables:
int[] row0 = [1, 2, 3];
int[] row1 = [4, 5, 6];
int[] row2 = [7, 8, 9];
int[][] twoDFromVariables = [row0, row1, row2];
```

*展开运算符*（集合表达式中的 `..`）可将其参数替换为该集合中的元素。 参数必须是集合类型。 以下示例演示了展开运算符的工作原理：

C#

```
int[] row0 = [1, 2, 3];
int[] row1 = [4, 5, 6];
int[] row2 = [7, 8, 9];
int[] single = [.. row0, .. row1, .. row2];
foreach (var element in single)
{
    Console.Write($"{element}, ");
}
// output:
// 1, 2, 3, 4, 5, 6, 7, 8, 9,
```

展开运算符的操作数是可以枚举的表达式。 展开运算符可计算枚举表达式的每个元素。

可以在需要元素集合的任何位置使用集合表达式。 它们可以指定集合的初始值，也可以作为参数传递给采用集合类型的方法。 可以在[有关集合表达式的语言参考文章](https://learn.microsoft.com/zh-cn/dotnet/csharp/whats-new/../language-reference/operators/collection-expressions)或[功能规范](https://learn.microsoft.com/zh-cn/dotnet/csharp/whats-new/../language-reference/proposals/csharp-12.0/collection-expressions)中详细了解集合表达式。

[](#ref-readonly-parameters)

## `ref readonly` 参数

C# 添加了 `in` 参数作为传递只读引用的方法。 `in` 参数既允许变量和值，并且无需对参数进行任何注释即可使用。

添加 `ref readonly` 参数可以更清楚地了解可能使用 `ref` 参数或 `in` 参数的 API：

- 即使参数未修改，在 `in` 引入之前创建的 API 也可能会使用 `ref`。 可以使用 `ref readonly` 更新这些 API。 对于调用方来说，这不会是一项中断性变更，就像参数 `ref` 更改为 `in` 那样。 示例为 [System.Runtime.InteropServices.Marshal.QueryInterface](https://learn.microsoft.com/zh-cn/dotnet/api/system.runtime.interopservices.marshal.queryinterface)。
- 采用 `in` 参数但逻辑上需要变量的 API。 值表达式不起作用。 示例为 [System.ReadOnlySpan&lt;T&gt;.ReadOnlySpan&lt;T&gt;(T)](https://learn.microsoft.com/zh-cn/dotnet/api/system.readonlyspan-1.-ctor#system-readonlyspan-1-ctor%28-0@%29)。
- 使用 `ref` 的 API，因为它们需要变量，但不改变该变量。 示例为 [System.Runtime.CompilerServices.Unsafe.IsNullRef](https://learn.microsoft.com/zh-cn/dotnet/api/system.runtime.compilerservices.unsafe.isnullref)。

若要了解有关 `ref readonly` 参数的详细信息，请参阅有关语言参考中的[参数修饰符](https://learn.microsoft.com/zh-cn/dotnet/csharp/whats-new/../language-reference/keywords/method-parameters#ref-readonly-modifier)的文章，或[参考只读参数](https://learn.microsoft.com/zh-cn/dotnet/csharp/whats-new/../language-reference/proposals/csharp-12.0/ref-readonly-parameters)功能规范。

[](#default-lambda-parameters)

## 默认 Lambda 参数

现在可以为 Lambda 表达式的参数定义默认值。 语法和规则与将参数的默认值添加到任何方法或本地函数相同。

可以在有关 [Lambda 表达式](https://learn.microsoft.com/zh-cn/dotnet/csharp/whats-new/../language-reference/operators/lambda-expressions#input-parameters-of-a-lambda-expression)的文章中详细了解 Lambda 表达式上的默认参数。

[](#alias-any-type)

## 任何类型的别名

可以使用 `using` 别名指令创建任何类型的别名，而不仅仅是命名类型。 这意味着可以为元组类型、数组类型、指针类型或其他不安全类型创建语义别名。 有关详细信息，请参阅[功能规范](https://learn.microsoft.com/zh-cn/dotnet/csharp/whats-new/../language-reference/proposals/csharp-12.0/using-alias-types)。

[](#inline-arrays)

## 内联数组

运行时团队和其他库作者使用内联数组来提高应用的性能。 内联数组使开发人员能够创建固定大小的 `struct` 类型数组。 具有内联缓冲区的结构应提供类似于不安全的固定大小缓冲区的性能特征。 你可能不会声明自己的内联数组，但当它们从运行时 API 作为 [System.Span&lt;T&gt;](https://learn.microsoft.com/zh-cn/dotnet/api/system.span-1) 或 [System.ReadOnlySpan&lt;T&gt;](https://learn.microsoft.com/zh-cn/dotnet/api/system.readonlyspan-1) 对象公开时，你将透明地使用这些数组。

*内联数组*的声明类似于以下 `struct`：

C#

```
[System.Runtime.CompilerServices.InlineArray(10)]
public struct Buffer
{
    private int _element0;
}
```

它们的用法与任何其他数组类似：

C#

```
var buffer = new Buffer();
for (int i = 0; i < 10; i++)
{
    buffer[i] = i;
}

foreach (var i in buffer)
{
    Console.WriteLine(i);
}
```

区别在于编译器可以利用有关内联数组的已知信息。 你可能会像使用任何其他数组一样使用内联数组。 有关如何声明内联数组的详细信息，请参阅有关 [`struct` 类型](https://learn.microsoft.com/zh-cn/dotnet/csharp/whats-new/../language-reference/builtin-types/struct#inline-arrays)的语言参考。

[](#experimental-attribute)

## Experimental 属性

可以使用 [System.Diagnostics.CodeAnalysis.ExperimentalAttribute](https://learn.microsoft.com/zh-cn/dotnet/api/system.diagnostics.codeanalysis.experimentalattribute) 来标记类型、方法或程序集，以指示实验性特征。 如果访问使用 [ExperimentalAttribute](https://learn.microsoft.com/zh-cn/dotnet/api/system.diagnostics.codeanalysis.experimentalattribute) 注释的方法或类型，编译器将发出警告。 用 `Experimental` 特性标记的程序集中包含的所有类型都是实验性的。 可以在有关[编译器读取的常规属性](https://learn.microsoft.com/zh-cn/dotnet/csharp/whats-new/../language-reference/attributes/general#experimental-attribute)的文章或[功能规范](https://learn.microsoft.com/zh-cn/dotnet/csharp/whats-new/../language-reference/proposals/csharp-12.0/experimental-attribute)中阅读详细信息。

[](#interceptors)

## 拦截器

警告

拦截器是一项试验性功能，在 C# 12 的预览模式下提供。 在将来的版本中，该功能可能会发生中断性变更或被删除。 因此，不建议将其用于生产或已发布的应用程序。

若要使用拦截器，用户项目必须指定属性 `<InterceptorsPreviewNamespaces>`。 这是允许包含拦截器的命名空间的列表。

例如：`<InterceptorsPreviewNamespaces>$(InterceptorsPreviewNamespaces);Microsoft.AspNetCore.Http.Generated;MyLibrary.Generated</InterceptorsPreviewNamespaces>`

*拦截器*是一种方法，该方法可以在编译时以声明方式将对*可拦截*方法的调用替换为对其自身的调用。 通过让拦截器声明所拦截调用的源位置，可以进行这种替换。 拦截器可以向编译中（例如在源生成器中）添加新代码，从而提供更改现有代码语义的有限能力。

在源生成器中使用*拦截器*修改现有编译的代码，而非向其中添加代码。 源生成器将对可拦截方法的调用替换为对拦截器方法的调用

。

如果你有兴趣尝试拦截器，可以阅读[功能规范](https://github.com/dotnet/roslyn/blob/main/docs/features/interceptors.md)来了解详细信息。 如果使用该功能，请确保随时了解此实验功能的功能规范中的任何更改。 最终确定功能后，我们将在本站点上添加更多指导。
