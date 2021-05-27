---
layout:     post
title:      什么是mysql脏页
subtitle:   什么是mysql脏页
date:       2021-05-27
author:     锐玩道
header-img: img/bg_img/post-bg-cook.jpg
catalog:      true
theme:      smartblue
tags:
    - mysql
---

# 什么是mysql脏页

> 如果❤️我的文章有帮助，欢迎点赞、关注。这是对我继续技术创作最大的鼓励。[更多往期内容在个人博客](https://coderdao.github.io/)

## 脏页（内存页）
- 干净页：内存和磁盘中的数据一致
- 脏页：内存和磁盘中的数据不一致


## 为什么会出现 脏页
平时很快的更新操作，都是在写内存和日志。
他并`不会马上同步`到磁盘数据页,这时内存数据页跟磁盘数据页内容不一致,我们称之为`脏页`。

这里面就涉及 mysql 的内存管理机制
### 内存管理机制简述
缓冲区中包含这三大类列表。分别为：`LRUList`、`FreeList`、`FlushList`。

在数据库刚启动时，LRUlist中`没有数据页`。FreeList存放空闲页。
- 当需要读取某个页时，会从FreeList中获取一个空闲页，读入数据后，放入LRUlist中
- 如果FreeList中没有空闲页了，那么根据LRU算法淘汰Lru列表中末位的页
- 当LRUlist中的页被修改后，页就变成了脏页，这个页也会被加入FlushList中
> 注意：这时这个页既在LRUlist中，又在FlushList中。

总结：LRUList（管理已经被读取的页）和FreeList（管理空闲的页）用来管理页的可用性；FlushList（管理脏页）用来管理脏页的刷新

在脏页数据同步到磁盘过程中，如果对该磁盘数据页执行 SQL 语句。执行速度就会变慢

### 数据修改和读取只依赖缓冲区行不行
如果数据修改和读取只依赖内存的缓冲区，那么一旦数据库宕机，内存中的数据都会丢失。所以MySQL使用之前讲过的redo log来实现异常重启的数据恢复，redolog相关介绍可以看篇文章：[MySQL-redo log 和 binlog](https://www.jianshu.com/p/e441d398d14d)

简单来说，就是在更新缓冲区之前，先写入redo log，保证异常重启之后可以正常恢复缓冲区中的数据。

## 为什么脏页一定要刷新
- 上面说了 数据只放在缓冲区，会出现`数据库宕机，内存数据丢失`。所以需要刷新到磁盘。
- redo log如果无限大或者有许多个文件的话，系统中有大量的修改操作，一旦宕机，恢复的时间也会非常长。

所以自然而然，我们就一定**需要把内存中的脏页按照某种规则刷新到磁盘**中，有了刷新这个操作，缓冲区的大小问题和redo log的大小问题都可以解决。

- 缓冲区不需要无限大了，因为可以持久化到磁盘
- redo log也不需要无限大了，因为一旦持久化到磁盘，redo log中对应的那部分数据就可以释放。

## 刷脏页有下面4种场景

1. 当 `redo log 写满`，mysql就会`暂停所有更新`操作，将同步这部分日志对应的`脏页同步到磁盘`。

2. 系统`内存不足`时，需要`淘汰`一部分数据页，如果淘汰的是`脏页`，就要`先将脏页同步到磁盘`。

3. MySQL 认为系统`空闲`的时候,有机会就`同步`内存数据到磁盘，这种没有性能问题。

4. `MySQL 正常关闭`，MySQL 会把`内存的脏页都同步到磁盘`上，这样下次 MySQL 启动的时候，就可以直接从磁盘上读数据，启动速度会很快。这种没有性能问题。

### 会造成的影响

**1 如果是 redo log 写满了**

要尽量避免`redo log 写满`。否则整个系统的更新都会停止。`此时写的性能变为 0`，必须等待该日志对应`脏页同步完成`后才能更新,这时就会导致 sql 语句 执行的很慢。

## 如何刷新呢？
MySQL中，刷新的规则叫checkpoint机制。

在InnoDB存储引擎中，有两种checkpoint：

- sharp checkpoint：在数据库关闭时，刷新所有的脏页到磁盘，这里有参数控制，默认是开启的
- fuzzy checkpoint：刷新一部分脏页到磁盘中。
### fuzzy checkpoint
关于Fuzzy checkpoint，InnoDB存储引擎中可能包括如下几种：
- 定时刷新 - master thread做
- Flush LRUlist checkpoint -page cleaner thread做
- async/sync checkpoint -page cleaner thread做
- dirty too much checkpoint -？谁做？

刷新机制用更通俗易懂的角度来分析，上面的四种类型可以对应下面的
- 无论如何，定时刷新
- 当LRU中列表中空闲页不足时，强制LRU删除一些末尾的页，如果存在脏页，那么需要checkpoint刷新
- 使用innodb_lru_scan_depth来控制最少空闲页的数量
- 当重做日志不够用时，从flush 列表中选择一些页，强制checkpoint刷新
    - 重做日志有两个水位：async水位 75% * innodb的总大小；sync水位：90* innodb大小。
    - 当未刷新的数据大小 小于 低水位，不需要刷新
    - 当未刷新的数据大小 大于 低水位，小于高水位，异步刷新，保证刷新后小于 低水位
    - 当未刷新的数据大小 大于 高水位，同步阻塞刷新，保证刷新后小于 低水位。
- 关注系统中的整体脏页比例，如果达到一定比例，强制刷新
    - 使用 innodb_dirty_page_pct来控制这个比例数值，默认时75%

### master thread中的定时刷新机制
#### InndoDB1.0.x版本之前的master thread。
每秒，会进行一次 dirty too much checkpoint。
每10秒
- 判断过去10秒的IO操作是否小于200次，如果是，刷100个脏页；
- 判断系统当前脏页比例，如果超过70%，刷新100个；如果小于70%，刷新脏页的10%
#### InndoDB1.2.x版本之前的master thread。
在1.0.x存在硬编码，每秒最多只会刷新100个脏页到磁盘中，这种规定其实限制了性能更高的SSD磁盘。

在1.0.x版本，可以使用innodb_io_capacity来表示磁盘io的吞吐量。刷新脏页的数量由innodb_io_capacity来控制，默认是200。


## 总结
了解脏页刷新机制以及相应的参数是很有必要的，当数据库系统某些性能问题时，要考虑是否是脏页刷新相关的配置参数不合理导致的。

根据实际业务，考虑缓冲区的大小，redo log的大小，最少空闲页，脏页比例，io吞吐量相关参数是否配置合理，根据优化相关参数，解决系统问题。
