---
title: "Transparent Application Failover"
date: 2018-01-22 12:44:57
---
[TOC]

## Concept
TAF提供遇到实例失败时，会话故障转移的功能。但
* TAF只支持select语句，DML语句只会失败且回滚
* TAF不支持jdbc

## TAF参数
TAF可通过TNS中加入failover_mode实现，且有以下几个可选参数：
```
TYPE: 'session', failover后需要重新连接
       'select', 可重新利用其他实例的open cursor，而不需要重新连接
       'none', 关闭failover

METHOD: 'Basic', 只有发生实例失败时候才重新连接到其他实例
        'preconnect', 在这个模式下，应用会事先在一个backup instance上建立一个backup session，这样就能加快failover的速度
```

