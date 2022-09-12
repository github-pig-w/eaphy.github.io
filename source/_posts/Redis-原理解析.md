---
title: Redis 原理解析
tags:
  - redis
categories:
  - 中间件
abstract: <font color=red>ʅ（´◔౪◔）ʃ &nbsp&nbsp&nbsp  文章已加密，有缘自会相见 ！！！</font>
message: 执意要看，请输入暗号 ！
date: 2019-07-29 23:46:59
password:
---

# 数据类型
## String
string 类型通过int、sds作为数据结构，int用来存储整型数据，sds存放字节、字符串、浮点型数据。redis3.2分支引入了五种sdshdr类型，目的是为了满足不同长度字符串可以使用不同大小的Header，从而节省内存，每次在创建一个sds时根据sds的实际长度判断应该选择什么类型的sdshdr。
## List
list 可以存储一个有序的字符串列表，常用的操作是向列表两端添加元素。

<!-- more -->

redis3.2之前，List类型的value对象内部以linkedlist或者ziplist来实现, 当list的元素个数和单个元素的长度比较小 的时候，Redis会采用ziplist（压缩列表）来实现来减少内存占用。否则就会采用linkedlist（双向链表）结构。

redis3.2之后，采用的一种叫quicklist的数据结构来存储list，列表的底层都由quicklist实现。 这两种存储方式都有优缺点，双向链表在链表两端进行push和pop操作，在插入节点上复杂度比较低，但是内存开 销比较大； ziplist存储在一段连续的内存上，所以存储效率很高，但是插入和删除都需要频繁申请和释放内存；

quicklist仍然是一个双向链表，只是列表的每个节点都是一个ziplist，其实就是linkedlist和ziplist的结合，quicklist 中每个节点ziplist都能够存储多个数据元素

## Hash
map提供两种结构来存储，一种是hashtable、另一种是前面讲的ziplist，数据量小的时候用ziplist. 
## Set
Set在的底层数据结构以intset或者hashtable来存储。当set中只包含整数型的元素时，采用intset来存储，否则， 采用hashtable存储，但是对于set来说，该hashtable的value值用于为NULL。通过key来存储元素 。
## Sorted-Set
在集合类型的基础上，有序集合类型为集合中的每个元素都关联了一个分数，这使得我们不仅可以完成插入、删除 和判断元素是否存在等集合类型支持的操作，还能获得分数最高(或最低)的前N个元素、获得指定分数范围内的元 素等与分数有关的操作。虽然集合中每个元素都是不同的，但是他们的分数却可以相同

zset类型的数据结构就比较复杂一点，内部是以ziplist或者skiplist+hashtable来实现，这里面最核心的一个结构就 是skiplist，也就是跳跃表 

# 过期删除原理

 1. 周期性随机抽取20个设置了过期时间的key，判断是否过期，过期则删除。
 2. get 时，判断key是否过期，过期则删除。

# 持久化
## RDB
当符合条件时，fork子进程生成一个dump.rdb的快照文件。有可能丢失数据
**条件1：**
修改配置文件：

```
save 900 1		//900s之内有一个key发生改变，触发快照
save 300 10		//300s之内有10个key发生改变，触发快照
save 60  10000		//60s之内有10000个key发生改变，触发快照
```
**条件2：**
执行 `save` 命令  （阻塞所有请求），触发快照
执行 `bgsave` 命令 （不会阻塞请求），触发快照

**条件3：**
执行命令 `flushall`，触发快照

**条件4:**
主从复制时，触发快照

## AOF
在aof文件中保存增删改的的操作指令。
当aof文件过大时，可以进行压缩，把内存中的数据生成一个临时的aof文件，然后覆盖原文件。

# 内存回收策略
**no-enviction：**内存不足时，报错

**allkeys-lru：**挑选最近最少使用的数据淘汰

**allkeys-random：**随机选择数据淘汰

**volatile-lru：**从已设置过期时间的数据中挑选最近最少使用的数据淘汰

**volatile-ttl：**从已设置过期时间的数据中挑选将要过期的数据淘汰

**volatile-random：**从已设置过期时间的数据中随机选择数据淘汰

# 主从架构
## 主从复制原理
当启动一个 slave 节点的时候，它会发送一个 psync 命令给 master 节点，如果是第一次连接 master，那么会触发一次全量复制，mater 在后台启动一个线程，在内存中生成一个 RDB 快照文件，同时还会将所有收到的写命令缓存在内存中，然后将 RDB 发送给slave节点，slave节点会先将快照写入本地磁盘，然后再从本地磁盘加载到内存中，然后 master 节点将内存中缓存的写命令发送给 slave，slave 也会同步这些数据。如果不是第一次连接 master ，那么 master 仅仅复制 slave 部分缺少的数据。

## 主从复制的断点续传
如果主从复制中，网络连接断掉了，可以接着上次复制的地方，继续复制下去。mater 节点会在内存中有一个backlog，mater 和 slave 都会保存一个 offset 和 mater run id，如果网络连接断开了，slave 会判断 mater run id 是否一样，一样的话让 mater 从上次的 offset 开始继续复制，如果没有找到 offset ，会进行一次全量复制。 mater run id 不一样就重新全量复制。

## 无磁盘化复制
master 直接在内存中创建 rdb ，然后发送给 slave ，不会在自己本地落盘

repl-diskless-sync
repl-diskless-sync-delay   延迟复制，因为要等多 slave 节点的连接成功。

## 过期 key 处理
slave 不会过期 key，只会等待 master 过期key，如果 master 过期了一个key，或者通过lru淘汰了一个key，那么会模拟一条del命令发送给 slave。


