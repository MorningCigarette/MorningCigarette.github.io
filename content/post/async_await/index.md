---
title: "最容易踩坑的6个async/await 使用错误"
description: 
slug: "async—await"
date: 2025-07-15
categories:
    - C#
---

## 📌 引言：async/await 很强大，但也很容易用错！

在 .NET 中使用 `async` 和 `await` 进行异步编程，确实让代码看起来更简洁、逻辑更清晰。但正因为写起来太“顺手”，很多开发者（包括我自己）都曾不小心掉进过各种“坑”里。

比如程序卡死、性能下降、异常丢失、资源耗尽……这些问题往往不是语法错误造成的，而是对异步机制的理解不到位。

今天我就来总结一下，**.NET 开发者最容易犯的 6 个 async/await 使用错误**，并告诉你正确的做法是什么。希望你看了之后能少走弯路，写出真正高效又稳定的异步代码。

------

## 🔥 常见错误一：用了 `.Result` 或 `.Wait()` 阻塞异步方法

### ❌ 错误示例：

```
var  result = GetDataAsync().Result;
```

或者：

```
GetDataAsync().Wait();
```

### ⚠️ 为什么有问题？

这看似只是等一个结果，但在某些上下文环境中（比如 UI 线程或 ASP.NET 请求线程），这样做会导致**死锁**！

原因在于：

- `await` 默认会尝试回到原来的上下文线程去继续执行。
- 如果主线程被 `.Result` 或 `.Wait()` 阻塞了，就没人去释放它，导致死循环。

### ✅ 正确做法：

如果当前方法支持异步，就把它也标记为 `async`，然后用 `await`：

```
var  result =  await  GetDataAsync();
```

### 💡 小贴士：

> ★
>
> 在 **ASP.NET Core、WPF、WinForms、Blazor Server** 这类框架中，一定要避免使用 `.Result` 或 `.Wait()`，否则很容易引发死锁问题。

------

## 🔥 常见错误二：写了 `async` 方法，却没用 `await`

### ❌ 错误示例：

```
public  async  Task  DoWorkAsync()
{
        Task.Delay(1000);  // 啥也没干
}
```

### ⚠️ 有什么问题？

这个方法虽然加了 `async`，但没有用 `await`，所以它其实是一个**同步方法**，只不过多了一个状态机包装而已。

更糟的是，编译器并不会报错，你可能还以为自己写了个异步方法。

### ✅ 正确做法：

要用 `await` 才能真正进入异步流程：

```
public  async  Task  DoWorkAsync()
{
        await  Task.Delay(1000);
}
```

### 💡 小贴士：

> ★
>
> 每一个 `async` 方法都应该至少有一个 `await`；如果没有，那就不应该加 `async`。

------

## 🔥 常见错误三：使用 `async void`（除了事件处理）

### ❌ 错误示例：

```
public  async  void  SaveDataAsync()
{
        await  Task.Delay(500);
}
```

### ⚠️ 为什么危险？

`async void` 方法就像“幽灵”一样，你无法等待它完成，也无法捕获它的异常。

一旦抛出异常，就会直接崩溃整个应用程序 —— 即使你在外面写了 `try-catch` 也没用！

### ✅ 正确做法：

除非是事件处理函数（比如按钮点击），否则一律返回 `Task`：

```
public  async  Task  SaveDataAsync()
{
        await  Task.Delay(500);
}
```

这样就可以被 `await` 调用，并且能正确处理异常。

### 💡 小贴士：

> ★
>
> 除了事件处理器，永远不要写 `async void` 方法。它就像是“裸奔”的异步方法，非常不安全。

------

## 🔥 常见错误四：明明不需要异步，却还加 `async`

### ❌ 错误示例：

```
public  async  Task<int>  GetNumberAsync()
{
        return  42;
}
```

### ⚠️ 有什么问题？

这个方法根本没有做任何异步操作，但却加了 `async` 关键字，白白引入了状态机，增加了性能开销。

这不是“为了异步而异步”，而是“为了装样子而异步”。

### ✅ 正确做法：

如果你的方法就是同步返回数据，那就不要用 `async`，直接返回已完成的 `Task`：

```
public  Task<int>  GetNumberAsync()
{
        return  Task.FromResult(42);
}
```

### 💡 小贴士：

> ★
>
> 只有当你真的在调用 I/O、数据库、网络请求等异步操作时，才需要用 `async/await`，否则就别滥用。

------

## 🔥 常见错误五：每次调用都 new HttpClient

### ❌ 错误示例：

```
public  async  Task<string>  GetData()
{
        using  var  client =  new  HttpClient();    // 每次都新建一个
        return  await  client.GetStringAsync("https://api.example.com/data");
}
```

### ⚠️ 有什么风险？

每次创建 `HttpClient` 实际上都会打开一个新的 TCP 连接，而且关闭后不会立刻释放端口，容易造成**端口耗尽（Socket Exhaustion）**。

尤其是在高并发场景下，这种写法可能会让你的应用突然“挂掉”。

### ✅ 正确做法：

把 `HttpClient` 当作共享资源来使用，推荐通过依赖注入的方式获取：

```
private  readonly  HttpClient _httpClient;

public  MyService(HttpClient httpClient)
{
        _httpClient = httpClient;
}

public  async  Task<string>  GetData()
{
        return  await  _httpClient.GetStringAsync("https://api.example.com/data");
}
```

这样不仅复用了连接，还能更好地控制生命周期。

### 💡 小贴士：

> ★
>
> `HttpClient` 是设计用来长期使用的，频繁 new 它是一种“反模式”。建议配合 `IHttpClientFactory` 或服务注入一起使用。

------

## 🔥 常见错误六：忽略 ConfigureAwait(false)，导致上下文捕获引发死锁

### ❌ 错误示例：

```
// 默认捕获当前上下文，可能导致线程阻塞
public  async  Task  DoWorkAsync()
{
        await  SomeIoOperationAsync();  // 默认会尝试回到原始上下文
}
```

### ⚠️ 有什么问题？

在很多异步库或框架中（如 ASP.NET 或 WPF），`await` 默认会尝试捕获当前的同步上下文（Synchronization Context），并在任务完成后回到这个上下文继续执行后续代码。

这在 UI 应用中是有意义的，但在非 UI 层（如类库、服务层）中，这种行为反而可能带来不必要的性能开销，甚至在某些情况下引发**死锁**。

### ✅ 正确做法：

在非 UI 代码中，建议加上 `.ConfigureAwait(false)` 来避免上下文捕获：

```
// 避免上下文捕获，提高性能并防止死锁
public  async  Task  DoWorkAsync()
{
        await  SomeIoOperationAsync().ConfigureAwait(false);
}
```

### 💡 小贴士：

> ★
>
> 在类库、通用方法、后台服务中，建议始终加上 `.ConfigureAwait(false)`，除非你确实需要回到原始上下文。

------

## 🧾 总结：async/await 的最佳实践清单

| 问题                           | 推荐做法                                    |
| :----------------------------- | :------------------------------------------ |
| ❌ 使用 `.Result` 或 `.Wait()`  | ✅ 改成 `async/await`，保持异步链            |
| ❌ 写了 `async` 却不用 `await`  | ✅ 该删就删，不该加就别加                    |
| ❌ 乱用 `async void`            | ✅ 仅限事件处理，其他一律用 `Task`           |
| ❌ 不需要异步却加了 `async`     | ✅ 用 `Task.FromResult()` 替代               |
| ❌ 每次都 new HttpClient        | ✅ 全局复用或通过 DI 获取                    |
| ❌ 忽略 `ConfigureAwait(false)` | ✅ 非 UI 代码中加上 `.ConfigureAwait(false)` |

------

## 🎯 写在最后

`async/await` 是 .NET 中非常强大的工具，但也是一把双刃剑。用得好，能让你的应用响应更快、吞吐更高；用不好，轻则性能下降，重则系统崩溃。

这篇文章列出的六个常见错误，都是我们在实际项目中最容易踩到的“地雷”。希望你能从中吸取经验教训，写出更稳定、更高效的异步代码。
