---
title: C++日期和时间
date: 2021-06-10
categories:
  - C++
cover: https://tva1.sinaimg.cn/large/008i3skNly1gtc0jv6g2fj31nc0qwwhf.jpg
---

![](https://tva1.sinaimg.cn/large/008i3skNly1gtc0jv6g2fj31nc0qwwhf.jpg)



### 保存时间日期的结构体

- C++ 中延用C的时间日期函数,需要引入头文件`<ctime>`头文件
  - 有四个与时间相关的类型 `clock_t`,`time_t`,`size_t`,`tm`
  - `clock_t`,`time_t`,`size_t` 这三个类型可以把系统时间和日期表示为整数

```c++
struct tm {
  int tm_sec;   // 秒，正常范围从 0 到 59，但允许至 61
  int tm_min;   // 分，范围从 0 到 59
  int tm_hour;  // 小时，范围从 0 到 23
  int tm_mday;  // 一月中的第几天，范围从 1 到 31
  int tm_mon;   // 月，范围从 0 到 11
  int tm_year;  // 自 1900 年起的年数
  int tm_wday;  // 一周中的第几天，范围从 0 到 6，从星期日算起
  int tm_yday;  // 一年中的第几天，范围从 0 到 365，从 1 月 1 日算起
  int tm_isdst; // 夏令时
}
```

### 相关的函数

```c
time_t time(time_t *time);
//该函数返回系统的当前日历时间，自 1970 年 1 月 1 日以来经过的秒数。如果系统没有时间，则返回 -1
```

```c
#include <stdio.h>
#include <time.h>

int main(void)
{	
  time_t seconds;
  seconds = time(NULL);
  printf("自1970-01-01 起的小时数:%ld\n",seconds/3600);

	return 0;
}

//out:
// 自1970-01-01 起的小时数:452388
```



-------

```c
char *ctime(const time_t *time);
//该返回一个表示当地时间的字符串指针，字符串形式 day month year hours:minutes:seconds year\n\0。
```

```c
#include <stdio.h>
#include <time.h>
 
int main ()
{
   time_t curtime;
 
   time(&curtime);
 
   printf("当前时间 = %s", ctime(&curtime));
 
   return(0);
}

```



--------

```c
struct tm *localtime(const time_t *time);

//该函数返回一个指向表示本地时间的 tm 结构的指针。

```

```c
#include <stdio.h>
#include <time.h>
 
int main ()
{
   time_t rawtime;
   struct tm *info;
 
   time( &rawtime );
 
   info = localtime( &rawtime );
   printf("当前的本地时间和日期：%s", asctime(info));
 
   return(0);
}
```

------

```c
clock_t clock(void);
//该函数返回程序执行起（一般为程序的开头），处理器时钟所使用的时间。如果时间不可用，则返回 -1。
```

```c

#include <time.h>
#include <stdio.h>
 
int main()
{
   clock_t start_t, end_t;
   double total_t;
   int i;
 
   start_t = clock();
   printf("程序启动，start_t = %ld\n", start_t);
    
   printf("开始一个大循环，start_t = %ld\n", start_t);
   for(i=0; i< 100000000; i++)
   {
   }
   end_t = clock();
   printf("大循环结束，end_t = %ld\n", end_t);
   
   total_t = (double)(end_t - start_t) / CLOCKS_PER_SEC;
   printf("CPU 占用的总时间：%f\n", total_t  );
 
   return(0);
}

/*
程序启动，start_t = 4710
开始一个大循环，start_t = 4710
大循环结束，end_t = 144218
CPU 占用的总时间：0.139508
*/
```

----------



```c
char *asctime(const struct tm *time);
//该函数返回一个指向字符串的指针，字符串包含了 time 所指向结构中存储的信息，返回形式为：day month date hours:minutes:seconds year\n\0。

```



```c
#include <stdio.h>
#include <string.h>
#include <time.h>

int main()
{
   struct tm t;

   t.tm_sec    = 10;
   t.tm_min    = 20;
   t.tm_hour   = 11;
   t.tm_mday   = 22;
   t.tm_mon    = 9;
   t.tm_year   = 20;
   t.tm_wday   = 5;

   puts(asctime(&t));
   
   return(0);
}
//Fri Oct 22 11:20:10 1920
```



------



```c
struct tm*gmtime(const time_t *time);
//该函数返回一个指向 time 的指针，time 为 tm 结构，用协调世界时（UTC）也被称为格林尼治标准时间（GMT）表示。
```

```c
#include <stdio.h>
#include <time.h>
 
#define BST (+1)
#define CCT (+8)
 
int main ()
{
 
   time_t rawtime;
   struct tm *info;
 
   time(&rawtime);
   /* 获取 GMT 时间 */
   info = gmtime(&rawtime );
   
   printf("当前的世界时钟：\n");
   printf("伦敦：%2d:%02d\n", (info->tm_hour+BST)%24, info->tm_min);
   printf("中国：%2d:%02d\n", (info->tm_hour+CCT)%24, info->tm_min);
 
   return(0);
}
```

---------



```c
time_t mktime(struct tm*time);
//该函数返回日历时间，相当于 time 所指向结构中存储的时间。

```

```c

/* 输入日期判断是周几 */
#include <stdio.h>      /* printf, scanf */
#include <time.h>       /* time_t, struct tm, time, mktime */
 
int main ()
{
    time_t rawtime;
    struct tm * timeinfo;
    int year, month ,day;
    const char * weekday[] = { "周日", "周一","周二", "周三","周四", "周五", "周六"};
 
    /* 用户输入日期 */
    printf ("年: "); fflush(stdout); scanf ("%d",&year);
    printf ("月: "); fflush(stdout); scanf ("%d",&month);
    printf ("日: "); fflush(stdout); scanf ("%d",&day);
 
    /* 获取当前时间信息，并修改用户输入的输入信息 */
    time ( &rawtime );
    timeinfo = localtime ( &rawtime );
    timeinfo->tm_year = year - 1900;
    timeinfo->tm_mon = month - 1;
    timeinfo->tm_mday = day;
 
    /* 调用 mktime: timeinfo->tm_wday  */
    mktime ( timeinfo );
 
    printf ("那一天是：%s\n", weekday[timeinfo->tm_wday]);
 
    return 0;
}

```

---------



```c
double difftime(time_t time2,time_t time1);
//该函数返回 time1 和 time2 之间相差的秒数。
```

```c
#include <stdio.h>
#include <time.h>
#ifdef _WIN32
#include <Windows.h>
#else
#include <unistd.h>
#endif
 
int main ()
{
   time_t start_t, end_t;
   double diff_t;
 
   printf("程序启动...\n");
   time(&start_t);
 
   printf("休眠 5 秒...\n");
   sleep(5);
 
   time(&end_t);
   diff_t = difftime(end_t, start_t);
 
   printf("执行时间 = %f\n", diff_t);
   printf("程序退出...\n");
 
   return(0);
}
```

-----



```c
size_t strftime();
//该函数可用于格式化日期和时间为指定的格式。
size_t strftime(char *str, size_t maxsize, const char *format, const struct tm *timeptr)
```

- **str** -- 这是指向目标数组的指针，用来复制产生的 C 字符串。
- **maxsize** -- 这是被复制到 str 的最大字符数。
- **format** -- 这是 C 字符串，包含了普通字符和特殊格式说明符的任何组合。这些格式说明符由函数替换为表示 tm 中所指定时间的相对应值。格式说明符是：

| 说明符 | 替换为                                                    | 实例                     |
| :----- | :-------------------------------------------------------- | :----------------------- |
| %a     | 缩写的星期几名称                                          | Sun                      |
| %A     | 完整的星期几名称                                          | Sunday                   |
| %b     | 缩写的月份名称                                            | Mar                      |
| %B     | 完整的月份名称                                            | March                    |
| %c     | 日期和时间表示法                                          | Sun Aug 19 02:56:02 2012 |
| %d     | 一月中的第几天（01-31）                                   | 19                       |
| %H     | 24 小时格式的小时（00-23）                                | 14                       |
| %I     | 12 小时格式的小时（01-12）                                | 05                       |
| %j     | 一年中的第几天（001-366）                                 | 231                      |
| %m     | 十进制数表示的月份（01-12）                               | 08                       |
| %M     | 分（00-59）                                               | 55                       |
| %p     | AM 或 PM 名称                                             | PM                       |
| %S     | 秒（00-61）                                               | 02                       |
| %U     | 一年中的第几周，以第一个星期日作为第一周的第一天（00-53） | 33                       |
| %w     | 十进制数表示的星期几，星期日表示为 0（0-6）               | 4                        |
| %W     | 一年中的第几周，以第一个星期一作为第一周的第一天（00-53） | 34                       |
| %x     | 日期表示法                                                | 08/19/12                 |
| %X     | 时间表示法                                                | 02:50:06                 |
| %y     | 年份，最后两个数字（00-99）                               | 01                       |
| %Y     | 年份                                                      | 2012                     |
| %Z     | 时区的名称或缩写                                          | CDT                      |
| %%     | 一个 % 符号                                               | %                        |

- 返回值
  - 如果产生的 C 字符串小于 size 个字符（包括空结束字符），则会返回复制到 str 中的字符总数（不包括空结束字符），否则返回零。

```c
#include <stdio.h>
#include <time.h>
 
int main ()
{
   time_t rawtime;
   struct tm *info;
   char buffer[80];
 
   time( &rawtime );
 
   info = localtime( &rawtime );
 
   strftime(buffer, 80, "%Y-%m-%d %H:%M:%S", info);
   printf("格式化的日期 & 时间 : |%s|\n", buffer );
  
   return(0);
}


/*

//当前时间戳
    struct timeval tv;
    struct tm* ptm;
    char  *time_str  = (char*)malloc(sizeof(char)*128);
    memset(time_str,0,128);
    

    //使用函数gettimeofday()函数来得到时间。它的精度可以达到微妙
    gettimeofday(&tv, NULL);

    ptm = localtime(&tv.tv_sec);//把从1970-1-1零点零分到当前时间系统所偏移的秒数时间转换为本地时间
    //strftime() 函数根据区域设置格式化本地时间/日期，函数的功能将时间格式化，或者说格式化一个时间字符串
    strftime(time_str, sizeof(time_str), "%Y-%m-%d %H:%M:%S", ptm);

    return time_str;
    
*/
```



[参考网站](https://www.json.cn/jiaocheng/co-395.html)
