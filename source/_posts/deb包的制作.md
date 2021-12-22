---
title: 如何制作deb包
date: 2021-10-13
categories:
  - Linux
tags:
  - dpkg
cover: https://tva1.sinaimg.cn/large/008i3skNly1gvh264sogbj60cb06ljrb02.jpg
---

1. 准备工作：
``` shell
mkdir extract  #存放解包后的文件
mkdir extract/ex #存放deb包的控制信息
```
2. 解包命令
``` shell
#解压出xxx包中的文件放到extract目录
dpkg -X xxx.deb ./extract/
#解压出xxx包中的控制信息，放到extract/ex/目录中
dpkg -e xxx.deb extract/ex/
```
3. 解包后的目录结构
``` text
第一部分：opt目录，存放的是软件的相关文件
第二部分：ex目录，deb的控制信息，主要分为下面几个文件
    1.control(必须有):描述软件包的`Package`,版本`Version`，描述`Description`
    --->具体的信息：
    其中可能会有下面的字段：
-- Package 包名
-- Version 版本
-- Architecture：软件包结构，如基于i386, amd64,m68k, sparc, alpha, powerpc等
-- Priority：申明软件对于系统的重要程度，如required, standard, optional, extra等
-- Essential：申明是否是系统最基本的软件包（选项为yes/no），如果是的话，这就表明该软件是维持系统稳定和正常运行的软件包，不允许任何形式的卸载（除非进行强制性的卸载）
-- Section：申明软件的类别，常见的有utils, net, mail, text, devel 等
-- Depends：软件所依赖的其他软件包和库文件。如果是依赖多个软件包和库文件，彼此之间采用逗号隔开
-- Pre-Depends：软件安装前必须安装、配置依赖性的软件包和库文件，它常常用于必须的预运行脚本需求
-- Recommends：这个字段表明推荐的安装的其他软件包和库文件
-- Suggests：建议安装的其他软件包和库文件
-- Description：对包的描述
-- Installed-Size：安装的包大小
-- Maintainer：包的制作者，联系方式等

    2.postinst:软件进行正常的目录文件拷贝后，需要执行的配置工作，
    比如：目录创建，文件权限设置，文件信息拷贝
    3.prerm:卸载软件前需要执行的脚本
    4.poetrm:卸载软件后，需要执行的脚本            


4. 制作包
`dpkg -b ./extract/ xxx.deb`
#dpkg -b 包目录 包名称

5. 其他命令
安装：`dpkg -i xxx.deb`
卸载: `dpkg -r xxx.deb`