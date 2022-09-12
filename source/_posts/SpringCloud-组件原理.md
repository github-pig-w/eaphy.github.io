---
title: SpringCloud 组件原理
tags:
  - eureka
  - hystrix
categories:
  - springcloud
abstract: <font color=red>ʅ（´◔౪◔）ʃ &nbsp&nbsp&nbsp  文章已加密，有缘自会相见 ！！！</font>
message: 执意要看，请输入暗号 ！
date: 2020-03-06 22:51:30
password:
---

# Eureka
## 设计原则
Eureka 采用纯 java 实现，其设计原则是AP，即可用性和分区容错性。它保证了注册中心的可用性，但舍弃了数据一致性，各节点上的数据有可能是不一致的，但是会最终一致。

<!-- more -->

## 存储结构

<img src="/img/20030601.png"  align=center/>

eureka 数据存储结构有两层，第一层是数据存储区，第二层是缓存区。

 1. 数据存储区，其实内部是一个双层的ConcurrentHashMap，第一层的key是spring.application.name，value是第二层ConcurrentHashMap；第二层ConcurrentHashMap的key是服务的InstanceId，value是Lease对象；Lease对象包含了服务详情和服务治理相关的属性。
 2. 双级缓存区，两者都用于服务信息的对外输出。

## 运作机制
服务提供者向eureka注册、更新、删除服务时，会写入、更新、删除注册表以及全部写入最近修改队列，如果二级缓存有数据，那么清空二级缓存。

服务提供者，默认每隔30s向eureka发送心跳，表明自己活着，并且更新 Lease对象的 lastUpdateTimestamp。

服务消费者从eureka获取注册信息时，首先从一级缓存获取，如果一级缓存没有数据，那么一级缓存去二级缓存拉取数据，二级缓存也没有数据的话，二级缓存去注册表拉取数据，然后二级缓存返回给一级缓存，一级缓存返回给消费者。

eureka 自身存在一个Evict Task，会定期清理无效的服务，然后清空二级缓存，二级缓存本身是一个guava缓存，有失效机制，隔一段时间二级缓存会自己失效。eureka 自身还存在一个TimerTask，定期将二级缓存数据同步到一级缓存。

服务消费者自身也有一个缓存，用来缓存拉取的注册信息，首次拉取时，是全量从eureka缓存拉取，之后靠其自身的定时任务从eureka最近修改队列增量拉取，如果增量拉取失败再全量拉取。

# Hystrix

## 基本原理
1.可以通过线程池和信号量两种方式实现资源隔离
2.command key 代表了一类 command，一般对应服务的一个接口，command group 代表了一个线程池，command group 一般就对应一个服务，一个command group 可以包含多个 command key，多个接口共享一个线程池，如果 command key 要用自己的线程池，可以定义自己的 threadpool key 。
3.线程池的大小，默认为10。
4.queueSizeRejectionThreshold 用来设置等待队列大小，默认为5，超过线程池大小之后，先进入该队列，队列满，执行拒绝策略。

## 执行流程
<img src="/img/20041401.png"  align=center/>

1.执行command的四种方式：

HystrixCommand，仅仅返回一个结果

 - execute() ；           			阻塞，同步调用
 - queue()；					直接返回一个future，异步调用
 
HystrixObservableCommand，可能返回多个结果
 - observer()；  立即执行它的construct方法，得到结果
 - toObervable() ；返回一个observable对象，不会立即执行它的construct方法，订阅这个对象才会执行，得到结果