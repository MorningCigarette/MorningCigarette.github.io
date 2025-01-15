---
title: "Newtonsoft反序列化Json常用方式的性能对比"
description: 
slug: "DeserializeJson"
date: 2025-01-15
categories:
   

---

## 反序列化Json常用方式

1. `JsonConvert.DeserializeObject<dynamic>(string)`
2. `JObject.Parse(string)`



​    这里采用enchmarkDotNet进行测试，我们直接上代码

```
#load "BenchmarkDotNet"

void Main()
{
    RunBenchmark(); return;  // Uncomment this line to initiate benchmarking.
}

string complexJsonString = @"
        {
            ""id"": 12345,
            ""name"": ""Sample Name"",
            ""isActive"": true,
            ""details"": {
                ""description"": ""This is a sample description."",
                ""createdAt"": ""2025-01-13T00:00:00Z"",
                ""tags"": [""example"", ""performance"", ""test""]
            },
            ""metrics"": [1, 2, 3, 4, 5]
        }";

string easyString = @"{
            ""id"": 12345,
            ""name"": ""Sample Name""
        }";
[Benchmark]
public void JsonConvert_DeserializeObject_complexJsonString() => JsonConvert.DeserializeObject<dynamic>(complexJsonString);
[Benchmark]
public void JObject_Parse_complexJsonString() => JObject.Parse(complexJsonString);

[Benchmark]
public void JsonConvert_DeserializeObject_easyString() => JsonConvert.DeserializeObject<dynamic>(easyString);
[Benchmark]
public void JObject_Parse_easyString() => JObject.Parse(easyString);


[GlobalSetup]
public void BenchmarkSetup()
{
    // optional setup code...
}
```

​    我们来看结果
![34ac03add12f281099b443508ac96f39.png](https://s2.loli.net/2025/01/15/ZoL7KkCu1UvYEQR.png)

## 结论

​    可以看出**不论是复杂Json还是简单Json，JObject.Parse都是最优选择**
