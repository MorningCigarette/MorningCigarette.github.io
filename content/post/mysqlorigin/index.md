---
title: "Windows开启远程访问MySQL"
description: 
slug: "mysqlorigin"
date: 2024-08-20
categories:
    - 小技巧

---

## 进入MySQL安装目录 \\bin 下

![](https://s2.loli.net/2024/08/20/ujTIHxi1GU29JPz.png)

在目录输入”cmd“然后回车

## 第二步 **开启远程访问权限**

在步骤一执行成功后的mysql>下，依次输入命令 use mysql、update user set host = '%' where user = 'root';、flush privileges;  **（分号别漏了）**

```
use mysql
```

```
update user set host = '%' where user = 'root';
```

```
flush privileges;
```

![](https://s2.loli.net/2024/08/20/my4CzBkaQIX3UxZ.png)
