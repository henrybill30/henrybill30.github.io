---
title: redis学习（1）
date: 2021-09-16 13:17:27
tags: [redis, 数据库]
categories: 数据库
---
# Redis基本介绍
官网原文

>Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache, and message broker. Redis provides data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, geospatial indexes, and streams. Redis has built-in replication, Lua scripting, LRU eviction, transactions, and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster.

- 在内存中的数据存储，kv结构
- 可以作为内存数据库、缓存、消息broker
- 提供多种数据结构，后面详细介绍
- 复制、Lua脚本、LRU置换、事务、不同级别的持久化
- 通过Redis哨兵机制和自动分片在集群中提供高性能服务

# Redis Key
可以用任何二进制序列来表示redis的key，有几个规则：
- 不要过长，不仅占内存也会在查找时消耗性能
- 不要过短
- 试着确定一个key的模式(schema)
- key的最大size为512MB
# 5种基本数据结构
## String
最简单的数据结构，和key一样为string类型，主要是两个命令**SET**和**GET**

SET命令
```bash
set key value [EX second | PX millisecond | EXAT timestamp | PXAT timestamp] [NX | XX] [GET]
```
- EX 多少秒后过期
- PX 多少毫秒后过期
- EXAT 在多少秒过期 timestamp格式
- PXAT 在多少毫秒过期 timestamp格式
- NX 如果key存在则不set
- XX 不论key是否存在都可以set，若存在就覆盖
- GET 返回之前的value，若没有则为nil

GET命令
```bash
GET key
```
获取对应key的value

其他命令

- INCR, INCRBY 增加一个数字 64bit数字
- DECR, DECRBY 减少一个数字 64bit数字
- GETSET set一个值并返回旧值
- MGET, MSET 批量set和get
- EXISTS 是否存在value
- DEL 删除key
- TYPE 返回value的类型

## List
基于Linked List实现的list数据结构，主要命令LPUSH、RPUSH、LPOP、RPOP、RLANGE

- LPUSH 向列表头添加元素
- RPUSH 向列表尾添加元素
- RLANGE key start end   查看列表元素
- LPOP 从列表头去除元素
- RPOP 从列表尾去除元素

以上为list的基本操作，list的用途有以下两种

- 用作存储用户最近更新的内容，如最新发布的10篇文章等
- 用作两个进程间的通信，如作为生产者/消费者中的队列

以下为一些进阶操作

- LTRIM 可以限制list的长度，使用
```bash
ltrim key start end # start为list的起始，end为list的终止
```
trim的原理是取范围的元素组成新的list，再删除原来的list

- BRPOP 等待弹出元素

作为阻塞队列时使用，在生产者/消费者模式中，如果生产者一直没有生产，消费者取不出来时就需要不断通过轮询来判断是否有新元素，这样比较消耗性能，可以用brpop命令，具体为：
```bash
brpop key sec # sec为等待的时间，如果在这个时间内获取不到元素就返回nil，0为无限等待
```

redis中会自动创建和销毁list，主要有以下三个规则：

- 向一个list中添加元素时，如果key不存在，自动创建

![list1](lsit1.png)

- 从一个list中弹出元素时，如果list为空，则自动删除

![list2](list2.png)

- 对于只读命令或是删除命令，如果key值不存在，则返回的结果均按照空列表处理

![lsit3](lsit3.png)

## Hash
可以理解为Map<String, Map<String, Object>>，有以下几个基本命令：

- HSET
- HMSET
- HGET
- HMGET

![map](map1.png)
具体看上面图片就行，更多命令可以参考[官网](https://redis.io/commands#hash).

## Set
类似一个HashSet，主要可以用作存储“标签类数据”，主要的命令有：

- SADD key value1 value2 value3
- SMEMBERS key 查看所有元素
- SISMEMBER key value 查看元素是否在set中
- SINTER key1 [key2 key3 ...] 交集
- SUNION key1 [key2 ...] 并集
- SDIFF key1 [key2 ...] 差集
- SUNIONSTORE key [key1, key2...] 把并集复制到key中
- SDIFFUNION key [key...]
- SINTERUNION key [key...]
- SPOP key 随机弹出元素
- SCARD key 我理解就是set中的元素个数

## Sorted Set
有些类似一个map结构，map的key是一个score，用score来进行排序，如果score相等，则按value来排序，主要命令

- ZADD key [NX | XX] [GT | LT] [CH] [INCR] score member [...] 添加元素
- ZREM key member [...] 删除元素
- ZRANGE key start end [WITHSCORES] 查看元素
- ZREVRANGE key start end [WITHSCORES] 反向输出
- ZRANGEBYSCORE key min max [WITHSCORE] [LIMIT offset count] 按score范围获取
- ZREMRANGEBYSCORE key min max 按范围删除
- ZRANK key member 返回该member在zset中的位置
- ZRANGEBYLEX key min max [limit offset count] 按value的字典序查找元素

# 3种特殊数据结构
## Bitmaps
位图，就是设置一个512MB的String的每一bit的值，只能取0或1，主要命令：

- SETBIT key postion bit
- GETBIT key postion
- BITOP operation deskey key1 [...] 把两个String做逻辑操作，operation有AND、OR、NOT
- BITPOS key bit [start [end]] 返回一个String中第一个0或1的位置
- BITCOUNT key 返回一个String中1的个数
  
bitmap的用处

- 任何形式的实时分析
- 存储Boolean类型信息

其他两个有HyperLogLogs和Geospatial

# 底层原理
命令：

- OBJECT ENCODEING key [...] 存储的一个数据的内部实现
![data1](data1.jpg)

数据类型底层实现

- String类型：底层为简单动态字符串(Simple Dynamic String SDS) embstr
- List类型：底层为双向链表，无环，带长度计数器，实现多态 quicklist
- Hash类型：底层为压缩列表 ziplist
- Set类型： 底层为table，哈希冲突通过链表法解决，扩容机制为先创建一个2倍大小的table，再重新计算hash值再插入。当数据量大时会采用渐进式扩容，获取和更新操作在两个table中进行，插入操作在新table中进行。扩容时机：没有持久化时负载因子大于等于1，持久化时负载因子大于等于5 hashtable
- Sorted Set类型：底层为压缩列表