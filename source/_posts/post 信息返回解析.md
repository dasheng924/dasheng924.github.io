---
title: POST信息解析
date: 2021-03-28
tag:
  - 数据解析
categories:
  - C++
cover: https://tva1.sinaimg.cn/large/008i3skNly1gsmi6zxbwij317w0na14p.jpg
highlight_shrink: true
---

### 原来的文本信息

![post后收到的信息](https://tva1.sinaimg.cn/large/008i3skNly1gsmh9xt4rxj31ea0u0199.jpg)



### 解析后的内容

![这是一张图片，可以直接被打开成图片](https://tva1.sinaimg.cn/large/008i3skNly1gsmhbbe8luj31eb0u0qi8.jpg)



### post 信息返回解析

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <ctype.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

//fcgi 程序需要的头文件
#include "fcgi_config.h"
#include "fcgi_stdio.h"


/**
 * @brief  去掉一个字符串两边的空白字符
 *
 * @param inbuf确保inbuf可修改
 *
 * @returns   
 *      0 成功
 *      -1 失败
 */

int trim_space(char *inbuf)
{
    if(inbuf == NULL)
        return -1;

    char * tmp = inbuf;
    int i = 0;
    int j = strlen(tmp) -1 ;

    //"###user##"
    //i = 3
    //j = 6
    while(isspace(tmp[i]) && tmp[i] != '\0') //从左边开始去空格
    {
        i++;
    }
    while(isspace(tmp[j]) && j > i)//从右边开始去空格
    {
        j--;
    }
    int count = j-i+1; //6-3+1-->4 //得出去掉左右空格后的真实字符个数
    strncpy(inbuf,tmp+i,count);
    inbuf[count] = '\0';

    return 0;
}

/**
 * @brief  在字符串full_data中查找字符串substr第一次出现的位置
 *
 * @param full_data 	  源字符串首地址
 * @param full_data_len 源字符串长度
 * @param substr        匹配字符串首地址
 *
 * @returns   
 *      成功: 匹配字符串首地址
 *      失败：NULL
 */
//这个函数的功能和strstr是类似的 
char* memstr(char* full_data, int full_data_len, char* substr) 
{ 
	//异常处理
    if (full_data == NULL || full_data_len <= 0 || substr == NULL) 
	{ 
        return NULL; 
    } 

    if (*substr == '\0')
	{ 
        return NULL; 
    } 

	//匹配子串的长度
    int sublen = strlen(substr); 

    int i; 
    char* cur = full_data; 
    int last_possible = full_data_len - sublen + 1; //减去匹配子串后的长度
  	//last_possible 需要循环匹配的最大次数
    for (i = 0; i < last_possible; i++) 
	{ 
        if (*cur == *substr) //如果*cur的字符和 *substr中的第一个字符对的上，就开始整个子串的匹配
		{ 
            if (memcmp(cur, substr, sublen) == 0) //整个子串的匹配
			{ 
                //found  
                return cur; //找见返回源串中 包含子串的起始位置
            } 
        }
		
        cur++; 
    } 

    return NULL; 
} 






int main(void)
{

//1.get data len from nginx web content
while (FCGI_Accept() >= 0)
    {
        //==========> 获取post数据的长度 <===========
        char *contentLength = getenv("CONTENT_LENGTH");
        long len;
        int ret = 0;

        printf("Content-type: text/html\r\n\r\n");

        if (contentLength != NULL)
        {
            len = strtol(contentLength, NULL, 10); //字符串转long， 或者atol
        }
        else
        {
            len = 0;
        }

        if (len <= 0)
        {
            printf("No data from standard input\n");
            ret = -1;
        }
        printf("len=%d",len);

        //==========> 开辟存放文件的 内存 <===========
        //malloc space to save conten
        char *web_content = (char *)malloc(sizeof(char)*len);
        if(web_content == NULL)
        {
            perror("malloc error!");
            return -1;
        }
        memset(web_content,0,len);

        ret = fread(web_content,1,len,stdin); //保存返回的数据
        if(ret == 0)
        {
            perror("get web content error!");
            return -1;
        }

        FILE *fp = fopen("./check_info.txt","w+");//把数据写到文件里面，等待验证

        fwrite(web_content,1,ret,fp);

        fclose(fp);

        //===========> 开始处理前端发送过来的post数据格式 <============
        char *begin = web_content;    //内存起点
        char *p = begin;

        /*
       ------WebKitFormBoundary88asdgewtgewx\r\n
       Content-Disposition: form-data; user="mike"; filename="xxx.jpg"; md5="xxxx"; size=10240\r\n
       Content-Type: application/octet-stream\r\n
       \r\n
       真正的文件内容\r\n
       ------WebKitFormBoundary88asdgewtgewx
       */

        //get boundary 得到分界线, ------WebKitFormBoundary88asdgewtgewx
        p = strstr(begin,"\r\n"); //p到第一行的末尾
        if(p == NULL)
        {
            perror("get first_line error!");
            return -1;
        }
        char first_line[128] = {0}; //保存第一行的内容，也就是分界线
        strncpy(first_line,begin,p-begin);
        first_line[p-begin] = '\0';
        printf("%s\n",first_line);

        //jump \r\n
        p += 2;
         //new len，保存剩余的需要处理的数据长度
        len -= (p-begin); 
        //new begin
        begin = p;
        
        //----------------------------------------get file_name  begin--------
        //to second_line,get file_name
  			//第二行的处理，这里面会保存文件的各种信息
        char file_name[128] = {0};
    
        char *q = begin; //begin 指向第二行的起始位置
        q = strstr(q,"filename="); 
        //filename="xxxxx"
        //^
        q += strlen("filename=");
        //filename="xxxxx"
        //         ^
        q++;
        //filename="xxxxx"
        //          ^ 
        char *k = strchr(q,'"');
        //filename="xxxxx"
        //               ^
        strncpy(file_name,q,k-q);
        file_name[k-q] = '\0';
        trim_space(file_name);
        printf("%s\n",file_name);
        //----------------------------------------get file_name  end --------

        p = strstr(begin,"\r\n"); //到第二行的 \r\n 位置
        if(p == NULL)
        {
            perror("get second_line error!");
            return -1;
        }

        p += 2; //thired line start 第三行的起始位置
        len -= (p- begin) ;
        begin = p; //thired line start

        p = strstr(p,"\r\n"); //thired line end 第三行的末尾
        p += 4;//\r\n\r\n // jump 3 line,4 line --->to 5 line 跳过第三行的末尾和第四行
        len -= (p-begin) ; //odd len

        //下面才是文件的真正内容
        /*
        真正的文件内容\r\n
       ------WebKitFormBoundary88asdgewtgewx
       */

        begin = p; //begin=p --->5 line start
        //printf("%s",p) ;

        p = memstr(begin, len, first_line);// 获取到标志再次出现的位置
        //printf("%s",p) ;
        if(p == NULL)
        {
            perror("find second flag_line error!");
            return -1;
        }

        p -= 2; //to real content end

        //real content  = p-begin ;
        //=====> 此时begin-->p两个指针的区间就是post的文件二进制数据
        //======>将数据写入文件中,其中文件名也是从post数据解析得来  <===========
        //OKKKKKK!!!!!!
        int fd = open("./content.txt", O_CREAT|O_WRONLY, 0644);
        if (fd < 0)
        {
           
            free(web_content);
            return -1;
        }

        //ftruncate会将参数fd指定的文件大小改为参数length指定的大小
        ftruncate(fd, (p-begin));
        write(fd, begin, (p-begin));
        close(fd);
    }

    return 0;
}
```

