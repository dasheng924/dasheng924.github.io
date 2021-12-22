---
title: HEXO发生error:spawn failed错误的解决方法
date: 2021-05-18
tags:
  - hexo
  - git
categories:
  - Tools
cover: https://pic4.zhimg.com/v2-9c5e558982dbf8fc02cfdee6b001e101_1200x500.jpg
---

### 解决方法

- 删除根目录下面的`.deploy_git`

- 输入 `git config --global core.autocrlf false`
- 然后依次输入 `hexo clean && hexo g && hexo d`

 



