---
title: "Python中*"
description: 
slug: "xinghaoinpy"
date: 2025-10-28
categories:
    - Python
---

------

## 🧩 一、单个星号 `*` —— 接收「任意数量的位置参数」

```python
def foo(*args):
    print(args)
```

调用：

```python
foo(1, 2, 3)
# 输出: (1, 2, 3)
```

📘 含义：

- `*args` 会把传入的多个位置参数**打包成一个元组 (tuple)**；
- 名字 `args` 只是惯例，你可以写成任何名字，比如 `*numbers`。

✅ 示例：

```python
def add(*nums):
    return sum(nums)

print(add(1, 2, 3, 4))  # 10
```

------

## 🧩 二、两个星号 `**` —— 接收「任意数量的关键字参数」

```python
def foo(**kwargs):
    print(kwargs)
```

调用：

```python
foo(a=1, b=2, c=3)
# 输出: {'a': 1, 'b': 2, 'c': 3}
```

📘 含义：

- `**kwargs` 会把所有的关键字参数打包成一个字典 `dict`；
- 名字 `kwargs` 也是惯例，可以换成任意合法标识符。

✅ 示例：

```python
def print_info(**info):
    for k, v in info.items():
        print(f"{k} = {v}")

print_info(name="Alice", age=25)
```

------

## 🧩 三、函数定义中单独的 `*` —— **限制后面参数必须用关键字传递**

```python
def foo(a, b, *, c, d):
    print(a, b, c, d)
```

调用：

```python
foo(1, 2, c=3, d=4)  # ✅ 正确
foo(1, 2, 3, 4)      # ❌ 错误：c、d 必须用关键字传递
```

📘 含义：

- 单独一个 `*` 表示“从此处开始的参数必须使用关键字指定”；
- 常用于提高函数调用的**可读性和安全性**。

✅ 示例：

```python
def move(x, y, *, speed=1):
    print(f"Moving to ({x},{y}) at speed {speed}")

move(10, 20, speed=5)  # ✅
move(10, 20, 5)        # ❌ 报错
```

------

## 🧩 四、在调用函数时使用 `*` / `**` —— 参数「解包」

这时 `*` / `**` 的作用是 **把序列或字典拆包成参数列表**。

### 示例 1：解包列表或元组

```python
def add(a, b, c):
    print(a + b + c)

nums = [1, 2, 3]
add(*nums)  # 等价于 add(1, 2, 3)
```

### 示例 2：解包字典

```python
def print_person(name, age):
    print(name, age)

person = {"name": "Alice", "age": 25}
print_person(**person)  # 等价于 print_person(name="Alice", age=25)
```

------

## 🧩 五、总结对比表

| 形式       | 定义中含义               | 调用中含义                 |
| ---------- | ------------------------ | -------------------------- |
| `*args`    | 收集多个位置参数为元组   | 解包序列为多个位置参数     |
| `**kwargs` | 收集多个关键字参数为字典 | 解包字典为多个关键字参数   |
| 单独 `*`   | 限制后续参数必须用关键字 | ——（不能单独在调用中使用） |

------

✅ **一句话总结：**

> `*` → 打包或解包“位置参数”；
>  `**` → 打包或解包“关键字参数”；
>  单独的 `*` → 标识“后面参数必须用关键字传入”。

