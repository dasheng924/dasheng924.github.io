---
title: shell脚本简单总结
date: 2020-05-18
tag:
  - 基础
  - shell
categories:
  - Linux
keywords: "shell,脚本"
cover: https://tva1.sinaimg.cn/large/008i3skNly1gsa3299d7jj30c106lt8p.jpg
---
### 1.shell脚本的执行

```shell
1.给脚本赋予执行权限后，./xxx.sh

2. . xxx.sh

3. source xxx.sh
```

### 2.shell变量

```shell
1.环境变量

	printenv 显示当前shell进程下的环境变量


2.本地变量

	设置本地变量 VAR=10 VAR1="hello"
	使用本地变量 $VAR  $VAR1
	本地变量变成环境变量   export VAR=10
	取消本地变量 unset VAR

set 查看shell进程的本地变量和环境变量

没有定义的变量取值是空字符串

```

### 3.shell代换

#### 3.1文件名代换

```shell
* 匹配0个或多个  ls a*
？ 匹配一个		ls a?
[]列表中的任意一个字符的一次匹配 ls a[123]   ls a[1-3]

匹配的文件名是由shell展开的，还没有传给程序之前，已经展开了，传递的是文件名，不是字符串
```

#### 3.2命令代换

```shell
1. `命令`，如 DATE=`date`
2. $(命令)，如 DATE=$(date)
```

#### 3.3 算术代换

```shell
1. $(()),如 VAR=10，$(($VAR+1)),输出11

2. $[],如 VAR=10，$[VAR+1]（或者 $[$VAR+1] ），输出11

	$[base#n],以多少进制输出，如 $[2#10+11]代表2进制的10加上10进制的11


```

#### 3.4转义代换

```shell
输出原意字符
echo \$VAR,输出$VAR

echo \\,输出\

1. 如创建一个文件，名字为$ $
	 touch \$\ \$

特殊：-

2.如创建一个文件，名字为 -hello

	touch -- -hello.  OR    touch ./-hello


3. \ 还有续行的含义

```

### 4.单引号和双引号

```shell 
1.''   保持引号内字符的字面值

	如： echo '$VAR' ,out: $VAR

	    echo  'abc\(enter)de',out: abc\(enter)de
	  
2."" 被视为单一字符串，允许变量扩展

	如：
		DATE=$(date)
		echo "DATE" ,out:输出时间
		echo 'DATE' ,out:输出 DATE 字面值
```

### 5.条件测试

```shell
1.  test

	eg: var=2
		test $var -gt 1
		echo $?  (查看上一个进程的返回值，0为真，1位假)

2. []

	eg: 
		[$var -gt 1]
	
	
3.
	-d 文件
	-f 普通文件
	-z string string长度为0，则为真
	-n string string长度非0为真
	=  相同为真
	!= 不同为真

	ARG1 OP ARG2：
		OP：
			-eq 等于
			-ne 不等于
			-lt 小于
			-gt 大于
			-ge 大于等于
			-le 小于等于
		
	！ 非
	-a 与
	-o 或

		

```

### 6.分支

```shell
if [ condition1 ];then
	cmd1
elif [ condition2 ];then
	cmd2
else
	cmd3
fi


-----------------------------------------------------------------------------------

case "$VAR" in
condition1)
	cmd1;;
conition2)
	cmd2;;(多条命令，最后一条命令加;;)
esac
```

### 7.循环

```shell
for VAR in list;do
	cmd
done

-----------------------------------------------------------------------------------
while [ conition1 ];do
	cmd
done


break[n]可以指定跳出几层循环；continue跳过本次循环，但不会跳出循环
```

### 8.位置参数和特殊变量

```shell
$0 -->argv[0]
$1,$2...  --->位置参数, argv[1],argv[2]...
$# ---> argc-1
$@ ---> 参数列表
$* ---> 参数列表
$? ---> 上一条命令的退出值
$$ ---> 当前进程号
```

### 9.echo

```shell
显示文本或者变量，把字符串输入到文件

-e 解析转义字符
-n 不回车换行，默认后面跟一个回车换行

```

### 10.管道

```shell
| 把前面命令的输出做为后面命令的输入

cat myfile | more 

ls -l | grep "myfile"

df -k | awk `{print $1}`

```

### 11.文件重定向

```shell
cmd > file 把标准输出重定向到新文件中
cmd >> file 追加

cmd > file 2>&1 标准出错也重新定向到1所指的file中   所有的东西都进入文件file
cmd >> file 2>&1

cmd < file1 > file2 从file1读，写到file2中

cmd < &fd 把文件描述符fd作为标准输入，从文件中读
cmd > &fd  把文件描述符fd作为标准输出，输入到文件

cmd < &- 关闭标准输入


```
