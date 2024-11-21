---
title: "SourceTree使用教程"
description: 
slug: "gitskill"
date: 2024-11-15
categories:
    - Git

---

## 前言：

俗话说的好工欲善其事必先利其器，Git分布式版本控制系统是我们日常开发中不可或缺的。目前市面上比较流行的Git可视化管理工具有SourceTree、Github Desktop、TortoiseGit，综合网上的一些文章分析和自己的日常开发实践心得个人比较推荐开发者使用SourceTree，因为SourceTree同时支持Windows和Mac，并且界面十分的精美简洁，大大的简化了开发者与代码库之间的Git操作方式。该篇文章主要是对日常开发中使用SourceTree可视化管理工具的一些常用操作进行详细讲解。

## SourceTree | Github Desktop | TortoiseGit 可视化管理工具对比：

>  https://blog.csdn.net/hmllittlekoi/article/details/104504406/

## SourceTree介绍和Atlassian账号注册和登录教程：

> https://www.cnblogs.com/Can-daydayup/p/13128511.html

## 连接Gitee or GitHub，获取代码:

**注意：这里介绍的是使用SSH协议获取关联远程仓库的代码，大家也可以直接使用过HTTPS协议的方式直接输入账号密码获取关联代码！**

### 全面概述Gitee和GitHub生成/添加SSH公钥：

> https://www.cnblogs.com/Can-daydayup/p/13063280.html

### 在SourceTree中添加SSH密钥：

**工具=>选择：**

![img](https://s2.loli.net/2024/11/15/b4q9pwtYcZSHLVW.png)

 

**添加SSH密钥位置：C:\Users\xxxxx\.ssh\id_rsa.pub：**

![img](https://s2.loli.net/2024/11/15/usHvRrWzE2yPotD.png)

**SSH客户端选择OpenSSH：**

![img](https://s2.loli.net/2024/11/15/VNYaA1EnvS7yZqT.png)

### Clone对应托管平台仓库（以Gitee为例）：

**打开码云，找到自己需要Clone的仓库！**

![img](https://s2.loli.net/2024/11/15/89OvVyKjcu4B7xg.png)![img](https://s2.loli.net/2024/11/21/QNRk7flcY3qOtAX.png)![img](https://s2.loli.net/2024/11/15/UnzM3BVZhvHCeGq.png)

## SourceTree设置默认工作目录：

由上面我们可以发现每次Clone克隆项目的时候，克隆下来的项目默认存储位置都是在C盘，因此每次都需要我们去选择项目存放的路径，作为一个喜欢偷懒的人而言当然不喜欢这种方式啦，因此我们可以设置一个默认的项目存储位置。

### 设置SourceTree默认项目目录：

**点击工具=>选项=>一般=>找到项目目录设置Clone项目默认存储的位置：**

![img](https://s2.loli.net/2024/11/21/kYqo8iUSJBaWHwd.png)

## SourceTree代码提交：

### 首先切换到需要修改功能代码所在的分支：

![img](https://s2.loli.net/2024/11/15/wcD2a4nK8dAFJBp.png)

![图片](https://s2.loli.net/2024/11/15/xVbGHNhsID4mOyJ.png)

### 将修改的代码提交到暂存区：

![img](https://s2.loli.net/2024/11/15/KujbJ6MYvlh71sn.png)

### 将暂存区中的代码提交到本地代码仓库：

**注意：多人同时开发项目的时候，不推荐默认选中立即推送变更到origin/develop，避免一些不必要的麻烦！**

![img](https://s2.loli.net/2024/11/15/1d4OKkCRj7LYWB6.png)

### 代码拉取更新本地代码库，并将代码推送到远程仓库：

![img](https://s2.loli.net/2024/11/15/VvGWX4AZhszLJPa.png)

 **勾选需要推送的分支，点击推送到远程分支：**

![img](https://s2.loli.net/2024/11/15/5gb4uspM3nWVNZq.png)

 

**代码成功推送到远程代码库：**

![img](https://s2.loli.net/2024/11/15/nEOaRj5iyD1zGYM.png)

### 在Gitee中查看推送结果：

![img](https://s2.loli.net/2024/11/15/OA9DEqKYFU4kifm.png)

## SourceTree分支切换，新建，合并：

### 分支切换：

**双击切换：**

![img](https://s2.loli.net/2024/11/15/pFgtQ2J4GrSdU75.png)

**单击鼠标右键切换：**

![图片](https://s2.loli.net/2024/11/15/VbKr1zlxhEoJ9XH.png)

### 新建分支：

**注意：在新建分支时，我们需要在哪个主分支的基础上新建分支必须先要切换到对应的主分支才能到该主分支上创建分支，如下我们要在master分支上创建一个feature-0613分支：**

![img](https://s2.loli.net/2024/11/15/uzWCAF8hJp1Kqt2.png)![img](https://s2.loli.net/2024/11/15/h9qdToGs6jFRMQB.png)

### 合并分支:

**注意：在合并代码之前我们都需要将需要合并的分支拉取到最新状态（\**避免覆盖别人的代码，或者丢失一些重要文件）!!!**

在master分支上点击右键，选择合并feature-0613至当前分支即可进行合并:

![img](https://s2.loli.net/2024/11/15/to2uxwHLUhpv9fg.png)

分支合并成功：

![img](https://s2.loli.net/2024/11/15/hqctx82zp9ZaSM1.png)

## SourceTree代码冲突解决：

### 首先我们需要制造一个提交文件遇到冲突的情景：

在SoureceTree中在Clone一个新项目，命名为pingrixuexilianxi2，如下图所示：

![img](https://s2.loli.net/2024/11/15/dVJ27x5SzeniETc.png)

我们以项目中的【代码合并冲突测试.txt】文件为例：

![img](https://s2.loli.net/2024/11/15/XBqOQDEylId7ogk.png)

 

在pingrixuexilianxi2中添加内容，并提交到远程代码库，添加的内容如下：

![img](https://s2.loli.net/2024/11/15/UHXVatQdO95ZNDi.png)

在pingrixuexilianxi中添加内容，提交代码（不选择立即推送变更到origin/master），拉取代码即会遇到冲突：

![img](https://s2.loli.net/2024/11/15/xTZh9QkoCqSDK78.png)![img](https://s2.loli.net/2024/11/15/s7zcxuhONKPUADS.png)

 ![图片](https://s2.loli.net/2024/11/15/Qy6SJZ5dkhLDsRE.png)

**冲突文件中的内容：**

![img](https://s2.loli.net/2024/11/15/6wSv1YLoz2VXWIR.png)

### 直接打开冲突文件手动解决冲突：

由下面的冲突文件中的冲突内容我们了解到：

```
`<<<<<<< HEAD``6月19日 pingrixuexilianxi添加了内容``=======``6月18日 pingrixuexilianxi2修改了这个文件哦``>>>>>>> a8284fd41903c54212d1105a6feb6c57292e07b5`
`<<<<<<< HEAD到 =======里面的【6月19日 pingrixuexilianxi添加了内容】是自己刚才的Commit提交的内容``=======到 >>>>>>> a8284fd41903c54212d1105a6feb6c57292e07b5里面的【6月18日 pingrixuexilianxi2修改了这个文件哦】是远程代码库更新的内容（即为pingrixuexilianxi2本地代码库推送修改内容）。`
```

 

手动冲突解决方法：

> 根据项目需求删除不需要的代码就行了，假如都需要的话我们只需要把 <<<<<<< HEAD=======   >>>>>>> a8284fd41903c54212d1105a6feb6c57292e07b5都删掉冲突就解决了（注意，在项目中最后这些符号都不能存在，否则可能会报异常）。

 

最后将冲突文件标记为已解决，提交到远程仓库：

![img](https://s2.loli.net/2024/11/15/vnFNBDE3kKT1zfp.png)

### 采用外部文本文件对比工具Beyond Compare解决冲突:

#### **SourceTree配置文本文件对比工具Beyond Compare:**

工具=>选项=>比较：

 ![图片](https://s2.loli.net/2024/11/15/vS9uGwNBYiPThay.png)

![img](https://s2.loli.net/2024/11/15/PKdi9FBqZxLp1lk.png)

#### 使用Beyond Compare解决冲突：

Beyond Compare使用技巧：

官方全面教程：https://www.beyondcompare.cc/jiqiao/

SourceTree打开外部和合并工具：

![img](https://s2.loli.net/2024/11/21/47O6tSJTkbi5gcI.png)

**注意：第一次启动Beynod Compare软件需要一会时间，请耐心等待：**

***\**\*![图片](https://s2.loli.net/2024/11/15/5yspA9x1UmECcGJ.png)\*\**\***

Beynod Compare进行冲突合并：

![img](https://s2.loli.net/2024/11/21/Cm7QiNljaPDoV1n.png)

点击保存文件后关闭Beynod Compare工具，SourceTree中的冲突就解决了，在SourceTree中我们会发现多了一个 .orig 的文件。接着选中那个.orig文件，单击右键 => 移除，最后我们推送到远程代码库即可：

![img](https://s2.loli.net/2024/11/15/Xi2LCZwuEGHa7c8.png)

## Sourcetree中的基本名词说明：

> 克隆/新建(clone)：从远程仓库URL加载创建一个与远程仓库一样的本地仓库。
>
> 提交(commit)：将暂存区文件上传到本地代码仓库。
>
> 推送(push)：将本地仓库同步至远程仓库，一般推送（push）前先拉取（pull）一次，确保一致（十分注意：这样你才能达到和别人最新代码同步的状态，同时也能够规避很多不必要的问题）。
>
> 拉取(pull)：从远程仓库获取信息并同步至本地仓库，并且自动执行合并（merge）操作（git pull=git fetch+git merge）。
>
> 获取(fetch)：从远程仓库获取信息并同步至本地仓库。
>
> 分支(branch)：创建/修改/删除分枝。
>
> 合并(merge)：将多个同名文件合并为一个文件，该文件包含多个同名文件的所有内容，相同内容抵消。
>
> 贮藏(git stash)：保存工作现场。
>
> 丢弃(Discard)：丢弃更改,恢复文件改动/重置所有改动,即将已暂存的文件丢回未暂存的文件。
>
> 标签(tag)：给项目增添标签。
>
> 工作流(Git Flow)：团队工作时，每个人创建属于自己的分枝（branch）��确定无误后提交到master分支。
>
> 终端(terminal)：可以输入git命令行。
>
> 每次拉取和推送的时候不用每次输入密码的命令行：git config credential.helper osxkeychain sourcetree。
>
> 检出(checkout)：切换不同分支。
>
> 添加（add）：添加文件到缓存区。
>
> 移除（remove）：移除文件至缓存区。
>
> 重置(reset)：回到最近添加(add)/提交(commit)状态。

## Git分布式版本控制器常用命令和使用：

当然作为一个有逼格的程序员， 一些常用的命令我们还是需要了解和掌握的，详情可参考我之前写过的文章：

> https://www.cnblogs.com/Can-daydayup/p/10134733.html

## SourceTree如何提交PR(Pull Request)：

Pull Request提交相关操作参考该篇文章：

> https://www.jianshu.com/p/b365c743ec8d

### fork 项目：

![img](https://s2.loli.net/2024/11/21/DOxV29PK4ZnqsAc.png)

 

### 克隆本地

![img](https://s2.loli.net/2024/11/21/dfTeBGkIJ15Pql2.png)

 打开Git Bash输入仓库克隆命令：

```
git clone https://github.com/liangtongzhuo/taro-ui.git
```

### 根据文档创建分支

拖进 SourceTree，基于 dev 创建分支如下图：

![img](https://s2.loli.net/2024/11/15/teqPI4bYFUHR1rZ.png)

 

### 提交修改的代码到远程代码库

文章上面已经提到了使用SourceTree提交的相关操作，可参考：

https://www.cnblogs.com/Can-daydayup/p/13128633.html#\_label5（或者Ctrl F：SourceTree代码提交）

当然也可以使用git命令提交：

```
`git add .  --提交所有修改的文件到本地暂存区``git commit -m"fix(dos):修正文字 "   --提交到本地代码库``git push  --提交到github中的远程代码库`
```

### 提交 Pull Request

第四步提交成功后，进入原来fork的仓库，点击 Compare

![img](https://s2.loli.net/2024/11/15/2kwyM8ApsPLOmIu.png)

 提交你的说明，选择合并的分支即可，剩下等待合并。

![img](https://s2.loli.net/2024/11/15/G1gVLm4wCk6jUAX.png)
