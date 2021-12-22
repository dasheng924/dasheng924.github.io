---
title: redis的简单使用(一)
date: 2021-03-22
tag:
  - 非关系型数据库
  - redis
  - hiredis
categories:
  - Tools
keywords: "redis,缓存数据库"
cover: https://tva1.sinaimg.cn/large/008i3skNly1gsi1y87n53j30aa06vweq.jpg
highlight_shrink: false
sticky: 3
---

### 关系型数据库

- NoSQL 数据库的四大分类
  - KV键值，典型就是 redis
  - 文档型数据库MongoDB 
  - 列存储数据库 HBase
  - 图关系数据库 Neo4J

### redis 是什么？

- 开源的key-value 的存储系统，将大部分数据存储在内存中。
- redis基于内存操作，读写数据很快，作为内存型缓存服务器，搭配mysql可以做到数据持久化
- redis基于C语言开发，不需要过多依赖；客户端提供各种语言版本。      

### redis优点

- 完全在内存中保存数据库，使用磁盘为了持久化;速度异常快速
- 有丰富的数据类型，string，list，set，sorted set,hash
- 操作都是原子的，操作不会在执行完毕前被打断。从而确保当两个客户同时访问 redis 服务器得到的是更新后的值

### 相关资源

{% btn 'http://redis.cn',中文网站,far fa-hand-point-right %}

### redis 安装

#### 下载对应软件包

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsi2rwxb4ej30di07ot8v.jpg" alt="redis 安装包" style="zoom:50%;" />

#### 解压后，开始编译

```bash
make
sudo make install
```

#### 配置文件redis.cnf 在安装包里面

> 配置文件分成几大块：
>
> 1) 通用(general)
>
> 2) 快照(snapshotting)
>
> 3) 复制(replication)
>
> 4) 安全(security)
>
> 5) 限制(limits)
>
> 6) 追加模式(append only mode)
>
> 7) LUA脚本(lua scripting)
>
> 8) 慢日志(slow log)
>
> 9) 事件通知(event notification).       

```ini
daemonize no
守护进程
默认情况下，redis不是在后台运行的。如果需要在后台运行，把该项的值更改为yes。

pidfile /var/run/redis.pid
当redis在后台运行的时候，redis默认会把pid文件放在/var/run/redis.pid，你可以配置到其他位置。当运行多个redis服务时，需要指定不同的pid文件和端口。

port 6379
指定redis运行的端口，默认是6379。

bind 127.0.0.1
指定redis只接收来自于该IP地址的请求看，如果不进行设置，那么将处理所有请求。在生产环境中最好设置该项。
远程连接的话，需要把这行注释掉

		timeout 多长时间的等待，就会断连接,0表示永远连着
		
		keepalive 心跳检测

protected-mode no
远程访问需要设置为 no

loglevel debug
指定日志记录级别，其中redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose。
1．debug表示记录很多信息,用于开发和测试
2．verbose表示记录有用的信息, 但不像debug会记录那么多
3．notice表示普通的verbose，常用于生产环境
4．warning 表示只有非常重要或者严重的信息会记录到日志

logfile /var/log/redis/redis.log
配置log文件地址,默认值为stdout。若后台模式会输出到/dev/null。

databases 16
可用数据库数，默认值为16，默认数据库为0，数据库范围在0~15之间切换，彼此隔离。

save
保存数据到磁盘，格式为save，指出在多长时间内，有多少次更新操作，就将数据同步到数据文件rdb。相当于条件触发抓取快照，这个可以多个条件配合。 

 save 900 1 -- 900秒之内有1个keys发生变化时
 save 300 10 -- 300秒之内有10个keys发生变化时
 save 60 10000 -- 60秒之内有10000个keys发生变化时

rdbcompression yes
存储至本地数据库时(持久化到rdb文件)是否压缩数据，默认为yes。

dbfilename dump.rdb
本地持久化数据库文件名，默认值为dump.rdb。

dir ./
工作目录，数据库镜像备份的文件放置的路径。这里的路径跟文件名要分开配置是因为redis在进行备份时，先会将当前数据库的状态写入到一个临时文件中，等备份完成时，再把该临时文件替换为上面所指定的文件。 而这里的临时文件和上面所配置的备份文件都会放在这个指定的路径当中，AOF文件也会存放在这个目录下面。 注意这里必须指定一个目录而不是文件。

slaveof
主从复制，设置该数据库为其他数据库的从数据库。设置当本机为slave服务时，设置master服务的IP地址及端口。 在redis启动时,它会自动从master进行数据同步。

masterauth
当master服务设置了密码保护时(用requirepass制定的密码)slave服务连接master的密码。

slave-serve-stale-data yes
当从库同主机失去连接或者复制正在进行，从机库有两种运行方式：
如果slave-serve-stale-data设置为 yes(默认设置)，从库会继续相应客户端的请求。
如果slave-serve-stale-data是指为no，除去INFO和SLAVOF命令之外的任何请求都会返回一个错误"SYNC with master in progress"。

repl-ping-slave-period 10
从库会按照一个时间间隔向主库发送PING，可以通过repl-ping-slave-period设置这个时间间隔,默认是10秒。

repl-timeout 60
设置主库批量数据传输时间或者ping回复时间间隔，默认值是60秒，一定要确保repl-timeout大于repl-ping-slave-period。

requirepass foobared
设置客户端连接后进行任何其他指定前需要使用的密码。因为redis速度相当快，所以在一台比较好的服务器平台下, 一个外部的用户可以在一秒钟进行150K次的密码尝试，这意味着你需要指定非常强大的密码来防止暴力破解。

rename command CONFIG ""
命令重命名，在一个共享环境下可以重命名相对危险的命令，比如把CONFIG重名为一个不容易猜测的字符：
rename-command CONFIG  b840fc02d524045429941cc15f59e41cb7be6c52

如果想删除一个命令，直接把它重命名为一个空字符""即可：rename-command CONFIG ""。

maxclients 128
设置同一时间最大客户端连接数，默认无限制。redis可以同时打开的客户端连接数为redis进程可以打开的最大文件描述符数。 

如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，redis会关闭新的连接并向客户端返回max number of clients reached错误信息。

maxmemory 
指定redis最大内存限制。redis在启动时会把数据加载到内存中，达到最大内存后，redis会先尝试清除已到期或即将到期的key，redis同时也会移除空的list对象。当此方法处理后,仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。

注意：redis新的vm机制，会把key存放内存，value会存放在swap区。

maxmemory-policy volatile-lru
当内存达到最大值的时候redis会选择删除哪些数据呢？有五种方式可供选择：
1．volatile-lru代表利用LRU算法移除设置过期时间的key(LRU：最近使用LeastRecentlyUsed)
2．allkeys-lru代表利用LRU算法移除任何key
3．volatile-random代表移除设置过过期时间的随机key
4．allkeys_random代表移除一个随机的key
5． volatile-ttl代表移除即将过期的key(minor TTL)
6． noeviction代表不移除任何key，只是返回一个写错误

注意：对于上面的策略，如果没有合适的key可以移除，写的时候redis会返回一个错误。

appendonly no
默认情况下，redis会在后台异步的把数据库镜像备份到磁盘，但是该备份是非常耗时的，而且备份也不能很频繁。 如果发生诸如拉闸限电、拔插头等状况，那么将造成比较大范围的数据丢失，所以redis提供了另外一种更加高效的数据库备份及灾难恢复方式。

开启append only模式之后，redis会把所接收到的每一次写操作请求都追加到appendonly. aof文件中。当redis重新启动时，会从该文件恢复出之前的状态，但是这样会造成appendonly. aof文件过大，所以redis还支持BGREWRITEAOF指令对appendonly.aof。

appendfilename appendonly.aof
AOF文件名称，默认为"appendonly.aof"。

appendfsync everysec
redis支持三种同步AOF文件的策略：
1．no代表不进行同步,系统去操作
2．always代表每次有写操作都进行同步
3．everysec代表对写操作进行累积，每秒同步一次，默认是"everysec"，按照速度和安全折中这是最好的

slowlog-log-slower-than 10000
记录超过特定执行时间的命令。执行时间不包括I/O计算，比如连接客户端，返回结果等。只是命令执行时间，可以通过两个参数设置slow log：一个是告诉Redis执行超过多少时间被记录的参数slowlog-log-slower-than(微妙)，另一个是slow log 的长度。

当一个新命令被记录的时候最早的命令将被从队列中移除，下面的时间以微妙微单位，因此1000000代表一分钟。注意制定一个负数将关闭慢日志，而设置为0将强制每个命令都会记录。

hash-max-zipmap-entries 512 && hash-maxz-ipmap-value 64
当hash中包含超过指定元素个数并且最大的元素没有超过临界时，hash将以一种特殊的编码方式(大大减少内存使用)来存储，这里可以设置这两个临界值。Redis Hash对应Value内部实际就是一个HashMap，实际这里会有2种不同实现。这个Hash的成员比较少时redis为了节省内存会采用类似一维数组的方式来紧凑存储，而不会采用真正的HashMap结构，对应的value redisObject的encoding为zipmap。当成员数量增大时会自动转成真正的HashMap，此时encoding为ht。

hash-max-zipmap-entries 512 512
list数据类型多少节点以下会采用去指针的紧凑存储格式。

list-max-ziplist-value 64
数据类型节点值大小小于多少字节会采用紧凑存储格式。

setmaxintsetentries 512
set数据类型内部数据如果全部是数值型,且包含多少节点以下会采用紧凑格式存储。

zsetmaxziplistentries 128
zsort数据类型多少节点以下会采用去指针的紧凑存储格式。

zsetmaxziplistvalue 64
zsort数据类型节点值大小小于多少字节会采用紧凑存储格式。

activerehashing yes
redis将在每100毫秒时使用1毫秒的CPU时间来对redis的hash表进行重新hash，可以降低内存的使用。 

当你的使用场景中，有非常严格的实时性需要，不能够接受redis时不时的对请求有2毫秒的延迟的话，把这项配置为no。如果没有这么严格的实时性要求，可以设置为yes，以便能够尽可能快的释放内存。

			#设置密码
			config set requirepass 密码
			
			#用密码登录
			auth 密码
```

### redis 可视化工具

Mac版 `https://www.macwk.com/soft/redis-desktop-manager` 

### redis 启动服务

#### 启动server

`redis-server`

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshkkun5znj31cw0u0gw6.jpg" alt="image-20210715132834049" style="zoom:67%;" />

#### 也可以指定配置文件启动

`redis-server ./redis.conf`

![image-20210715133359786](https://tva1.sinaimg.cn/large/008i3skNly1gshkqhqqt0j31mm06awhp.jpg)

#### 打开就客户端连接服务器

`redis-cli`

#### 也可以远程连接，前提是配置过redis.conf,并以这个配置文件启动

`redis-cli -h 192.168.33.33 -p 8899`

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshmmlan8uj319g08qt9u.jpg" alt="远程连接redis server ，关闭保护模式" style="zoom:67%;" />

### redis 数据类型

| redis数据类型 | 含义     |
| ------------- | -------- |
| string        | 字符串   |
| list          | 列表     |
| hash          | 字典     |
| set           | 集合     |
| sorted set    | 有序集合 |
| pub/sub       | 订阅     |
| transactions  | 事务     |

{% btn 'http://redis.cn/commands.html',数据类型相关操作指令参考文档,far fa-hand-point-right %}

{% btn 'http://doc.redisfans.com',命令速查,far fa-hand-point-right %}

#### key的操作

>DBSIZE 	key的个数
>
>FIUSHDB   删当前库
>
>FLUSHALL  删所有库
>
>keys pattern  查找所有符合给定模式pattern （正则表达式）的key
>
>keys *  所有的key
>
>exists key 存在返回 1 ，不存在返回 0
>
>del key 删除对应的key值
>
>type key 返回key对应value值的数据类型
>
>expire key seconds     seconds 秒后key自动删除
>
>TTL key 查看key剩余的时间,-1 表示永不过期
>
>persist key 清除生存时间
>
>pexpire key milliseconds 设置生存时间为x x x毫秒
>
>rename key newkey key的重命名

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshnaa0m88j30vs0k0n0o.jpg" alt="image-20210715150212683" style="zoom:50%;" />

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsi2qw1uc4j30s60i075x.jpg" alt="image-20210715235709133" style="zoom:50%;" />

#### string 字符串

- string是简单的key-value 的类型，value不仅是string，也可以是数字。
- string是二进制安全的，可以包含任意的数据类型，string 可以看作是byte 数组，上限为1G字节。

```c
struct redis_string
{
	long len;
	long free;
	char buf[];
};
```

- len 数组buf的长度
- free 数组中可用的字节数
- buf char类型数组，用于存储实际的字符串内容

> set key value 设置key对应的值为string类型的value
>
> get key 获取key对应的value值，key不存在返回nil
>
> append key value

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshn46bfldj30pu0as75s.jpg" alt="image-20210715145620961" style="zoom:50%;" />

>setnx key value   key存在什么也不做，key不存在，相当于set
>
>msetnx key1 value1 key2 value2 有一个存在就不成功
>
>setex key seconds value 设置key对应字符串value，并且设置key在给定的seconds时间之后超时过期

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshngy25itj30us0j8q6e.jpg" alt="image-20210715150837301" style="zoom:50%;" />

>setrange key offset value  覆盖key对应的string的一部分，从指定的offset处开始，覆盖value的长度。如果offset比当前key对应string还要长，那这个string后面就补0以达到offset。
>
>incr key 1
>
>incrby  key 3  一次性加3
>
>dear key 1   
>
>nearby key 3
>
>setrange  key 0  10 从头开始，前两个字符设置为10
>
>getrange key start end 获取头到尾的字符

<img src="/Users/sunguosheng/Library/Application%20Support/typora-user-images/image-20210715153432030.png" alt="image-20210715153432030" style="zoom: 50%;" />

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gsho8vjd4pj30ym0dmwh1.jpg" alt="image-20210715153527907" style="zoom:50%;" />

> 批量的存取         
>
> mset key1 value1 key2 value2
>
> mget key1 key2 key3           

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshodd5w2sj317m08yq4j.jpg" alt="image-20210715153946272" style="zoom:50%;" />

### list 列表

- 本质双端链表
- 我们可以轻松地实现最新消息排行等功能(比如新浪微博的TimeLine )。List 的另一个应用就是消息队列，可以利用List的 PUSH操作，将任务存在List中，然后工作线程再用POP操作将任务取出进行执行。

>lpush key value1 value2 value3 (每次从左面插入新的value)
>
>lpushx key value1 value2 要求当且仅当key存在并且是一个列表
>
>lpop key 从左边移除第一个元素
>
>lrange key start stop 返回存储在key ,范围为start 到 stop 的所有元素，起始为0，最后为-1
>
>ltrim key start stop 让这个key对应的list只保留start到stop范围的元素  ，先截取再赋值
>
>lien key. 查看key 对应的value长度 

![image-20210715185643622](https://tva1.sinaimg.cn/large/008i3skNly1gshu2amclfj31jm0p6ju0.jpg)

>list 可以从右端进行插入
>
>rpush key value1 value2
>
>rpop key
>
>可以利用list来模拟栈和队列
>
>队列，尾入，头出
>
>rpush		lpop
>
>栈
>
>lpush  lpop 或者 rpush rpop             

### set 无序集合

- 集合指一堆不重复值的组合
- redis为集合提供了求交集和并集，差集的操作
- 比如在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。因为 Redis 非常人性化的为集合提供了求交集、并集、差集等操作，那么就可以非常方便的实现如共同关注、共同喜好、二度好友等功能，对上面的所有集合操作，你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。

> sadd key member1 member2 member3 添加一个或者多个member到集合key中
>
> smembers key 返回key集合所有的元素
>
> srem key member 删除key集合中指定的元素        
>
> SRANDMEMBER key num key中随机num个
>
> spop key 随机出
>
> smove source dest num
>
> 



<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshvsft7vlj312y0hugn0.jpg" alt="image-20210715195627033" style="zoom:67%;" />

>sinter key1 key2 key3   求集合的交集
>
>sunionstore dest key1 key2 返回给定的多个集合的并集中的所有成员，将结果存储destination集合
>
>sunion key1 key2 求并集
>
>sdiff key1 key2 求差集（A-B 和 B-A 是不一样的）    

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshw0ncqp9j317m0u0q4z.jpg" alt="image-20210715200420420" style="zoom:67%;" />

### sorted set 有序集合

- 在set的基础上加了一个权重参数 score，排序的依据就是这个score
- 比如一个存储全班同学成绩的 Sorted Sets，其集合 value 可以是同学的学号，而 score 就可以是其考试得分，这样在数据插入集合的时候，就已经进行了天然的排序。另外还可以用 Sorted Sets 来做带权重的队列，比如普通消息的 score 为1，重要消息的score 为2，然后工作线程可以选择按 score 的倒序来获取工作任务，让重要的任务优先执行。

>zadd key score1 member1 score2 member2 score3 member3
>
>zrange key start stop withscores 从小到大
>
>zrevrange key start stop withscores  从大到小       
>
>zrangebyscore key start_score end_score 筛选出start_score 到end_score 的value
>
>ZRANGEBYSCORE zset01 (20 (60 	大于20小于60
>
>ZRANGEBYSCORE zset01 (20 (60 limit 2 1  大于20小于60的结果集从第三个开始取一个
>
>ZREM zset01 v1 删除zset01中的v1
>
>ZCOUNT zset01 60 80 取分数再60-80的值的个数
>
>ZRANK zset01 v4 取V4在zset01中的下标
>
>ZREVRANK zset01 v4  取V4在zset01中的逆序下标
>
>ZSCORE zset01 v4 取v4 对应的分数

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshw8ayij0j31au0e2q49.jpg" alt="image-20210715201142224" style="zoom:80%;" />

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshw8wfnrfj310209odgd.jpg" alt="image-20210715201216665" style="zoom: 50%;" />

### hash 字典

- 键值key 对应的是 field字段名称  value 字段值的结合(也就是还是一个键值对)

>hset key field value 设置key指定的哈希集中的字段和字段值
>
>hmset key field1 value1 field2 value2
>
>hget key field key键值里面field 字段对应的字段值
>
>hmget key field1 field2
>
>hgetall key  获取所有的字段和字段值 ，注意和HVALS key --->只获取所有的字段值，不获取对应的字段
>
>hkeys key 返回 key 指定的哈希集中所有字段的名字
>
>hdel key field 删除key指定的哈希集中字段为field的字段
>
>hlen key 返回key指定的哈希集中字段的数量          
>
>hexists key field  判断对应的字段值是否存在
>
>HVALS key 取所有的字段值
>
>

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshwrfu84wj31el0u042h.jpg" alt="image-20210715203005524" style="zoom:80%;" />   

### pub/sub 发布/订阅

- Pub/Sub 从字面上理解就是发布(Publish)与订阅(Subscribe)。发件人(在 Redis 中的术语称为发布者)发送邮件，而接收器(订户)接收它们。信息传输的链路称为通道。Redis 一个客户端可以订阅任意数量的通道。
- 在 Redis 中，你可以设定对某一个key 值进行消息发布及消息订阅，当一个 key 值上进行了消息发布后，所有订阅它的客户端都会收到相应的消息。这一功能最明显的用法就是用作实时消息系统，比如普通的即时聊天、群聊等功能。

>subscribe channel 订阅给指定频道的信息。一旦客户端进入订阅状态，客户端就只可接受订阅相关的命令。
>
>publish channel message 将信息 message 发送到指定的频道 channel。     

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshx0letjij313808k3zb.jpg" alt="image-20210715203853735" style="zoom:67%;" />

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshx0zg4q5j30zk0gsabd.jpg" alt="image-20210715203915977" style="zoom:67%;" />

### transactions 事务

- 事务允许一组命令在单一步骤中执行，也就是一组命令的集合，也就是一次性批处理
- redis 事务具有原子性，所有命令要么都执行，要么都不执行
- 事务中的所有命令作为单个独立的操作  顺序执行
- 一个队列中，一次性的，顺序性的，拍他性的执行一组命令
- redis的事务性是部分的，在一些情况下是不保证完整性的，下面的后两种状态就显示出来了这个特点

>multi 标记事务的开始
>
>exec 执行事务中排队等待的指令并将链接状态恢复正常
>
>discard 放弃本次的批处理操作
>
>watch 监视一个或者多个key(这个类似于乐观锁，当中间有人修改之后，就会报错；需要重新 unwatch ，拿下数据，再去操作)
>
>unwatch 取消对所有key的监控         

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshxa004egj315t0u0jtv.jpg" alt="image-20210715204755336" style="zoom:80%;" />

#### 事务的几种状态

- 全部执行成功，每一条命令都入队，然后执行.       

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt79wxeiomj30he0ie0tu.jpg" alt="image-20210806190413020" style="zoom:33%;" />

- 放弃事务           

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt79yhuny7j30is0d0dgq.jpg" alt="image-20210806190546419" style="zoom:33%;" />



- 有一个错误，全部就丢失，这个类似于 编译时的报错

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt7a0oihjsj319o0agt9u.jpg" alt="image-20210806190752712" style="zoom:33%;" />

- 那个错误就找错误的那个，其余的能执行成功（这个类似于运行时的错误）

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt7aq93i4mj30z00i00u8.jpg" alt="image-20210806193226867" style="zoom:33%;" />

#### redis的主从复制

- 主机数据更新后，根据配置和策略自动同步到备机的master/slaver机制，Master主要为写，Slave以读为准
- 主要干的事情就是 读写复制和容载备份
- 怎么用
  - 配从库不配主库 `slaveof 主库IP 主库PORT`，每次和`mastre`需要重新连接
  - 修改配置文件中的端口，log目录，rdb 文件名，对应端口，用来做区分
  - `slaveof ip port`后，从机会复制主机所有的内容；只有主机可以写的，从机只能读；当主机宕机后，从机不会变成主机，主机重新启动之后，会继续当主机的身份
  - `info repcalition` 查看机器`replication`信息，需要关注`role`字段值

#### 一主二仆

- init.        

 <img src="https://tva1.sinaimg.cn/large/008i3skNly1gt8gntzmm3j311w0fa0xe.jpg" alt="image-20210807194315072" style="zoom:50%;" />

- 一个Master ，两个Slaves

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt8goh2xbjj61280i6aff02.jpg" alt="image-20210807194354594" style="zoom:50%;" />

- 主机日志需要关注的点

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt8gpj0np5j31140d676p.jpg" alt="image-20210807194454785" style="zoom:50%;" />

- 备机日志需要关注的点

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt8gq08fggj311i0bk40i.jpg" alt="image-20210807194522564" style="zoom:50%;" />

- 几个需要关注的问题
  - 切入点问题？slave1，slave2 是从头开始备份，还是从切入点开始复制？
    - --》从头开始复制
  - 从机是否可写？
    - --》主机可写，从机不可写
  - 主机shutdown后，从机是变成主机还是原地等待主机？
    - 从机原地等待
  - 主机又回来后，主机新增的记录，从机是否可以顺利的复制？
    - 从机可以自动与主机相连，也就可以自动同步主机新增的数据 。
  - 如果slave与Master断开之后，后面还能跟上master的脚步吗？
    - slave与master断开之后，不会自动重新连接，需要配置进 `redis.conf ####REPLICATION####`

#### 薪火相传

- 去中心化的模式（减轻了master的负担），一个传递一个
- 上一个的slave可以是下一个的master；中途方向变更后，会重新复制最新的数据
- `slaveof ip port`            

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt8nk2kqy2j31gm0kyjvy.jpg" alt="image-20210807234151934" style="zoom:67%;" />



#### 反客为主

- `SLAVEOF no one`使当前数据库停止与其他数据库的同步，转成主数据库

#### 复制的原理

- slave连接到master后，会发出一个`sync`的命令
- master接收到命令会启动一个存盘进程，后台收集所有用于修改的命令，收集完毕后，会把整个数据传送给slave
- 全量复制：slave服务在接收到数据库文件数据后，将其存盘并加载到内存中
- 增量复制：Master继续将新的所有收集到的修改命令依次传给slave,完成同步
- 只要是重新连接master，一次完全同步（全量复制)将被自动执行



#### 哨兵模式

- 一组sentinel能同时监控多个master
- 反客为主的自动版，能够监控主机是否故障，故障之后按照投票数将从库变为主库
- 设置
  - 调整结构，一个主库带着两个从库
  - 新建`sentinel.conf`文件，名字绝不能错
  - 配置哨兵
    - `sentinel monitor 被监控的数据库名字（自己起）监控的库ip 监控的库port  1（要大于的票数）`
    - 新恢复的master就变成了slave，也就是master已经易主了
  - 启动哨兵 `redis-sentinel /sentinel.conf`

#### 复制的缺点

- master到slave会有一些延迟，在业务繁忙的时候就会有更多的延迟

  

### redis 持久化

#### 备份数据

- `save`命令即可创建当前redis 的数据的备份，成功之后，会在服务器启动的目录生成对应的dump.rdb文件，文件名和文件路径都可以进行修改，在配置文件中可以设置对应的属性值，可以用 `CONFIG GET dir`命令张查看redis的启动目录

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshydgo13pj31110u0go0.jpg" alt="image-20210715212551022" style="zoom:67%;" />

- 会在对应目录 生成dump.rdb

#### 数据恢复

- redis 启动的时候会自动加载备份文件

#### RDB(redia database)持久化方式 （默认）

- 通过快照完成，dump.rdb 文件里面放的是真实的数据，也就是将内存中所有的数据进行快照到硬盘上面
- 快照的频率在 redis.conf 配置文件里面

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gshyjbl1dkj31e90u0gpv.jpg" alt="image-20210715213128304" style="zoom:80%;" />

- 多个频率设定的条件，只要有一个满足就会备份
- 如果不想备份，注释掉 save的参数
- 备份的文件名子和路径可以通过 dir 和 dbfilename 参数设定
- FLUSHALL 都会触发RDB的备份,但是产生的dump.rdb 文件是空的

>redis实现快照的过程：
>
>redis使用fork函数复制一份当前进程(父进程)的副本(子进程)，父进程继续接收并处理客户端发来的命令，而子进程开始将内存中的数据写入硬盘中的临时文件，当子进程写入完所有数据后会用该临时文件替换旧的RDB文件，至此一次快照操作完成。在执行fork的时候操作系统(类Unix操作系统)会使用写时复制(copy-on-write)策略，即fork函数发生的一刻父子进程共享同一内存数据，当父进程要更改其中某片数据时(如执行一个写命令)，操作系统会将该片数据复制一份以保证子进程的数据不受影响，所以新的RDB文件存储的是执行fork一刻的内存数据。
>
>redis在进行快照的过程中不会修改RDB文件，只有快照结束后才会将旧的文件替换成新的，也就是说任何时候RDB文件都是完整的。这使得我们可以通过定时备份RDB文件来实现redis数据库备份。RDB文件是经过压缩(可以配置rdbcompression参数以禁用压缩节省CPU占用)的二进制格式，所以占用的空间会小于内存中的数据大小，更加利于传输。
>
>除了自动快照，还可以手动发送SAVE或BGSAVE命令让redis执行快照，两个命令的区别在于，前者是由主进程进行快照操作, 会阻塞住其他请求，后者会通过fork子进程进行快照操作。
>
>redis启动后会读取RDB快照文件，将数据从硬盘载入到内存。根据数据量大小与结构和服务器性能不同，这个时间也不同。通常将一个记录一千万个字符串类型键、大小为1GB的快照文件载入到内存中需要花费20~30秒钟。
>
>通过RDB方式实现持久化，一旦redis异常退出，就会丢失最后一次快照以后更改的所有数据。这就需要开发者根据具体的应用场合，通过组合设置自动快照条件的方式来将可能发生的数据损失控制在能够接受的范围。如果数据很重要以至于无法承受任何损失，则可以考虑使用AOF方式进行持久化.              

#### AOF(Append only file) 数据持久化

- 默认没有开启AOF方式
- `appendonly yes` 开启AOF 方式备份 
- AOF方式会把执行的每一条命令写指令存进磁盘；恢复的时候，会执行命令，恢复数据。
- 恢复的时候却需要先执行命令，才会恢复数据，相比RDB比较费时间

- AOF 文件名设定

![image-20210715214354527](https://tva1.sinaimg.cn/large/008i3skNly1gshyw95f1mj31a408ugmh.jpg)

- 刷新硬盘缓存频率

![image-20210715214416705](https://tva1.sinaimg.cn/large/008i3skNly1gshywmog4oj30wy08uq3j.jpg)

- 自动重写的条件

  - 当AOF文件大小是上次rewrite大小的一倍且文件大于64M时就会重写

  - rewrite 重写机制，就优化命令，减下磁盘占用

    

![image-20210715214507252](https://tva1.sinaimg.cn/large/008i3skNly1gshyxi13icj616206i74n02.jpg)



#### AOF 和 RDB共存的问题

- AOF和RDB会共存
- 先去加载的是 AOF，当AOF中有错误的时候，`redis-server`会启动失败，需要修复AOF文件，使用redis 带的修复工具进行修复 `redis-check-aof --fix xxx.aof`

#### AOF 和 RDB 对比

- AOF
  - 优点：备份数据较完成，最多丢失2秒的数据
  - 缺点：磁盘占用较大和恢复较慢
- RDB
  - 优点：恢复数据较快
  - 缺点：备份频率不容易控制，会丢失最后一次的RDB之后的数据



### redis 客户端编程

{% btn 'https://github.com/redis/hiredis',下载地址,far fa-hand-point-rigth %}

```bash
make 
sudo make install
```

#### API 说明

```c
redisContext *redisConnect(const char *ip,int port)
功能：连接redis数据库
参数：IP 和 port
返回值： 成功：redisContext 指针 ；失败 NULL
---------------------------
void *redisCommand(redisContext *c,const char * formate,...)
功能：执行命令
参数：redisConnect的返回值
  	命令......
返回值： 成功：void *,会转换成redisReply类型 ；失败 NULL
  
typedef struct redisReply {
	int type; /* REDIS_REPLY_* */
	long long integer; /* The integer when type is REDIS_REPLY_INTEGER */
	size_t len; /* Length of string */
	char *str; /* Used for both REDIS_REPLY_ERROR and REDIS_REPLY_STRING */
	size_t elements; /* number of elements, for REDIS_REPLY_ARRAY */
	struct redisReply **element; /* elements vector for REDIS_REPLY_ARRAY */
} redisReply;
---------------------------
void freeReplyObject(void *reply);
功能：释放redisCommand执行后返回的redisReply所占用的内存
参数：redisCommand执行后返回的redisReply
返回值：无
---------------------------
void redisFree(redisContext *c);
功能：释放redisConnect()所产生的连接
参数：redisConnect()所产生的连接
返回值：无
```

- redisReply ->type 中字段的标识 

| 状态标识            | 含义                                                         |
| ------------------- | ------------------------------------------------------------ |
| REDIS_REPLY_STATUS  | 表示状态，内容通过str字段查看，字符串长度是len字段           |
| REDIS_REPLY_ERROR   | 表示出错，查看出错信息，如上的str，len字段                   |
| REDIS_REPLY_INTEGER | 返回整数，从integer字段获取值                                |
| REDIS_REPLY_NIL     | 没有数据返回                                                 |
| REDIS_REPLY_STRING  | 返回字符串，查看str，len字段                                 |
| REDIS_REPLY_ARRAY   | 返回一个数组，查看elements的值(数组个数)，通过element[index] 的方式访问数组元素，每个数组元素是一个redisReply对象的指针。 |

- redisReply->errata 字段查看

| 错误状态标识       | 含义                               |
| ------------------ | ---------------------------------- |
| REDIS_OK           | 正常                               |
| REDIS_ERR_IO       | IO读/写出现异常，通过errno查看原因 |
| REDIS_ERR_EOF      | 服务器关闭了链接，读结束           |
| REDIS_ERR_PROTOCOL | 分析redis协议内容出错              |
| EDIS_ERR_OTHER     | 其他未知的错误                     |



#### 测试用例

```c
#include <stdio.h>
#include <stdlib.h>
#include <stddef.h>
#include <stdarg.h>
#include <string.h>
#include <assert.h>
#include </usr/local/include/hiredis/hiredis.h>

#define IP "10.211.55.11"
#define PORT 8899

void doWork()
{
    //port 8899 ip 10.211.55.11
    redisContext * c = redisConnect(IP,PORT);
    if(c->err)
    {
        redisFree(c);
        perror("connect redis server fail!");
        return;
    }

    puts("connect redis server ok!");
//----------------
    const char * command1 = "set myname sungs";
    redisReply * r = (redisReply*)redisCommand(c,command1);
    if(r == NULL)
    {
        redisFree(c);
        perror("command1 error!");
        return;
    }

    if(!(r->type == REDIS_REPLY_STATUS && strcasecmp(r->str,"ok") == 0))
    {
        printf("Failed to execute command[%s]\n",command1);
		freeReplyObject(r);
		redisFree(c);
		return;
    }

    freeReplyObject(r);
	printf("Succeed to execute command[%s]\n", command1);
//-----------------
    const char *command2 = "strlen myname";

    r = (redisReply *)redisCommand(c,command2);
    if(r == NULL)
    {
        redisFree(c);
        perror("command2 error!");
        return ;
    }

    if(r->type != REDIS_REPLY_INTEGER)
    {
        freeReplyObject(r);
        redisFree(c);
        perror("REDIS_REPLY_INTEGE error");
        return;
    }

    int length = r->integer;
    freeReplyObject(r);
    printf("Succeed to execute command[%s]--->[%d]\n", command2,length);

    const char* command3 = "get myname";
	r = (redisReply*)redisCommand(c, command3);
	if ( r->type != REDIS_REPLY_STRING)
	{
		printf("Failed to execute command[%s]\n",command3);
		freeReplyObject(r);
		redisFree(c);
		return;
	} 

	printf("The value of 'myname' is %s\n", r->str);
	freeReplyObject(r);
	printf("Succeed to execute command[%s]\n", command3);

	const char* command4 = "get key2";
	r = (redisReply*)redisCommand(c, command4);
	if ( r->type == REDIS_REPLY_NIL)
	{
		printf("Failed to execute command[%s]\n",command4);
		freeReplyObject(r);
		redisFree(c);
		return;
	} 
	
	freeReplyObject(r);
	printf("Succeed to execute command[%s]\n", command4);
	redisFree(c);

}



int main(void)
{
	doWork();
	return 0;
}
```

![image-20210715231818738](https://tva1.sinaimg.cn/large/008i3skNly1gsi1mh7socj31bo0butak.jpg)

### 接口封装

```c
/**
 * @file   redis_api.h
 * @brief  redis 封装接口
 */

#ifndef _REDIS_OP_H_
#define _REDIS_OP_H_

#include <hiredis.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
#include "make_log.h"


#define REDIS_LOG_MODULE          "database"
#define REDIS_LOG_PROC            "redis"

#define REDIS_COMMAND_SIZE        300            /* redis Command 指令最大长度 */
#define FIELD_ID_SIZE            100            /* redis hash表field域字段长度 */
#define VALUES_ID_SIZE           1024            /* redis        value域字段长度 */
typedef char (*RCOMMANDS)[REDIS_COMMAND_SIZE];/* redis 存放批量 命令字符串数组类型 */
typedef char (*RFIELDS)[FIELD_ID_SIZE];        /* redis hash表存放批量field字符串数组类型 */
typedef char (*RVALUES)[VALUES_ID_SIZE];    /* redis 表存放批量value字符串数组类型 */


/* -------------------------------------------*/
/**
 * @brief  redis tcp模式链接
 *
 * @param ip_str	redis服务器ip
 * @param port_str	redis服务器port
 *
 * @returns   
 *			成功返回链接句柄 
 *			失败返回NULL
 */
/* -------------------------------------------*/
redisContext* rop_connectdb_nopwd(char *ip_str, char* port_str);


/* -------------------------------------------*/
/**
 * @brief  redis tcp模式链接
 *
 * @param ip_str    redis服务器ip
 * @param port_str  redis服务器port
 * @param pwd       redis服务器密码
 *
 * @returns   
 *            成功返回链接句柄 
 *            失败返回NULL
 */
/* -------------------------------------------*/
redisContext* rop_connectdb(char *ip_str, char* port_str, char *pwd);

/* -------------------------------------------*/
/**
 * @brief  redis unix域模式链接
 *
 * @param ip_str    unix域sock文件
 * @param  pwd      redis服务器密码
 *
 * @returns   
 *            成功返回链接句柄 
 *            失败返回NULL
 */
/* -------------------------------------------*/
redisContext* rop_connectdb_unix(char *sock_path, char *pwd);

/* -------------------------------------------*/
/**
 * @brief  tcp 链接redis超时等待模式，timeval链接超时
 *            返回
 *
 * @param ip_str        redis 服务器ip
 * @param port_str        redis 服务器端口
 * @param timeval        最大超时等待时间
 *
 * @returns   
 *        成功返回链接句柄
 *        失败返回NULL
 */
/* -------------------------------------------*/
redisContext* rop_connectdb_timeout(char* ip_str, char *port_str, struct timeval *timeout);


/* -------------------------------------------*/
/**
 * @brief  关闭指定的链接句柄
 *
 * @param conn    已建立好的链接
 */
/* -------------------------------------------*/
void rop_disconnect(redisContext* conn);

/* -------------------------------------------*/
/**
 * @brief  选择redis中 其中一个数据库
 *
 * @param conn        已链接的数据库链接
 * @param db_no        redis数据库编号
 *
 * @returns   
 *            -1 失败
 *            0  成功
 */
/* -------------------------------------------*/
int rop_selectdatabase(redisContext *conn, unsigned int db_no);

/* -------------------------------------------*/
/**
 * @brief            清空当前数据库所有信息(慎用)
 *
 * @param conn        已链接的数据库链接
 *
 * @returns   
 *            -1 失败
 *            0  成功
 */
/* -------------------------------------------*/
int rop_flush_database(redisContext *conn);

/* -------------------------------------------*/
/**
 * @brief  判断key值是否存在
 *
 * @param conn        已经建立的链接
 * @param key        需要寻找的key值
 *
 * @returns   
 *                -1 失败
 *                1 存在
 *                0 不存在
 */
/* -------------------------------------------*/
int rop_is_key_exist(redisContext *conn, char* key);

/* -------------------------------------------*/
/**
 * @brief            删除一个key
 *
 * @param conn        已经建立的链接
 * @param key        
 *
 * @returns   
 *                -1 失败
 *                0 成功
 */
/* -------------------------------------------*/
int rop_del_key(redisContext *conn, char *key);


/* -------------------------------------------*/
/**
 * @brief  打印库中所有匹配pattern的key
 *
 * @param conn        已建立好的链接
 * @param pattern    匹配模式，pattern支持glob-style的通配符格式，
 *                    如 *表示任意一个或多个字符，
 *                       ?表示任意字符，
 *                    [abc]表示方括号中任意一个字母。
 */
/* -------------------------------------------*/
void rop_show_keys(redisContext *conn, char* pattern);

/* -------------------------------------------*/
/**
 * @brief  设置一个key的删除时间 ，系统到达一定时间
 *            将会自动删除该KEY
 *
 * @param conn                已经建立好的链接
 * @param delete_time        到期事件 time_t 日历时间
 *
 * @returns   
 *        0    SUCC
 *        -1  FAIL
 */
/* -------------------------------------------*/
int rop_set_key_lifecycle(redisContext *conn, char *key, time_t delete_time);

/* -------------------------------------------*/
/**
 * @brief            创建或者覆盖一个HASH表
 *
 * @param conn                已建立好的链接
 * @param key                hash 表名
 * @param element_num        hash 表区域个数
 * @param fields            hash 表区域名称数组char(*)[FIELD_ID_SIZE]
 * @param values            hash 表区域值数组  char(*)[VALUES_ID_SIZE]
 *
 * @returns   
 *            0   成功    
 *            -1  失败
 */
/* -------------------------------------------*/
int rop_create_or_replace_hash_table(redisContext* conn,
                                     char* key, 
                                     unsigned int element_num, 
                                     RFIELDS fields, 
                                     RVALUES values);

/* -------------------------------------------*/
/**
 * @brief  给指定的hash表 指定的field对应的value自增num
 *
 * @param conn			已建立好的链接
 * @param key			hash表名
 * @param field			hash表下的区域名
 *
 * @returns
 *			0		succ
 *			-1		fail
 */
/* -------------------------------------------*/
int rop_hincrement_one_field(redisContext *conn, char *key, char *field, unsigned int num);


/* -------------------------------------------*/
/**
 * @brief  批量执行链表插入命令 插入链表头部
 *
 * @param conn        已建立好的链接
 * @param key        链表名
 * @param values    封装好的域名
 * @param values    封装好的值数组
 * @param val_num    值个数
 *
 * @returns   
 *            0        succ
 *            -1        FAIL
 */
/* -------------------------------------------*/
int rop_hash_set_append(redisContext *conn, char *key, RFIELDS fields, RVALUES values, int val_num);

/* -------------------------------------------*/
/**
 * @brief  想一个hash表中添加一条 key-value 数据
 *
 * @param conn  redis连接
 * @param key   哈希表名
 * @param field
 * @param value
 *
 * @returns
 *            0        succ
 *            -1        FAIL
 */
/* -------------------------------------------*/
int rop_hash_set(redisContext *conn, char *key, char *field, char *value);

/* -------------------------------------------*/
/**
 * @brief  从一个hash表中取出一条 key-value 数据
 *
 * @param conn  redis连接
 * @param key   哈希表名
 * @param field 字段名称
 * @param value 得到的数据， 需要先开辟内存
 *
 * @returns
 *            0        succ
 *            -1        FAIL
 */
/* -------------------------------------------*/
int rop_hash_get(redisContext *conn, char *key, char *field, char *value);



/* -------------------------------------------*/
/**
 * @brief        将指定的zset表，对应的成员，值自增1
 *                （key 或 成员不存在 则创建）
 *
 * @param conn        已建立的链接
 * @param key        zset表名
 * @param member    zset成员名
 *
 * @returns   
 *            0            succ
 *            -1            fail
 */
/* -------------------------------------------*/
int rop_zset_increment(redisContext *conn, char* key, char* member);


//得到zset一个member的score
int rop_zset_get_score(redisContext *conn, char *key, char *member);

/* -------------------------------------------*/
/**
 * @brief     批量将指定的zset表，对应的成员，值自增1
 *                （key 或 成员不存在 则创建）
 *
 * @param conn        已建立好的链接
 * @param key        有序集合名称
 * @param values    封装好的成员数组
 * @param val_num    数据个数
 *
 * @returns   
 *            0        succ
 *            -1        FAIL
 */
/* -------------------------------------------*/
int rop_zset_increment_append(redisContext *conn, char *key, RVALUES values, int val_num);

/* -------------------------------------------*/
/**
 * @brief  批量执行链表插入命令 插入链表头部
 *
 * @param conn        已建立好的链接
 * @param key        链表名
 * @param values    封装好的值数组
 * @param val_num    值个数
 *
 * @returns   
 *            0        succ
 *            -1        FAIL
 */
/* -------------------------------------------*/
int rop_list_push_append(redisContext *conn, char *key, RVALUES values, int val_num);

/* -------------------------------------------*/
/**
 * @brief  单条数据插入链表
 *
 * @param conn        已建立好的链接
 * @param key        链表名
 * @param value        数据
 *
 * @returns   
 */
/* -------------------------------------------*/
int rop_list_push(redisContext *conn, char *key, char *value);

/* -------------------------------------------*/
/**
 * @brief  得到链表中元素的个数
 *
 * @param conn    链接句柄
 * @param key    链表名
 *
 * @returns   
 *            >=0 个数
 *            -1 fail
 */
/* -------------------------------------------*/
int rop_get_list_cnt(redisContext *conn, char *key);

/* -------------------------------------------*/
/**
 * @brief  按照一定范围截断链表中的数据
 *
 * @param conn        已经建立的链接
 * @param key        链表名
 * @param begin        阶段启示位置 从 0 开始
 * @param end        阶段结束位置 从 -1 开始
 *
 *                    这里的范围定义举例 
 *                    如果得到全部范围(0, -1)
 *                    除了最后一个元素范围(0, -2)
 *                    前20各数据范围(0, 19)
 *
 * @returns   
 *            0  SUCC
 *            -1 FAIL
 */
/* -------------------------------------------*/
int rop_trim_list(redisContext *conn, char *key, int begin, int end);

/* -------------------------------------------*/
/**
 * @brief          得到链表中的数据
 *
 * @param conn		已经建立的链接
 * @param key		链表名
 *
 * @returns   
 *			0  SUCC
 *			-1 FAIL
 */
/* -------------------------------------------*/
int rop_range_list(redisContext *conn, char *key, int from_pos, int count, RVALUES values, int *get_num);


/* -------------------------------------------*/
/**
 * @brief  批量执行已经封装好的redis 命令
 *
 * @param conn        已建立好的链接
 * @param cmds        封装好的命令数组
 * @param cmd_num    命令个数
 *
 * @returns   
 *            0        succ
 *            -1        FAIL
 */
/* -------------------------------------------*/
int rop_redis_append(redisContext *conn, RCOMMANDS cmds, int cmd_num);


/* -------------------------------------------*/
/**
 * @brief  执行单向命令 无返回值 命令自行输入
 *
 * @param conn        已建立的链接
 * @param cmd        封装好的命令
 *
 * @returns   
 *            0        succ
 *            -1        FAIL
 */
/* -------------------------------------------*/
int rop_redis_command(redisContext *conn, char *cmd);

/* -------------------------------------------*/
/**
 * @brief  测试一个reply的结果类型
 *            得到对应的类型用对应的方法获取数据
 *
 * @param reply        返回的命令结果
 */
/* -------------------------------------------*/
void rop_test_reply_type(redisReply *reply);


/* -------------------------------------------*/
/**
 * @brief  设置key对应的值为string类型的value
 *            
 * @param conn          已经建立好的链接
 * @param key        	key值
 * @param value         value值
 *
 * @returns   
 *        0    SUCC
 *        -1  FAIL
 */
/* -------------------------------------------*/
int rop_set_string(redisContext *conn, char *key, char *value);

/* -------------------------------------------*/
/**
 * @brief  获取key对应的value值
 *            
 * @param conn          已经建立好的链接
 * @param key        	key值
 * @param value         value值
 *
 * @returns   
 *        0    SUCC
 *        -1  FAIL
 */
/* -------------------------------------------*/
int rop_get_string(redisContext *conn, char *key, char *value);

#endif

```

```c
/**
 * @file redis_op.c
 * @brief  redis 操作基本接口和key的操作实现
 */

#include "redis_op.h"


/* -------------------------------------------*/
/**
 * @brief  选择redis一个数据库
 *
 * @param conn		已链接的数据库链接
 * @param db_no		redis数据库编号
 *
 * @returns   
 *			-1 失败
 *			0  成功
 */
/* -------------------------------------------*/
int rop_selectdatabase(redisContext *conn, unsigned int db_no)
{
	int retn = 0;
	redisReply *reply = NULL;

	/* 选择一个数据库 */
	reply = redisCommand(conn, "select %d", db_no);
	if (reply == NULL) {
		fprintf(stderr, "[-][GMS_REDIS]Select database %d error!\n", db_no);
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Select database %d error!%s\n", db_no, conn->errstr);
		retn = -1;
		goto END;
	}

	printf("[+][GMS_REDIS]Select database %d SUCCESS!\n", db_no);
	LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[+][GMS_REDIS]Select database %d SUCCESS!\n", db_no);

END:
	freeReplyObject(reply);
	return retn;
}


/* -------------------------------------------*/
/**
 * @brief			清空当前数据库所有信息(慎用)
 *
 * @param conn		已链接的数据库链接
 *
 * @returns   
 *			-1 失败
 *			0  成功
 */
/* -------------------------------------------*/
int rop_flush_database(redisContext *conn)
{
	int retn = 0;	
	redisReply *reply = NULL;

	reply = redisCommand(conn, "FLUSHDB");
	if (reply == NULL) {
		fprintf(stderr, "[-][GMS_REDIS]Clear all data error\n");
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Clear all data error\n");
		retn = -1;
		goto END;
	}

	printf("[+][GMS_REDIS]Clear all data!!\n");
	LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC,"[+][GMS_REDIS]Clear all data!!\n");

END:
	freeReplyObject(reply);
	return retn;
}

/* -------------------------------------------*/
/**
 * @brief  判断key值是否存在
 *
 * @param conn		已经建立的链接
 * @param key		需要寻找的key值
 *
 * @returns   
 *				-1 失败
 *				1 存在
 *				0 不存在
 */
/* -------------------------------------------*/
int rop_is_key_exist(redisContext *conn, char* key)
{
	int retn = 0;	

	redisReply *reply = NULL;

	reply = redisCommand(conn, "EXISTS %s", key);
	//rop_test_reply_type(reply);
	if (reply->type != REDIS_REPLY_INTEGER) {
		fprintf(stderr, "[-][GMS_REDIS]is key exist get wrong type!\n");
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]is key exist get wrong type! %s\n", conn->errstr);
		retn = -1;
		goto END;
	}

	if (reply->integer == 1) {
		retn = 1;	
	}
	else {
		retn = 0;
	}

END:
	freeReplyObject(reply);
	return retn;
}

/* -------------------------------------------*/
/**
 * @brief			删除一个key
 *
 * @param conn		已经建立的链接
 * @param key		
 *
 * @returns   
 *				-1 失败
 *				0 成功
 */
/* -------------------------------------------*/
int rop_del_key(redisContext *conn, char *key)
{
	int retn = 0;
	redisReply *reply = NULL;

	reply = redisCommand(conn, "DEL %s", key);
	if (reply->type != REDIS_REPLY_INTEGER) {
		fprintf(stderr, "[-][GMS_REDIS] DEL key %s ERROR\n", key);
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS] DEL key %s ERROR %s\n", key, conn->errstr);
		retn = -1;
		goto END;
	}

	if (reply->integer > 0) {
		retn = 0;	
	}
	else {
		retn = -1;
	}

END:
	freeReplyObject(reply);
	return retn;
}

/* -------------------------------------------*/
/**
 * @brief  设置一个key的删除时间 ，系统到达一定时间
 *			将会自动删除该KEY
 *
 * @param conn				已经建立好的链接
 * @param delete_time		到期事件 time_t 日历时间
 *
 * @returns   
 *		0	SUCC
 *		-1  FAIL
 */
/* -------------------------------------------*/
int rop_set_key_lifecycle(redisContext *conn, char *key, time_t delete_time)
{
	int retn = 0;
	redisReply *reply = NULL;		
	
	reply = redisCommand(conn, "EXPIREAT %s %d", key, delete_time);
	if (reply->type != REDIS_REPLY_INTEGER) {
		fprintf(stderr, "[-][GMS_REDIS]Set key:%s delete time ERROR!\n", key);
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Set key:%s delete time ERROR! %s\n", key, conn->errstr);
		retn = -1;
	}
	if (reply->integer == 1) {
		/* 成功 */
		retn = 0;
	}
	else {
		/* 错误 */
		retn = -1;
	}


	freeReplyObject(reply);	
	return retn;
}

/* -------------------------------------------*/
/**
 * @brief  打印库中所有匹配pattern的key
 *
 * @param conn		已建立好的链接
 * @param pattern	匹配模式，pattern支持glob-style的通配符格式，
 *					如 *表示任意一个或多个字符，
 *					   ?表示任意字符，
 *				    [abc]表示方括号中任意一个字母。
 */
/* -------------------------------------------*/
void rop_show_keys(redisContext *conn, char* pattern)
{
	int i = 0;
	redisReply *reply = NULL;

	reply = redisCommand(conn, "keys %s", pattern);
	if (reply->type != REDIS_REPLY_ARRAY) {
		fprintf(stderr, "[-][GMS_REDIS]show all keys and data wrong type!\n");
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]show all keys and data wrong type! %s\n", conn->errstr);
		goto END;
	}

	for (i = 0; i < reply->elements; ++i) {
		printf("======[%s]======\n", reply->element[i]->str);
	}


END:
	freeReplyObject(reply);
}

/* -------------------------------------------*/
/**
 * @brief  批量执行已经封装好的redis 命令
 *
 * @param conn		已建立好的链接
 * @param cmds		封装好的命令数组
 * @param cmd_num	命令个数
 *
 * @returns   
 *			0		succ
 *			-1		FAIL
 */
/* -------------------------------------------*/
int rop_redis_append(redisContext *conn, RCOMMANDS cmds, int cmd_num)
{
	int retn = 0;
	int i = 0;
	redisReply *reply = NULL;


	/* 批量插入命令到缓冲命令管道 */
	for (i = 0; i < cmd_num; ++i) {
		retn = redisAppendCommand(conn, cmds[i]);
		if (retn != REDIS_OK) {
			fprintf(stderr, "[-][GMS_REDIS]Append Command: %s ERROR!\n", cmds[i]);
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Append Command: %s ERROR! %s\n", cmds[i], conn->errstr);
			retn = -1;
			goto END;
		}
		retn = 0;
	}

	/* 提交命令 */
	for (i = 0; i < cmd_num; ++i) {
		retn = redisGetReply(conn, (void**)&reply);
		if (retn != REDIS_OK) {
			retn = -1;
			fprintf(stderr, "[-][GMS_REDIS]Commit Command:%s ERROR!\n", cmds[i]);
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Commit Command:%s ERROR! %s\n", cmds[i], conn->errstr);
			freeReplyObject(reply);
			break;
		}
		freeReplyObject(reply);
		retn = 0;
	}
	
END:
	return retn;
}

/* -------------------------------------------*/
/**
 * @brief  执行单向命令 无返回值 命令自行输入
 *
 * @param conn		已建立的链接
 * @param cmd		封装好的命令
 *
 * @returns   
 *			0		succ
 *			-1		FAIL
 */
/* -------------------------------------------*/
int rop_redis_command(redisContext *conn, char *cmd)
{
	int retn = 0;

	redisReply *reply = NULL;

	reply = redisCommand(conn, cmd);
	if (reply == NULL) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Command : %s ERROR!%s\n", cmd, conn->errstr);
		retn = -1;
	}

	freeReplyObject(reply);

	return retn;
}

/* -------------------------------------------*/
/**
 * @brief  测试一个reply的结果类型
 *			得到对应的类型用对应的方法获取数据
 *
 * @param reply		返回的命令结果
 */
/* -------------------------------------------*/
void rop_test_reply_type(redisReply *reply)
{
	switch (reply->type) {
		case REDIS_REPLY_STATUS:
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[+][GMS_REDIS]=REDIS_REPLY_STATUS=[string] use reply->str to get data, reply->len get data len\n");
			break;
		case REDIS_REPLY_ERROR:
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[+][GMS_REDIS]=REDIS_REPLY_ERROR=[string] use reply->str to get data, reply->len get date len\n");
			break;
		case REDIS_REPLY_INTEGER:
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[+][GMS_REDIS]=REDIS_REPLY_INTEGER=[long long] use reply->integer to get data\n");
			break;
		case REDIS_REPLY_NIL:
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[+][GMS_REDIS]=REDIS_REPLY_NIL=[] data not exist\n");
			break;
		case REDIS_REPLY_ARRAY:
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[+][GMS_REDIS]=REDIS_REPLY_ARRAY=[array] use reply->elements to get number of data, reply->element[index] to get (struct redisReply*) Object\n");
			break;
		case REDIS_REPLY_STRING:
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[+][GMS_REDIS]=REDIS_REPLY_string=[string] use reply->str to get data, reply->len get data len\n");
			break;
		default:
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Can't parse this type\n");
			break;
	}
}


/* -------------------------------------------*/
/**
 * @brief  redis tcp模式链接
 *
 * @param ip_str	redis服务器ip
 * @param port_str	redis服务器port
 *
 * @returns   
 *			成功返回链接句柄 
 *			失败返回NULL
 */
/* -------------------------------------------*/
redisContext* rop_connectdb_nopwd(char *ip_str, char* port_str)
{
	redisContext *conn = NULL;
	uint16_t port = atoi(port_str);

	conn = redisConnect(ip_str, port);

	if (conn  == NULL) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Connect %s:%d Error:Can't allocate redis context!\n", ip_str, port);		
		goto END;
	}

	if (conn->err) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Connect %s:%d Error:%s\n", ip_str, port, conn->errstr);	
		redisFree(conn);
		conn = NULL;
		goto END;
	}
	
	LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC,"[+][GMS_REDIS]Connect %s:%d SUCCESS!\n", ip_str, port);

END:
	return conn;
}


/* -------------------------------------------*/
/**
 * @brief  redis tcp模式链接
 *
 * @param ip_str	redis服务器ip
 * @param port_str	redis服务器port
 *
 * @returns   
 *			成功返回链接句柄 
 *			失败返回NULL
 */
/* -------------------------------------------*/
redisContext* rop_connectdb(char *ip_str, char* port_str, char *pwd)
{
	redisContext *conn = NULL;
	uint16_t port = atoi(port_str);
    char auth_cmd[REDIS_COMMAND_SIZE];

	conn = redisConnect(ip_str, port);

	if (conn  == NULL) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Connect %s:%d Error:Can't allocate redis context!\n", ip_str, port);		
		goto END;
	}

	if (conn->err) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Connect %s:%d Error:%s\n", ip_str, port, conn->errstr);	
		redisFree(conn);
		conn = NULL;
		goto END;
	}

    redisReply *reply = NULL;
    sprintf(auth_cmd, "auth %s", pwd);

    reply = redisCommand(conn, auth_cmd);
	if (reply == NULL) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Command : auth %s ERROR!\n", pwd);
        conn = NULL;
        goto END;
	}
    freeReplyObject(reply);

	
	LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC,"[+][GMS_REDIS]Connect %s:%d SUCCESS!\n", ip_str, port);

END:
	return conn;
}

/* -------------------------------------------*
**
 * @brief  redis unix域模式链接
 *
 * @param ip_str	unix域sock文件
 *
 * @returns   
 *			成功返回链接句柄 
 *			失败返回NULL
 */
/* -------------------------------------------*/
redisContext* rop_connectdb_unix(char *sock_path, char *pwd)
{
	redisContext *conn = NULL;
    char auth_cmd[REDIS_COMMAND_SIZE];

	conn = redisConnectUnix(sock_path);
	if (conn  == NULL) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Connect domain-unix:%s Error:Can't allocate redis context!\n", sock_path);		
		goto END;
	}

	if (conn->err) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Connect domain-unix:%s Error:%s\n", sock_path, conn->errstr);	
		redisFree(conn);
		conn = NULL;
		goto END;
	}

    redisReply *reply = NULL;
    sprintf(auth_cmd, "auth %s", pwd);
    reply = redisCommand(conn, auth_cmd);
	if (reply == NULL) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Command : auth %s ERROR!\n", pwd);
        conn = NULL;
        goto END;
	}
    freeReplyObject(reply);
	
	LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC,"[+][GMS_REDIS]Connect domain-unix:%s SUCCESS!\n", sock_path);

END:
	return conn;
}

/* -------------------------------------------*/
/**
 * @brief  tcp 链接redis超时等待模式，timeval链接超时
 *			返回
 *
 * @param ip_str		redis 服务器ip
 * @param port_str		redis 服务器端口
 * @param timeval		最大超时等待时间
 *
 * @returns   
 *		成功返回链接句柄
 *		失败返回NULL
 */
/* -------------------------------------------*/
redisContext* rop_connectdb_timeout(char* ip_str, char *port_str, struct timeval *timeout)
{
	redisContext *conn = NULL;
	uint16_t port = atoi(port_str);


	conn = redisConnectWithTimeout(ip_str, port, *timeout);

	if (conn  == NULL) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Connect %s:%d Error:Can't allocate redis context!\n", ip_str, port);
		goto END;
	}

	if (conn->err) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Connect %s:%d Error:%s\n", ip_str, port, conn->errstr);	
		redisFree(conn);
		conn = NULL;
		goto END;
	}
	
	LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC,"[+][GMS_REDIS]Connect %s:%d SUCCESS!\n", ip_str, port);

END:
	return conn;
}

/* -------------------------------------------*/
/**
 * @brief  关闭指定的链接句柄
 *
 * @param conn	已建立好的链接
 */
/* -------------------------------------------*/
void rop_disconnect(redisContext* conn)
{
	if (conn == NULL) {
		return ;
	}
	redisFree(conn);
	
	LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC,"[+][GMS_REDIS]Disconnect SUCCESS!\n");
}

/* 封装一个 hmset 命令 */
static char* make_hmset_command(char* key, unsigned int element_num, RFIELDS fields, RVALUES values)
{
	char *cmd = NULL;	
	unsigned int buf_size = 0;
	unsigned int use_size = 0;
	unsigned int i = 0;

	cmd = (char*)malloc(1024*1024);
	if (cmd == NULL) {
		goto END;
	}
	memset(cmd, 0, 1024*1024);
	buf_size += 1024*1024;

	strncat(cmd, "hmset", 6);
	use_size += 5;
	strncat(cmd, " ", 1);
	use_size += 1;

	strncat(cmd, key, 200);
	use_size += 200;

	for (i = 0; i < element_num; ++i) {

		strncat(cmd, " ", 1);
		use_size += 1;
		if (use_size >= buf_size) {
			cmd = realloc(cmd, use_size + 1024*1024);
			if (cmd == NULL) {
				goto END;
			}
			buf_size += 1024*1024;
		}

		strncat(cmd, fields[i], FIELD_ID_SIZE);
		use_size += strlen(fields[i]);
		if (use_size >= buf_size) {
			cmd = realloc(cmd, use_size + 1024*1024);
			if (cmd == NULL) {
				goto END;
			}
			buf_size += 1024*1024;
		}


		strncat(cmd, " ", 1);
		use_size += 1;
		if (use_size >= buf_size) {
			cmd = realloc(cmd, use_size + 1024*1024);
			if (cmd == NULL) {
				goto END;
			}
			buf_size += 1024*1024;
		}

		strncat(cmd, values[i], VALUES_ID_SIZE);
		use_size += strlen(values[i]);
		if (use_size >= buf_size) {
			cmd = realloc(cmd, use_size + 1024*1024);
			if (cmd == NULL) {
				goto END;
			}
			buf_size += 1024*1024;
		}

	}

END:
	return cmd;
}


/* -------------------------------------------*/
/**
 * @brief  批量执行链表插入命令 插入链表头部
 *
 * @param conn		已建立好的链接
 * @param key		链表名
 * @param values	封装好的值数组
 * @param val_num	值个数
 *
 * @returns   
 *			0		succ
 *			-1		FAIL
 */
/* -------------------------------------------*/
int rop_hash_set_append(redisContext *conn, char *key, RFIELDS fields, RVALUES values, int val_num)
{
    int retn = 0;
    int i = 0;
    redisReply *reply = NULL;

	/* 批量插入命令到缓冲命令管道 */
    for (i = 0; i < val_num; ++i) {
        retn = redisAppendCommand(conn, "hset %s %s %s", key, fields[i], values[i]);
        if (retn != REDIS_OK) {
            LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]HSET %s %s %s ERROR![%s]\n", key, fields[i], values[i], conn->errstr);
            retn = -1;
            goto END;
        }
        retn = 0;
    }

	/* 提交命令 */
	for (i = 0; i < val_num; ++i) {
		retn = redisGetReply(conn, (void**)&reply);
		if (retn != REDIS_OK) {
			retn = -1;
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Commit HSET %s %s %s ERROR![%s]\n", key, fields[i], values[i], conn->errstr);
			freeReplyObject(reply);
			break;
		}
		freeReplyObject(reply);
		retn = 0;
	}
	
END:
	return retn;
}

/* -------------------------------------------*/
/**
 * @brief  想一个hash表中添加一条 key-value 数据
 *
 * @param conn  redis连接
 * @param key   哈希表名
 * @param field
 * @param value
 *
 * @returns
 *            0        succ
 *            -1        FAIL
 */
/* -------------------------------------------*/
int rop_hash_set(redisContext *conn, char *key, char *field, char *value)
{
    int retn = 0;
    redisReply *reply = NULL;

    reply =  redisCommand(conn, "hset %s %s %s", key, field, value);
    if (reply == NULL || reply->type != REDIS_REPLY_INTEGER) {
        LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]hset %s %s %s error %s\n", key, field, value,conn->errstr);
        retn =  -1;
        goto END;
    }


END:
    freeReplyObject(reply);

    return retn;
}

/* -------------------------------------------*/
/**
 * @brief  从一个hash表中取出一条 key-value 数据
 *
 * @param conn  redis连接
 * @param key   哈希表名
 * @param field 字段名称
 * @param value 得到的数据， 需要先开辟内存
 *
 * @returns
 *            0        succ
 *            -1        FAIL
 */
/* -------------------------------------------*/
int rop_hash_get(redisContext *conn, char *key, char *field, char *value)
{
    int retn = 0;
    int len = 0;

    redisReply *reply = NULL;

    reply =  redisCommand(conn, "hget %s %s", key, field);
    if (reply == NULL || reply->type != REDIS_REPLY_STRING) {
        LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]hget %s %s  error %s\n", key, field, conn->errstr);
        retn =  -1;
        goto END;
    }


    len = reply->len > VALUES_ID_SIZE? VALUES_ID_SIZE:reply->len ;

    strncpy(value, reply->str, len);

    value[len] = '\0';


END:
    freeReplyObject(reply);


    return retn;
}


/* -------------------------------------------*/
/**
 * @brief			创建或者覆盖一个HASH表
 *
 * @param conn				已建立好的链接
 * @param key				hash 表名
 * @param element_num		hash 表区域个数
 * @param fields			hash 表区域名称数组char(*)[FIELD_ID_SIZE]
 * @param values			hash 表区域值数组  char(*)[VALUES_ID_SIZE]
 *
 * @returns   
 *			0   成功	
 *			-1  失败
 */
/* -------------------------------------------*/
int rop_create_or_replace_hash_table(redisContext* conn,
									 char* key, 
									 unsigned int element_num, 
									 RFIELDS fields, 
									 RVALUES values)
{
	int retn = 0;
	redisReply *reply = NULL;			

	char *cmd = make_hmset_command(key, element_num, fields, values);		
	if (cmd == NULL) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]create hash table %s error\n", key);
		retn = -1;
		goto END_WITHOUT_FREE;
	}

	reply = redisCommand(conn, cmd);
//	rop_test_reply_type(reply);
	if (strcmp(reply->str, "OK") != 0) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Create hash table %s Error:%s,%s\n", key, reply->str, conn->errstr);
		
		retn = -1;
		goto END;
	}


END:
	free(cmd);
	freeReplyObject(reply);

END_WITHOUT_FREE:

	return retn;
}

/* -------------------------------------------*/
/**
 * @brief  给指定的hash表 指定的field对应的value自增num
 *
 * @param conn			已建立好的链接
 * @param key			hash表名
 * @param field			hash表下的区域名	
 *
 * @returns   
 *			0		succ
 *			-1		fail
 */
/* -------------------------------------------*/
int rop_hincrement_one_field(redisContext *conn, char *key, char *field, unsigned int num)
{
	int retn = 0;

	redisReply *reply = NULL;

	reply = redisCommand(conn, "HINCRBY %s %s %d", key, field, num);
	if (reply == NULL) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]increment %s %s error %s\n", key, field, conn->errstr);	
		retn =  -1;
		goto END;
	}

END:
	freeReplyObject(reply);

	return retn;
}


/* -------------------------------------------*/
/**
 * @brief  批量执行链表插入命令 插入链表头部
 *
 * @param conn		已建立好的链接
 * @param key		链表名
 * @param values	封装好的值数组
 * @param val_num	值个数
 *
 * @returns   
 *			0		succ
 *			-1		FAIL
 */
/* -------------------------------------------*/
int rop_list_push_append(redisContext *conn, char *key, RVALUES values, int val_num)
{
	int retn = 0;
	int i = 0;
	redisReply *reply = NULL;


	/* 批量插入命令到缓冲命令管道 */
	for (i = 0; i < val_num; ++i) {
		retn = redisAppendCommand(conn, "lpush %s %s", key, values[i]);
		if (retn != REDIS_OK) {
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]PLUSH %s %s ERROR! %s\n", key, values[i], conn->errstr);
			retn = -1;
			goto END;
		}
		retn = 0;
	}

	/* 提交命令 */
	for (i = 0; i < val_num; ++i) {
		retn = redisGetReply(conn, (void**)&reply);
		if (retn != REDIS_OK) {
			retn = -1;
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Commit LPUSH %s %s ERROR! %s\n", key, values[i], conn->errstr);
			freeReplyObject(reply);
			break;
		}
		freeReplyObject(reply);
		retn = 0;
	}
	
END:
	return retn;
}

/* -------------------------------------------*/
/**
 * @brief  单条数据插入链表
 *
 * @param conn		已建立好的链接
 * @param key		链表名
 * @param value		数据
 *
 * @returns   
 */
/* -------------------------------------------*/
int rop_list_push(redisContext *conn, char *key, char *value)
{
	int retn = 0;
	redisReply *reply = NULL;

	reply = redisCommand(conn, "LPUSH %s %s", key, value);
	//rop_test_reply_type(reply);	
	if (reply->type != REDIS_REPLY_INTEGER) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]LPUSH %s %s error!%s\n", key, value, conn->errstr);
		retn = -1;
	}

	freeReplyObject(reply);
	return retn;
}

/* -------------------------------------------*/
/**
 * @brief  得到链表中元素的个数
 *
 * @param conn	链接句柄
 * @param key	链表名
 *
 * @returns   
 *			>=0 个数
 *			-1 fail
 */
/* -------------------------------------------*/
int rop_get_list_cnt(redisContext *conn, char *key)
{
	int cnt = 0;

	redisReply *reply = NULL;

	reply = redisCommand(conn, "LLEN %s", key);
	if (reply->type != REDIS_REPLY_INTEGER) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]LLEN %s error %s\n", key, conn->errstr);
		cnt = -1;
		goto END;
	}

	cnt = reply->integer;

END:
	freeReplyObject(reply);
	return cnt;
}


/* -------------------------------------------*/
/**
 * @brief  按照一定范围截断链表中的数据
 *
 * @param conn		已经建立的链接
 * @param key		链表名
 * @param begin		阶段启示位置 从 0 开始
 * @param end		阶段结束位置 从 -1 开始
 *
 *					这里的范围定义举例 
 *					如果得到全部范围(0, -1)
 *					除了最后一个元素范围(0, -2)
 *					前20各数据范围(0, 19)
 *
 * @returns   
 *			0  SUCC
 *			-1 FAIL
 */
/* -------------------------------------------*/
int rop_trim_list(redisContext *conn, char *key, int begin, int end)
{
	int retn = 0;
	redisReply *reply = NULL;

	reply = redisCommand(conn, "LTRIM %s %d %d", key, begin, end);
	if (reply->type != REDIS_REPLY_STATUS) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]LTRIM %s %d %d error!%s\n", key, begin, end, conn->errstr);
		retn = -1;
	}

	freeReplyObject(reply);
	return retn;
}


/* -------------------------------------------*/
/**
 * @brief  rop_range_list 得到链表中的数据
 *
 *          返回数据为 区间为
 *              [from_pos, end_pos)
 *
 * @param conn
 * @param key       表名
 * @param from_pos  查找表的起始数据下标
 * @param end_pos   查找表的结尾数据下标
 * @param values    得到表中的value数据
 * @param get_num   得到结果value的个数
 *
 * @returns   
 *      0 succ, -1 fail
 */
/* -------------------------------------------*/
int rop_range_list(redisContext *conn, char *key, int from_pos, int end_pos, RVALUES values, int *get_num)
{
    int retn = 0;
    int i = 0;
    redisReply *reply = NULL;
    int max_count = 0;

    int count = end_pos - from_pos + 1;

    reply = redisCommand(conn, "LRANGE %s %d %d", key, from_pos, end_pos);
    if (reply->type != REDIS_REPLY_ARRAY || reply->elements == 0) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]LRANGE %s  error!%s\n", key, conn->errstr);
		retn = -1;
        goto END;
	}


    max_count = (reply->elements > count) ? count: reply->elements;
    *get_num = max_count;


    for (i = 0; i < max_count; ++i) {
        strncpy(values[i], reply->element[i]->str, VALUES_ID_SIZE-1);
    }

END:
    if(reply != NULL)
    {
        freeReplyObject(reply);
    }

	return retn;
}

/* -------------------------------------------*/
/**
 * @brief		将指定的zset表，对应的成员，值自增1
 *				（key 或 成员不存在 则创建）
 *
 * @param conn		已建立的链接
 * @param key		zset表名
 * @param member	zset成员名
 *
 * @returns   
 *			0			succ
 *			-1			fail
 */
/* -------------------------------------------*/
int rop_zset_increment(redisContext *conn, char* key, char* member)
{
	int retn = 0;	

	redisReply *reply = NULL;

	reply = redisCommand(conn, "ZINCRBY %s 1 %s", key, member);
	//rop_test_reply_type(reply);
	if (strcmp(reply->str, "OK") != 0) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Add or increment table: %s,member: %s Error:%s,%s\n", key, member,reply->str, conn->errstr);
		
		retn = -1;
		goto END;
	}

END:
	freeReplyObject(reply);
	return retn;
}

/* -------------------------------------------*/
/**
 * @brief	 批量将指定的zset表，对应的成员，值自增1
 *				（key 或 成员不存在 则创建）
 *
 * @param conn		已建立好的链接
 * @param key		有序集合名称
 * @param values	封装好的成员数组
 * @param val_num	数据个数
 *
 * @returns   
 *			0		succ
 *			-1		FAIL
 */
/* -------------------------------------------*/
int rop_zset_increment_append(redisContext *conn, char *key, RVALUES values, int val_num)
{
	int retn = 0;
	int i = 0;
	redisReply *reply = NULL;

	/* 批量命令到缓冲管道 */
	for (i = 0; i < val_num; ++i) {
		retn = redisAppendCommand(conn, "ZINCRBY %s 1 %s", key, values[i]);
		if (retn != REDIS_OK) {
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]ZINCRBY %s 1 %s ERROR! %s\n", key, values[i], conn->errstr);
			retn = -1;
			goto END;
		}
		retn = 0;
	}

	/* 提交命令 */
	for (i = 0; i < val_num; ++i) {
		retn = redisGetReply(conn, (void**)&reply);
		if (retn != REDIS_OK) {
			retn = -1;
			LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]Commit ZINCRBY %s 1 %s ERROR!%s\n", key, values[i], conn->errstr);
			freeReplyObject(reply);
			break;
		}
		freeReplyObject(reply);
		retn = 0;
	}

END: 
	return retn;
}

int rop_zset_get_score(redisContext *conn, char *key, char *member)
{
	int score = 0;

	redisReply *reply = NULL;

	reply = redisCommand(conn, "ZSCORE %s %s", key, member);
    rop_test_reply_type(reply);
    
	if (reply->type != REDIS_REPLY_STRING) {
		LOG(REDIS_LOG_MODULE, REDIS_LOG_PROC, "[-][GMS_REDIS]ZSCORE %s %s error %s\n", key, member,conn->errstr);
        score = -1;
		goto END;
	}

    score = atoi(reply->str);


END:
	freeReplyObject(reply);

	return score;
}
```

```c
#ifndef  _MAKE_LOG_H_
#define  _MAKE_LOG_H
#include "pthread.h"

int out_put_file(char *path, char *buf);
int make_path(char *path, char *module_name, char *proc_name);
int dumpmsg_to_file(char *module_name, char *proc_name, const char *filename,
                        int line, const char *funcname, char *fmt, ...);
#ifndef _LOG
#define LOG(module_name, proc_name, x...) \
        do{ \
		dumpmsg_to_file(module_name, proc_name, __FILE__, __LINE__, __FUNCTION__, ##x);\
	}while(0)
#else
#define LOG(module_name, proc_name, x...)
#endif

extern pthread_mutex_t ca_log_lock;

#endif



```

```c
#include<stdio.h>
#include<stdarg.h>
#include<string.h>
#include<fcntl.h>
#include<unistd.h>
#include<time.h>
#include<sys/stat.h>

#include"make_log.h"
#include<pthread.h>

#if 1

//pthread_mutex_t lock;
//pthread_mutex_init(lock);
/*void *comm_log(void *p)
{
    pthread_mutex_lock(&lock);
    pthread_mutex_unlock(&lock);
}
struct file_path{
    char *module_name;
    char *proc_name;
    const char *filename;
    int line;
    const char *funcname;
    char *fmt;

};
int lock_file(char *module, char *proc, const char *file,
                        int lines, const char *func, char *mt, ...)
{
    va_list ap;
    struct file_path path;
    path.module_name = module;
    path.proc_name = proc;
    path.filename = file;
    path.line = lines;
    path.funcname = func;
    //path.fmt  = mt;
    va_start(ap,mt);
    vsprintf(path.fmt,mt,ap);
    va_end(ap);
    pthread_mutex_init(&lock,0); 
    pthread_t comm;
        pthread_create(&comm,0,dumpmsg_to_file,&path);
    pthread_join(comm,0);
    pthread_mutex_destroy(&lock);
}*/

pthread_mutex_t ca_log_lock=PTHREAD_MUTEX_INITIALIZER;

//创建目录并写入内容
int dumpmsg_to_file(char *module_name, char *proc_name, const char *filename,
                        int line, const char *funcname, char *fmt, ...)
{
        char mesg[4096]={0};
        char buf[4096]={0};
	    char filepath[1024] = {0};
        time_t t=0;
        struct tm * now=NULL;                                                                                     
        va_list ap;                                                                                               
        //struct file_path *path;
        //path = (struct file_path*)paths;
        time(&t);                                                                                                 
        now = localtime(&t);                                       
        va_start(ap, fmt);                                                                               
        vsprintf(mesg, fmt, ap);                                                                       
        va_end(ap);                        
#if 1
        snprintf(buf, 4096, "[%04d-%02d-%02d %02d:%02d:%02d]--[%s:%d]--%s\n",
                                now -> tm_year + 1900, now -> tm_mon + 1,                                         
                                now -> tm_mday, now -> tm_hour, now -> tm_min, now -> tm_sec,                     
								filename, line, mesg);                                     
#endif

#if 0
        snprintf(buf, 4096, "===%04d%02d%02d-%02d%02d%02d,%s[%d]=== %s\n",
                                now -> tm_year + 1900, now -> tm_mon + 1,                                         
                                now -> tm_mday, now -> tm_hour, now -> tm_min, now -> tm_sec,
                                funcname, line, mesg);   
#endif								
        make_path(filepath, module_name, proc_name);
        
        pthread_mutex_lock(&ca_log_lock);
	    out_put_file(filepath, buf);     
        pthread_mutex_unlock(&ca_log_lock);

        return 0;     
}
#endif
//写入内容
int out_put_file(char *path, char *buf)
{
	int fd;                                                                                                   
    fd = open(path, O_RDWR | O_CREAT | O_APPEND, 0777);

    if(write(fd, buf, strlen(buf)) != (int)strlen(buf)) {                                      
            fprintf(stderr, "write error!\n");                           
            close(fd);                                                                                        
    } else {                                                                                                  
            //write(fd, "\n", 1);
            close(fd);                                                                                        
    }               

	return 0;
}
//创建目录
int make_path(char *path, char *module_name, char *proc_name)
{
	time_t t;
	struct tm *now = NULL;
	char top_dir[1024] = {"."};
	char second_dir[1024] = {"./logs"};
	char third_dir[1024] = {0};
	char y_dir[1024] = {0};
	char m_dir[1024] = {0};
	char d_dir[1024] = {0}; 
	time(&t);
        now = localtime(&t);
	snprintf(path, 1024, "./logs/%s/%04d/%02d/%s-%02d.log", module_name, now -> tm_year + 1900, now -> tm_mon + 1, proc_name, now -> tm_mday);
	
	sprintf(third_dir, "%s/%s", second_dir, module_name);
	sprintf(y_dir, "%s/%04d/", third_dir, now -> tm_year + 1900);
	sprintf(m_dir, "%s/%02d/", y_dir, now -> tm_mon + 1);
	sprintf(d_dir,"%s/%02d/", m_dir, now -> tm_mday);
	
	if(access(top_dir, 0) == -1) {
		if(mkdir(top_dir, 0777) == -1) {
			fprintf(stderr, "create %s failed!\n", top_dir);	
		} else if(mkdir(second_dir, 0777) == -1) {
			fprintf(stderr, "%s:create %s failed!\n", top_dir, second_dir);
		} else if(mkdir(third_dir, 0777) == -1) {
			fprintf(stderr, "%s:create %s failed!\n", top_dir, third_dir);
		} else if(mkdir(y_dir, 0777) == -1) {
                        fprintf(stderr, "%s:create %s failed!\n", top_dir, y_dir);                                                     
                } else if(mkdir(m_dir, 0777) == -1) {                                                             
                        fprintf(stderr, "%s:create %s failed!\n", top_dir, m_dir);                                                     
                }          	
	} else if(access(second_dir, 0) == -1) {
		if(mkdir(second_dir, 0777) == -1) {
			fprintf(stderr, "create %s failed!\n", second_dir);
		} else if(mkdir(third_dir, 0777) == -1) {
			fprintf(stderr, "%s:create %s failed!\n", second_dir, third_dir);
                } else if(mkdir(y_dir, 0777) == -1) {
                        fprintf(stderr, "%s:create %s failed!\n", second_dir, y_dir);
                } else if(mkdir(m_dir, 0777) == -1) {
                        fprintf(stderr, "%s:create %s failed!\n", second_dir, m_dir);
                }
	} else if(access(third_dir, 0) == -1) {
		if(mkdir(third_dir, 0777) == -1) {
			fprintf(stderr, "create %s failed!\n", third_dir);
		} else if(mkdir(y_dir, 0777) == -1) {
			fprintf(stderr, "%s:create %s failed!\n", third_dir, y_dir);
		} else if(mkdir(m_dir, 0777) == -1) {
			fprintf(stderr, "%s:create %s failed!\n", third_dir, m_dir);
		} 
	} else if (access(y_dir, 0) == -1) {
		if(mkdir(y_dir, 0777) == -1) {
			fprintf(stderr, "create %s failed!\n", y_dir);
		} else if(mkdir(m_dir, 0777) == -1) {
                        fprintf(stderr, "%s:create %s failed!\n", y_dir, m_dir);
                } 

	} else if (access(m_dir, 0) == -1) {
                if(mkdir(m_dir, 0777)) {
			fprintf(stderr, "create %s failed!\n", m_dir);
		} 
        }
	//printf("path:%s\n", path);
	return 0;
}

#if 0
int main(void)
{
	char path[1024] = {0};
	char proc_name[] = {"sys_guard"};
	char buf[] = {"12345\n"};
	make_path(path, proc_name);
	out_put_file(path, buf);
	return 0;
} 
#endif

```

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#include "redis_op.h"

int main(int argc, char *argv[])
{
    int ret = 0;
    //连接数据库
    redisContext *conn = NULL;

    conn = rop_connectdb_nopwd("127.0.0.1", "6379");

    //set foo  hello
    ret = rop_set_string(conn, "foo", "hello");
    if (ret == 0) 
	{
        printf("set succ!\n");
    }
    else 
	{
        printf("set fail\n");
    }
	
	char value[256] = {0};
	ret = rop_get_string(conn, "foo", value);
    if (ret == 0) 
	{
        printf("get succ: %s - %s\n", "foo", value);
    }
    else 
	{
        printf("get fail\n");
    }


    //释放数据库
    rop_disconnect(conn);


	return 0;
}

```

