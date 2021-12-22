---
title: 网络模型
date: 2020-06-20
tag:
  - 网络
  - 底层
categories:
  - Linux
keywords: "底层,网络"
cover: https://tva1.sinaimg.cn/large/008i3skNly1gsa1d6d63dj30nx0d9tan.jpg
---
#### 1.OSI 七层模型

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsa0oyuoylj618m0so48z02.jpg" style="zoom:60%;" />

* 1. 物理层：网线的类型，光纤的接口，网卡等硬件设备，主要作用：传输比特流（1，0转化为电流的强弱），这一层的数据叫做{% label 比特 blue %}
  2. 数据链路层：物理层面两个主机间的通信传输，将比特流划分为帧传输给对端数据单元为{% label 帧 pink %}
  3. 网络层：在网络上将数据传输到目的地址，负责寻址和路由的选择，传输路径的选择,数据单元为{% label 包 pink %}
  4. 传输层：提供数据的传输服务，在端到端之间进行数据传输，数据单元为{% label 段 pink %}
  5. 会话层：管理和协调不同主机上面各个进程间的通信，管理端到端的连接
  6. 表示层：数据格式的转换，如编码，格式转换，加密，解密
  7. 应用层：应用程序和网络之间的接口，直接向用户提供服务。比如电子邮件协议，远程登录协议，FTP协议

#### 2.TCP/IP四层模型

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsa1d6d63dj30nx0d9tan.jpg" style="zoom:80%;" />

##### 1.网络接入层：

```
对网络介质的管理，以及定义如何使用网络来通信。设备之间通过物理介质互连，互连的设备通过 {% btn 'https://baike.baidu.com/item/MAC地址',MAC地址,far fa-hand-point-right %}实现数据的传输。采用MAC地址就是为了识别同一个网络上面的设备。
```


##### 2.网络层：

```
使用IP协议，根据IP地址进行数据包的转发，也就是把数据包从源地址发送到目的地址。路由器就是通过网络层实现转发数据包的功能。每个节点会根据地址的信息来判断该报文由那个网卡发出来的。MAC寻址中会参考MAC地址转发表，IP寻址会参考路由控制表。
```


![](https://tva1.sinaimg.cn/large/008i3skNly1gsa1ul4uf3j60gq05cdg702.jpg)

* IP：IP作为主机的标识，使整个互联网都能收到数据。IP协议独立于底层介质，实现从源到目的端的数据传输。iP协议不具有重发机制，属于非可靠的传输协议，需要依靠上层的TCP协议来实现。
* {% btn 'https://baike.baidu.com/item/ICMP',ICMP,far fa-hand-point-right %}用于在IP主机中，路由之间传递控制消息的协议，用来诊断网络的健康状况。
* {% btn 'https://baike.baidu.com/item/ARP/609343',ARP,far fa-hand-point-right %}从数据包的IP地址中解析出MAC地址。

##### 3.传输层：

```
主要功能就是让应用程序直接相互通信，通过端口号进行识别应用程序。使用面向连接的{% label TCP协议 blue %}和面向无连接的{% label UDP协议 blue %}.面向连接就是打电话，需要双方建立连接，确认对方存在且正常。面向无连接就是寄信，只需要知道一个地址，不需要确认对方是否存在，也不需要确认对方是否会收到信。
```


* {% btn 'https://baike.baidu.com/item/TCP/33012',TCP,far fa-hand-point-right %} 面向连接的，可靠的流式传输协议,用于文件传输
* {% btn 'https://baike.baidu.com/item/UDP',UDP,far fa-hand-point-right %}面向无连接的，不可靠的报式传输协议，用于视频通话，直播

##### 5.应用层：

```
常见的协议都是应用层的协议，如HTTP，POP3，FTP
```


#### 4. 封装和解装

![封装](https://tva1.sinaimg.cn/large/008i3skNly1gsa2m2jc84j30u008lq4g.jpg)

数据发送之前，按照参考模型从上面到下面都会封装各自层的信息，包括数据和各层的协议信息。

![解封包](https://tva1.sinaimg.cn/large/008i3skNly1gsa2o7p4kyj30u008lwg0.jpg)

按照模型，从下到上，每过一层就去掉一层协议信息，到最后的应用层就会拿到数据
