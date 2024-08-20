---
title: "git add . 和 git add --all 的区别"
description: 
date: 2024-08-20T11:15:15+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
---
git add . 会把当前目录及子孙目录里的变动都加到暂存区；而 git add --all 会将项目里所有文件的变动都加到暂存区，也就是说该命令不论在项目的哪级目录执行，都有同样的效果。