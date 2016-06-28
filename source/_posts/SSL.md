---
title: IntelliJ IDEA SVN 更新报错
date: 2016-05-23 11:14:27
categories: [IntelliJ IDEA,SVN]
tags: [IntelliJ IDEA,SVN]
visitors:
---
## Error:Server SSL certificate rejected
### 问题
更新报错，SSL失效
<!-- more -->
### 解决
SVN 设置 -> 已保存数据 ->清除认证数据 -> 更新 -> 输入账户信息

![证物](http://zhoufangyuan.me/images/svnSet.png)
