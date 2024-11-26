---
title: "ObservableCollection和BindingList对比"
description: 
slug: "ObservableCollection_and_BindingList"
date: 2024-11-26
categories:
    - WPF&&MVVM
---

在 WPF 中，**`ObservableCollection`** 和 **`BindingList`** 都是常用的数据集合类型，但它们的用途和功能有一些不同。以下是两者的详细对比：

## **数据变更通知机制**

#### `ObservableCollection<T>`:

- **实现方式**: 实现了 `INotifyCollectionChanged` 和 `INotifyPropertyChanged` 接口。
- **功能**: 当集合中元素的添加、移除或整个集合的刷新等操作发生时，`ObservableCollection` 会触发 **集合变更通知**（CollectionChanged事件），比如 UI 会自动更新。但是一旦其中的元素发生了改变，将不会通知
- **优势**: 适合绑定到 WPF 的 `ItemsControl`（如 `ListBox`、`DataGrid`）时实时更新界面。

#### `BindingList<T>`:

- **实现方式**: 实现了 `IBindingList` 接口。
- **功能**: 提供了更细粒度的控制，包括数据绑定和排序功能，但默认情况下不支持批量通知。
- **通知**: 只会在集合的元素被添加或删除时触发事件，而 **元素属性变更**（ListChanged） 通知需要集合中的元素实现 `INotifyPropertyChanged`。

------

## **元素属性变化通知**

#### `ObservableCollection<T>`:

- **支持情况**: 集合本身不监听集合中元素的属性变化。如果需要更新绑定，则集合中每个元素需实现 `INotifyPropertyChanged`。
- **应用场景**: 适合动态变化的集合（添加/删除），但对于元素属性变化通知较弱。

#### `BindingList<T>`:

- **支持情况**: 原生支持 `INotifyPropertyChanged`。如果元素实现了该接口，则 `BindingList` 会响应元素属性的变化。
- **应用场景**: 更适合需要频繁更新单个元素属性的场景。

------

## **排序和筛选**

#### `ObservableCollection<T>`:

- **排序支持**: 不支持直接排序。需要手动实现，通常借助 `CollectionView` 或 `Linq` 实现排序或筛选。
- **灵活性**: 主要适合简单的数据绑定和动态更新。

#### `BindingList<T>`:

- **排序支持**: 内置支持排序功能（通过实现 `IBindingList` 提供 `ApplySort` 方法）。
- **使用场景**: 适合需要排序或筛选功能的集合操作。

------

## **性能**

- **`ObservableCollection<T>`**: 因为其设计用于 WPF 数据绑定，通知机制针对 WPF 优化，所以性能更高。
- **`BindingList<T>`**: 功能更通用，通知机制在 WinForms 环境中表现更优，在 WPF 中性能稍逊。

------

## **线程安全**

- **两者均不是线程安全的**，如果需要在多线程环境中操作集合，应使用同步机制（如 `Dispatcher` 或 `lock`）来确保线程安全。

------

## **总结与选择**

| 特性       | ObservableCollection                      | BindingList                                 |
| ---------- | ----------------------------------------- | ------------------------------------------- |
| 通知机制   | 集合变更通知 (`INotifyCollectionChanged`) | 支持元素变更通知 (`INotifyPropertyChanged`) |
| 排序和筛选 | 需借助 `CollectionView` 等实现            | 原生支持排序                                |
| 适用场景   | 动态数据绑定，实时 UI 更新                | 复杂的排序、筛选和 WinForms 数据绑定        |
| 性能       | 在 WPF 中表现优异                         | 较为通用，但在 WPF 中略逊一筹               |

## **推荐**

- - 使用 `ObservableCollection`

    : 如果主要是用于 WPF 中的动态绑定。

    - **使用 `BindingList`**: 如果需要内置排序和属性监听，或者需要兼容 WinForms 应用。
