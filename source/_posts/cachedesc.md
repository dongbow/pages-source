---
title: 缓存简单介绍
date: 2019-07-19 16:31:06
categories: 缓存
tags:  
	- cache
	- redis
	- memcached
	- 缓存
---
## 缓存概念
### 缓存穿透
缓存穿透是指攻击者不断发起查询缓存和数据库中都没有的数据，导致压力全部落在数据库，导致数据库压力过大。
#### 解决方案：
- 接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
- 从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击。  

### 缓存击穿
缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。
#### 解决方案
- 设置热点数据永远不过期。
- 读数据库时加互斥锁，写入缓存，其他等待线程就可以从缓存读取  

### 缓存雪崩
缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。
#### 解决方案：
- 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
- 如果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中。
- 设置热点数据永远不过期。

## 内存缓存框架
比较项 | ConcurrentHashMap | LRUMap | Ehcache | Guava Cache | Caffeine	 
---|---|---|---|---|---
读写性能 | 很好，分段锁 | 一般，全局加锁 | 好 | 好，需要做淘汰操作 | 很好
淘汰算法 | 无 | LRU，一般 | 支持多种淘汰算法,LRU,LFU,FIFO | LRU，一般 | W-TinyLFU, 很好
功能丰富程度 | 功能比较简单 | 功能比较单一 | 功能很丰富 | 功能很丰富，支持刷新和虚引用等 | 功能和Guava Cache类似
工具大小 | jdk自带类，很小 | 基于LinkedHashMap，较小 | 很大，最新版本1.4MB | 是Guava工具类中的一个小部分，较小 | 一般，最新版本644KB
是否持久化 | 否 | 否 | 是 | 否 | 否
是否支持集群 | 否 | 否 | 是 | 否 | 否  

- 对于ConcurrentHashMap来说，比较适合缓存比较固定不变的元素，且缓存的数量较小的。虽然从上面表格中比起来有点逊色，但是其由于是jdk自带的类，在各种框架中依然有大量的使用,比如我们可以用来缓存我们反射的Method,Field等等;也可以缓存一些链接，防止其重复建立。在Caffeine中也是使用的ConcurrentHashMap来存储元素。
- 对于LRUMap来说，如果不想引入第三方包，又想使用淘汰算法淘汰数据，可以使用这个。
- 对于Ehcache来说，由于其jar包很大，较重量级。对于需要持久化和集群的一些功能的，可以选择Ehcache。笔者没怎么使用过这个缓存，如果要选择的话，可以选择分布式缓存来替代Ehcache。
- 对于Guava Cache来说，Guava这个jar包在很多Java应用程序中都有大量的引入，所以很多时候其实是直接用就好了，并且其本身是轻量级的而且功能较为丰富，在不了解Caffeine的情况下可以选择Guava Cache。  

总结一下:如果不需要淘汰算法则选择ConcurrentHashMap，如果需要淘汰算法和一些丰富的API，这里推荐选择Caffeine。
## 分布式缓存框架
这里选取比较出名的分布式缓存来作为比较，Memcached，Redis. 不同的分布式缓存功能特性和实现原理方面有很大的差异，因此他们所适应的场景也有所不同。  
 
比较项 | Memcached | Redis
---|---|---
数据结构 | 只支持简单的Key-Value结构 | String,Hash, List, Set, Sorted Set 
持久化 | 不支持 | 支持 
容量大小 | 数据纯内存，数据存储不宜过多 | 数据全内存，资源成本考量不宜超过100GB
读写性能 | 很高 | 很高(RT0.5ms左右)  

- Memcached：其吞吐量较大，但是支持的数据结构较少，并且不支持持久化。
- Redis:支持丰富的数据结构，读写性能很高，但是数据全内存，必须要考虑资源成本，支持持久化。  

总结:如果服务对延迟比较敏感，Map/Set数据也比较多的话，比较适合Redis。