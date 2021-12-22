---
title: select和poll
date: 2021-06-18
Tags:
  - select
  - poll
categories: 
  - Linux
cover: https://tse3-mm.cn.bing.net/th/id/OIP-C.WM3IDwBeyCfAsEvqXQ_gTQAAAA?pid=ImgDet&rs=1
---

### 处理并发的方式

- 多进程
- 多线程
- IO多路复用
  - select
  - poll
  - epoll

>### 多进程/多线程 对比 IO多路复用
>
>- 多进程/多线程
>  - 主进程/主线程：`accept()`进行阻塞等待客户端的连接，有客户端连接就去建立连接，同时创建新的进程/线程来解决通信问题
>    - 子进程/子线程：主要用来和新建立的客户端进行通信，`write`和`send`发送数据给客户端，当写缓冲区满了之后，就阻塞当前线程/进程；`read`和`recv`接受客户端发来的消息，读缓冲区没有数据积极进行阻塞。             
>
>- IO多路复用的方式
>  - 委托内核进行文件描述符的监控（包括建立连接的文件描述符和进行通信的文件描述符），这个检测过程会在调用IO多路复用的函数处进行阻塞。如果有文件描述符就绪状态就解除阻塞，内核一般会返回活跃的文件描述符。
>  - 在IO多路复用的基础上，`accept()`，`recv(),read(),send(),write()`不会进行阻塞了。
>- 对比：
>  - IO多路复用让系统不必维护进程/线程的创建和销毁，这样就可以减小系统的开销。

### IO多路复用三种方式的对比

- `select`是跨平台的，`poll`和`epoll`只能在Linux使用
- `select`和`poll`底层是一个线性表，`epoll`底层是一个红黑树
- `select`连接数限制在1024个,`poll`和`epoll`连接数由用户配置

### 多个文件描述符的监听

- 文件描述符的读写检测由程序员转向了内核，内核也可以同时检测多个文件描述符，提高效率

### 解决一下端口复用的问题

```c
#include <sys/types.h>
#include <sys/socket.h>

// 设置套接字属性
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
	参数:
	- sockfd: 要操作的文件描述符
	- level: 级别 -> SOL_SOCKET (端口复用的级别)
        - optname: 端口复用的级别(二选一，随便选，对于端口复用都可以)
        	- SO_REUSEADDR
        	- SO_REUSEPORT
        - optval: 端口复用-> 对应的是整形数
        	- 1: 可以复用
        	- 0: 不能复用
        - optlen: 对参数4取地址
          
int num=1;          
setsockopt(sfd,SOL_SOCKET,SO_REUSEADDR,num,&num);
 
// 设置端口复用, 设置的时机: 服务器绑定端口之前, 设置端口复用
```

--------

### select

- 一个监听的文件描述符，N个连接的文件描述符，总共 N+1个,`select`文件描述符的个数最大为1024，可以编译内核修改这个最大的限制
- 可以借助内核同时监控多个文件描述符(这个过程是阻塞的)，哪个文件描述符就绪（可读或者可写）就返回，同时阻塞也就解除了。

### select API

```c
#include <sys/select.h>

struct timeval{
  time_t tv_sec;
  suseconds_t tv_usec;
};

int select(int nfds,fd_set *readfds,fd_set *writefds,fd_set *exceptfds,struct timeval *timeout);

//nfds --检测三个最大集合中文件描述符最大值+1（+1原因：文件描述符从0开始，最大可以指定1024）；内核根据这个参数进行线性的遍历，检测活跃的文件描述符
//readfds writefds ecceptfds 文件描述符集合，分别检测读缓冲区，写缓冲区，异常缓冲区
//timeval 超时时间，NULL 检测不到就一直阻塞；固定时长返回；0 不等待，函数不会阻塞
//返回值： >0 成功。-1 失败。==0 超时，没有检测到就绪的文件描述符


//参数2，3，4是一个传入传出参数，意味着先初始化传入，内核经过检测之后，就会传出；
//fd_set 是一个文件描述符的集合，128个字节，1024个bit位。32个int，和文件描述符表是一一对应的关系 
```

<img src="https://subingwen.cn/linux/select/image-20210324233252448.png" alt="img" style="zoom:30%;" />

- fd_set 集合中1代表检测这个文件描述符，0代表不检测
- 以读集合为例，当内核检测到被检测的文件描述符读缓冲区有数据，就把fd_set 中对应的位改成1；没有数据就把对应位上的值改为 0

<img src="https://subingwen.cn/linux/select/image-20210324234324388.png" alt="img" style="zoom:25%;" />

- 内核会把这个集合拷贝两次，第一次就是传入的时候，传入等到检测；第二次就是传出，也就是内核把结果返回的时候。

```c
//集合相关的操作
// 将文件描述符fd从set集合中删除 == 将fd对应的标志位设置为0        
void FD_CLR(int fd, fd_set *set);
// 判断文件描述符fd是否在set集合中 == 读一下fd对应的标志位到底是0还是1
int  FD_ISSET(int fd, fd_set *set);
// 将文件描述符fd添加到set集合中 == 将fd对应的标志位设置为1
void FD_SET(int fd, fd_set *set);
// 将set集合中, 所有文件文件描述符对应的标志位设置为0, 集合中没有添加任何文件描述符
void FD_ZERO(fd_set *set);
```

----

### 处理流程

- `socket()`

- `bind()`

- `listen()`

- 设置`fd_set`,存储用于监听的文件描述符

- 循环调用`select()`

- 解除阻塞，返回结果，对返回的集合进行判断
  - `FD_ISSET()`判断结合中的标识位是否为1
  - 如果是监听的文件描述符就开始建立连接，同时加入集合
  - 通信的文件描述符，就开始通信；通信完成断开连接，从集合清除掉
- 重复上一步

<img src="https://subingwen.cn/linux/select/image-20210324235635304.png" alt="img" style="zoom:50%;" />



```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/select.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <ctype.h>


int main()
{
    //socket
    int sfd = socket(AF_INET,SOCK_STREAM,0);
    if(sfd == -1)
    {
        perror("socket fail!");
        return -1;
    }
    //setsocketopt
    int ok = 1;
    int ret = setsockopt(sfd,SOL_SOCKET,SO_REUSEADDR,&ok,sizeof(ok));
    if(ret == -1)
    {
        perror("setsocketopt error!");
        exit(-1);
    }
    //bind
    struct sockaddr_in saddr;
    saddr.sin_port = htons(8888);
    saddr.sin_family = AF_INET;
    saddr.sin_addr.s_addr = INADDR_ANY;
    //inet_pton(AF_INET,"127.0.0.1",&saddr.sin_addr.s_addr);


    bind(sfd,(struct sockaddr*)&saddr,sizeof(saddr));

    //listen
    listen(sfd,128);

    //初始化集合
    fd_set readfds;
    fd_set tmpfds;
    FD_ZERO(&readfds);
    FD_ZERO(&tmpfds);
    FD_SET(sfd,&readfds);
    int maxfd = sfd;

    while(1)
    {
        tmpfds = readfds;//防止后续的fd加入到已经发生变化的集合中，readfds一直为需要检测的原始数据
        struct timeval time;
        time.tv_sec = 5;
        time.tv_usec = 0;

        int num = select(maxfd+1,&tmpfds,NULL,NULL,&time);

        if(FD_ISSET(sfd,&tmpfds)) //是通信的fd
        {
            //accept
            struct sockaddr_in caddr;
            socklen_t len =sizeof(caddr);

            int cfd = accept(sfd,(struct sockaddr*)&caddr,&len);
            if(cfd == -1)
            {
                perror("accept error!");
                exit(-1);

            }
            FD_SET(cfd,&readfds);
            maxfd = cfd>maxfd?cfd:maxfd;

            int port;
            port = ntohs(caddr.sin_port);
            char ip[32];
            inet_ntop(AF_INET,&caddr.sin_addr.s_addr,ip,sizeof(ip));
            printf("%s:%d\n connected!",ip,port);
        }
        for(int i = 0;i<maxfd+1;++i) //遍历就绪的文件描述符,i的值可以从3开始
        {
            printf("%d\n",i);
            if(i != sfd && FD_ISSET(i,&tmpfds)) //不是监听的fd,是通信的fd
            {
                printf("ok-->%d\n",i);
                char recv_buf[1024];
                bzero(recv_buf,sizeof(recv_buf));

                int len = recv(i,recv_buf,sizeof(recv_buf),0);
                if(len == 0)
                {
                    FD_CLR(i,&readfds);
                    close(i);
                }
                else if(len > 0)
                {
                    for(int i = 0;i<len;++i)
                    {
                        recv_buf[i] = toupper(recv_buf[i]);
                    }
                    send(i,recv_buf,len,0);
                }
                else{

                    close(i);
                    break;
                }

            }
        }


    }

    close(sfd);
    return 0;
}
```



```tex
0
1
2
3
127.0.0.1:51064 connected!
0
1
2
3
4
0
1
2
3
4
ok-->4
0
1
2
3
4
127.0.0.1:51065 connected!
0
1
2
3
4
5
0
1
2
3
4
5
ok-->5
```

### select几个缺陷

- 待检测集合需要不断的拷贝，在用户区和内核区频繁切换
- 内核对于select传进来的文件描述符是线性检测的，fd多了，效率就低了
- 1024个fd的限制

------

### poll

- `poll`没有文件描述符数量的限制，依据你机器的性能
- `poll`只能在`linux`平台使用

### API

```c
#include <sys/poll.h>

struct pollfd{
  int fd;
  short events;
  short revents;
};

struct pollfd myfd[100];
int poll(struct pollfd *fds,nfds_t nfds,int timeout);


```

- `fds` 是一个 `pollfd`类型的结构体数组

  - fd 委托内核检测的文件描述符
  - events 委托内核检测的fd的事件，每一个事件可以取多个值
  - revents  内核检测之后的结果

  ![img](https://subingwen.cn/linux/poll/1558308141721.png)

- nfds 数组中最后一个有效元素的下标 + 1（也可以指定参数 1 数组的元素总个数）
- timeout  -1 一直阻塞；0 不阻塞；>0  xx ms
- return val : -1 失败  ；>0  集合中已就绪的文件描述符的总个数

### 例子

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/poll.h>
#include <ctype.h>

int main()
{
	
	int sfd = socket(AF_INET,SOCK_STREAM,0);
	if(sfd == -1)
	{
		perror("socket error!");
		exit(-1);
	}
	int ok =1;
	setsockopt(sfd,SOL_SOCKET,SO_REUSEADDR,&ok,sizeof(ok));

	struct sockaddr_in saddr;
	saddr.sin_family = AF_INET;
	saddr.sin_port = htons(8888);
	saddr.sin_addr.s_addr = INADDR_ANY;
	//inet_pton(AF_INET,"127.0.0.1",&saddr.sin_addr.s_addr);

	int ret = bind(sfd,(struct sockaddr*)&saddr,sizeof(saddr));
	if(ret == -1)
	{
		perror("bind error!");
		exit(-1);
	}
	ret = listen(sfd,128);
	if(ret == -1)
	{
		perror("listen error!");
		exit(-1);
	}
	
	struct pollfd fds[1024];
	for(int i = 0;i<1024;++i)
	{
		fds[i].fd = -1;
		fds[i].events = POLLIN;
	}
	fds[0].fd = sfd;
	int maxfd = 0;

	while(1)
	{
		
		int num = poll(fds,maxfd+1,3);
		if(num == -1)
		{
			perror("poll fail!");
			exit(-1);
		}

		//new connect
		if(fds[0].revents & POLLIN)
		{
			
			//accept 
			struct sockaddr_in caddr;
			socklen_t len = sizeof(caddr);
			int cfd = accept(fds[0].fd,(struct sockaddr*)&caddr,&len);
			
			if(cfd == -1)
			{
				
				perror("accept fail!");
				exit(-1);
			}
			
			for(int i = 0;i<1024;++i)
			{
				if(fds[i].fd == -1)
				{
					fds[i].fd = cfd;
					break;
				}
			}

			maxfd = cfd>maxfd?cfd:maxfd;
		}

		for(int i = 1;i<maxfd+1;++i) //注意这个i从1开始，0是监听的文件描述符
		{
			
			if(fds[i].revents & POLLIN)
			{
				
				char recv_buf[1024];
				bzero(recv_buf,1024);

				int len = recv(fds[i].fd,recv_buf,sizeof(recv_buf),0);
				if(len == 0)
				{
					
					close(fds[i].fd);
					fds[i].fd = -1;
				}	
				else if(len > 0)
				{
					for(int i = 0;i<len;++i)
					{
						recv_buf[i] = toupper(recv_buf[i]);
					}
					send(fds[i].fd,recv_buf,sizeof(recv_buf),0);

				}
				else{
					close(fds[i].fd);
					exit(-1);
				}


			}
		}


	}

	close(sfd);

	exit(0);
}

```



### poll和select比较

- poll的实现思路和select一样，都是进行线性遍历活跃的文件描述符表
- 不同的是：select使用位图来存储文件描述符表；poll使用pollfd结构体数组来表示

----

