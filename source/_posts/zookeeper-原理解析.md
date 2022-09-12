---
title: zookeeper 原理解析
tags:
  - zookeeper
categories:
  - 开源框架
date: 2019-06-28 01:04:37
---




# zookeeper 简介

   ZooKeeper 是一个开源的分布式协调服务，是 Apache 顶级项目 Hadoop 中非常重要的组件。
# zookeeper 作用

 统一配置、分布式锁、集群管理、注册中心、分布式队列

 <!-- more -->

# zookeeper 结构
- **数据模型**


![zookeeper 数据结构](/img/zk.jpg)

ZooKeeper 数据模型的结构与Unix文件系统很类似，每个节点称作一个ZNode，ZNode 是以 key-value 形式存在的。

  - **重要概念**
	## Znode：
	zookeeper 中每个文件系统路径就是一个节点。节点有四种：
	- **PERSISTENT-持久化节点**
	客户端与zookeeper断开连接后，该节点依旧存在
	- **PERSISTENT_SEQUENTIAL-持久化顺序节点**
	客户端与zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号
	- **EPHEMERAL-临时节点**
	客户端与zookeeper断开连接后，该节点被删除
	- **EPHEMERAL_SEQUENTIAL-临时顺序节点**
	客户端与zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号
	
	## 会话：(Session)
Session 指的是 ZooKeeper 服务器与客户端会话，是一个 TCP 长连接。通过这个连接，客户端能够通过心跳检测与服务器保持有效的会话，同时还能够接收来自服务器的 Watch 事件通知。在为客户端创建会话之前，服务端首先会为每个客户端都分配一个 sessionID，且具有全局唯一性。

 ## 版本：
zookeeper 为数据节点引入了版本的概念。存在于一个Stat 的数据结构中，记录了这个 ZNode 的三个数据版本，分别是：
dataversion（当前 ZNode 的版本）、cversion（当前 ZNode 子节点的版本）、aclversion（当前 ZNode 的 ACL 版本）。用于保证分布式数据原子性操作。

 ## watcher：
Watcher（事件监听器），是 ZooKeeper 中的一个很重要的特性。ZooKeeper 允许用户在指定节点上注册一些 Watcher，并且在一些特定事件触发的时候，ZooKeeper 服务端会将事件通知到感兴趣的客户端上去，该机制是 ZooKeeper 实现分布式协调服务的重要特性。事件**监听是一次性**的，如果希望收到有关未来更改的通知，则必须设置另一个监听。
## ACL：
zookeeper 通过acl机制来实现对数据节点的权限控制。主要有五种权限：
- **create**	创建权限
- **delete** 子节点的删除权限
- **read**	读取权限
- **write**	更新权限
- **admin**	管理权限，能够进行ACL设置
	 
# zookeeper 特点
- **顺序一致性**：从同一客户端发起的事务请求，最终将会严格地按照顺序被应用到 ZooKeeper 中去。
- **原子性**：所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的，要么整个集群中所有的机器都成功应用了某一个事务，要么都没有应用。
- **单一系统映像**：无论客户端连到哪一个 ZooKeeper 服务器上，其看到的服务端数据模型都是一致的。
- **可靠性**：一旦一次更改请求被应用，更改的结果就会被持久化，直到被下一次更改覆盖。


# zookeeper 角色
- **Leader**	提供读写
- **Follower** 	 提供读
- **Observer**  	提供读，不参与选举


# zookeeper 原理
Zookeeper的核心是原子广播，这个机制保证了各个Server之间的同步，实现这个机制的协议叫做Zab协议。Zab协议有两种模式，它们分别是恢复模式（选主）和广播模式（同步）。当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数Server完成了和 leader的状态同步以后，恢复模式就结束了。状态同步保证了leader和Server具有相同的系统状态。

为了保证事务的顺序一致性，zookeeper采用了递增的事务id号（zxid）来标识事务。所有的提议（proposal）都在被提出的时候加上 了zxid。实现中zxid是一个64位的数字，它高32位是epoch用来标识leader关系是否改变，每次一个leader被选出来，它都会有一个 新的epoch，标识当前属于那个leader的统治时期。低32位用于递增计数。

每个Server在工作过程中有三种状态：

（1）LOOKING：当前Server不知道leader是谁，正在搜寻。
（2）LEADING：当前Server即为选举出来的leader。
（3）FOLLOWING：leader已经选举出来，当前Server与之同步。


- ## 选主流程
当leader崩溃或者leader失去大多数的follower，这时候zk进入恢复模式，恢复模式需要重新选举出一个新的leader，让所有的 Server都恢复到一个正确的状态。
Zk的选举算法有两种：一种是基于basic paxos实现的，一种是基于fast paxos实现的。系统默认为fast paxos。
（1）Basic paxos：当前Server发起选举的线程,向所有Server发起询问,选举线程收到所有回复,计算zxid最大Server,并推荐此为leader，若此提议获得n/2+1票通过,此为leader，否则重复上述流程，直到leader选出。
（2）Fast paxos：某Server首先向所有Server提议自己要成为leader，当其它Server收到提议以后，解决epoch和 zxid的冲突，并接受对方的提议，然后向对方发送接受提议完成的消息，重复这个流程，最后一定能选举出Leader。(即提议方解决其他所有epoch和 zxid的冲突,即为leader)。

- ## 同步流程
（1）Leader等待server连接；
（2）Follower连接leader，将最大的zxid发送给leader；
（3）Leader根据follower的zxid确定同步点，完成同步后通知follower 已经成为uptodate状态；
（4）Follower收到uptodate消息后，又可以重新接受client的请求进行服务了。
