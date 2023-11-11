# Redis学习笔记

Redis官网：https://redis.io/

### Redis简介

Redis：Remote dictionary server，一个开源的基于内存的数据存储系统，可用于数据库缓存和消息队列等各种场景，是目前最热门的 NoSQL 数据库之一

支持多种数据结构：字符串String、列表List、集合Set、有序集合SortedSet、哈希Hash、消息队列Stream、地理空间Geospatial、HyperLogLog、位图Bitmap、位域Bitfield

使用方式：CLI、API、GUI

### 字符串String

Redis中数据使用键值对存储，默认使用字符串存储数据，且为二进制安全

```
SET [key] [value] // 设置
GET [key] // 获取
DEL [key] / /删除
EXISTS [key] // 判断某个键是否存在
FLUSHALL // 删除所有键
KEYS * // 查看所有键
TTL [key] // 查看一个键的过期时间，-1表示未设置过期时间
EXPIRE [key] [time] // 设置过期时间
SETEX [key] [time] [value] // 设置过期时间
```

### 列表List

```
LPUSH [name] [value] // 将元素添加到列表头部
RPUSH [name] [value] // 将元素添加到列表尾部
LRANGE [key] [start] [stop] // 获取列表内容，stop为-1表示最后一个元素
LPOP [name] // 删除列表第一个元素
RPOP [name] // 删除列表最后一个元素
LTRIM [name] [start] [stop] // 删除列表指定范围以外的元素
```

### 集合Set

无序集合，元素不能重复。Redis中支持集合运算

```
SADD // 添加
SMEMBERS // 查看集合中的元素
SISMEMBER // 判断元素是否在集合中
SREM // 删除
```

### 有序集合SortedSet

有序集合的每个元素都会关联一个浮点类型的分数，按照该分数对集合中的元素进行**从小到大**排序

```
ZADD [key] [score] [member] // 添加
ZRANGE [key] [start] [end]
ZSCORE [key] [member] // 查看某个成员分数
ZRANK [key] [member] // 查看某个成员排名
```

### 哈希Hash

```
HSET [name] [key] [value]
HGET [name] [key]
HGETALL [name]
HEXISTS [name] [key] // 判断某个键值对是否存在
HKEYS
HLEN
```

### 发布订阅模式

```
SUBSCRIBE [channel] // 订阅
PUBLISH [Channel] [message] // 发布
```

### 消息队列Stream

Redis5.0中引入的新数据结构，解决发布订阅模式中**消息无法持久化、无法记录历史消息**等缺点

```
XADD [key] * [field] [value] // *表示自动生成一个消息的id
XLEN
XRANGE [key] + - // 查看所有消息
XDEL [key] [id]
XTRIM [key] MAXLEN 0 // 删除所有消息 
XREAD COUNT 2 BLOCK 1000 STREAMS [key] 0 // 从头读取两条消息，没有消息的话就阻塞1000ms再读
XREAD COUNT 2 BLOCK 1000 STREAMS [key] $ // 读取最新消息
XGROUP CREATE [key] [group] [id] // 设置消费者组
XINFO GROUPS [key] // 查看组信息
XREADGROUP GROUP [group] [consumer] COUNT [index] BLOCK [time] STREAMS [key] > // 读取最新消息
```

### 地理空间

```
GEOADD city 116.405285 39.904989 beijing
GEOPOS city beijing
GEODIST city beijing shanghai // 计算两个城市直线距离
GEOSEARCH city FROMEMBER shanghai BYRADUIS 300 KM
```

### HyperLogLog 

用于统计基数

```
PFADD [key] [value1] [value2]...
PFCOUNT [key]
PFMERGE [key] [key1] [key2] // 合并1和2
```

### 位图Bitmap

```
SETBIT [key] [offset] [0/1]
GETBIT [offset]
SET [key] [value]
BITCOUNT [key] // 统计某个key里面有多少个bit是1
```

### 位域Bitfield

```
BITFIELD player:1 set u8 #0 1
GET player:1
```

### 事务

在Redis中，不能保证事务同成功或同失败，某个任务执行失败后面的任务可能依然执行

在事务执行过程中，其他客户端提交的命令请求不会被插入到事务的执行序列中

```
MULTI // 开启事务
EXEC // 执行事务
```

### 持久化

Redis中的持久化主要有两种方式

- RDB：Redis Database，在指定时间内将内存中的数据快照写入磁盘。缺点：服务器在某次修改之后如果宕机，快照之后的所有修改内容都会丢失。所以RDB模式更适合做备份
- AOF：Append Only File，在执行写命令时，将命令写入内存的同时将命令写入到一个追加的文件中。当Redis重启之后就会重新执行AOF中的命令来重建数据库内容

### 主从复制

一般来说，主节点负责写操作，从节点负责读操作。主节点会将数据变化通过异步方式发送给从节点，从节点接收到消息后更新数据，从而实现数据一致性。可以通过修改Redis 配置文件实现

### 哨兵模式

用于实现故障自动转移。哨兵会以一个独立进程运行在Redis集群中，用来监控Redis集群中的各个节点是否运行正常，哨兵有以下功能：

- 监控：通过不断发送命令来检查Redis节点是否正常
- 通知：采用发布订阅模式通知其他节点
- 故障自动转移：将某个从节点升为主节点，再将其他从节点指向新的主节点

