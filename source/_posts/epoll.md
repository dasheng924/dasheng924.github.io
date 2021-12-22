---
title: epoll
date: 2021-06-17
Tags:
  - epoll
categories:
  - Linux
cover: https://static001.geekbang.org/infoq/32/3262c10fc151ee6fb518e29f338701f9.png
---

![img](https://static001.geekbang.org/infoq/32/3262c10fc151ee6fb518e29f338701f9.png)

### epoll

- 对于检测集合，select和poll采用线性的方式遍历；epoll采用红黑树的方式进行了管理。
- epoll采用事件回调的机制高效的
- select和poll中存在用户区和内核区的频繁拷贝问题；epoll采用mmap共享内存，省区了不必要的内存拷贝
- select和poll需要遍历返回集合或者数组才能知道哪个文件描述符触发；epoll直接返回那几个文件描述符触发
- epoll也没有文件描述符数量的限制

### 优点：

- epoll 创建的红黑树保存所有 fd，没有大小限制，且增删查的复杂度 O(logN)

- 基于 callback，利用系统内核触发感兴趣的事件

- 就绪列表为双线链表时间复杂度 O(1)

- 应用获取到的 fd 都是真实发生 IO 的 fd，与 select 和 poll 需要不断轮询判断是否可用相比，能避免无用的内存拷贝

### API

```c
 #include <sys/epoll.h>
// 创建epoll实例，通过一棵红黑树管理待检测集合
int epoll_create(int size);
// 管理红黑树上的文件描述符(添加、修改、删除)
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
// 检测epoll树中是否有就绪的文件描述符
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

```c
// 创建epoll实例，通过一棵红黑树管理待检测集合
int epoll_create(int size);

//内核2.6.8 忽略了参数size,填写>0的数即可

//成功 返回一个有效的文件描述符，通过这个文件描述符就可以访问epoll实例
//失败：-1
```

```c
// 联合体, 多个变量共用同一块内存        
typedef union epoll_data {
 	void        *ptr;
	int          fd;	// 通常情况下使用这个成员, 和epoll_ctl的第三个参数相同即可
	uint32_t     u32;
	uint64_t     u64;
} epoll_data_t;

struct epoll_event {
	uint32_t     events;      /* Epoll events */
	epoll_data_t data;   //调用epoll_wait时，这个值会被传出
};

// 管理红黑树上的文件描述符(添加、修改、删除)
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);


//epfd --》epoll_create()返回值
//op :
	//EPOLL_CTL_ADD：往 epoll 模型中添加新的节点
	//EPOLL_CTL_MOD：修改 epoll 模型中已经存在的节点
	//EPOLL_CTL_DEL：删除 epoll 模型中的指定的节点
//fd 参数2准备要操作的文件描述符
//event：epoll事件，修饰fd,表明检测这个文件描述符的什么事件

```

```c
// 检测epoll树中是否有就绪的文件描述符
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);

//epfd --》epoll_create()返回值
//events 传出参数，活跃的文件描述符和对应的事件都可知
//maxevents 修饰参数2，结构体数组的大小

```

### 操作流程

![img](https://img-blog.csdnimg.cn/20191224210045320.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1ODgzNDY0,size_16,color_FFFFFF,t_70)

#### socke

#### setsockopt

#### bind

#### listen

#### epoll_create

```c
int epfd = epoll_create(100);
```

#### epoll_ctl

```c
struct epoll_event ev;
ev.events = EPOLLIN;
ev.data.fd = sfd;

int ret = epoll_ctl(epfd,EPOLL_CTL_ADD,sfd,&ev);
```

#### epoll_wait

```c
struct epoll_event evs[1024];
int size = sizeof(evs)/sizeof(evs[0]);
int num = epoll_wait(epfd,evs,size,-1);
```

- 监听的文件描述符

```c
int cfd = accept(curfd, NULL, NULL);
ev.events = EPOLLIN;
ev.data.fd = cfd;
// 新得到的文件描述符添加到epoll模型中, 下一轮循环的时候就可以被检测了
epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &ev);
```

- 通信的文件描述符

```c
int len = recv(curfd, buf, sizeof(buf), 0);
if(len == 0)
{
    // 将这个文件描述符从epoll模型中删除
    epoll_ctl(epfd, EPOLL_CTL_DEL, curfd, NULL);
    close(curfd);
}
else if(len > 0)
{
    send(curfd, buf, len, 0);
}
```



### 例子

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/epoll.h>
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
	
    int epfd = epoll_create(100);
    if(epfd == -1)
    {
        perror("epoll_create fail!");
        exit(-1);

    }

    struct epoll_event ev;
    ev.events = EPOLLIN;
    ev.data.fd = sfd;

    ret = epoll_ctl(epfd,EPOLL_CTL_ADD,sfd,&ev);
    if(ret == -1)
    {
        perror("epoll_ctl fail!");
        exit(-1);
    }




    struct epoll_event evs[1024] ;
    int size = sizeof(evs)/sizeof(evs[0]);

	while(1)
	{
		
		int num = epoll_wait(epfd,evs,size,-1);
		if(num == -1)
		{
			perror("epoll fail!");
			exit(-1);
		}


        for(int i = 0;i<num;++i) //这里要循环去拿文件描述符，而不是直接判断，原因：无法做到像线性表那样顺序性的拿取，基于事件的回调，只能在一个循环里面判断
        {
            int curfd = evs[i].data.fd;
            if(curfd == sfd)
            {
                int cfd = accept(curfd,NULL,NULL);
                if(cfd == -1)
                {
                    perror("accept error!");
                    exit(-1);
                }

                ev.events = EPOLLIN;
                ev.data.fd = cfd;
                int ret = epoll_ctl(epfd,EPOLL_CTL_ADD,cfd,&ev);
                if(ret == -1)
                {
                    perror("epoll_ctl add cfd fail!");
                    exit(-1);
                }
                
            }
            else{
                char recv_buf[1024];
				bzero(recv_buf,1024);

				int len = recv(curfd,recv_buf,sizeof(recv_buf),0);
				if(len == 0)
				{
					epoll_ctl(epfd,EPOLL_CTL_DEL,curfd,NULL);
					close(curfd);
				}	
				else if(len > 0)
				{
					for(int i = 0;i<len;++i)
					{
						recv_buf[i] = toupper(recv_buf[i]);
					}
					send(curfd,recv_buf,sizeof(recv_buf),0);

				}
				else{
					close(curfd);
					exit(-1);
				}
            }

        }

    }
   
	close(sfd);

	exit(0);
}

```



### epoll 的LT和ET模式

- LT水平触发模式：缓冲区只要有数据，就一直通知
- ET边沿触发模式：缓冲区有数据只通知一次，等到新数据到来，再通知一次

### epoll 非阻塞模式

```c
#include <stdio.h>
#include <ctype.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <fcntl.h>
#include <errno.h>

// server
int main(int argc, const char* argv[])
{
    // 创建监听的套接字
    int lfd = socket(AF_INET, SOCK_STREAM, 0);
    if(lfd == -1)
    {
        perror("socket error");
        exit(1);
    }

    // 绑定
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(9999);
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);  // 本地多有的ＩＰ
    // 127.0.0.1
    // inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr.s_addr);
    
    // 设置端口复用
    int opt = 1;
    setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

    // 绑定端口
    int ret = bind(lfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
    if(ret == -1)
    {
        perror("bind error");
        exit(1);
    }

    // 监听
    ret = listen(lfd, 64);
    if(ret == -1)
    {
        perror("listen error");
        exit(1);
    }

    // 现在只有监听的文件描述符
    // 所有的文件描述符对应读写缓冲区状态都是委托内核进行检测的epoll
    // 创建一个epoll模型
    int epfd = epoll_create(100);
    if(epfd == -1)
    {
        perror("epoll_create");
        exit(0);
    }

    // 往epoll实例中添加需要检测的节点, 现在只有监听的文件描述符
    struct epoll_event ev;
    ev.events = EPOLLIN;    // 检测lfd读读缓冲区是否有数据
    ev.data.fd = lfd;
    ret = epoll_ctl(epfd, EPOLL_CTL_ADD, lfd, &ev);
    if(ret == -1)
    {
        perror("epoll_ctl");
        exit(0);
    }


    struct epoll_event evs[1024];
    int size = sizeof(evs) / sizeof(struct epoll_event);
    // 持续检测
    while(1)
    {
        // 调用一次, 检测一次
        int num = epoll_wait(epfd, evs, size, -1);
        printf("==== num: %d\n", num);

        for(int i=0; i<num; ++i)
        {
            // 取出当前的文件描述符
            int curfd = evs[i].data.fd;
            // 判断这个文件描述符是不是用于监听的
            if(curfd == lfd)
            {
                // 建立新的连接
                int cfd = accept(curfd, NULL, NULL);
                // 将文件描述符设置为非阻塞
                // 得到文件描述符的属性
                int flag = fcntl(cfd, F_GETFL);
                flag |= O_NONBLOCK;
                fcntl(cfd, F_SETFL, flag);
                // 新得到的文件描述符添加到epoll模型中, 下一轮循环的时候就可以被检测了
                // 通信的文件描述符检测读缓冲区数据的时候设置为边沿模式
                ev.events = EPOLLIN | EPOLLET;    // 读缓冲区是否有数据
                ev.data.fd = cfd;
                ret = epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &ev);
                if(ret == -1)
                {
                    perror("epoll_ctl-accept");
                    exit(0);
                }
            }
            else
            {
                // 处理通信的文件描述符
                // 接收数据
                char buf[5];
                memset(buf, 0, sizeof(buf));
                // 循环读数据
                while(1)
                {
                    int len = recv(curfd, buf, sizeof(buf), 0);
                    if(len == 0)
                    {
                        // 非阻塞模式下和阻塞模式是一样的 => 判断对方是否断开连接
                        printf("客户端断开了连接...\n");
                        // 将这个文件描述符从epoll模型中删除
                        epoll_ctl(epfd, EPOLL_CTL_DEL, curfd, NULL);
                        close(curfd);
                        break;
                    }
                    else if(len > 0)
                    {
                        // 通信
                        // 接收的数据打印到终端
                        write(STDOUT_FILENO, buf, len);
                        // 发送数据
                        send(curfd, buf, len, 0);
                    }
                    else
                    {
                        // len == -1
                        if(errno == EAGAIN)
                        {
                            printf("数据读完了...\n");
                            break;
                        }
                        else
                        {
                            perror("recv");
                            exit(0);
                        }
                    }
                }
            }
        }
    }

    return 0;
}

```



