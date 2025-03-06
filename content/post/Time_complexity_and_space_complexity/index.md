---
title: "时间复杂度与空间复杂度"
description: 
slug: "Time_complexity_and_space_complexity"
date: 2025-03-06
categories:
    - C#
---

## **常数时间复杂度 O(1)**
### **示例：访问数组元素**
```csharp
int[] arr = { 1, 2, 3, 4, 5 };
int firstElement = arr[0]; // 直接访问第一个元素
```
• **时间复杂度**：O(1)  
  直接通过索引访问数组元素，操作次数与输入规模无关。
• **空间复杂度**：O(1)  
  仅使用固定内存存储变量 `firstElement`，无额外空间占用。

**对比**：时间和空间均为常数级别，效率极高。

---

## **线性时间复杂度 O(n)**
### **示例1：遍历数组求和**
```csharp
int SumArray(int[] arr) {
    int sum = 0;
    foreach (int num in arr) { // 遍历每个元素
        sum += num;
    }
    return sum;
}
```
• **时间复杂度**：O(n)  
  需要遍历数组的每个元素，操作次数与输入规模 `n` 成线性关系。
• **空间复杂度**：O(1)  
  仅使用固定变量 `sum`，不随输入规模增加。

### **示例2：复制数组**
```csharp
int[] CopyArray(int[] arr) {
    int[] newArr = new int[arr.Length];
    for (int i = 0; i < arr.Length; i++) {
        newArr[i] = arr[i];
    }
    return newArr;
}
```
• **时间复杂度**：O(n)  
  遍历数组复制每个元素。
• **空间复杂度**：O(n)  
  需要额外创建一个长度为 `n` 的新数组。

**对比**：  
• 同样是 O(n) 时间，`SumArray` 的空间复杂度更优（O(1) vs O(n)）。  
• **空间与时间的权衡**：复制数组牺牲空间来保留原始数据。

---

## **平方时间复杂度 O(n²)**
### **示例：冒泡排序**
```csharp
void BubbleSort(int[] arr) {
    for (int i = 0; i < arr.Length; i++) {
        for (int j = 0; j < arr.Length - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
```
• **时间复杂度**：O(n²)  
  双重循环导致操作次数与 `n²` 成正比。
• **空间复杂度**：O(1)  
  原地排序，仅用临时变量 `temp`，无额外空间占用。

**对比**：时间效率低但空间效率高，适合内存受限的场景。

---

## **递归算法的复杂度分析**
### **示例1：斐波那契数列（递归实现）**
```csharp
int Fibonacci(int n) {
    if (n <= 1) return n;
    return Fibonacci(n - 1) + Fibonacci(n - 2);
}
```
• **时间复杂度**：O(2ⁿ)  
  递归调用指数级增长（每次调用产生两个子问题）。
• **空间复杂度**：O(n)  
  递归调用栈深度为 `n`，占用线性空间。

### **示例2：斐波那契数列（迭代实现）**
```csharp
int FibonacciIterative(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1, sum;
    for (int i = 2; i <= n; i++) {
        sum = a + b;
        a = b;
        b = sum;
    }
    return b;
}
```
• **时间复杂度**：O(n)  
  单层循环遍历 `n` 次。
• **空间复杂度**：O(1)  
  仅使用固定变量 `a`, `b`, `sum`。

**对比**：  
• 递归实现牺牲时间（O(2ⁿ)）和空间（O(n)）换取代码简洁性。  
• 迭代实现通过优化，时间降至 O(n)，空间降至 O(1)，但代码稍复杂。

---

## **5. 时间与空间的权衡**
### **哈希表：以空间换时间**
```csharp
bool HasDuplicate(int[] arr) {
    HashSet<int> seen = new HashSet<int>();
    foreach (int num in arr) {
        if (seen.Contains(num)) return true;
        seen.Add(num);
    }
    return false;
}
```
• **时间复杂度**：O(n)  
  `HashSet` 的插入和查询操作平均为 O(1)。
• **空间复杂度**：O(n)  
  需要额外存储所有元素的哈希表。

**对比**：  
通过牺牲 O(n) 的空间，将时间从暴力法的 O(n²) 优化到 O(n)。

---

## **总结对比**
| **场景**         | **时间复杂度** | **空间复杂度** | **核心权衡**               |
| ---------------- | -------------- | -------------- | -------------------------- |
| 直接访问数组元素 | O(1)           | O(1)           | 无                         |
| 遍历数组求和     | O(n)           | O(1)           | 时间线性增长，空间高效     |
| 复制数组         | O(n)           | O(n)           | 空间换数据完整性           |
| 冒泡排序         | O(n²)          | O(1)           | 时间低效，空间高效         |
| 斐波那契递归     | O(2ⁿ)          | O(n)           | 时间和空间均低效，代码简洁 |
| 斐波那契迭代     | O(n)           | O(1)           | 时间高效，空间高效         |
| 哈希表查重       | O(n)           | O(n)           | 空间换时间                 |

**核心思想**：  
• **时间复杂度**关注操作次数的增长率，**空间复杂度**关注内存占用的增长率。  
• 实际开发中需根据场景选择：  
  • **内存敏感**（如嵌入式系统）优先优化空间复杂度。  
  • **时间敏感**（如实时计算）优先优化时间复杂度。  
  • 递归需警惕栈溢出风险，迭代更可控。
