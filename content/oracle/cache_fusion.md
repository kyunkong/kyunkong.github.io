---
title: "Cache fusion"
date: 2018-01-22 12:32:39
---

[TOC]
## concept
在rac中，服务器进程读取数据块过程中，为了保证数据一致性及高效性而采取的一种机制：
* 在各实例间共享由global locks协调和保护的资源
* 通过私有网络，任何一个实例都能访问到最新的master copy

如实例A想更新一个数据，发现这个块的ownership是在B下，A发送请求给GCS，B从GCS接到请求，释放属主关系，通过通过私有网络将数据块发送给A。
在以上的操作中，并没有发生磁盘读写，实例间通过缓存融合的机制能高效的访问数据。但在多事务并发中，多个实例同时访问同一数据块的情况并不少见，因此引进以下几个服务以保证数据的可靠性。

### 1. GCS Global Cache Service
主要负责实例间读写数据块的请求和共享

### 2. GES Global Enqueue Service
负责管理library locks，dictionary locks，事务锁及表锁等。

### 3. GRD Global Resource Directory
跟踪记录资源属主关系和状态

如在三节点rac中，实例A发出一个请求，GCS接收请求，通过GRD发现B是属主，再从GES发现当前最新块为实例C拥有，然后GCS通过LMS进程将对应块发送给A，并在A接收后，由GCS更新GRD的相关信息。

## background processes
### 1. LMS
负责节点间块的传输，构建CR块

### 2. LMON
管理GES中锁资源的分配

### 3. LMD
管理其他节点的远程锁请求

### 4. LCK
负责管理非缓存融合相关的锁请求，如row cache或者library cache

