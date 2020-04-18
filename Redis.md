# 数据类型

## 字符串

（概率计数器	hyperloglog）

+ int：8字节整型
+ embstr：不长于39
+ raw：长于39



## 哈希

+ ziplist（压缩列表）：短于 `hash-max-ziplist-entries` （512），且每个元素都小于 `hash-max-ziplist-value` （64字节）
+ hashtable（哈希表）：上面的条件无法满足时



## 列表

（慢查询）

+ ziplist（压缩列表）：短于 `list-max-ziplist-entries` （512），且每个元素都小于 `list-max-ziplist-value` （64字节）
+ linkedlist（链表）：上面的条件无法满足时



## 集合

+ intset（整数集合）：所有元素都是整数，且短于 `set-max-intset-entries` （512）

+ hashtable（哈希表）：上面的条件无法满足时



## 有序集合

（地理位置geo）

+ ziplist（压缩列表）：短于 `zset-max-ziplist-entries` （128），且每个元素都小于 `zset-max-ziplist-value` （64字节）
+ skiplist（跳跃表）：上面的条件无法满足时



# 缓冲区

## 输入缓冲区

临时保存命令

每个客户端一个，最大1G，不可更改，且不受 `maxmemory` 控制

所以缓冲区占用过大可能会超出 `maxmemory` ，导致数据丢失，键值淘汰，OOM



## 输出缓冲区

临时保存命令执行结果

分为固定（byte[]，返回小结果）和动态（list，返回大结果），并存

1. 普通客户端输出缓冲区
2. 发布订阅输出缓冲区
3. 主从复制输出缓冲区





# 淘汰策略

惰性删除 定时任务删除

1. volatile-lru：从设置过期时间的数据集（server.db[i].expires）中挑选出最近最少使用的数据淘汰。没有设置过期时间的key不会被淘汰，这样就可以在增加内存空间的同时保证需要持久化的数据不会丢失。

2. volatile-ttl：除了淘汰机制采用LRU，策略基本上与volatile-lru相似，从设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰，ttl值越大越优先被淘汰。

3. volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰。当内存达到限制无法写入非过期时间的数据集时，可以通过该淘汰策略在主键空间中随机移除某个key。

4. allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰，该策略要淘汰的key面向的是全体key集合，而非过期的key集合。

5. allkeys-random：从数据集(server.db[i].dict）中选择任意数据淘汰。

6. no-enviction：禁止驱逐数据，也就是当内存不足以容纳新入数据时，新写入操作就会报错，请求可以继续进行，线上任务也不能持续进行，采用no-enviction策略可以保证数据不被丢失，这也是系统默认的一种淘汰策略。



上述是Redis的6种淘汰策略，关于使用这6种策略，开发者还需要根据自身系统特征，正确选择或修改驱逐。

1. 在Redis中，数据有一部分访问频率较高，其余部分访问频率较低，或者无法预测数据的使用频率时，设置allkeys-lru是比较合适的。

2. 如果所有数据访问概率大致相等时，可以选择allkeys-random。

3. 如果研发者需要通过设置不同的ttl来判断数据过期的先后顺序，此时可以选择volatile-ttl策略。

4. 如果希望一些数据能长期被保存，而一些数据可以被淘汰掉时，选择volatile-lru或volatile-random都是比较不错的。

5. 由于设置expire会消耗额外的内存，如果计划避免Redis内存在此项上的浪费，可以选用allkeys-lru 策略，这样就可以不再设置过期时间，高效利用内存了。





# 阻塞

持久化fork阻塞

持久化io阻塞

aof刷盘阻塞（与上次同步时间超过2秒）只要部分从节点开启持久化功能就行了，不要以及备份



cpu竞争（cpu绑定）

IO

网络



主从复制（不要一起全量复制，低峰）

宕机引起的哨兵选举下线，复制



bigKey `redis-cli --bigkeys`

慢API（keys）

慢查询（慢日志）

内存满

内存交换（swap）

开了monitor

缓冲区满



## 规避全量复制

slave of 在低峰进行

保持runid匹配（可以通过哨兵保证）

保证积压缓冲区充足



## 规避复制风暴

用2叉树代替n叉树，但是运维变复杂





# INFO命令

server 服务

clients 客户端

memory 内存

persistence 持久化

stats 统计（连接 命令 网络 过期 同步）

replication 主从复制

cpu 进程cpu小号

commandstats 命令统计

cluster 集群

keyspace 键值统计



# 持久化

## RDB 

1. bgsave，已经有子进程则直接返回
2. fork出子进程（会阻塞）
3. bgsave返回，结束阻塞
4. 子进程创建rdb文件，根据父进程生成临时快照文件，完成后对原文件进行**原子**替换
5. 通知父进程已经完成，父进程更新统计信息

优点：

+ 压缩文件
+ 恢复数据快

缺点：

+ 非实时的，会有数据丢失
+ 因压缩格式而老旧不兼容

## AOF

+ 所有写入命令会追加到 `aof_buf` aof缓冲区
+ aof缓冲区根据配置的策略（aways，everysec，no）向硬盘做同步操作（write -> fsync）（2s刷盘阻塞）
+ aof文件太大会触发重写机制

### 重写

重写期间aof不受影响（旧的aof文件大小一直在增加），可以手动或者自动触发

1. 请求重写，如果正在重写则直接返回
2. fork出子进程（会阻塞）
3. COW，父子只共享此时的内存快照，所以新的写入命令会暂存到aof重写缓冲区
4. 子进程合并规则，写到新的aof文件（分批，防止IO阻塞）
5. 写完后子通知父，父更新统计信息
6. 父把aof重写缓冲区内的新命令写入到新的aof文件中
7. 新aof文件替换旧的aof文件



+ 文件较大
+ 期间消耗性能（写命令，磁盘同步，偶尔还要重写）
+ 数据比较完整



# 单线程

​		因为Redis是基于内存的操作，CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存的大小或者网络带宽。既然单线程容易实现，而且CPU不会成为瓶颈，那就顺理成章地采用单线程的方案了（毕竟采用多线程会有很多麻烦！）Redis利用队列技术将并发访问变为串行访问

1. 绝大部分请求是纯粹的内存操作（非常快速）
2. 采用单线程,避免了不必要的上下文切换和竞争条件
3. NIO，IO多路复用

非阻塞IO优点：

1. 速度快，因为数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)
2. 支持丰富数据类型，支持string，list，set，sorted set，hash
3.支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行
4. 丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除如何解决redis的并发竞争key问题



# 缓存击穿

hot key 永不过期

布隆过滤器 拦截

默认值（加载中。。。）

设置 set -1 null（给不存在的值设置缓存）

互斥锁：分布式锁 set nx expire

```
从 Redis 2.6.12 版本开始， SET 命令的行为可以通过一系列参数来修改：

EX second ：设置键的过期时间为 second 秒。 SET key value EX second 效果等同于 SETEX key second value 。
PX millisecond ：设置键的过期时间为 millisecond 毫秒。 SET key value PX millisecond 效果等同于 PSETEX key millisecondvalue 。
NX ：只在键不存在时，才对键进行设置操作。 SET key value NX 效果等同于 SETNX key value 。
XX ：只在键已经存在时，才对键进行设置操作。
```



二级（多级）缓存

随机过期时间



# 主从复制（Replication）

`slave of`

`slave of no one` 不会抛弃原有数据

`debug reload` 不会更改runid（40位十六进制）

复制积压缓冲区（1MB）



1. 保存主节点信息
2. 建立socket连接，直到成功
3. ping，超时断开
4. 权限验证，密码
5. 同步数据集，全量复制
6. 命令持续复制（异步）



？ -1 //全量

## 全量复制

1. bgsave
2. 网络传输rdb文件（）
3. 清空旧数据
4. 加载rdb
5. 立刻aof（如果设置了的话）



## 部分复制

1. 如果超出repl-timeout，则会断开
2. 恢复后从->主 psync {runid} {offset} //部分
3. 主收到后，核对runid
4. 如果复制积压缓冲区里有，则发送，否则退化为全量复制

### 心跳

主->从 10秒一ping

从->主 1秒一replconf ack {offset}

+ 实时监测主从节点网络状态
+ 上报偏移量
+ 记录统计延迟性

### 1. 读写分离

### 2. 延迟

zookeeper监听

遇到高延迟就切换节点连接，自动路由



### 3. 独到过期数据

从节点只会被动删除

+ 惰性删除

  读到僵尸时，删除僵尸

+ 定时任务删除（慢模式超时25ms，快模式超时1ms且2s内只能运行1次）

  每秒10次的任务

  1. 每个数据库随机查20个key
  2. 删除其中的僵尸（同时把删除命令同步给从节点），如果僵尸超过25%则重复，直到不足25%或超时
  3. 如果超时，则在redis触发内部事件之前再开始快任务





# 哨兵（Sentinel）

**故障转移**

config里加 sentinel monitor

`redis-sentinel` / `redis-server --sentinel`

主节点下线 `sentinel failover {master name}`

+ 每个se定期（1秒）向数据节点和其他se发ping `down-after-milliseconds`

检查可达性，节点失败判定的重要依据

+ se每过10秒向M和S发info，获取最新拓扑结构

所有se不用配置S信息，有S新加入或发生故障，se都能感知到，更新拓扑信息

+ se每过2秒向数据节点的 `_sentinel_:hello` 频道发送信息，同时订阅这个频道

通过这个，se之间可以交换意见，并能够感知新加入的se，客观下线和领导选举的依据

### 主观下线

ping/1秒超过 `down-after-milliseconds` ，一家之言，误判

### 客观下线

如果se发现主观下线的是M，se会通过 `sentinel is-master-down-by-addr` 向其他se交换意见，同意者超过 `quorum` ，则客观下线

### 领导选举

选举活动至少需要`max(quorum, se_num / 2 + 1)` ，raft算法 P259

### 故障转移

1. 领导在S中找一个合适的新M

+ 过滤不健康：5秒内没回复ping，与M失联超过 `down-after-milliseconds`
+ `slave-priority` 中选最高的，返回，不存在则下一步
+ offset最大（复制最完整），不存在则下一步
+ runid最小

2. 领导对新M执行 `slaveof no one`
3. 领导让其他S认新主（新M）， `parallel-syncs`
4. se集合将旧M更新为S，等它恢复健康后，命令它去复制新M





# 集群（Cluster）

```
cluster-enabled yes
cluster-node-timeout 15000
cluster-config-file "node-6379.conf" //如果不配会自动生成，不需要手动修改，内有节点ID，40位十六进制，保存在文件中，重启不会丢失更改
```



哈希

**==槽 `slot` （0 ～ 16383）==**

+ 解耦数据和节点之间的关系，简化扩容和收缩
+ 节点自身维护槽的映射关系，无需客户端或第三方代理服务操心
+ 支持键（key）、槽（slot）、节点（node）的映射查询，用于数据路由和在线伸缩

### 功能限制（缺点）

+ mset，mget只支持相同slot的key（可以用{}大括号解决）
+ 涉及不同节点，无法使用事务
+ 无法将bigkey，比如hash、list分配到不同节点
+ pipeline也受限制（可以用{}大括号解决）
+ 发布订阅引发大量网络IO
+ 分成多次请求造成大量网络IO

### 节点握手，通信

**Gossip（流言）协议**（频繁交换信息，信息扩散），感知对方的过程，单独TCP通道（基础端口 + 1000）

定时任务选举出本次联系人（每秒10次）

**接收到消息后：**新节点 ? meet : 更新该节点状态到本地; 回复pong消息;

+ meet
+ ping
+ pong
+ fail

**消息头**（clusterMsg，包含分区槽，复制偏移，消息类型，nodeId，端口等）+ **消息体**（clusterMsgData，包含自身信息和十分之一的其他节点信息（集群不能过大））

异步命令：`cluster meet {ip} {port}`

`cluser replicate {nodeId}`

对方用pong来相应meet

之后双方用ping pong进行正常的节点通信

握手状态会通过消息在集群内传播，其他节点会自动发现新加入的节点，并发起握手流程

### 分配槽

**==只有当16383个槽全部分配给节点后，集群才进入在线状态 fail->ok==**

否则任何键命令都会返回错误 `(error) CLUSTERDOWN Hash slot not served`

`cluster addslots {0..5416}` 分配槽

`cluster keyslot {key}` 返回key所对应的槽

### 伸缩

1. 运行新节点（集群模式）
2. 加入集群（meet）
3. 迁移槽（setslot importing migrating migrate）

下线的节点应该被剩余节点忘记

`cluster forget {downNodeId}`

`cluster node` 中不再包含 {downNodeId}

### 请求路由

收到任何key相关命令，首先计算对应槽（slot），再根据槽找出对应节点

如果节点为自身，则自行处理

否则回复 `(error) MOVE {ip} {slot}` 重定向错误，通知客户端请求正确的节点

`redis-cli -p 6379 -c`     -c可以让客户端支持自动重定向（此功能在redis-cli内部维护，而非redis数据节点）



### 故障转移

**主观下线：** 流言ping pong会记录最近通讯时间，当超过 `cluster-node-time-out`，标记此节点为主观下线

**客观下线：** 主观下线消息在集群内留言传播（每个节点有张表维护），当大半**主节点**收到此消息时，触发尝试客观下线流程

1. 统计认为此节点挂了的节点数量（通过表）
2. 若大于一般，则确认客观下线
3. 广播 `fail {nodeId}` ，通知所有节点客观下线，立即生效，通知下线节点的从节点启动故障转移

**故障转移：**

1. 资格检查：排除掉不合格的（与主短线超过阈值）
2. 准备选举时间：（延迟选举，通过不同的延迟时间来解决优先级问题（offset））
3. 选举：更新配置纪元，广播选举消息（超时投票会作废，重新投票）
4. 选举投票：主节点才能投（cluster的新M选举=se的领导者选举，从数量无需>=3）
   1. 替换主节点：从节点取消复制（slave of）变为主节点，`clusterDelSlot; clusterAddSlot` ，向集群广播 `pong` 来通知
5. 统计耗时



### 故障转移失败：

+ 主从同时故障
+ 所有从都不合格
+ 网络问题导致（投票一直超时作废）
+ 一半以上主节点挂了



# 端口：6379