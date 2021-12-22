---
title: git学习
date: 2020-08-21
tag:
  - 基础
  - git
categories:
  - Tools
keywords: "git,版本控制"
cover: https://tva1.sinaimg.cn/large/008i3skNly1gsau9qmjxvj30pa0ak757.jpg
---
{% btn 'https://www.bilibili.com/video/BV19E411f76x/',直接看视频学习,far fa-hand-point-right %}

### 什么是git?

1. 分布式
2. 版本控制
3. 软件

- 针对这三个名词的解释

> + **软件**：git是一个软件，需要你安装在电脑本地才可以使用。
> + **版本控制**：你的一篇论文，需要反复修改和保留历史版本，比如论文有修改版，完成版，终极完成版。做好版本控制，有利于回滚和修改
> + **分布式**：版本不但在本地存在一份，远端也完全存在一份一样的。每一个安装了git的主机上都有一个版本备份，这个备份包括所有的历史版本和修改版本。

- 安装git

```
参见：
```

[安装git](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)

### 我接下来要干啥？

1. 新建一个文件夹，或者进入你要做版本控制的目录
2. 执行初始化命令

   ``git init ``
3. 查看目录下的文件状态

   `git status`(新增和修改过的文件是红色的)
4. 管理指定文件

   `git add 文件名`

   `git  add . `
5. 个人信息的配置：用户名和邮箱

   `git config --global user.name "your name"`

   `git config --global user.email "you@example.com"`
6. 生成版本

   `git commit -m '版本描述信息'`
7. 查看版本记录

   `git log `

   `git log --graph`以图形化的形式查看版本记录

   `git log --graph --pretty=format:"%h %s"`简略版的版本记录，包括哈希值和版本描述

### 我想回退指定版本？

1. 回滚到之前版

   ```
   git log
   git reset --hard 版本号   
   ```
2. 回滚到之后版本

   ```
   git reflog
   git reset --hard 版本号
   ```

### 一个master分支不够怎么办？

* 分支可以给我们提供另外一个环境，从master主线脱离出来，可以形成master专注于上线版本，新的dev 分支可以用于开发分支。

> **基本的命令：**
>
> * 查看分支
>
>   ```
>   git branch
>   ```
> * 创建新的分支
>
>   ```
>   git branch 分支名称
>   ```
> * 切换分支
>
>   ```
>   git checkout 分支名称
>   ```
> * 分支合并（可能产生冲突）
>
>   ```
>   git merge 要合并的分支
>   ```
> * 删除分支
>
>   ```
>   git branch -d 分支名称
>   ```

### 我想去公司继续搞代码？

- 注册GitHub账号
- 在GitHub新建一个仓库，记下仓库的地址

#### 去公司之前你需要先把代码上传到自己的GitHub

```
1.给你的GitHub仓库起一个别名
	git remote add origin 远程仓库地址
2.向GitHub推送你的代码
	git push -u origin 分支（后面可以简写成git push,这样默认的是master）
```


#### 到公司了，是一台新电脑，我该怎么办？

```
1.克隆远程仓库
	git clone 远程仓库名称（内部已经实现了给远程仓库起别名）

2.切换分支
	可以直接切换分支，本地新电脑已经有了所有分支
	git checkout 分支名称
```

* 下面展现你在公司新电脑上的一套命令流程：

  ```
  1.切换到dev分支进行开发
  	git checkout dev
  2.把master分支合并到dev分支（只需要做一次，以保持dev分支的最新版本）
  	git merge master
  3.修改代码
  4.提交代码到远程仓库
  	git add .
  	git commit -m '版本信息'
  	git push origin dev (提交dev分支)
  ```

#### 下班回家继续开发

* 下面展现你回到家继续写代码

  ```
  1.切换到dev分支进行开发
  	git checkout dev
  2.从远程仓库拉下代码
  	git pull origin dev
  3.继续开发

  4.提交代码
  	git add .
  	git commit -m '版本描述'
  	git push origin dev
  ```

#### 第二天又上班去

下面展示你第二天上班需要做的事情

```
1.切换到dev 分支进行开发
	git checkout dev
2.拉最新代码，这是一个更新的步骤，不需要再去clone
	git pull origin dev 
3.继续开发

4.提交代码
	git add .
	git commit -m '版本描述'
	git push origin dev 
```

#### 开发完毕要上线

```
1.将dev 分支合并到master分支
	git checkout master
	git merge dev
	git push origin master

2.把dev推到远程仓库
	git checkout dev
	git merge master
	git push origin dev
```

### 约女票吃大餐，公司代码忘提交，回家该咋办？

#### 吃完大餐，在家继续工作

```
1.拉代码
	git pull origin dev
2.继续开发新功能

3.长记性，一定提交代码
	git add .
	git commit -m '版本描述'
	git push origin dev
```

#### 第二天起床去公司赶紧拉昨天的代码

```
1.拉代码，把昨天晚上在家写的代码拉到本地(昨天在公司写的代码和昨晚在家写的代码之间有合并，冲突)
	git push origin dev
2.有冲突解决冲突

3.继续开发剩余功能

4.把dev分支一定记得推到远端
	git add .
	git commit -m '版本描述'
	git push origin dev
		>上面一句命令有两个功能：拉下来和合并，使用下面两句命令可以替换
		git fetch origin dev
		git merge origin/dev
```

### 借助软件快速解决冲突的方法？

1. 安装beyond compare
2. 在git配置

   ```
   git config --global merge.tool bc3
   git config --global mergetool.path '/usr/local/bin/bcomp'
   git config --global meregtool.keepBackup false
   ```
3. 使用beyond compare解决冲突

   ```
   git mergetool
   ```

### 使用Tag替代commit，完成版本管理

```
git tag -a v1.0 -m '版本描述'            创建本地Tag信息
git tag -d v1.0  删除Tag
git push origin --tags 	将本地Tag推送到远端仓库
git pull origin --tags  更新本地Tag版本信息

git checkout v1.0  	切换Tag					
git clone -b v1.0 	指定Tag下载代码								
```

### 杂项

* fork ,开源项目源代码拷贝到远程仓库
* 给源代码的作者提交bug申请（pull request)
* .gitignore文件，让git不再管理当前目录下的某些文件，参见[忽略文件](https://github.com/github/gitignore)
* issues,文档以及任务管理
* Wiki，项目文档

### 感谢

本文档主要参考{% btn 'https://www.bilibili.com/video/BV19E411f76x/',B站 老男孩视频,far fa-hand-point-right %}

视频原笔记：[原文档参考i3um](https://pan.baidu.com/s/1-E8_Kcn8i8tYCRT6vepMzw "密码i3um")
