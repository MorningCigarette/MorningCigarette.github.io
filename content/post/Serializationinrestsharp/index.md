---
title: "System.Text.Json 和 Newtonsoft.Json 默认序列化时的大小写行为差异"
description: 
slug: "Serializationinrestsharp"
date: 2025-10-11
categories:
    - C#
---

在使用 **RestSharp** 时，很多人会遇到一个常见问题：**序列化成 JSON 时字段的大小写被改变（比如属性名变成小写或 camelCase）**。这通常与 **RestSharp 默认使用的 JSON 序列化器（System.Text.Json 或 Newtonsoft.Json）** 有关。

| 序列化器                                 | 默认属性命名策略          | 结果示例                | 说明                         | 关闭大小写变化的方法                               |
| ---------------------------------------- | ------------------------- | ----------------------- | ---------------------------- | -------------------------------------------------- |
| **System.Text.Json** (.NET Core 3+ 默认) | ✅ camelCase（首字母小写） | `UserName` → `userName` | 更符合前端 JavaScript 习惯   | `options.PropertyNamingPolicy = null`              |
| **Newtonsoft.Json** (Json.NET)           | ❌ 保留原有大小写          | `UserName` → `UserName` | 更传统，更适合后端系统间通信 | `ContractResolver = new DefaultContractResolver()` |

 
