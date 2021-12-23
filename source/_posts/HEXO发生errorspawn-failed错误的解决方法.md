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

 

### 解决办法二

- 在博客的根目录输入`git remote set-url origin git@github.com:名字/仓库名.git`

### 重置SSH

#### 1.重置用户名和密码

```bash
git config --global user.name "yourname"
git config --global user.email "your@email.com"
```

#### 2.删除.ssh 目录下面的`known_hosts`

### 3.生成新的秘钥

`ssh-keygen -t rsa -C "your@email.com"`

#### 4.github设置你的ssh public_key

#### 5.git 测试连接

`ssh -T git@github.com`
