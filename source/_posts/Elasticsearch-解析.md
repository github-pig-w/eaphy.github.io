---
title: Elasticsearch 解析
tags:
  - elasticsearch
categories:
  - 开源框架
abstract: <font color=red>ʅ（´◔౪◔）ʃ &nbsp&nbsp&nbsp  文章已加密，有缘自会相见 ！！！</font>
message: 执意要看，请输入暗号 ！
date: 2019-09-01 13:50:06
password:
---

## 一、shard
1. es 中的 shard 分为 primary shard 和 replica shard，后者是前者的副本，负责容错，承担读请求
2. 每个 shard 都可以独立完成创建索引，处理请求，是一个最小工作单元
3. 每个 document 只会存在一个 primary shard 和 其对应的 replica shard 中
4. primary shard  在索引创建时固定，默认为 5，创建后不可修改， replica shard ，默认为1，可以修改
5. 一个 index 包含多个 shard


<!-- more -->

<img src="/img/20040501.png"  align=center/>

## 二、扩容、容错
 由于 primary shard  在索引创建后不可修改，所以 es 的扩容只能扩展 replica shard 的数量

 1. 一台mater node节点宕机后，其 primary shard  丢失，status 变红
 2. 集群进行master选举，自动选举一个新的node作为master
 3. 新的mater会将丢失的 primary shard  的某个 replica shard 提升为primary shard ，status 变黄，因为少一个 replica shard 
 4. 重启宕机的 node ，新的master会将宕机后缺失的数据复制到该node上，原来的数据正常使用，status 变绿


## 三、全量替换、部分更新
```java
PUT /eaphy/student/1
{
  "id":1,
  "age":11,
  "name":"小花",
  "city":"北京",
  "desc":"我是一个北京人"
}

POST /eaphy/student/1/_update
{
  "doc": {
    "age":11
  }
}
```
1.全量替换是先查询到document，修改信息，把信息全部发送到后台，后台发送put请求，进行全量替换，再把老的document标记为deleted，然后创建一个新的document
2.部分更新是先查询到document，将更新的数据更新到document中，再把老的document标记为deleted，然后创建一个新的document，这个过程是在一个shard中完成的
3.部分更新，减少了网络请求，减少了并发冲突。

## 四、document 路由
一个index存在多个shard中，当一个document传进来时，其路由到某个具体shard的算法是： shard = hash(routing num) % number_of_primary_shards

ps: 一个index有3个primary shard，每增删改查一个document时，会带过来一个routing num，默认是 document 的 id，然后求出hash值，与 3 求余，得到其应该定位到哪个shard中。

这就是 primary shard 数量不可修改的原因，会导致路由错误。

## 五、写一致性、quorum 机制
我们在发送一个增删改操作时，可以带上一个consistency参数，指明我们想要的写一致性是什么。

```bash
put index/type/id?consistency=one
put index/type/id?consistency=all
put index/type/id?consistency=quorum
```
one: 只要有一个primary shard 是活跃的就可以执行
all： 必须所有 primary shard 和 replica shard是活跃的，才可执行
quorum：必须大部分shard是活跃的，才能执行

quorum = int（（primary_num+replica_num）/ 2）+1，当replica_num>1时生效
ps: 3 个primary shard ，1个 replica shard ，共 3+3*1=6，那么 quorum = int((3+1)/2)+1 = 3 ,即要求6个shard中，至少3个shard活跃才能执行

quorum 不齐全时，默认会等待1分钟，也可以自己加参数 timeout 去设定