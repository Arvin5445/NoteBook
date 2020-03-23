# 数据类型

## 字符串

概率计数器

hyperloglog



## 哈希



## 列表

慢查询



## 集合



## 有序集合

地理位置geo





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

aof刷盘阻塞（与上次同步时间超过2秒）

cpu竞争（cpu绑定）

bigKey

慢API（keys）

慢查询（慢日志）

内存满

内存交换（swap）

开了monitor

缓冲区满



# INFO

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

RDB AOF

预热

击穿



# 单线程

因为Redis是基于内存的操作，CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存的大小或者网络带宽。既然单线程容易实现，而且CPU不会成为瓶颈，那就顺理成章地采用单线程的方案了（毕竟采用多线程会有很多麻烦！）Redis利用队列技术将并发访问变为串行访问

1. 绝大部分请求是纯粹的内存操作（非常快速）
2. 采用单线程,避免了不必要的上下文切换和竞争条件

非阻塞IO优点：

1. 速度快，因为数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)
2. 支持丰富数据类型，支持string，list，set，sorted set，hash
3.支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行
4. 丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除如何解决redis的并发竞争key问题



# 击穿

hot key 永不过期

布隆过滤器 拦截

默认值

设置 -1，null

分布式锁 set nx expire

二级缓存

随机过期时间