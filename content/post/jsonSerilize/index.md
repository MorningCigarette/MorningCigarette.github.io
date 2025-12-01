---
title: "避免Newtonsoft.Json反序列化出现重复列表项"
description: 
slug: "jsonSerilize"
date: 2025-12-01
categories:
    - C#
---

## 问题描述

我们公司的项目是一个基于.NET的Web应用，它需要和一些第三方的API交互数据。为了方便地处理JSON格式的数据，我们使用了Newtonsoft.Json这个框架，它可以让我们轻松地将.NET对象序列化为JSON字符串，或者将JSON字符串反序列化为.NET对象。  
在项目中，我们定义了一些实体类来表示我们需要处理的数据。其中有一个类叫做Product，它有三个属性：Property，MemberVariable和Array。Property是一个列表类型的属性，它有一个get和set方法。MemberVariable是一个列表类型的成员变量，它没有get和set方法。Array是一个数组类型的成员变量，它也没有get和set方法。在Product的构造函数中，这三个属性都被初始化为一些整数值。代码如下：

```
public class Product
{
public List<int> Property { get; set; }
public List<int> MemberVariable;
public int[] Array;
public Product()
{
Property = new List<int>() { 1, 2, 3 };
MemberVariable = new List<int>() { 4, 5, 6 };
Array = new int[] { 7, 8, 9 };
}
}
```

在项目中，我们需要将Product对象序列化为JSON字符串，并发送给第三方的API。然后，我们需要从第三方的API接收JSON字符串，并反序列化为Product对象。我们使用了Newtonsoft.Json的SerializeObject和DeserializeObject方法来实现这个功能。代码如下：

```
Product product = new Product();
string json = JsonConvert.SerializeObject(product);
// 发送json给第三方API
// 接收json从第三方API
Product deserializedProduct = JsonConvert.DeserializeObject<Product>(json);
```

按照我们的预期，当我们序列化Product对象时，应该得到如下的JSON字符串：

```
{"Property":[1,2,3],"MemberVariable":[4,5,6],"Array":[7,8,9]}
```

当我们反序列化JSON字符串时，应该得到一个和原来相同的Product对象。  
然而，在测试过程中，我们发现了一个奇怪的现象。当我们反序列化JSON字符串时，得到的Product对象并不是和原来相同的。它的Property和MemberVariable属性的值都被重复了，而Array属性的值没有变化。例如：

```
{"Property":[1,2,3,1,2,3],"MemberVariable":[4,5,6,4,5,6],"Array":[7,8,9]}
```

这就是我们遇到的问题：为什么使用Newtonsoft.Json反序列化时，列表项会被重复呢？

## 问题分析

为了找出问题的原因，我们首先查看了Newtonsoft.Json的文档，并搜索了一些相关的资料 。我们发现了以下几点：

- Newtonsoft.Json在反序列化时，会先调用对象的构造函数，然后再根据JSON字符串中的键值对来设置对象的属性或字段。
- 如果对象的属性或字段已经在构造函数中被初始化了，那么反序列化时就会把JSON字符串中的值追加到原来的值上，而不是替换掉它们。
- 这种行为只会发生在列表类型的属性或字段上，因为列表类型有一个Add方法，可以把新的值添加到列表中。而数组类型没有这个方法，所以反序列化时会直接替换掉原来的值。  
    根据这些信息，我们就可以理解为什么我们的问题会发生了。当我们反序列化JSON字符串时，Newtonsoft.Json会先调用Product的构造函数，这时Property和MemberVariable属性就被初始化为一些整数值。然后，Newtonsoft.Json会根据JSON字符串中的键值对来设置Property和MemberVariable属性，这时它会把JSON字符串中的值添加到原来的值上，导致列表项重复。而Array属性没有这个问题，因为它是一个数组类型，反序列化时会直接替换掉原来的值。

## 问题解决

既然我们知道了问题的原因，那么我们就可以想办法解决它了。在网上搜索后，我们发现了几种可能的方法：

- 一种方法是在对象的构造函数中不要初始化属性或字段，而是在属性的get方法中检查是否为空，并在需要时进行初始化。这样就可以避免在反序列化时调用构造函数导致的重复。例如：

```
public class Product
{
private List<int> _property;
public List<int> Property
{
get
{
if (_property == null)
{
_property = new List<int>() { 1, 2, 3 };
}
return _property;
}
set
{
_property = value;
}
}
// 省略其他属性和构造函数
}
```

- 另一种方法是使用\[JsonIgnore\]特性来忽略不需要序列化或反序列化的属性或字段。这样就可以避免在反序列化时设置对象的属性或字段导致的重复。例如：

```
public class Product
{
[JsonIgnore]
public List<int> Property { get; set; }
// 省略其他属性和构造函数
}
```

- 还有一种方法是使用\[ObjectCreationHandling\]设置来控制反序列化时对象的创建方式。这个设置有三个选项：Auto，Reuse和Replace。Auto是默认的选项，它会根据对象的类型来决定是重用还是替换。Reuse表示会重用已经存在的对象，并把JSON字符串中的值追加到对象上。Replace表示会创建一个新的对象，并把JSON字符串中的值赋给对象。这样就可以避免在反序列化时追加列表项导致的重复。例如：

```
Product deserializedProduct = JsonConvert.DeserializeObject<Product>(json, new JsonSerializerSettings
{
ObjectCreationHandling = ObjectCreationHandling.Replace
});
```

我们根据项目的实际需求，选择了第三种方法，并修改了代码。经过测试，完美解决问题，反序列化后得到的Product对象和原来相同了。
