---
title: nginx简单使用
date: 2021-04-01
tags: 
  - nginx
categories:
  - Tools
cover: https://tva1.sinaimg.cn/large/008i3skNly1gsn6el6z53j30cs031jrc.jpg
---

### nginx 

- http服务器

  - 能够处理http协议

- 反向代理服务器

- 邮件服务器

  - IMAP/POP3/SMTP

  ### nginx 优势

  - 更快-数以万计的并发时，nginx可以比其他web服务器更快的响应
  - 高扩展-低耦合的设计
  - 高可靠-大量的网站都在使用
  - 低内存的消耗-对内存的消耗很小
  - 热部署-提供不间断的服务器升级

### nginx 的相关资源

- {% btn 'http://nginx.org',nginx维护包,far fa-hand-point-right,outline %}
- {% btn 'http://tengine.taobao.org/documentation_cn.html',nginx中文文档,far fa-hand-point-right,outline %}

### nginx 相关配置

#### ubuntu 的软件依赖项

> {% label gcc,g++编译器 green%}
>
> `sudo apt-get install gcc`
>
> `sudo apt-get install g++`
>
> `sudo apt-get install build-essential`
>
> `sudo apt-get install libtool`          
>
> {% label pcre库 green %}
>
> {% btn 'ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/',pcre包下载,far fa-hand-point-right,outline %}
>
> - `./configure`
> - `make`
> - `sudo make install`
>
> {% label zlib green %}
>
> {% btn 'http://zlib.net/',zlib,far fa-hand-point-right,outline %}
>
> - `./configure`
> - `make`
> - `sudo make install`
>
> {% label openssl green %}
>
> {% btn 'https://www.openssl.org/source/',openssl,far fa-hand-point-right,outline %}
>
> - `./configure`
> - `make`
> - `sudo make install`



### nginx相关操作

- 进入到安装目录
  - `/usr/local/nginx/sbin/`       
- 启动
  - `sudo ./nginx`         
  - `sudo ./nginx -c nginx配置文件`
- 停止
  - `sudo ./nginx -s stop`  --暴力
  - `sudo ./nginx -s quit` --柔和
- 重新加载
  - `sudo ./nginx -s reload`    
- 测试配置信息
  - `sudo ./nginx -t`

### nginx 的相关配置

- nginx 的配置文件由主配置文件和辅助配置文件构成

![nginx 配置文件](https://tva1.sinaimg.cn/large/008i3skNly1gsn5k0xyvlj31u605mwhg.jpg)

- nginx 的配置文件结构

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsn5mnaftoj31720u0dhg.jpg" alt="image-20210720092437398" style="zoom: 25%;" />



> `main`:	nginx运行时的参数，与业务没有关系，如工作进程数
>
> `http`: 提供与http服务相关的参数
>
> `server`: http 服务支持多个server，每一个server都可以有自己的配置项
>
> `locatio`: 	 http服务中，对URL对应的配置，包括资源访问路径，文件，命令

### nginx 配置静态页面

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsn62hs5pxj60qe0b00t502.jpg" alt="image-20210720093954168" style="zoom: 50%;" />



- `root	`: 放置静态网页的根目录
- `index`: 要访问的网页资源

### nginx 的反向代理配置

- 反向代理：指代理服务器来接受internet的请求，然后将请求转发给内部的服务器，服务器的结果也是通过代理给客户端。	代理服务器来均衡的分配和协调服务器资源的使用。

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsn5s56w73j31rs0u0tbz.jpg" alt="image-20210720092956542" style="zoom:33%;" />



<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsn5wazi96j30zq0u0n20.jpg" alt="image-20210720093356778" style="zoom:25%;" />





