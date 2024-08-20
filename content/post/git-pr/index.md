---
title: "如何提交PR（Pull Request）"
description: 
slug: "git-pr"
date: 2024-08-20
categories:
    - Git

---

# **前言**

本文尽量使用图形工具介绍如何向开源项目提交 Pull Request，一次亲身经历提交 PR

### fork 项目**

![](https://s2.loli.net/2024/08/20/yUnX7F4sY9OuxZP.webp)

image

### 克隆本地**

![](https://s2.loli.net/2024/08/20/rlQEXRqTsn3LWfK.webp)

image

```php
git clone https://github.com/liangtongzhuo/taro-ui.git
```

### 根据文档创建分支**

拖进 SourceTree，基于 dev 创建分支

  

![](https://s2.loli.net/2024/08/20/qT9eUzQAwbP74ml.webp)

image

### 提交的自己仓库**

```csharp
git add . && git commit -m"fix(dos):修正文字 " && git push

```

### 提交 Pull Request**

第四步提交成功后，进入原来的仓库，点击 compare

  

![](https://s2.loli.net/2024/08/20/CT1VYWo68I4GPwJ.webp)

image

提交你的说明，选择合并的分支即可，剩下等待合并。

  

![](https://s2.loli.net/2024/08/20/hPlkA2E7KDTfNmo.webp)

image

# **结尾**

开源的魅力就在于协同工作，提高效率。

  

![](https://s2.loli.net/2024/08/20/Yv2X4VGLxWCPtAm.webp)
