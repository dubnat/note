# Redis
## 为什么使用Nosql
> 1、单机MySql

一个基本的网站访问量一般不会太大，单个数据库完全足够。
那个时候，更多使用静态页面Html，服务器根本没有太大压力。
思考，这种情况下，整个网站的瓶颈是什么？
1、数据量如果太大，一个机器放不下了
2、数据的索引(B+Tree)，300万数据以上就一定要建索引 
3、访问量（读写混合）、一个服务器承受不了

> 2、Memcached(缓存) + MySql + 垂直拆分 + 读写分离

网站80%的情况下都是在读，每次都需要去查询数据库的话就十分麻烦！所以说希望减轻数据库的压力，我们可以用缓存来保证效率。
发展过程：优化数据结构和索引 --> 使用文件缓存(IO)--> Memcached(当时最热门的技术！)

> 3、分库分表 + 水平拆分 + MySql集群

技术和业务在发展的同时，对人的要求也越来越高！
==本质：数据库（读、写）==
早些年MyISAM：表锁，十分影响效率，高并发下就会出现严重的锁问题。
转战Innodb：行锁
慢慢的就开始使用分库分表来解决写的压力！MySql在那个年代推出了表分区，这个并没有多少公司使用。
不同业务使用不同的数据库就行管理。

> 4、如今

MySql等关系型数据库就不够用了，数据量很多，变化很快
MySql有的使用它来存储一些比较大的文件、博客、图片！数据库表很大，效率就低了！如果有一种数据表专门来处理这种数据，MySql的压力就会变得十分小（研究如何处理这些问题），大数据的IO压力下，表几乎没法更大！

## 什么是NoSQL
> NoSQL

Not Only SQL!
关系型数据库：表格，行，列

很多的数据类型如用户的个人信息，社交网络，地理位置。这些数据类型的存储不需要一个固定的格式！不需要多余的操作就可以横向扩展的！Map<String,Object>

> NoSQL特点

解耦
1. 方便扩展（数据之间没有关系，很好扩展）
2. 大数据量高性能（Redies一秒可以写八万次，读取十一万，NoSQL的缓存记录级是一种细粒度的缓存，性能比较高）
3. 数据类型是多样性的（不需要事先设计数据库，随取随用）
4. 传统的RDBMS和NoSQL
```
传统的RDBMS
- 结构化组织
- SQL
- 数据和关系都存在单独的表中
- 严格的一致性
- 基础的事物
- ...

NoSql
- 不仅仅是数据库
- 没有固定的查询语言
- 键值对存储，列存储，文档存储，图形数据库（社交关系）
- 最终一致性
- CAP定理和BASE（异地多活）
- 高性能，高可用，高可扩展
- ...
```

> 了解：3V + 3高

大数据时代的3V：主要是描述问题的
 1. Volume 海量
 2. Variety 多样
 3. Velocity 实时
 
大数据时代的3V：主要是对程序的要求
1. 高并发
2. 高可拓展（随时可以水平拆分）
3. 高性能

真正在公司的实践：NoSQL + RDBMS
==技术没有高低之分，就看如何使用==

## NoSQL的四大分类
**KV键值对：**
+ 新浪：Redis
+ 美团：Redis + Tair
+ 阿里、百度：Redis + memecache

**文档型数据库（bson格式和json一样）**
+ MongoDB(一般必须要掌握)
    - MongoDB是一个基于分布式文件存储的数据库，C++编写，主要用来处理大量的文档
    - MongoDB是一个介于关系型数据库和非关系型数据库中间的产品！MongoDB是非关系型数据库中功能最丰富最像关系型数据库的！
+ ConthDB

**列存储数据库**
+ HBase
+ 分布式文件系统

**图关系数据库**
+ 不是存图形，放的是关系，比如：朋友圈社交网络

## Redis入门
### 概述
> Redis是什么？

Redis（Remote Dictionary Server )，即远程字典服务!
是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。
是当下最热门的NoSQL技术之一！也被人们称为结构化数据库。

> Redis能干嘛？

1、 内存存储、持久化，内存中是断电即失、所以说持久化很重要（rdb,aof）
2、 效率高、可以用于高速缓存
3、 发布订阅系统
4、 地图信息分析
5、 计时器、计数器（浏览量）
6、 ...

> 特性

1、多样的数据类型
2、持久化
3、集群
4、事务
5、...

> windows使用Redis

端口号6379

127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set name chuyi
OK
127.0.0.1:6379> get name
"chuyi"

### 基础知识
有16个数据库，默认使用的是第0个，可以使用select进行切换
127.0.0.1:6379> select 3                 # 切换数据库
OK
127.0.0.1:6379[3]> dbsize                # 查看数据库大小
(integer) 0
127.0.0.1:6379[3]> keys *                # 查看所有的key
1） "name"
127.0.0.1:6379[3]> flushdb               # 清空的当前数据库
OK
127.0.0.1:6379> flushall                 # 清空全部数据库
OK

> Redis是单线程的

明白Redis是很快的，官方表示，Redis是基于内存操作，CPU不是Redis性能瓶颈，Redis的瓶颈是根据机器的内存和网络带宽，既然可以使用单线程来实现，所以就使用了单线程。
Redis是c语言写的，官方提供的数据是100000+的QPS，完全不比同样是使用key-value的Menecache差！

**Redis为什么单线程还这么快？**
1、误区1：高性能的服务器一定是多线程的
2、误区2：多线程（CPU上下文切换）一定比单线程效率高
CPU > 内存 > 硬盘的速度
核心：Redis是将所有的数据全部放在内存中的，所以说使用单线程去操作效率就是最高的，多线程（CPU上下文会切换：耗时的操作），对于内存系统来说，如果没有上下文切换效率就是最高的！多次读写都是在一个cpu上的，在内存情况下这个就是最佳方案。

## 五大数据类型
> 官方文档

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作==数据库==、==缓存==和==消息中间件MQ==。 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。

### Redis-Key
127.0.0.1:6379> exists name             # 判断当前key是否存在
(integer) 1                             
127.0.0.1:6379> move name 1             # 移除当前的key到数据库1
(integer) 1
127.0.0.1:6379> expire name 10          # 设置当前key的过期时间(s)
(integer) 1
127.0.0.1:6379> ttl name                # 查看当前key的剩余时间
(integer) -2
127.0.0.1:6379> type name               # 查看当前key的一个类型
string

### String（字符串）
90%的java程序员用redis只会使用String类型！
``` bash
127.0.0.1:6379> append key1 "hello"   # 在当前key后追加字符串，如果当前key不存在，则相当于set key
(integer) 7                           # 返回当前key的长度
127.0.0.1:6379> strlen key1           # 返回当前key的长度
(integer) 7
#####################################################################
127.0.0.1:6379> set views 0
OK
127.0.0.1:6379> incr views            # 自增1 浏览量增1
(integer) 1
127.0.0.1:6379> decr views            # 自减1 浏览量减1
(integer) 0
127.0.0.1:6379> incrby views 10       # 可以设置步长，指定增量
(integer) 11
127.0.0.1:6379> decrby views 5        # 可以设置步长，指定减量
(integer) 6
#####################################################################
127.0.0.1:6379> set key1 "hello,chuyi"
OK
127.0.0.1:6379> get key1
"hello,chuyi"
127.0.0.1:6379> getrange key1 0 3   # 截取字符串[0,3]
"hell"
127.0.0.1:6379> getrange key1 0 -1  # 获取全部的字符串，和get key是一样的
"hello,chuyi"

# 替换！
127.0.0.1:6379> set key2 abcdefg
OK
127.0.0.1:6379> setrange key2 1 xx  # 替换指定位置开始的字符串
(integer) 7
127.0.0.1:6379> get key2
"axxdefg"
#####################################################################
# setex(set with expire)        # 设置过期时间
# setnx(set if not exist)       # 不存在则设置（在分布式锁中会常常使用）


127.0.0.1:6379> setex key3 30 hello     # 设置key3的值为hello,并且30秒后过期
OK
127.0.0.1:6379> ttl key3    
(integer) 24
127.0.0.1:6379> get key3
"hello"
127.0.0.1:6379> setnx mykey "redis"     # 如果mykey 不存在，创建mykey
(integer) 1
127.0.0.1:6379> ttl key3
(integer) -2
127.0.0.1:6379> setnx mykey "mongodb"   # mykey存在，创建失败
(integer) 0
127.0.0.1:6379> get mykey
"redis"
#####################################################################
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3  # 同时设置多个值
OK
127.0.0.1:6379> keys *
1) "k3"
2) "k2"
3) "k1"
127.0.0.1:6379> mget k1 k2 k3           # 同时获取多个值
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> msetnx k1 v1 k4 v4      #msetnx 是一个原子性的操作，要么一起成功，要么一起失败
(integer) 0
127.0.0.1:6379> get k4
(nil)

# 对象
set uesr:1{name:chuyi,age:3} # 设置一个user:1 对象，值为json字符串来保存一个对象

# 这里的key是一个巧妙的设计： user:{id}:{filed}， 如此设计在redis中是完全ok了！

127.0.0.1:6379> mset user:1:name chuyi user:1:age 2
OK
127.0.0.1:6379> mget user:1:name user:1:age
1) "chuyi"
2) "2"

#####################################################################
getset      # 先get 再set

127.0.0.1:6379> getset db redis     
(nil)
127.0.0.1:6379> get db
"redis"
127.0.0.1:6379> getset db mongodb       # 如果存在值则先获取原来的值，并设置新的值
"redis"
127.0.0.1:6379> get db
"mongodb"
```
String类似的使用场景：value除了是字符串还可以是数字！
+ 计数器
+ 统计多单位的数量
+ 粉丝数
+ 对象缓存存储

### List
基本的数据类型，列表
在redis里面，我们可以把list玩成栈、队列、阻塞队列！
所有的list命令都是l开头的

``` bash
127.0.0.1:6379> lpush list one          # 将一个值或者多个值，插入列表头部（左边）
(integer) 1
127.0.0.1:6379> lpush list two  
(integer) 2
127.0.0.1:6379> lpush list three
(integer) 3
127.0.0.1:6379> lrange list 0 -1        # 获取list中的值
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> lrange list 0 1         # 通过区间获取具体的值
1) "three"
2) "two"
127.0.0.1:6379> rpush list right        # 将一个值或者多个值，插入列表尾部（右边） 
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"
3) "one"
4) "right"
#####################################################################
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"
3) "one"
4) "right"
127.0.0.1:6379> lpop list           # 移除列表的第一个元素
"three"
127.0.0.1:6379> rpop list           # 移除最后一个元素
"right"
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "one"
#####################################################################
127.0.0.1:6379> lindex list 1       # 通过下标获得list中的某一个值
"one"
127.0.0.1:6379> lindex list 0
"two"
127.0.0.1:6379> llen list           # 返回列表的长度
(integer) 2

#####################################################################
移除指定的值
取关 uid

lrem
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "three"
3) "two"
4) "one"
5) "zero"
127.0.0.1:6379> lrem list 1 one         # 移除list集合中指定个数的value，精确匹配
(integer) 1
127.0.0.1:6379> lrange list 0 -1        
1) "three"
2) "three"
3) "two"
4) "zero"
127.0.0.1:6379> lrem list 2 three
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "zero"

#####################################################################
trim

127.0.0.1:6379> lrange list1 0 -1
1) "hello"
2) "hello1"
3) "hello2"
4) "hello3"
5) "hello4"
127.0.0.1:6379> ltrim list1 1 2
OK
127.0.0.1:6379> lrange list1 0 -1       #通过下标截取指定的长度，这个list已经被改变了，只剩下截取的元素
1) "hello1"
2) "hello2"

#####################################################################
rpoplpush # 移除列表最后一个元素，将它移动到新的列表中

127.0.0.1:6379> rpush mylist "hello1"
(integer) 1
127.0.0.1:6379> rpush mylist "hello2"
(integer) 2
127.0.0.1:6379> rpush mylist "hello3"
(integer) 3
127.0.0.1:6379> rpoplpush mylist myotherlist
"hello3"
127.0.0.1:6379> lrange mylist 0 -1
1) "hello1"
2) "hello2"
127.0.0.1:6379> lrange myotherlist 0 -1
1) "hello3"

#####################################################################
lset 将列表中指定下标的值替换为另外一个值，更新操作

127.0.0.1:6379> exists list         # 判断这个列表是否存在
(integer) 0
127.0.0.1:6379> lset list 0 item1   # 如果不存在我们去更新就会报错
(error) ERR no such key
127.0.0.1:6379> lpush list v1       
(integer) 1
127.0.0.1:6379> lrange list 0 -1
1) "v1"
127.0.0.1:6379> lset list 0 i1      # 如果存在，则更新当前下标的值
OK
127.0.0.1:6379> lrange list 0 -1
1) "i1"
127.0.0.1:6379> lset list 1 other   # 如果不存在，则会报错
(error) ERR index out of range

#####################################################################
linsert     # 将某个具体的值插入到列表中某个元素的前面或后面

127.0.0.1:6379> rpush list1 hello
(integer) 1
127.0.0.1:6379> rpush list1 world
(integer) 2
127.0.0.1:6379> linsert list1 before "world" "other"
(integer) 3
127.0.0.1:6379> lrange list1 0 -1
1) "hello"
2) "other"
3) "world"
127.0.0.1:6379> linsert list1 after "world" "new"
(integer) 4
127.0.0.1:6379> lrange list1 0 -1
1) "hello"
2) "other"
3) "world"
4) "new"
```

> 小结

+ 实际上是一个链表，before Node after, left, right 都可以插入值
+ 如果key不存在，创建新的链表
+ 如果key存在，新增内容
+ 如果移除了所有的值，空链表，也代表不存在
+ 在两边插入或者改动值，效率最高！中间元素，相对来说效率会低一点

消息排队！消息队列！队列(Lpush Rpop), 栈（Lpush Lpop） 

### Set(集合)
set中的值是不能重复的
``` bash
127.0.0.1:6379> sadd myset helo     # set集合中添加元素
(integer) 1
127.0.0.1:6379> sadd myset chuyi     
(integer) 1
127.0.0.1:6379> sadd myset lovechuyi
(integer) 1
127.0.0.1:6379> smembers myset      # 查看指定set中的所有值
1) "lovechuyi"
2) "chuyi"
3) "helo"
127.0.0.1:6379> sismember myset helo    # 判断某一个值是不是在set集合中
(integer) 1
127.0.0.1:6379> sismember myset hello
(integer) 0
127.0.0.1:6379> scard myset             # 获取set集合中的内容元素个数
(integer) 3

#####################################################################
srem    

127.0.0.1:6379> srem myset helo     # 移除set集合中的指定元素
(integer) 1
127.0.0.1:6379> smembers myset
1) "lovechuyi"
2) "chuyi"

#####################################################################
set 无序不重复集合，抽随机

127.0.0.1:6379> SRANDMEMBER myset       # 随机抽出一个元素
"chuyi"
127.0.0.1:6379> SRANDMEMBER myset 2     # 随机抽出指定个数的元素
1) "lovechuyi"
2) "world"

#####################################################################
# 删除指定的key，随机删除key

127.0.0.1:6379> SMEMBERS myset
1) "world"
2) "lovechuyi"
3) "hello"
4) "chuyi"
127.0.0.1:6379> spop myset      # 随机删除一个set集合中的元素
"hello"
127.0.0.1:6379> SPOP myset
"chuyi"
127.0.0.1:6379> SMEMBERS myset
1) "world"
2) "lovechuyi"

#####################################################################
# 将一个指定的值，移动到另外一个set集合中

127.0.0.1:6379> sadd myset hello
(integer) 1
127.0.0.1:6379> SADD myset world
(integer) 1
127.0.0.1:6379> sadd myset chuyi
(integer) 1
127.0.0.1:6379> sadd myset2 set2
(integer) 1
127.0.0.1:6379> smove myset myset2 chuyi    # 将一个指定的值，移动到另一个set集合中
(integer) 1
127.0.0.1:6379> smembers myset
1) "world"
2) "hello"
127.0.0.1:6379> smembers myset2
1) "set2"
2) "chuyi"

#####################################################################
微博，b站，共同关注！（交集）
数字集合类：
 - 差集
 - 交集
 - 并集

127.0.0.1:6379> sdiff set1 set2     # 差集
1) "a"
2) "b"
127.0.0.1:6379> sinter set1 set2    # 交集，共同好友就可以这么实现
1) "c"
127.0.0.1:6379> sunion set1 set2    # 并集
1) "e"
2) "c"
3) "b"
4) "d"
5) "a"
```

微博，A用户将所有关注的人放在一个set集合中！将它的粉丝也放在一个集合中！
共同关注，共同好友，二度好友，推荐好友！（六度分隔理论）

### Hash
Map集合，key-map!这时候这个值是map集合！本质和string类型没有太大区别，还是一个简单的key:value，只是这个值变成了map
set myhash filed kuangshen

``` bash
127.0.0.1:6379> hset myhash field1 chuyi        # set一个具体的key-value
(integer) 1
127.0.0.1:6379> hget myhash field1              # 获取一个字段值
"chuyi"
127.0.0.1:6379> hmset myhash field1 hello field2 world      # set多个key-value
OK
127.0.0.1:6379> hmget myhash field1 field2      # 获取多个字段值
1) "hello"
2) "world"
127.0.0.1:6379> hgetall myhash                  # 获取全部的数据
1) "field1"
2) "hello"
3) "field2"
4) "world"
127.0.0.1:6379> hdel myhash field1          # 删除hash指定key字段，对应的value值也就消失了
(integer) 1
127.0.0.1:6379> hgetall myhash
1) "field2"
2) "world"

#####################################################################
hlen

127.0.0.1:6379> hgetall myhash
1) "field2"
2) "world"
3) "field1"
4) "hello"
127.0.0.1:6379> hlen myhash         # 获取hash表的字段数量
(integer) 2

#####################################################################
127.0.0.1:6379> hexists myhash field1       # 判断hash中指定字段是否存在
(integer) 1
127.0.0.1:6379> hexists myhash field3       
(integer) 0

#####################################################################
# 只获得所有的字段
# 只获得所有的值
127.0.0.1:6379> hkeys myhash
1) "field2"
2) "field1"
127.0.0.1:6379> hvals myhash
1) "world"
2) "hello"

#####################################################################
127.0.0.1:6379> hset myhash field3 5
(integer) 1
127.0.0.1:6379> hincrby myhash field3 1
(integer) 6
127.0.0.1:6379> hincrby myhash field3 -1
(integer) 5

127.0.0.1:6379> hsetnx myhash field4 hello4         # 如果不存在则添加
(integer) 1
127.0.0.1:6379> hsetnx myhash field4 world          # 存在则不添加
(integer) 0
```

变更的数据 user name age，尤其是用户信息之类的，经常变动的信息！hash更适合于对象的存储，String更加适合字符串的存储

### Zset(有序集合)
在set的基础上，增加了一个值，set k1 v1, zset k1 score v1
``` bash
127.0.0.1:6379> zadd myset 1 one        # 添加一个值
(integer) 1
127.0.0.1:6379> zadd myset 2 two 3 three 4 four     # 添加多个值
(integer) 3
127.0.0.1:6379> zrange myset 0 -1
1) "one"
2) "two"
3) "three"
4) "four"

#####################################################################
127.0.0.1:6379> zadd salary 2500 xh
(integer) 1
127.0.0.1:6379> zadd salary 5000 zs
(integer) 1
127.0.0.1:6379> zadd salary 100000000 cy
(integer) 1
127.0.0.1:6379> zrangebyscore salary -inf +inf      # 显示所有的用户，从小到大排序
1) "xh"
2) "zs"
3) "cy"
127.0.0.1:6379> zrangebyscore salary -inf +inf withscores   # 显示所有的用户（带score），从小到大排序
1) "xh"
2) "2500"
3) "zs"
4) "5000"
5) "cy"
6) "100000000"  
127.0.0.1:6379>  zrevrangebyscore salary +inf -inf      # 从大到小排序
1) "cy" 
2) "zs"
127.0.0.1:6379> zrangebyscore salary -inf 2500      # 显示工资小于2500员工的升序排序
1) "xh"

#####################################################################
# 移除元素 rem

127.0.0.1:6379> zrange salary 0 -1
1) "xh"
2) "zs"
3) "cy"
127.0.0.1:6379> zrem salary xh          # 移除有序集合中的指定元素
(integer) 1
127.0.0.1:6379> zrange salary 0 -1
1) "zs"
2) "cy"
127.0.0.1:6379> zcard salary            # 获取有序集合中的个数
(integer) 2

#####################################################################



```
## 三种特殊数据类型
### geospatial
### hyperloglog
### bitmaps




# --------------------------------------------------------------------------------
# Java基础知识
## 面向对象的三大特征
+ 封装
+ 继承
+ 多态
　　多态同一个行为具有多个不同表现形式或形态的能力。是指一个类实例（对象）的相同方法在不同情形有不同表现形式。多态机制使具有不同内部结构的对象可以共享相同的外部接口。这意味着，虽然针对不同对象的具体操作不同，但通过一个公共的类，它们（那些操作）可以通过相同的方式予以调用。
　　**多态的优点：**
　　1.消除类型之间的耦合关系
　　2.可替换性
　　3.可扩充性
　　4.接口性
　　5.灵活性
　　6.简化性
　　**多态存在的三个必要条件：**
　　1.继承
　　2.重写（子类继承父类后对父类方法进行重新定义）
　　3.父类引用指向子类对象
　　简言之，多态其实是在继承的基础上的。比如说今天我们要去动物园参观动物，那么你说我们去参观兔子、参观绵羊、参观狮子、参观豹子都是对的，但你不能说我们去参观汽车。在这个例子中，子类具有多态性：除了使用自己的身份，还能充当父类。
 **在多态中，一个对象的实际类型是确定的，引用类型就不确定了（父类的引用可以指向子类对象）**
 **对象能执行哪些方法，主要看对象的引用类型，当父类的引用指向子类对象时，子类重写了父类的方法，就执行子类的方法**

1. static方法不能被重写，属于类不属于实例
2. final 不能重写
3. private 不能重写
### 代码块
执行顺序： 静态代码块->匿名代码块->构造方法
静态代码块只执行一次。
匿名代码块常被用来赋初值。
静态导入包：import static java.lang.Math.random;

## 抽象类与接口
### 抽象类
*abstract*
1. 不能new抽象类，只能靠子类去实现它
2. 抽象类中可以写普通的方法
3. 抽象方法必须在抽象类中
### 接口
*interface*
+ 普通类：只有具体实现
+ 抽象类：具体实现和规范（抽象方法）都有
+ 接口：只有规范，自己无法写方法~专业的约束！约束和实现分离：面向接口编程。

**作用**
1. 约束
2. 定义一些方法，让不同的人实现
3. 方法都是public abstract
4. 属性都是public static final
5. 接口不能被实例化，接口没有构造方法
6. 实现接口必须要重写接口中的方法

## 异常
+ Java把异常当做对象来处理，并定义一个基类java.lang.Throwable作为所有异常(Error和Exception)的超类。
+ 在Java API中已经定义了许多异常类，这些异常类分为两大类，错误*ERROE*和异常*Exception*。
+ Error类对象由Java虚拟机生成并抛出，大多数错误与代码编写者所执行的操作无关。
+ Error和Exception的区别：Error通常是灾难性的致命的错误，是程序无法控制的，当出现这些异常时，JVM一般会选择终止线程；Exception通常情况下是可以被程序处理的，并且在程序中应该尽可能的去处理这些异常。

### 异常处理机制
*throw* *throws* *try* *catch* *finally*
+ 抛出
    throw: 方法中
    throws: 方法上
+ 捕获

## Java集合类
常用的集合用List、Set和Map集合，其中List和Set继承了Collection接口。
### 泛型
<T>
+ 泛型类  
+ 泛型方法
+ 泛型接口 

#### Collections
Collections是针对集合进行操作的工具类，都是静态方法。
**Collections和Collection的区别？**
Collection是单列集合的顶层接口，有子接口List和Set。
Collections是针对集合操作的工具类，有对集合进行排序和二分查找的方法。

void sort(List<?> list) 排序
int binarySearch(List<?> list, T key) 二分查找
T max(Collection<?> coll) 最大值
void reverse(List<?> list) 反转
void shuffle(List<?> list) 随机置换

#### 泛型通配符
+ ？:任意类型，如果没有明确，那么就是Object以及任意的Java类了
+ ？ extends E :向下限定，E及其子类
+ ? super E :向上限定，E及其父类

### List
List是有序（存储顺序和取出顺序一致），可重复
#### ArrayList
底层数据结构是数组，查询快，增删慢，线程不安全，效率高
#### Vector
底层数据结构是数组，查询快，增删慢，线程不安全，效率低
Vector的特有功能
1. 添加功能
    public void addElement(Object onj)
2. 获取功能
    public Object elementAt(int index)
    public Enumeration elements()
    boolean hasMoreElements()
    Object nextElement()

#### LinkedList
底层数据结构是链表，查询慢，增删块，线程不安全，效率高
1. 添加
    public void addFirst(Object e)
    public void addLast(Object e)
2. 获取
    public Object getFirst()
    public Object getLast()
3. 删除
    public Object removeFirst()
    public Object removeLast()

### Set
Set是无序（存储顺序和取出顺序不一致），唯一。
虽然Set集合无序，但是作为集合来说他肯定有自己的顺序，而你的顺序恰好与他的一致，这不代表有序，可以多存储一些数据，就能看到效果。
#### HashSet
底层数据结构是哈希表。
对集合的迭代次序不作任何保证; 特别是，它不能保证该顺序在一段时间内保持不变。

#### LinkedHashSet
具有可预知迭代顺序的Set接口的哈希表和链接列表实现。

#### TreeSet
底层是红黑树（红黑树是一种自平衡的二叉树），使用元素的自然顺序对元素进行排序，或者根据创建的set时提供的Comparator进行排序，具体取决于使用的构造方法。

*元素是如何存储进去的呢？*
第一个元素存储的时候，直接作为根节点存储，从第二个元素开始，每个元素从根节点开始比较，比根节点大就作为右孩子，小就作为左孩子，相等就不添加。
*元素是如何出来的呢？（前序/中序/后序遍历）*
从根节点开始按照左中右的原则一次取出元素即可。

### Map
Map<K,V>
K键值是唯一的，V值可以重复。每个键最多只能映射到一个值。键是无序的。
Map和Collection集合的区别？
+ Map集合元素是成对出现的，Map集合的键是唯一的，值是可重复的。
+ Collection集合存储元素是单独出现的，Collection的儿子Set是唯一的，List是可重复的。
+ Map集合的数据结构针对键有效，与值无关，Collection集合的数据结构针对元素有效。

**Map集合功能概述**
1. 添加
    V put(K key, V value):添加元素，如果键是第一次存储，则直接存储元素，并返回Null。如果有键不是第一次存储，则替换值，返回以前的值。
2. 删除
    void clear():移除所有的键值对元素
    V remove(Object key):根据键删除键值对并返回值
3. 判断
    boolean containsKey(Object key):判断集合是否包含指定的键
    boolean containsValue(Object value):判断集合是否包含指定的值
    boolean isEmpty():判断集合是否为空
4. 获取
    Set<Map.Entry<K,V>> entrySet():?
    V get(Object key):根据键获取值
    Set<K> keySet():获取集合中所有键集合
    Collection<V> values():获取集合中所有值得集合
5. 长度
    int size():返回集合中键值对的个数

#### HashMap
    基于哈希表的Map接口实现，哈希表的作用是用来保证键的唯一性。
#### TreeMap
    基于红黑树的map接口实现，保证键的排序和唯一。
#### LinkedHashMap
    是Map接口的哈希表和链接列表实现，具有可预知的迭代顺序。由哈希表保证键的唯一性，由链表保证键的有序（存储和取出的顺序一致）。
    
## 注解与反射
### Annotation注解
Annotation的作用：
+ 不是程序本身，可以对程序做出解释（这点和注释没有什么区别）
+ 可以被其他程序（如：编译器等）读取。

Annotation可以在哪里使用？
+ 可以附加在package,class,method,filed等上面，相当于给他们添加了额外的辅助信息，我们可以通过反射机制编程实现对这些元数据的访问。

常见的注解
@Override   // 重写的注解
@Deprecated // 不推荐程序员使用，但是可以使用，或者存在更好的方式
@SuppressWarnings("all") // 抑制编译时的警告信息

元注解
元注解的作用就是负责注解其他注解，Java定义了4个标准的meta-annotation类型，他们被用来提供对其他annotation类型进行说明。
+ @Target 用于描述注解的使用范围
+ @Retention 表示需要在什么级别保存该注释信息，用于描述注解的生命周期（source < class < runtime）
+ @Document 说明该注解将被包含在javadoc中
+ Inherited 说明子类可以继承父类中的该注解

自定义注解
+ 使用@Interface自定义注解，自动继承Annotation接口

### 反射机制
动态语言就是在运行时代码可以根据某些条件改变自身结构。
Java不是动态语言，但是Java可以称之为”准动态语言“。即java具有一定的动态性，我们可以利用反射机制获取类似动态语言的特性。

反射是Java中被视为动态语言的关键，反射机制允许程序在执行期借助于反射取得任何类的内部信息，并且能够直接操作任意对象的内部属性及方法。

加载完类之后，在堆内存的方法区中就产生了一个Class类型的对象（一个类只有一个Class对象），这个对象就包含了完整的类的结构信息。我们可以通过这个对象看到类的结构。

正常方式：引入需要的”包类“名称 -> 通过new实例化 -> 取得对象实例化
反射方式: 实例化对象 -> getClass()方法 -> 得到完整的包类名称

反射的优点：可以实现动态创建对象和编译，体现出很大的灵活性。
反射的缺点：使用反射基本上是一种解释操作，我们可以告诉JVM，我们希望做什么并且它满足我们的要求。这类操作总是慢于直接执行相同的操作。

### 什么时候会发生类初始化
+ 类的主动引用（一定会发生类的初始化）
    - 当虚拟机启动时，先初始化main方法所在的类
    - new 一个类的对象
    - 调用类的静态成员（除了final常量）和静态方法
    - 使用java.lang.reflect包的方法对类进行反射调用
    - 当初始化一个类，如果其父类没有初始化，会先初始化它的父类
+ 类的被动引用（不会发生类的初始化）
    - 当访问一个静态域时，只有真正声明这个域的类才会被初始化。如：当通过子类引用父类的静态变量，不会导致子类初始化
    - 通过数组定义类引用，不会触发此类的初始化
    - 引用常量不会触发此类的初始化（常量在链接阶段就存入调用类的常量池中了）

### 类加载器的作用
+ 类加载的作用：将class字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构，然后在堆中生成一个代表这个类的java.lang.Class对象，作为方法区中类数据的访问入口。
+ 类缓存：标准的JavaSE类加载器可以按要求查找类，但一旦某个类被加载到类加载器中，它将维持加载（缓存）一段时间。不过JVM的垃圾回收机制可以回收这些Class对象。

*.java文件 -> Java编译器 -> .class文件 -> 类加载器 -> 字节码校验器 -> 解释器 -> 操作系统平台*

类加载器作用是把类(class)装载进内存的。JVM规范定义了如下类型的类加载器。
引导类(根)加载器：用C++编写的，是JVM自带的类加载器，负责Java平台核心库，用来装载核心类库，该加载器无法直接获取。
扩展类加载器：负责jre/lib/ext目录下的jar包或 -D java.ext.dirs指定目录下的jar包装入工作库。
系统类加载器：负责java -classpath或-D java.class.path所指的目录下的类与jar包装入工作，是最常用的加载器。

### setAccessible
+ Method和Field、Constructor对象都有setAccessible()方法
+ setAccessible作用是启动和禁止访问安全检查的开关
+ 参数值为true则指示反射的对象在使用时应该取消Java语言访问检查
    - 提高反射的效率，如果代码中必须使用反射，而该句代码需要频繁的被调用，那么请设置为true
    - 使得原本无法访问的私有成员也可以访问
+ 参数值为false则指示反射的对象应该实施Java语言访问检查

## 多线程
https://www.cnblogs.com/java1024/archive/2019/11/28/11950129.html
### 进程(Process)与线程(Thread)的区别
+ 程序是指令和数据的有序集合，其本身没有任何运行的含义，只是一个静态的概念。
+ 而进程则是执行程序的一次执行过程，他是一个动态的概念。是系统资源分配的单位。
+ 通常在一个进程中可以包含若干个线程，当然一个进程中至少有一个线程，不然没有存在的意义。线程是cpu调度和执行的单位。

进程是程序的一次动态执行过程，它需要经历从代码加载，代码执行到执行完毕的一个完整的过程，这个过程也是进程本身从产生，发展到最终消亡的过程。多进程操作系统能同时达运行多个进程（程序），由于 CPU 具备分时机制，所以每个进程都能循环获得自己的CPU时间片。由于CPU执行速度非常快，使得所有程序好像是在同时运行一样。

注意：很多多线程是模拟出来的，真正的多线程是指有多个cpu，即多核，如服务器。如果是模拟出来的多线程，即在一个cpu的情况下，在同一个时间点，cpu只能执行一个代码，因为切换的很快，所以就有同时执行的错觉。

在 Java 中实现多线程有三种手段，继承 Thread 类，实现 Runnable 接口和实行Callable接口。

Thread类实现了Runnable接口。

推荐使用Runnable，避免单继承局限性，灵活方便，方便同一个对象被多个线程使用。

### 静态代理
真实对象和代理对象都要实现同一个接口
代理对象要代理真实角色

好处：
+ 代理对象可以做很多真实对象做不了的事情
+ 真实对象专注做自己的事情

### 线程状态
+ 创建状态：
    Thread t = new Thread()线程对象一旦创建就进入到了新生状态
+ 就绪状态
    当调用start()方法，线程立即进入就绪状态,但并不意味着立即调度执行
+ 阻塞状态
    当调用sleep,wait或同步锁定时,线程进入阻塞状态，就是代码不往下执行，阻塞事件解除后，重新进入就绪状态，等待cpu调度执行
+ 运行状态
    进入运行状态，线程才真正执行线程体的代码块
+ 死亡状态
    线程中断或者结束，一旦进入死亡状态，就不能再次启动

### 线程方法
#### 停止线程
1. 建议线程正常停止--->利用次数，不建议死循环
2. 建议使用标志位--->设置一个标志位
3. 不要使用stop或者destroy等过时或者JDK不建议使用的方法

#### 线程休眠
+ sleep(时间)指定当前线程阻塞的毫秒数
+ sleep存在异常InterruptedException
+ sleep时间达到后线程进入就绪状态
+ sleep可以模拟网络延时，倒计时等
+ 每个对象都有一个锁，sleep不会释放锁

#### 线程礼让(yield)
+ 礼让线程，让当前正在执行的线程暂停，但不阻塞
+ 将线程从运行状态转换为就绪状态
+ 让cpu重新调度，礼让不一定成功！看cpu心情

#### 线程插队(join)

### 线程优先级
+ Java提供一个线程调度器来监控程序中启动后进入就绪状态的所有线程，线程调度器按照优先级决定应该调度哪个线程来执行。
+ 线程的优先级用数字来表示，范围从1~10.
    - Thread.MIN_PRIORITY = 1;
    - Thread.MAX_PRIORITY = 10;
    - Thread.NORM_PRIORITY = 5;
+ 使用以下方式改变或获取优先级
    - getPriority(),setPriority(int xxx)

线程的优先级低只是意味着获得调度的概率低，并不是优先级低就不会被调用了，这都是看cpu的调度。

### 守护(daemon)线程
+ 线程分为用户线程和守护线程
+ 虚拟机必须确保用户线程执行完毕
+ 虚拟机不用等待守护线程执行完毕
+ 如，后台记录操作日志，监控内存，垃圾回收等待

### 线程同步(synchronized)
多个线程操作同一个资源
**并发：同一个对象被多个线程同时操作。**

处理多线程问题时，多个线程访问同一个对象，并且某些线程还想修改这个对象，这个时候我们就需要线程同步，线程同步其实就是一种等待机制，多个需要同时访问此对象的线程进入这个对象的 等待池 形成队列，等待前面线程使用完毕，下一个线程再使用。

线程同步形成条件：队列+锁

由于同一个进程的多个线程共享同一块存储空间，在带来方便的同事，也带来了访问冲突问题，为了保证数据在方法中被访问时的正确性，在访问时加入锁机制synchronized，当一个线程获得对象的排它锁，独占资源，其他线程必须等待，使用后释放锁即可。存在以下问题：
+ 一个线程持有锁会导致其他所有需要此锁的线程挂起
+ 在多线程竞争下，加锁，释放锁会导致比较多的上下文切换 和 调度延时，因此性能问题
+ 如果一个优先级高的线程等待一个优先级低的线程释放锁 会导致优先级倒置，引起性能问题

synchronized方法和synchronized块
synchronized方法锁的是this
synchronized块锁的是对象

### 死锁
多个线程各自占有一些共享资源，并且互相等待其他线程占有的资源才能运行，而导致两个或者多个线程都在等待对方释放资源，都停止执行的情形，同一个同步块同时拥有“两个以上对象的锁”时，就有可能会发生“死锁”的问题。

产生死锁的四个必要条件：
1. 互斥条件：一个资源每次只能被一个线程使用。
2. 请求与保持条件：一个线程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺资源：线程已获得的资源，在未使用完之前，不能强行剥夺。
4. 循环等待条件：若干线程间形成一种头尾相接的循环等待资源条件。

### synchronized与lock对比
+ lock是显式锁（手动开启和关闭锁）synchronized是隐式锁，出了作用域自动释放
+ lock只有代码块锁，synchronized有代码块锁和方法锁
+ 使用lock锁，JVM将花费较少的时间来调度线程，性能更好。并且有更好的扩展性
+ 优先使用顺序：locl -> 同步代码块（已经进入了方法体，分配了相应资源） -> 同步方法（在方法体之外）

### 线程协作
生产者消费者问题（同步和通信）
解决方式1：（管程法）
生产者：负责生产数据的模块（可能是方法，对象，线程，进程）
消费者：负责处理数据的模块（可能是方法，对象，线程，进程）
缓冲区：消费者不能直接使用生产者的数据，他们之间有个“缓冲区”
生产者将生产好的数据放入缓冲区，消费者从缓冲区取出数据

解决方法2：（信号灯法）
通过标志位来判断

### 使用线程池
+ 背景：经常创建和销毁、使用量特别大的资源，比如并发情况下的下城，对性能影响很大。
+ 思路：提前创建好多个线程，放入线程池中，使用时直接获取，使用完放回池中。可以避免频繁创建销毁、实现重复利用。类似生活中的交通工具。
+ 好处：
    - 提高响应速度（减少了创建新线程的时间）
    - 降低资源消耗（重复利用线程池中线程，不需要每次都创建）
    - 便于线程管理
        + corePoolSize: 核心池的大小
        + maximumPoolSize: 最大线程数
        + keepAliveTime: 线程没有任务时最多保持多长时间后会终止

## 十进制2任何进制
```java
public static String dec2Radix(long dec, int radix){
    String s = new String("");
    long mod = 0;
    while (dec > 0) {
        mod = dec % radix;
        dec = dec / radix;
        char a = val_code_dict.get((int)mod);   // val_code_dict数字字母对应表，10-A...
        s += a;
    }

    StringBuilder sb = new StringBuilder();
    for (int i = s.length() - 1; i >= 0; i--) {
        sb.append(s.charAt(i));
    }

    String result = sb.toString();
    return result;
}
```

## 任何进制2十进制
    // 31进制字符串转int值
    public static long radix2Dec(String val, int radix) {
        long result = 0;
        for (int i = val.length()-1; i >= 0; i--) {
            char a = val.charAt(i);
            int dict = code_val_dict.get(a);
            result += (dict * Math.pow(radix, i));
        }
        return result;
    }

## Lambda表达式
+ 函数式接口的定义：
    - 任何接口，如果只包含唯一一个抽象方法，那么他就是一个函数式接口。
        ```java
        @FunctionalInterface
        public interface Runnable {
            public abstract void run();
        }
        ```
    - 对于函数式接口，我们可以通过lambda表达式来创建该接口的对象。

+ 总结：
    lambda表达式在只有一行代码的情况下才能简化成为一行，如果有多行，那么就用代码块包裹。
    前提是接口为函数式接口
    多个参数也可以去掉参数类型，要去掉就都去掉，必须加上括号