---
title: "Efcore Update-Database显示证书链是由不受信任"
description: 
slug: "EFCore-Notice"
date: 2024-08-07
categories:
    - EFCore

---
&nbsp;   在学习EF Core的时候使用Update-Database时候报错显示证书链是由不受信任的颁发机构颁发的：

```
A connection was successfully established with the server, but then an error occurred during the login process. (provider: SSL Provider, error: 0 - 证书链是由不受信任的颁发机构颁发的。)
```

**解决方法：**

直接在数据库连接字符串最后面”增加证书信任的配置。

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("server=***;Database=***; user=***; Password=***;MultipleActiveResultSets = True;TrustServerCertificate=true");
        base.OnConfiguring(optionsBuilder);
    }

最终成功执行：

```
PM> Update-database
Build started...
Build succeeded.
Done.
```