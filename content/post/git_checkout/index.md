---
title: "Git checkout 签出或切换分支"
description: 
slug: "git_checkout"
date: 2024-11-21
categories:
    - Git
---

# Git checkout 签出或切换分支

本章节是对git checkout 命令的介绍。在Git的术语中，“checkout”是指在目标实体的不同版本之间切换的行为。git checkout命令操作三个不同的实体：文件、提交和分支。除了“checkout”的定义之外，短语“签出”通常用于暗示执行`git checkout`命令的行为。

在[git 撤消更改和提交](https://www.jiyik.com/w/git/git-undoing-changes)一篇中，我们看到了如何使用 git checkout 查看旧提交。 本文档大部分内容的重点将是分支的 “checkout” 操作。

检出分支类似于检出旧的提交和文件，因为工作目录会更新以匹配选定的分支； 但是，新的更改会保存在项目历史记录中——也就是说，它不是只读操作。

------

## 签出分支

git checkout 命令允许我们在 [git branch](https://www.jiyik.com/w/git/git-branch) 创建的分支之间进行切换。 签出分支会更新工作目录中的文件以匹配该分支中存储的版本，并告诉 Git 记录该分支上的所有新提交。 将其视为选择你正在从事的开发工作线的一种方式。

为每个新功能都新建一个专门的分支是对传统 SVN 工作流程的巨大转变。 它使尝试新实验变得非常容易，而不必担心破坏现有功能，并且可以同时处理许多不相关的功能。 此外，分支机构还促进了多个协作工作流程。

git checkout 命令有时可能会与 [git clone](https://www.jiyik.com/w/git/git-clone) 混淆。 这两个命令之间的区别在于 clone 用于从远程仓库获取代码，或者 checkout 用于在本地系统上已有的代码版本之间切换。

------

## git checkout 用法

下面我们看一下 git checkout 常见的两种用法

### 已经存在的分支

假设你正在使用的仓库已经创建了几个分支，那么可以使用 git checkout 在这些分支之间进行切换。 要找出哪些分支可用以及当前分支名称是什么，请执行 [git branch](https://www.jiyik.com/w/git/git-branch)。

```
$ git branch
```

![git branch 查看的分支列表](https://s2.loli.net/2024/11/21/md8cF1EW2koIQUT.jpg)

绿色的分支 master 表示的是我们当前所在的活跃分支。下面我们是用 git checkout 命令切换到 jiyik 分支

```
$ git checkout jiyik

$ git branch
```

![git checkout 切换分支](https://s2.loli.net/2024/11/21/v8LlZgw47MybBGI.jpg)

我们看到，现在绿色的分支为 jiyik。说明我们当前切换到了jiyik分支。

### 新分支

Git checkout 与 [git branch](https://www.jiyik.com/w/git/git-branch) 协同工作。 git branch 命令可用于创建新分支。 当你想开发一个新功能时，可以使用 `git branch new_branch` 在 master 之外创建一个新分支。 创建后，可以使用 `git checkout new_branch` 切换到该分支。 此外， git checkout 命令后面可以跟一个 `-b` 参数，这样就不用使用 git branch命令新建分支了。它会自动创建一个新分支并立即切换到这个分支。 我们可以使用 git checkout 在单个仓库中切换多个功能分支之间进行切换来处理这些功能。

```
git checkout -b <new-branch>
```

下面我们使用上面给出的命令创建一个新的分支，首先我们看一下当前有哪些分支

```
$ git branch
```

![git checkout 创建新分支之前查看当前的分支](https://s2.loli.net/2024/11/21/YHaxQDdMruL9Uic.jpg)

接下来使用git checkout 创建分支并切换到它

```
$ git checkout -b new_branch
$ git branch
```

![git checkout 创建新分支并切换到了新的分支](https://s2.loli.net/2024/11/21/8s9dCWR6kuPBHEO.jpg)

默认情况下 `git checkout -b` 将基于当前 HEAD 指向的分支创建新分支。 一个可选的附加分支参数可以传递给 git checkout。

```
git checkout -b <new-branch> <existing-branch>
```

在上面的例子中，git checkout 命令后面跟着 <existing-branch> 参数，然后新分支基于该指定的分支而不是当前 HEAD。

### git checkout 签出远程分支

与团队合作时，通常使用远程仓库。 这些仓库可能是托管和共享的，也可能是其他同事的本地副本。 每个远程仓库都将包含自己的一组分支。 为了签出远程分支，我们必须首先获取分支的内容。

```
$ git fetch --all
```

然后我们看一下本地分支和远程分支的差异

```
$ git branch
$ git branch -r
```

![git checkout 本地分支和远程分支](https://s2.loli.net/2024/11/21/gdALBZptHoyOXGr.jpg)

在现代版本的 Git 中，您可以像签出本地分支一样签出远程分支。

```
$ git checkout jiyik_branch
```

![git checkout 签出远程分支](https://s2.loli.net/2024/11/21/GXAOpgBhU7jVsIq.jpg)

如果你的Git版本比较旧，则需要使用下面的命令

```
$ git checkout -b jiyik_branch origin/jiyik_branch
```

此外，您可以签出一个新的本地分支并将其重置为上次提交的远程分支。这里需要用到 [git reset 命令](https://www.jiyik.com/w/git/git-reset)

```
$ git checkout -b new_branch
$ git reset --hard origin/new_branch
HEAD is now at f88bed1 增加迹忆客教程地址
```

------

## 分离 HEAD 状态

现在我们已经看到了 git checkout 在分支上的三个主要用途，讨论“分离的 HEAD”状态很重要。

> `记住` - HEAD 是 Git 引用当前快照的方式。

在内部， git checkout 命令只是更新 HEAD 以指向指定的分支或提交。 当它指向一个分支时，Git 不会有什么变化，但是当你检出一个提交时，它会切换到“分离的 HEAD”状态。

这是一个警告，告诉我们所做的一切都与项目开发的其余部分“分离”。 如果你在处于“分离的 HEAD 状态”时开始开发功能，则不会有任何分支允许你返回到它。 当不可避免地检查另一个分支（例如，将我们的功能合并到其中）时，将无法引用我们的功能：

git checkout分离的HEAD状态

关键是，你的开发应该总是在一个分支上进行——而不是在一个独立的 HEAD 上。 这确保您始终可以引用新提交。 但是，如果您只是查看旧提交，那么您是否处于分离的 HEAD 状态并不重要。

------

## git checkout 总结

本章节主要介绍更改分支时 git checkout 命令的使用。 总之，git checkout 在分支上使用时会改变 HEAD 引用的目标。 它可用于创建分支、切换分支和检出远程分支。 git checkout 命令是标准 Git 操作的必备工具。 它和 [git merge](https://www.jiyik.com/w/git/git-merge) 是对应的。 git checkout 和 git merge 命令是启用 git 工作流的关键工具。
