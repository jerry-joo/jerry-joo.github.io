---
title: linux
mathjax: true
categories: 
tags: 
---
# Linux权限管理

<!--more-->

## windows wsl
用vscode在wsl中无法直接操作文件，每次都需要sudo，

解决方法：
```
#查看当前用户
whoami 
#给用户分配文件夹操作权限
sudo chown -R username /path
```