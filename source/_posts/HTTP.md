---
title: HTTP 简单使用
date: 2021-06-02
tags:
  - http
  - 协议
categories:
  - Tools
cover: https://tva1.sinaimg.cn/large/008i3skNly1gt9wxe3uylj309g0a0glj.jpg
---



### HTTP简介

​		HTTP 协议工作在应用层，端口号是80.HTTP协议用于两台计算机的通信，相较于TCP/IP这些底层协议，HTTP协议更像是高层次的标记语言，浏览器根据从服务器得到的HTTP响应体中得到的<b>报文头，响应头和信息体(HTML正文)</b>，然后将HTML解析显示在浏览器上。



​		当我们在浏览器上输入网址，浏览器会自动帮我们组织好<b>报文头，请求头，[信息体]</b>，之后通过网络发送给服务器，服务器根据请求的内容准备数据。一个完整的HTTP协议，需要一个WEB服务器和一个浏览器，一侧生成响应，一侧生成请求信息。

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt9u12g7jwj30hm0dkwep.jpg" alt="image-20210809001121638" style="zoom:33%;" />



​		HTTP在应用层，TCP协议为其提供网络传输服务，故在HTTP请求前，客户机和服务器之间一定通过了三次握手建立了连接，其中套接字服务器一侧的端口号是80。当HTTP协议承载于TLS或者SSL协议层之上，就形成了HTTPS。请求和传输的过程中，数据也是有区别的，一个页面上不只有文本数据，大多有图片的嵌套。HTTP1.0采用对页面的每一个文件都建立一次TCP连接，一次传输完断开连接，多次的操作来获取到所有数据，这种做法需要多次连接，效率会低很多。HTTP2.0采取传输完一个资源不断开，在同一个连接上一直传输数据，直到数据传输完毕，但长时间的占用服务器资源，降低了吞吐率。	

#### HTTPS

- HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。

- https和http的对比：
  - https需要ca证书，进行身份认证
  - http是超文本传输协议，信息是明文传输，https则是具有安全性的SSL加密传输协议
  - http的端口是80，https的端口是443
  - http的连接简单；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

------

### HTTP工作流程



1. 建立起客户端和服务器的连接
2. 建立连接后，客户机发送一个请求给服务器
3. 服务器接收到请求给予响应
4. 客户端浏览器将收到的消息进行解析

-------

#### HTTP的特点

- 支持C/S架构
- 简答快速，客户端向服务器请求服务时，只需要传送请求方法和路径
- HTTP允许传输任意类型的数据对象
- HTTP是无连接的，每次连接只能处理一个请求。**请求时建连接、请求完释放连接，以尽快将资源释放出来服务其他客户端**（这个在最开始的时代，网络请求大都以文字为主）；后面随着网站的图片越来月多，每次访问都需要建立一次TCP的连接效率比较低，keep-Alive就被提出来解决这个问题，**Keep-Alive 功能使客户端到服务器端的连接持续有效，当出现对服务器的后继请求时，Keep-Alive 功能避免了建立或者重新建立连接**。但对于负担较重的网络，长时间的占用连接，不断开，会使资源的使用更加拘谨。
- 无状态：是指协议对于事务处理没有记忆能力，服务器不知道客户端是什么状态。即我们给服务器发送 HTTP 请求之后，服务器根据请求，会给我们发送数据过来，但是，发送完，不会记录任何信息。缺少状态就导致要处理前面的信息，就需要重传。HTTP 协议这种特性有优点也有缺点，优点在于解放了服务器，每一次请求“点到为止”不会造成不必要连接占用，缺点在于每次请求会传输大量重复的内容信息。        

{% btn 'https://blog.csdn.net/tennysonsky/article/details/44562435',无连接｜无状态,far fa-hand-point-right %}

------

### HTTP协议结构

#### <b>请求报文，分为四个部分，每一行数据必须通过'\r\n'分割</b>

![image-20210809005541914](https://tva1.sinaimg.cn/large/008i3skNly1gt9vb784wcj30ye0dcwia.jpg)

![image-20210809005605695](https://tva1.sinaimg.cn/large/008i3skNly1gt9vbmeo7sj31z40rrq83.jpg)



#### **报文头**（只有一行）

格式： METHOD  URL  VERSION



```
METHOD:HTTP的请求方法，一共有9种，其中GET和POST使用频率。

GET：向特定资源发起请求，也能提交数据，数据以明文方式在URL中。

POST：向特定资源提交数据进行处理，数据在请求体中，较安全

URL：指代请求的文件路径

VERSION：HTTP协议的版本，有HTTP/1.0和 HTPP/1.1

```



#### **请求头**（多行）

在HTTP/1.1中，除了host都是可选的。



* HOST：指定请求资源的主机和端口号，默认80
* Connection:值为keep-alive和close。keep-alive使客户端到服务器的连接持续有效，不需要每次重新连接
* Accept：浏览器可接受的MIME类型。text/html表示服务器发回的数据类型为text/html。如果服务器无法返回这种类型，返回406错误。
* Cache-control：缓存控制，Public内容可以被任何缓存所缓存，Private内容只能被缓存到私有缓存。none-cache所有内容都不会被缓存
* Content-Length：请求消息正文长度

```tex
User-Agent 请求的浏览器类型
Accept "*"用于按范围将类型分组；“*/*”表示可以接受全部类型；“type/*“ 表示可以接受type类型的所有子类型
Accept-Language	客户端可接受的自然语言
Accept-Encoding	客户端可接受的编码压缩格式
Accept-Charset	可接受的应答的字符集
Host	请求的主机名，允许多个域名同处一个IP 地址，即虚拟主机
connection	连接方式(close或keepalive)
Cookie	存储于客户端扩展字段，向同一域名的服务端发送属于该域的cookie
```



* **小结请求报文是告知服务器请求的内容，而请求头提供了关于客户端浏览器的一些基本信息**            

  

#### 空行（一行）

#### **可选消息体**（多行）

- 请求包体不在GET方法中使用，而是POST方法中使用。
- POST方法适用于需要客户填写表单的场合。与请求包体相关的最常使用的是包体类型`Content-Type`和包体长度`Content-Length`。

--------

#### 响应报文

#### 	<b>响应报文是服务器对客户端请求资源的响应，数据也是以<b>'\r\n'</b>来分割</b>

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt9vlewiykj60qg0dugo102.jpg" alt="image-20210809010531249" style="zoom:80%;" />

![image-20210809010627230](https://tva1.sinaimg.cn/large/008i3skNly1gt9vmdzq4nj30wa0d6gn4.jpg)



#### **报文头（1行）**

​	格式：<b>version    status_code   status_message</b>


```
version:所遵循的HTTP版本

status_code：状态码，表明请求处理的状态，常见的如下

	202 成功
	301 内容已移动
	400 请求不能被服务器理解
	404 不能找到请求文件
	500 服务器内部错误
	501 服务器不支持请求的方法
	505 服务器不支持请求的版本
	
status_message:显示和状态码等价的英文描述

```

[更多的状态码，请点击查看](http://blog.csdn.net/tennysonsky/article/details/45332345)



#### **响应头（多行）**

```
只展示一部分：
Date:表示信息发送的时间
Server：Web 服务器用来处理请求的软件信息
Content-Encoding:Web服务器表明自己用什么压缩方式压缩对象
Content-Length:服务器告知浏览器自己响应的对象长度
Content-type:告知浏览器响应对象类型

```

#### **空行（1行）**

- 最后一个响应头部之后是一个空行，发送回车符和换行符，通知服务器以下不再有响应头部。

#### **响应包体（多行）**

- 服务器返回给客户端的文本信息。

-------

### HTTP通信过程

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt9wmnvxl3j30u00uugnc.jpg" alt="image-20210809014118996" style="zoom:67%;" />

```c
int main()
{
	// 创建通信端点：套接字
	int sockfd = socket(AF_INET, SOCK_STREAM, 0);
	if (sockfd < 0)
	{
		perror("socket");
		return -1;
	}

	// 设置本地地址结构体  
	struct sockaddr_in my_addr;
	bzero(&my_addr, sizeof(my_addr));   // 清空      
	my_addr.sin_family = AF_INET;   	// ipv4  
	my_addr.sin_port = htons(8000);   // 端口  
	my_addr.sin_addr.s_addr = htonl(INADDR_ANY); // ip  

	// 绑定  
	int err_log = bind(sockfd, (struct sockaddr*)&my_addr, sizeof(my_addr));
	if (err_log != 0)
	{
		perror("binding");
		close(sockfd);
		return -1;
	}

	listen(sockfd, 10); // 监听，监听套接字改为被动  

	printf("listen client @port=%d...\n", 8000);

	int connfd;
	connfd = accept(sockfd, NULL, NULL);    // 等待连接  

	char recv_buf[8 * 1024] = { 0 };
	read(connfd, recv_buf, sizeof(recv_buf));
	printf("%s", recv_buf);

	//获取客户端需要的网页内容
	char filename[200] = { 0 };
	sscanf(recv_buf, "GET /%[^ ]", filename); //获取文件名字
	printf("filename = %s\n", filename);

	int fd;
	fd = open(filename, O_RDONLY);//只读方式打开
	if (fd < 0)//打开文件失败
	{
		//HTTP 响应报文由状态行、响应头部、空行、响应包体4个部分组成
		char err[] = "HTTP/1.1 404 Not Found\r\n"	//状态行
			"Content-Type: text/html\r\n"			//响应头部
			"\r\n"							//空行
			"<HTML><BODY>File not found</BODY></HTML>";  //响应包体           

		perror("open");
		send(connfd, err, strlen(err), 0);//发送失败的响应报文头

		close(connfd);
		return -1;
	}

	//HTTP 响应报文由状态行、响应头部、空行、响应包体4个部分组成
	char head[] = "HTTP/1.1 200 OK\r\n" 	//状态行
		"Content-Type: text/html\r\n"		//响应头部
		"\r\n";						//空行
	send(connfd, head, strlen(head), 0); //发送成功的响应报文头

	//发送响应包体
	int len;
	char file_buf[4 * 1024];
	//循环读取并发送文件，读多少，发多少
	while ((len = read(fd, file_buf, sizeof(file_buf))) > 0)
	{
		send(connfd, file_buf, len, 0);
	}

	close(fd);
	close(connfd);

	while (1)
	{
		NULL;
	}

	return 0;
}
```



![image-20210809014225571](https://tva1.sinaimg.cn/large/008i3skNly1gt9wntl4guj30yw0gyq7e.jpg)

-------

### CGI和环境变量

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt9vvc8h1rj317w0u0mzu.jpg" alt="image-20210809011502986" style="zoom:80%;" />

#####  web服务器收到客户端(浏览器)的请求Http Request，启动CGI程序，并通过环境变量、标准输入传递数据

##### CGI进程启动解析器、加载配置(如业务相关配置)、连接其它服务器(如数据库服务器)、逻辑处理等

##### CGI进程将处理结果通过标准输出、标准错误，传递给web服务器

##### web服务器收到CGI返回的结果，构建Http Response返回给客户端，并杀死CGI进程



**web服务器与CGI通过环境变量、标准输入、标准输出、标准错误互相传递数据**

---------

#### CGI程序：放在服务器中的一段可执行程序

​	服务器为客户端提供动态服务首先需要得到用户提供的参数再根据参数返回信息。为了和客户端进行交互，服务器需要创建子进程，子进程执行相应的程序为客户端服务。CGI帮我们解决参数获取和结果输出等问题。通常在服务器中会留出一个单独的目录（cgi-bin）来存放所有的CGI程序。

--------

#### 在遇到用户连接请求：

##### 先要创建CGI子进程，然后CGI子进程处理请求，处理完事退出这个子进程：fork-and-execute

##### CGI方式是客户端有多少个请求，就开辟多少个子进程，每个子进程都需要启动自己的解释器、加载配置，连接其他服务器等初始化工作，这是CGI进程性能低下的主要原因。当用户请求非常多的时候，会占用大量的内存、cpu等资源，造成性能低下。

##### CGI使外部程序与Web服务器之间交互成为可能。CGI程序运行在独立的进程中，并对每个Web请求建立一个进程，这种方法非常容易实现，但效率很差，难以扩展。面对大量请求，进程的大量建立和消亡使操作系统性能大大下降。此外，由于地址空间无法共享，也限制了资源重用。

--------

#### 环境变量

CGI环境变量在创建时被初始化，结束时被销毁。当CGI程序被服务器调用时，服务器会fork出子进程，这个子进程继承了父进程的环境变量，这些环境变量包含了基本信息，请求头和响应头中的信息也在其中（客户端主机名，客户端IP，浏览信息），CGI参数也在其中

GET请求，它将数据打包放置在环境变量`QUERY_STRING`中，CGI从环境变量

`QUERY_STRING`中获取数据。

 

常见的环境变量如下表所示：

| 环境变数          | 含义                                                         |
| ----------------- | ------------------------------------------------------------ |
| AUTH_TYPE         | 存取认证类型                                                 |
| CONTENT_LENGTH    | 由标准输入传递给CGI程序的数据长度，以bytes或字元数来计算     |
| CONTENT_TYPE      | 请求的MIME类型                                               |
| GATEWAY_INTERFACE | 服务器的CGI版本编号                                          |
| HTTP_ACCEPT       | 浏览器能直接接收的Content-types, 可以有HTTP Acceptheader定义 |
| HTTP_USER_AGENT   | 递交表单的浏览器的名称、版本和其他平台性的附加信息           |
| HTTP_REFERER      | 递交表单的文本的URL，不是所有的浏览器都发出这个信息，不要依赖它 |
| PATH_INFO         | 传递给CGI程序的路径信息                                      |
| QUERY_STRING      | 传递给CGI程序的请求参数，也就是用"?"隔开，添加在URL后面的字串 |
| REMOTE_ADDR       | client端的host名称                                           |
| REMOTE_HOST       | client端的IP位址                                             |
| REMOTE_USER       | client端送出来的使用者名称                                   |
| REMOTE_METHOD     | client端发出请求的方法(如get、post)                          |
| SCRIPT_NAME       | CGI程序所在的虚拟路径，如/cgi-bin/echo                       |
| SERVER_NAME       | server的host名称或IP地址                                     |
| SERVER_PORT       | 收到request的server端口                                      |
| SERVER_PROTOCOL   | 所使用的通讯协定和版本编号                                   |
| SERVER_SOFTWARE   | server程序的名称和版本                                       |



```
GET下的参数获取：

	服务器把接受到的参数数据编码到环境变量QUERY_STRING，在请求时只需要直接把参数写到URL后面，如http:127.0.0.1:80/cgi-bin/test?a=1&b=2&c=3,表示请求cgi-bin目录下test程序。‘？’后面为参数，多个参数用‘&’分割。
	
CGI程序中用getenv()获取环境变量：

char * value = getenv("QUERY_STRING");


POST方法获取参数：

	POST方法下，CGI直接从服务器的标准输入获取数据，不过先要CONTENT_LENGTH这个环境变量中得到POST参数的长度

```

​		

#### stdin

环境变量的大小是有一定的限制的，当需要传送的数据量大时，储存环境变量的空间可能会

不足，造成数据接收不完全，甚至无法执行CGI程序。

 

因此后来又发展出另外一种方法：POST，也就是利用I/O重新导向的技巧，让CGI程序可以由`stdin`和`stdout`直接跟浏览器沟通。

 

当我们指定用这种方法传递请求的数据时，web服务器收到数据后会先放在一块输入缓冲区

中，并且将数据的大小记录在`CONTENT_LENGTH`这个环境变量，然后调用CGI程序并将

CGI程序的`stdin`指向这块缓冲区，于是我们就可以很顺利的通过`stdin`和环境变数`CONTENT_LENGTH`得到所有的信息，再没有信息大小的限制了。

----

#### FastCGI

##### 什么是FastCGI

- 快速通用网关接口(Fast Common Gateway Interface／FastCGI)是通用网关接口(CGI)的改进，描述了客户端和服务器程序之间传输数据的一种标准。
- FastCGI致力于减少Web服务器与CGI程式之间互动的开销，从而使服务器可以同时处理更多的Web请求。与为每个请求创建一个新的进程不同，FastCGI使用持续的进程来处理一连串的请求。这些进程由FastCGI进程管理器管理，而不是web服务器。

##### FastCGI 处理流程

![image-20210809012911243](https://tva1.sinaimg.cn/large/008i3skNly1gt9wa1szapj31bq0u0dit.jpg)

- Web 服务器启动时载入初始化FastCGI执行环境。
- FastCGI进程管理器自身初始化，启动多个CGI解释器进程并等待来自Web服务器的连接。启动FastCGI进程时，可以配置以ip和UNIX 域socket两种方式启动。
- 当客户端请求到达Web 服务器时， Web 服务器将请求采用socket方式转发FastCGI主进程，FastCGI主进程选择并连接到一个CGI解释器。Web 服务器将CGI环境变量和标准输入发送到FastCGI子进程。
- FastCGI子进程完成处理后将标准输出和错误信息从同一socket连接返回Web 服务器。当FastCGI子进程关闭连接时，请求便处理完成。
- FastCGI子进程接着等待并处理来自Web 服务器的下一个连接。

------

>由于FastCGI程序并不需要不断的产生新进程，可以大大降低服务器的压力并且产生较高的应用效率。它的速度效率最少要比CGI 技术提高 5 倍以上。它还支持分布式的部署，即
>
>FastCGI 程序可以在web 服务器以外的主机上执行。
>
> 
>
>CGI 是所谓的短生存期应用程序，FastCGI 是所谓的长生存期应用程序。FastCGI
>
>像是一个常驻(long-live)型的CGI，它可以一直执行着，不会每次都要花费时间去fork一次(这
>
>是CGI最为人诟病的fork-and-execute 模式)。



#### spawn-fcgi

##### 什么是spawn-fcgi

- Nginx不能像Apache那样直接执行外部可执行程序，但Nginx可以作为代理服务器，将请求转发给后端服务器，这也是Nginx的主要作用之一。其中Nginx就支持FastCGI代理，接收客户端的请求，然后将请求转发给后端FastCGI进程。
- 由于FastCGI进程由FastCGI进程管理器管理，而不是Nginx。这样就需要一个FastCGI进程管理器，管理我们编写FastCGI程序。
-  spawn-fcgi是一个通用的FastCGI进程管理器，简单小巧，原先是属于lighttpd的一部分，后来由于使用比较广泛，所以就迁移出来作为独立项目。
- spawn-fcgi使用pre-fork 模型，功能主要是打开监听端口，绑定地址，然后fork-and-exec创建我们编写的FastCGI应用程序进程，退出完成工作。FastCGI应用程序初始化，然后进入死循环侦听socket的连接请求。

##### spawn-fcgi 参数信息

spawn-fcgi参数信息：

| 参数           | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
| f <fcgiapp>    | 指定调用FastCGI的进程的执行程序位置                          |
| -a <addr>      | 绑定到地址addr                                               |
| -p <port>      | 绑定到端口port                                               |
| -s <path>      | 绑定到unix domain socket                                     |
| -C <childs>    | 指定产生的FastCGI的进程数，默认为5(仅用于PHP)                |
| -P <path>      | 指定产生的进程的PID文件路径                                  |
| -F <childs>    | 指定产生的FastCGI的进程数(C的CGI用这个)                      |
| -u和-g FastCGI | 使用什么身份(-u用户、-g用户组)运行，CentOS下可以使用apache用户，其他的根据情况配置，如nobody、www-data等 |



##### 测试

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include "fcgi_stdio.h"

int main(int argc, char *argv[])
{
	int count = 0;

	//阻塞等待并监听某个端口，等待Nginx将数据发过来
	while (FCGI_Accept() >= 0)
	{
		//如果想得到数据，需要从stdin去读，实际上从Nginx上去读
		//如果想上传数据，需要往stdout写，实际上是给Nginx写数据
		printf("Content-type: text/html\r\n");
		printf("\r\n");
		printf("<title>Fast CGI Hello!</title>");
		printf("<h1>Fast CGI Hello!</h1>");
		//SERVER_NAME：得到server的host名称
		printf("Request number %d running on host <i>%s</i>\n", ++count, getenv("SERVER_NAME"));
	}

	return 0;
}
```

- `gcc test.c -o test -lfcgi`

- `spawn-fcgi -a 127.0.0.1 -p 8001 -f ./test`

- ```ini
  #监听用户的test请求，通过fastcgi_pass交给本地8001端口处理
  #此时spwan-cgi已经将8001端口交给之前我们写好的test进程处理
  location /test {
  	fastcgi_pass 127.0.0.1:8001;
  	fastcgi_index test;
  	include fastcgi.conf;
  }
  ```

- 当Nginx收到http://localhost/test请求时，会匹配到location test块，将请求传到后端的FastCGI应用程序处理：
