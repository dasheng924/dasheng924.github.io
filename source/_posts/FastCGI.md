---
title: fastcgi简单使用
date: 2021-04-02
tags: 
  - fastcgi
categories:
  - Tools
cover: https://tva1.sinaimg.cn/large/008i3skNly1gsn8jaaj62j30zk0k0t8w.jpg
---

### FastCGI

- nginx 对于动态请求，无法直接处理，需要调用第三方模块去处理。
- 动态请求，一般可以使用CGI程序去处理
- 我们需要编写CGI程序，完成对应数据的处理

### CGI

- CGI是一段程序，运行在服务器上，帮助服务器来处理动态请求
- 处理步骤
  - 通过internet把用户请求发送到WEB服务器
  - WEB服务器收到动态请求提交给CGI程序
  - CGI进行处理，把结果反馈给WEB服务器
  - WEB服务器把结果反馈给用户

![nginx CGI ](https://tva1.sinaimg.cn/large/008i3skNly1gsn71boctoj31ni0hy77f.jpg)

- WEB服务器没收到一个动态请求，就会创建一个子进程
- 子进程CGI处理完请求后，就会被WEB服务器杀死
- 大量的请求就需要不断的创建子进程，不断的杀死子进程，这样的效率就很低下
- 为了解决子进程大量创建销毁带来的巨大开销，选用FastCGI 

### FastCGI

- 主要将CGI程序在内存中进行了管理，也就是不会频繁的创建子进程和销毁子进程。

- WEB服务器启动时会启动FastCGI

- FastCGI 开始启动几个CGI程序，等待WEB服务器的请求到来

- 有请求过来，FastCGI 就处理，处理完之后，不销毁CGI程序，让其阻塞等待。等到下一次任务到来时就继续工作。

  

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsn7og7yf1j61820mgdhv02.jpg" alt="image-20210720103511539" style="zoom:33%;" />

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsn7po6cboj31ak0iegnt.jpg" alt="image-20210720103633517" style="zoom: 33%;" />



### spawn-fcgi

- FastCGI 与nginx 是相互分离的，所以需要`spawn-fcgi`来做中间的桥梁

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsn7s2eq27j31cs0ka0uy.jpg" alt="image-20210720103904653" style="zoom: 33%;" />

- `spawn-fcgi`相当于一个代理工具，用于`nginx`和`fastcgi`之间的通信

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsn7whne3wj31jy0pidju.jpg" alt="image-20210720104319230" style="zoom:67%;" />

- nginx 中关于spawn-fcgi 的配置

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsn7zkjxxsj30km05qwev.jpg" alt="image-20210720104617057" style="zoom:50%;" />

>fastcgi_pass  web 服务器把数据要转发到的spawn-fcgi 的IP和PORT

- spawn-fcgi 的使用

  ```shell
  spawn-fcgi -a IP -p PORT -f fastcgi程序	
  
  IP 服务器的IP
  PORT 服务器将数据发送到的端口
  fastcgi: 	spawn-fcgi 需要启动的fastcgi的程序
  
  ```

### fastcgi程序的使用

#### fastcgi 在 nginx 的辅助配置文件

```ini

fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING       $query_string; #请求的资源
fastcgi_param  REQUEST_METHOD     $request_method;#请求的方法
fastcgi_param  CONTENT_TYPE       $content_type;#文件的类型
fastcgi_param  CONTENT_LENGTH     $content_length;#请求的内容长度

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;#要处理的指令，nginx中server对应的location
fastcgi_param  REQUEST_URI        $request_uri;#浏览器中请求的资源，不包括IP PORT
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
```

#### fastcgi 举例

```c
/*
 * echo.c --
 *
 *	Produce a page containing all FastCGI inputs
 *
 *
 * Copyright (c) 1996 Open Market, Inc.
 *
 * See the file "LICENSE.TERMS" for information on usage and redistribution
 * of this file, and for a DISCLAIMER OF ALL WARRANTIES.
 *
 */
#ifndef lint
static const char rcsid[] = "$Id: echo.c,v 1.5 1999/07/28 00:29:37 roberts Exp $";
#endif /* not lint */

#include "fcgi_config.h"

#include <stdlib.h>

#ifdef HAVE_UNISTD_H
#include <unistd.h>
#endif

#ifdef _WIN32
#include <process.h>
#else
extern char **environ;
#endif

#include "fcgi_stdio.h"


static void PrintEnv(char *label, char **envp)
{
    printf("%s:<br>\n<pre>\n", label);
    for ( ; *envp != NULL; envp++) {
        printf("%s\n", *envp);
    }
    printf("</pre><p>\n");
}

int main ()
{
    char **initialEnv = environ;
    int count = 0;

    while (FCGI_Accept() >= 0) {
        char *contentLength = getenv("CONTENT_LENGTH");
        int len;

	printf("Content-type: text/html\r\n"
	    "\r\n"
	    "<title>FastCGI echo</title>"
	    "<h1>FastCGI echo</h1>\n"
            "Request number %d,  Process ID: %d<p>\n", ++count, getpid());

        if (contentLength != NULL) {
            len = strtol(contentLength, NULL, 10);
        }
        else {
            len = 0;
        }

        if (len <= 0) {
	    printf("No data from standard input.<p>\n");
        }
        else {
            int i, ch;

	    printf("Standard input:<br>\n<pre>\n");
            for (i = 0; i < len; i++) {
                if ((ch = getchar()) < 0) {
                    printf("Error: Not enough bytes received on standard input<p>\n");
                    break;
		}
                putchar(ch);
            }
            printf("\n</pre><p>\n");
        }

        PrintEnv("Request environment", environ);
        PrintEnv("Initial environment", initialEnv);
    } /* while */

    return 0;
}

```



```c++

#include <stdlib.h>
#ifdef _WIN32
#include <process.h>
#else
#include <unistd.h>
extern char ** environ;
#endif
#include "fcgio.h"
#include "fcgi_config.h"  // HAVE_IOSTREAM_WITHASSIGN_STREAMBUF

using namespace std;

// Maximum number of bytes allowed to be read from stdin
static const unsigned long STDIN_MAX = 1000000;

static void penv(const char * const * envp)
{
    cout << "<PRE>\n";
    for ( ; *envp; ++envp)
    {
        cout << *envp << "\n";
    }
    cout << "</PRE>\n";
}

static long gstdin(FCGX_Request * request, char ** content)
{
    char * clenstr = FCGX_GetParam("CONTENT_LENGTH", request->envp);
    unsigned long clen = STDIN_MAX;

    if (clenstr)
    {
        clen = strtol(clenstr, &clenstr, 10);
        if (*clenstr)
        {
            cerr << "can't parse \"CONTENT_LENGTH="
                 << FCGX_GetParam("CONTENT_LENGTH", request->envp)
                 << "\"\n";
            clen = STDIN_MAX;
        }

        // *always* put a cap on the amount of data that will be read
        if (clen > STDIN_MAX) clen = STDIN_MAX;

        *content = new char[clen];

        cin.read(*content, clen);
        clen = cin.gcount();
    }
    else
    {
        // *never* read stdin when CONTENT_LENGTH is missing or unparsable
        *content = 0;
        clen = 0;
    }

    // Chew up any remaining stdin - this shouldn't be necessary
    // but is because mod_fastcgi doesn't handle it correctly.

    // ignore() doesn't set the eof bit in some versions of glibc++
    // so use gcount() instead of eof()...
    do cin.ignore(1024); while (cin.gcount() == 1024);

    return clen;
}

int main (void)
{
    int count = 0;
    long pid = getpid();

    streambuf * cin_streambuf  = cin.rdbuf();
    streambuf * cout_streambuf = cout.rdbuf();
    streambuf * cerr_streambuf = cerr.rdbuf();

    FCGX_Request request;

    FCGX_Init();
    FCGX_InitRequest(&request, 0, 0);

    while (FCGX_Accept_r(&request) == 0)
    {
        // Note that the default bufsize (0) will cause the use of iostream
        // methods that require positioning (such as peek(), seek(),
        // unget() and putback()) to fail (in favour of more efficient IO).
        fcgi_streambuf cin_fcgi_streambuf(request.in);
        fcgi_streambuf cout_fcgi_streambuf(request.out);
        fcgi_streambuf cerr_fcgi_streambuf(request.err);

#if HAVE_IOSTREAM_WITHASSIGN_STREAMBUF
        cin  = &cin_fcgi_streambuf;
        cout = &cout_fcgi_streambuf;
        cerr = &cerr_fcgi_streambuf;
#else
        cin.rdbuf(&cin_fcgi_streambuf);
        cout.rdbuf(&cout_fcgi_streambuf);
        cerr.rdbuf(&cerr_fcgi_streambuf);
#endif

        // Although FastCGI supports writing before reading,
        // many http clients (browsers) don't support it (so
        // the connection deadlocks until a timeout expires!).
        char * content;
        unsigned long clen = gstdin(&request, &content);

        cout << "Content-type: text/html\r\n"
                "\r\n"
                "<TITLE>echo-cpp</TITLE>\n"
                "<H1>echo-cpp</H1>\n"
                "<H4>PID: " << pid << "</H4>\n"
                "<H4>Request Number: " << ++count << "</H4>\n";

        cout << "<H4>Request Environment</H4>\n";
        penv(request.envp);

        cout << "<H4>Process/Initial Environment</H4>\n";
        penv(environ);

        cout << "<H4>Standard Input - " << clen;
        if (clen == STDIN_MAX) cout << " (STDIN_MAX)";
        cout << " bytes</H4>\n";
        if (clen) cout.write(content, clen);

        if (content) delete []content;

        // If the output streambufs had non-zero bufsizes and
        // were constructed outside of the accept loop (i.e.
        // their destructor won't be called here), they would
        // have to be flushed here.
    }

#if HAVE_IOSTREAM_WITHASSIGN_STREAMBUF
    cin  = cin_streambuf;
    cout = cout_streambuf;
    cerr = cerr_streambuf;
#else
    cin.rdbuf(cin_streambuf);
    cout.rdbuf(cout_streambuf);
    cerr.rdbuf(cerr_streambuf);
#endif

    return 0;
}
```

