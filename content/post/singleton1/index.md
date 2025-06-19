---
title: "Windows桌面程序单例"
description: 
slug: "桌面程序单例"
date: 2025-06-19
categories:
    - C#
---

### 使用 `EventWaitHandle`

- **原理**：通过命名的事件对象（`EventWaitHandle`）检测是否已有实例运行。

- 

  特点：

  - 适合跨进程通信（IPC）。
  - 可以通过 `Set()` 和 `WaitOne()` 唤醒其他实例（如你的代码中唤醒主窗口）。

```
 public bool EnsureAppSingletion()
 {
     EventWaitHandle singletonEvent = new EventWaitHandle(false, EventResetMode.AutoReset, "SlimeNull/OpenGptChat", out bool createdNew);

     if (createdNew)
     {
         Task.Run(() =>
         {
             while (true)
             {
                 // wait for the second instance of OpenGptChat
                 singletonEvent.WaitOne();

                 Dispatcher.Invoke(() =>
                 {
                     ShowApp();
                 });
             }
         });

         return true;
     }
     else
     {
         singletonEvent.Set();
         return false;
     }
 }
 public static void ShowApp()
{
            Window mainWindow = Application.Current.MainWindow;
            if (mainWindow == null)
                return;

            mainWindow.Show();

            if (mainWindow.WindowState == WindowState.Minimized)
                mainWindow.WindowState = WindowState.Normal;

            if (!mainWindow.IsActive)
                mainWindow.Activate();
}
 public bool EnsureAppSingletion()
 {
     EventWaitHandle singletonEvent = new EventWaitHandle(false, EventResetMode.AutoReset, "SlimeNull/OpenGptChat", out bool createdNew);

     if (createdNew)
     {
         Task.Run(() =>
         {
             while (true)
             {
                 // wait for the second instance of OpenGptChat
                 singletonEvent.WaitOne();

                 Dispatcher.Invoke(() =>
                 {
                     ShowApp();
                 });
             }
         });

         return true;
     }
     else
     {
         singletonEvent.Set();
         return false;
     }
 }
 public static void ShowApp()
{
            Window mainWindow = Application.Current.MainWindow;
            if (mainWindow == null)
                return;

            mainWindow.Show();

            if (mainWindow.WindowState == WindowState.Minimized)
                mainWindow.WindowState = WindowState.Normal;

            if (!mainWindow.IsActive)
                mainWindow.Activate();
}
```

### **使用 `Mutex`（互斥锁）**

- **原理**：通过命名的互斥锁检测是否已有实例运行。
- 特点：
  - 更轻量级，适合单纯判断单例（不涉及复杂通信）。
  - 系统会自动释放资源（`Mutex` 继承自 `WaitHandle`，有 `Dispose()` 机制）。

 

```
public bool EnsureAppSingleton()
{
    bool isFirstInstance;
    Mutex mutex = new Mutex(true, "SlimeNull/OpenGptChat", out isFirstInstance);

    if (isFirstInstance)
    {
        // 首次运行，保持 Mutex 不释放（通过不调用 ReleaseMutex）
        return true;
    }
    else
    {
        mutex.Dispose(); // 立即释放资源
        return false;
    }
}
public bool EnsureAppSingleton()
{
    bool isFirstInstance;
    Mutex mutex = new Mutex(true, "SlimeNull/OpenGptChat", out isFirstInstance);

    if (isFirstInstance)
    {
        // 首次运行，保持 Mutex 不释放（通过不调用 ReleaseMutex）
        return true;
    }
    else
    {
        mutex.Dispose(); // 立即释放资源
        return false;
    }
}
```

### **两种方式的对比**

| **特性**         | **`EventWaitHandle`**           | **`Mutex`**              |
| ---------------- | ------------------------------- | ------------------------ |
| **主要用途**     | 跨进程通信 + 单例判断           | 单例判断（轻量级）       |
| **资源释放**     | 需手动 `Dispose()` 或 `Close()` | 自动释放（`using` 语句） |
| **唤醒其他实例** | 支持（通过 `Set()`）            | 不支持                   |
| **性能**         | 稍重（适合复杂场景）            | 更轻量                   |

------

### **如何选择？**

- 如果只需要判断单例（不涉及唤醒其他实例），优先用 **`Mutex`**（代码更简洁）。
- 如果需要唤醒已有实例（如你的代码中唤醒主窗口），用 **`EventWaitHandle`**。
