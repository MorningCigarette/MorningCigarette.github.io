---
title: "Efcore Update-Database显示证书链是由不受信任"
description: 
slug: "EFCore-Notice"
date: 2024-08-07
categories:
    - EFCore

---
&nbsp;   在学习EF Core的时候使用Update-Database时候报错显示证书链是由不受信任的颁发机构颁发的：![](file:///C:/Users/llc/.config/joplin-desktop/resources/d49b0040d1e4421184a8afb4132c4680.png)

**错误描述：**

> &nbsp;       A connection was successfully established with the server, but then an error occurred during the login process. ([provider](https://so.csdn.net/so/search?q=provider&spm=1001.2101.3001.7020): SSL Provider, error: 0 - 证书链是由不受信任的颁发机构颁发的。)

**解决方法：**

直接在“[数据库](https://so.csdn.net/so/search?q=%E6%95%B0%E6%8D%AE%E5%BA%93&spm=1001.2101.3001.7020)连接字符串最后面”增加证书信任的配置。TrustServerCertificate=true![](file:///C:/Users/llc/.config/joplin-desktop/resources/ddea8e3fdb334db9b8dbfcb463b4e96e.png)

最终成功执行：![](file:///C:/Users/llc/.config/joplin-desktop/resources/f07eb6be10ec4a62b8dd3ac91b923df3.png)

&nbsp;