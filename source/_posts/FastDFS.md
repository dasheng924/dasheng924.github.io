---
title: FastDFS配置与使用
date: 2021-05-09
cover: https://tva1.sinaimg.cn/large/008i3skNly1gsenfdjj13j31bi0u0kjw.jpg
tag:
  - FastDFS
  - 文件系统
categories:
  - Tools
keywords: "fdfs"
---

### 分布式文件系统

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsenbno4kdj30rr0isacq.jpg" style="zoom:67%;" />

#### 文件系统 - 存储数据

#### {% label 分布式 blue %}- 分布式存储简单的来说，就是将数据分散存储到多个存储设备(服务器)上。

- 它可以看做是基于文件的 key/value 存储系统，key 为文件 ID，value 为文件内容，因此称作分布式文件存储服务更为合适。
- 属于应用级文件系统，通过API进行访问，该文件系统通过C语言进行实现。可以实现冗余备份，负载均衡

### 数据库和文件系统的区别：

#### 文件系统：存储数据

- 数据库：把一些数据组织成一定的表结构放到文件系统里面（云盘的数据会放到这个文件系统里面）；数据库里面放的是用户和密码和用户对应的所属文件名称（也就是文件存在的记录，不是实体文件）

#### 文件系统分为三个角色：{% label 客户端client pink %}  {% label 追踪器tracker pink %}   {% label 存储节点storage pink %}。  tracker和storage属于服务器端

#### 三个角色之间的关系

![image-20210712005744350](https://tva1.sinaimg.cn/large/008i3skNly1gsdi0z6m4rj30s40d2ta0.jpg)





- Tracker Server 与 Storage Server 之间不直接通信，其基本的信息由配置文件在系统启动加

  载时获知。

- client和storage会主动联系tacker，storage会主动向tracker汇报信息(l跟踪器主要做调度工作，在访问上起负载均衡的作用)

  - 磁盘剩余情况
  - 文件同步情况
  - 文件上传和下载次数

- storage会启动一个单独的线程和tacker进行沟通(存储节点存储文件，完成文件管理的所有功能)

  - 连接和定时报告信息

- 一个组包含的storage不是通过配置文件设定的，而是通过tracker获取到的         

  

  #### stacker集群就是轮询的工作方式(多台 Tracker Server 之间保证了 Tracker 的分布式，Tracker Server 之间是对等的，防止了单点故障)

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gseaf5pxp9j30go0ba3za.jpg" alt="image-20210712172020225" style="zoom: 67%;" />

- 一个组里的数据容量取决于最小的节点存储大小，各个节点之间就是数据备份。这个就是纵向扩容。横向扩容就是再加组的个数，增加容量的大小。每个 Group 中的Storage 都是互相备份的，也就是说，如果 Group1 有 Storage1、Storage2、Storage3，其容量分别是100GB、200GB、300GB，那么 Group1 的存储能力是 100GB，而不是 300GB，这就是互相备份的意思。
- 横向扩容：增加容量--添加group组的个数,可以使数据流量足够分散
- 纵向扩容：数据备份，当前组容量的最大值是存储节点的最小值；所有存储节点的组名是一样的。



### FastDFS 的安装

- {% label 第一步 pink %}：如果是FastDFS是2.0之前的版本，请先安装好libevent环境（新版本不需要安装）。

  {% btn 'http://libevent.org',libevent下载地址,far fa-hand-point-right %}

- {% label 第二步 pink %}：libfastcommon为FastDFS操作的基础库，FastDFS没有使用标准的库，而是封装了一套和FastDFS相关的库。

  {% btn 'https://github.com/happyfish100/libfastcommon',libfastcommon下载,far fa-hand-point-right %}

  >
  >
  >1.解压 tar -xzvf libfastcommon-1.0.7.tar.gz
  >
  >2.进入解压目录，./make.sh
  >
  >3.sudo ./make.sh install 
  >
  >4./usr/lib64/ 查看生成的动态库
  >
  >5.若使用过程中找不见动态库，sudo ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so

- {% label 第三步 pink %}：下载源码

  {% btn 'l https://github.com/happyfish100/fastdfs.git',FastDFS下载地址,far fa-hand-point-right %}

- {% label 解压，编译，运行 pink %}

  >tar -zxvf xxx
  >
  >./make.sh
  >
  >sudo ./make.sh install
  >
  >/usr/bin/fdfs* 看安装成功的可执行文件               

### FastDFS 配置

#### tracker配置

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gseo4u623gj310i0d4n4h.jpg" style="zoom:67%;" />

```tex
#配置文件目录 /etc/fdfs/
#先把原有的后缀为 .sample 的文件拷贝一份，变成后缀为 .conf
#依次对配置文件修改
//1.修改IP，为空绑定主机的所有地址
bind_addr=10.211.55.11
//2.设置Tracker Server 的端口，sudo netstat ano | grep "22122" 
port=22122
//3.log	输出目录和仓库数据存储位置
base_path=/home/parallels/fdfs/tracker
```

#### storage配置

```tex
//1.组名设置，默认OK
group_name=group1
//2.storage所在服务器的IP
bind_addr=10.211.55.11
//3.storage的端口
port=23000
//4.storage日志目录
base_path=/home/parallels/fdfs/storage
//5.真正存储数据的位置
store_path0=/home/parallels/fdfs/storage
//6.tracker的IP和端口
tracker_server=10.211.55.11:22122
//7.数据仓库的数量
store_path_count=1
```

#### client的配置

```tex
base_path=/home/parallels/fdfs/client
#  "host:port", host can be hostname or ip address
tracker_server=10.211.55.11:2212
```

### 启动服务

#### tracker的启动fdfs服务

```bash
sudo fdfs_trackerd /etc/fdfs/tracker.conf
```

![image-20210713013340673](https://tva1.sinaimg.cn/large/008i3skNly1gseooe3g2oj31ga03ego8.jpg)

#### storage启动服务

```bash
sudo fdfs_storaged /etc/fdfs/storage.conf
ps aux|grep -v grep|grep fdfs
```



### 客户端启动

#### storage状态检测

```bash
fdfs_monitor /etc/fdfs/client.conf
```

![image-20210713014542308](https://tva1.sinaimg.cn/large/008i3skNly1gsep0x1rhdj31xb0u0amk.jpg)

#### storage 状态分类

|   状态    | 含义                                   |
| :-------: | -------------------------------------- |
|   INIT    | 初始化，尚未得到同步已有数据的源服务器 |
| WAIT_SYNC | 等待同步，已得到同步已有数据的源服务器 |
|  SYNCING  | 同步中                                 |
|  DELETED  | 已删除，该服务器从本组中摘除           |
|  OFFLINE  | 离线                                   |
|  ONLINE   | 在线，尚不能提供服务                   |
|  ACTIVE   | 在线，可以提供服务                     |

### 客户端使用

#### 文件上传

```tex
fdfs_upload_file /etc/fdfs/client.conf xxxxx
```

![文件上传](https://tva1.sinaimg.cn/large/008i3skNly1gsepdneejoj321c0cwn1f.jpg)

![仓库矩阵](https://tva1.sinaimg.cn/large/008i3skNly1gsepii22q8j31wk0aeq9k.jpg)

![文件存放具体位置](https://tva1.sinaimg.cn/large/008i3skNly1gsepjiymvvj313s04kju7.jpg)
- 启动tracker，storage会启动，然后定时返回storage的存储信息，包括剩磁盘余情况，磁盘分布情况
- client 发出上传请求
- tracker查询可用的storage信息
- 返回可用的storage的IP和端口
- 开始进行socket通信，上传文件
- storage生成file_id 的信息
- 将上传的信息写入磁盘
- storage返回对应的file_id（路径信息和文件名）
- 存储文件信息（程序员需要把这些信息放到数据库）


#### 文件下载

```bash
fdfs_download_file /etc/fdfs/client.conf  文件唯一串（上图那个）
```

![文件下载](https://tva1.sinaimg.cn/large/008i3skNly1gseph9jvv5j3218084797.jpg)

- storage定时向tracker发送状态信息
- 下载连接请求
- 查询可用的storage
- 返回storage的IP和端口
- 根据file_id（组名，路径，文件名），查找文件
- 返回file_content
  
#### 文件删除

```bash
fdfs_delete_file /etc/fdfs/client.conf 文件唯一串
```

![文件删除](https://tva1.sinaimg.cn/large/008i3skNly1gseplzyxloj322a08e442.jpg)

