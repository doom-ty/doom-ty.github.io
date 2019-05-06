---
title: Linux常用命令笔记
date: 2019-03-19 09:04:28
tags:
    - linux
categories: Linux  
---
#### 1、释放cache缓存
```shell
echo 3 > /proc/sys/vm/drop_caches
```