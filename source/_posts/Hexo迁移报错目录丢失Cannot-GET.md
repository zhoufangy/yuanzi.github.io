title: Hexo迁移报错目录丢失Cannot GET /
author: Yuanzi
tags:
  - Hexo
categories:
  - Hexo
date: 2020-02-15 11:56:00
---
#### 问题
迁移Hexo后部署主页丢失，访问目录报Cannot GET /。index页面可单独访问

#### 解决办法
检查缺失组建
npm audit fix
更新
npm install