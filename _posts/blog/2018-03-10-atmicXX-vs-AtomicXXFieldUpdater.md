---
layout: post
title: AtomicXX VS. AtomicXXFieldUpdater
categories: Java
description: java里面的bridge method
keywords: bridge method, 桥方法
---

看Netty源代码或者Druid连接池代码发现有很多地方都使用到了AtomicLongFieldUpdater这个类，如Druid里面的JdbcSqlStat，Netty里面的DefaultChannelHandlerContext。

先看下面两段代码：

DefaultChannelHandlerContext部分代码
```
    private static final AtomicIntegerFieldUpdater<AbstractChannelHandlerContext> HANDLER_STATE_UPDATER =
           AtomicIntegerFieldUpdater.newUpdater(AbstractChannelHandlerContext.class, "handlerState");

    private static final int INIT = 0;

    private volatile int handlerState = INIT;

```

JdbcSqlStat类部分代码

```
    private volatile long                               executeBatchSizeTotal;
    private volatile int                                executeBatchSizeMax;

    private volatile long                               executeSuccessCount;
    private volatile long                               executeSpanNanoTotal;
    private volatile long                               executeSpanNanoMax;
    private volatile int                                runningCount;
    private volatile int                                concurrentMax;
    private volatile long                               resultSetHoldTimeNano;
    private volatile long                               executeAndResultSetHoldTime;

    final static AtomicLongFieldUpdater<JdbcSqlStat>    executeBatchSizeTotalUpdater                    = AtomicLongFieldUpdater.newUpdater(JdbcSqlStat.class,
                                                                                                                                            "executeBatchSizeTotal");
    final static AtomicIntegerFieldUpdater<JdbcSqlStat> executeBatchSizeMaxUpdater                      = AtomicIntegerFieldUpdater.newUpdater(JdbcSqlStat.class,
                                                                                                                                               "executeBatchSizeMax");

    final static AtomicLongFieldUpdater<JdbcSqlStat>    executeSuccessCountUpdater                      = AtomicLongFieldUpdater.newUpdater(JdbcSqlStat.class,
                                                                                                                                            "executeSuccessCount");
    final static AtomicLongFieldUpdater<JdbcSqlStat>    executeSpanNanoTotalUpdater                     = AtomicLongFieldUpdater.newUpdater(JdbcSqlStat.class,
                                                                                                                                            "executeSpanNanoTotal");
    final static AtomicLongFieldUpdater<JdbcSqlStat>    executeSpanNanoMaxUpdater                       = AtomicLongFieldUpdater.newUpdater(JdbcSqlStat.class,
                                                                                                                                            "executeSpanNanoMax");
    final static AtomicIntegerFieldUpdater<JdbcSqlStat> runningCountUpdater                             = AtomicIntegerFieldUpdater.newUpdater(JdbcSqlStat.class,
                                                                                                                                               "runningCount");
    final static AtomicIntegerFieldUpdater<JdbcSqlStat> concurrentMaxUpdater                            = AtomicIntegerFieldUpdater.newUpdater(JdbcSqlStat.class,
                                                                                                                                               "concurrentMax");
    final static AtomicLongFieldUpdater<JdbcSqlStat>    resultSetHoldTimeNanoUpdater                    = AtomicLongFieldUpdater.newUpdater(JdbcSqlStat.class,
                                                                                                                                            "resultSetHoldTimeNano");
    final static AtomicLongFieldUpdater<JdbcSqlStat>    executeAndResultSetHoldTimeUpdater              = AtomicLongFieldUpdater.newUpdater(JdbcSqlStat.class,
                                                                                                                                            "executeAndResultSetHoldTime");

```

对于AtomicXXFieldUpdater这种类的作用及用法不多做解释。就是为了实现对volatile字段原子操作，那么实现原子操作我们为什么不选择AtomicInteger、AtomicLong、AtomicReference这种更好理解的类呢？

## 主要使用场景
大家考虑下在Netty这种系统里面会有大量的并发连接，而每一个并发连接都需要创建一个Channel,每一个Channel对应一个 ChannelPipeline，而一个ChannelPipeline对应由多个ChannelHandlerContext组成的链。这个时候就会有大量的ChannelHandlerContext实例存在，并且是长时间存活着的对象。这个时候使用AtomicXXFieldUpdater就比AtomicXX合适了。

## 为什么
因为这里多创建了一个AtomicXX对象，而在64位平台上每创建一个对象至少要花费16个Byte，如果这个数量特别大内存消耗可不小。
