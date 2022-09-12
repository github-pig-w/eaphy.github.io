---
title: mysql 解析
tags:
  - mysql
categories:
  - 数据库
abstract: <font color=red>ʅ（´◔౪◔）ʃ &nbsp&nbsp&nbsp  文章已加密，有缘自会相见 ！！！</font>
message: 执意要看，请输入暗号 ！
date: 2019-07-10 12:52:21
password:
---
# 索引是什么
索引是为了加快对表中数据行的检索而创建的一种分散存储的数据结构。

# 为什么用索引

 1. 索引能极大的减少存储引擎需要扫面的数据量
 2. 索引可以把随机IO变成顺序IO
 3. 索引可以帮助我们在进行分组、排序等操作时，避免使用临时表

<!-- more -->
# 为什么是 B+Tree 
## 二叉查找树
<img src="/img/19071001.png"   align=center/>


>  二叉查找树的缺点就是容易发生极端化，导致某个节点太长。

## 平衡二叉查找树
<img src="/img/19071002.png"  align=center/>


>  平衡二叉查找树的缺点就是数据太多时，导致树的深度太高，IO操作次数增多，耗时大，其次，每个节点存储的数据太少。

## 多路平衡二叉查找树（B Tree）
<img src="/img/19071003.png"  align=center/>




## 加强版多路平衡二叉查找树（B+ Tree）
<img src="/img/19071004.png"   align=center/>


# 索引的重要知识

## 列的离散性

列的离散性 = count（distinct col）/ count(col) ,值越大，离散性越好，选择性越高。

## 最左匹配原则
对索引中的关键字进行比对，一定是按照从左往右进行，且不可跳过。

## 联合索引
联合索引列选择原则：

 1. 经常用的列优先（最左匹配）
 2. 离散性高的列优先（离散高原则）
 3. 宽度小的列优先（最少空间原则）

## 覆盖索引
如果查询列可通过索引节点中的关键字直接返回，则该索引称之为覆盖索引。覆盖索引可以减少数据库的IO ，将随机IO变为顺序IO，可提高查询性能。

ps：
索引列：（name+phone）
查询语句：select name,phone from user where name = ? ;

