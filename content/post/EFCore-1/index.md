---
title: "EFcore"
description: 
slug: "EFCore"
date: 2024-08-09
categories:
    - EFCore
 
---
## 安装
* EF.Sqlsever（sqlserver数据库提供程序）
* EF Tools-->EF Design(安装到启动项)
## 跟踪查询
默认情况下，跟踪返回实体类型的查询。 跟踪查询意味着对实体实例所做的任何更改都由 SaveChanges持久化。 在以下示例中，检测到对博客分级的更改，并在 期间 SaveChanges保存到数据库中：
**Blogs上一定要指定主键**
``` 
var blog = context.Blogs.SingleOrDefault(b => b.BlogId == 1);
blog.Rating = 5;
context.SaveChanges();
```

在跟踪查询中返回结果时，EF Core 会检查实体是否已在上下文中。 如果 EF Core 找到现有实体，则返回相同的实例，这可能使用更少的内存，并且比无跟踪查询更快。 EF Core 不会使用数据库值覆盖条目中实体属性的当前值和原始值。 如果在上下文中找不到实体，EF Core 会创建一个新的实体实例并将其附加到上下文。 查询结果不会包含任何已添加到上下文但尚未保存到数据库中的实体。
## 非跟踪查询
在只读方案中使用结果时，非跟踪查询十分有用。 通常可以更快速地执行非跟踪查询，因为无需设置更改跟踪信息。 如果不需要更新从数据库检索到的实体，则应使用无跟踪查询。 单个查询可以设置为不跟踪。 无跟踪查询还会根据数据库中的内容提供结果，而不考虑任何本地更改或添加的实体。

``` 
var blogs = context.Blogs
    .AsNoTracking()
    .ToList();
```

	
## 字段部分更新

```
var thiscarcom = await db.CarCommons.AsNoTracking()
.FirstOrDefaultAsync(f => f.CarName == thiscarbase.CarName);
db.CarCommons.Attach(thiscarcom);   
db.Entry(thiscarcom).Property(p => p.TaskStatus).IsModified = true;
db.SaveChanges();
```

``` 
db.Attach(item);
db.Entry(item).State = EntityState.Modified;//将所有字段设置为要更新状态
db.Entry(item).Property(p => p.TaskStatus).IsModified = false
db.Entry(item).Property(p => p.RobotStatus).IsModified = false;
db.Entry(item).Property(p => p.ArmStatus).IsModified = false;
db.Entry(item).Property(p => p.CurrentTraName).IsModified = false;
db.Entry(item).Property(p => p.CanLedTask).IsModified = false;  
db.Entry(item).Property(p => p.Current_map).IsModified = false; 
db.SaveChanges();								   
```

