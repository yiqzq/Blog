# Redis学习笔记

[toc]

## 命名规范

1. redis 的key 区分大小写
2. 建议使用：分割，用来命名字段，比如 user:101:zhangsan

## redis的常用命令

### DEL

用法 ：**DEL ke  [key …]**

删除给定的一个或多个 `key` 。不存在的 `key` 会被忽略。

### **EXISTS**

用法 ：**EXISTS key**

检查给定 `key` 是否存在

## EXPIRE

用法：**EXPIRE key seconds**

为给定 `key` 设置生存时间，当 `key` 过期时(生存时间为 `0` )，它会被自动删除

## KEYS

用法：**KEYS pattern**

查找所有符合给定模式 `pattern` 的 `key` 。

`KEYS *` 匹配数据库中所有 `key` 。

`KEYS h?llo` 匹配 `hello` ， `hallo` 和 `hxllo` 等。

`KEYS h*llo` 匹配 `hllo` 和 `heeeeello` 等。

`KEYS h[ae]llo` 匹配 `hello` 和 `hallo` ，但不匹配 `hillo` 。

特殊符号用 `\` 隔开

## TTL

用法：**TTL key**

以秒为单位，返回给定 `key` 的剩余生存时间(TTL, time to live)。

## TYPE

用法：**TYPE key**

返回 `key` 所储存的值的类型。

**返回值：**

`none` (key不存在)

`string` (字符串)

`list` (列表)

`set` (集合)

`zset` (有序集)

`hash` (哈希表)

## String类型

### 赋值语法

**SET KEY_NAME VALUE**
Redis SET 命令用于设置给定 key 的值。如果 key 已经存储值， SET 就覆写旧值，且无视类型

**SETNX key value** 

解决分布式锁 方案之一
只有在 key 不存在时设置 key 的值。Setnx（SET if Not eXists） 命令在指定的 key 不存在时，为 key 设置指定的值

**MSET key value [key value …]**
同时设置一个或多个 key-value 对

**SETEX key seconds value**

将值 `value` 关联到 `key` ，并将 `key` 的生存时间设为 `seconds` (以秒为单位)。

如果 `key` 已经存在， SETEX 命令将覆写旧值。

SEXEX是一个原子操作

### 取值语法

**GET KEY_NAME**
Redis GET命令用于获取指定 key 的值。如果 key 不存在，返回 nil 。如果key 储存的值不是字符串类型，返回一个错误。

**GETRANGE key start end**
用于获取存储在指定 key 中字符串的子字符串。字符串的截取范围由 start 和 end 两个偏移量决定(包括 start 和 end 在内)

**MGET key1 [key2…]**
获取所有(一个或多个)给定 key 的值

**STRLEN key**
返回 key 所储存的字符串值的长度

**GETSET KEY_NAME VALUE**
Getset 命令用于设置指定 key 的值，并返回 key 的旧值,当 key 不存在时，返回 nil

### 自增/自减

**INCR KEY_Name**
Incr 命令将 key 中储存的数字值增1。如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作
**INCRBY KEY_Name** 增量值
Incrby 命令将 key 中储存的数字加上指定的增量值
**DECR KEY_NAME** 或 **DECYBY KEY_NAME** 减值
decR 命令将 key 中储存的数字减1

## Hash类型

Redis `hash` 是一个`string`类型的`field`和`value`的映射表，hash特别适合用于存储对象。

hash类型类似于Java中对象的概念，允许有多个字段

### 赋值语法

`HSET KEY FIELD VALUE` 

为指定的KEY，设定FILD/VALUE
`HMSET KEY FIELD VALUE [FIELD1,VALUE1]……` 

同时将多个 field-value (域-值)对设置到哈希表 key 中

### 取值语法

`HGET KEY FIELD` 

获取存储在HASH中的值，根据FIELD得到VALUE
`HMGET key field[field1]` 

获取key所有给定字段的值
`HGETALL key `

返回HASH表中所有的字段和值

### 删除语法

`HDEL KEY field1[field2]` 

删除一个或多个HASH表字段

### 其它语法

`HSETNX key field value`
只有在字段 field 不存在时，设置哈希表字段的值

`HINCRBY key field increment`
为哈希表 key 中的指定字段的整数值加上增量 increment 。

`HINCRBYFLOAT key field increment`
为哈希表 key 中的指定字段的浮点数值加上增量 increment 。

`HEXISTS key field` 

查看哈希表 key 中，指定的字段是否存在

## List类型

### 赋值语法

`LPUSH key value1 [value2]` 

将一个或多个值插入到列表头部(从左侧添加)
`RPUSH key value1 [value2]` 

在列表中添加一个或多个值(从右侧添加)
`LPUSHX key value` 

将一个值插入到已存在的列表头部。如果列表不在，操作无效
`RPUSHX key value` 

一个值插入已存在的列表尾部(最右边)。如果列表不在，操作无效。

### 取值语法

`LLEN key` 

获取列表长度
`LINDEX key index` 

通过索引获取列表中的元素
`LRANGE key start stop` 

获取列表指定范围内的元素

描述： 返回列表中指定区间内的元素，区间以偏移量 `START` 和 `END` 指定。 其中 0 表示列表的第一个元素， 1 表示列表的第二个元素，以此类推。也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。
`start:` 页大小*(页数-1)
`stop :` (页大小*页数)-1

### 删除语法

`LPOP key` 

移出并获取列表的第一个元素(从左侧删除)
`RPOP key` 

移除列表的最后一个元素，返回值为移除的元素(从右侧删除)

`BLPOP key1 [key2 ] timeout`
移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
实例
`redis 127.0.0.1:6379> BLPOP list1 100`
在以上实例中，操作会被阻塞，如果指定的列表 key list1 存在数据则会返回第一个元素，否则在等待100秒后会返回 nil 。

`BRPOP key1 [key2 ] timeout`
移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

`LTRIM key start stop`

 对一个列表进行修剪(`trim`)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。

### 修改语法

`LSET key index value` 

通过索引设置列表元素的值
`LINSERT key BEFORE|AFTER world value` 

在列表的元素前或者后插入元素
描述：将值 `value` 插入到列表 `key` 当中，位于值 `world` 之前或之后。

如果有多个相同的值，取第一个

### 高级语法

`RPOPLPUSH source destination`
移除列表的最后一个元素，并将该元素添加到另一个列表并返回
示例描述：
`RPOPLPUSH a1 a2` //a1的最后元素移到a2的左侧
`RPOPLPUSH a1 a1` //循环列表，将最后元素移到最左侧

`BRPOPLPUSH source destination timeout`
从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

## Set类型

### 赋值语法

`SADD key member1 [member2]` 向集合添加一个或多个成员

### 取值语法

`SCARD key` 

获取集合的成员数
`SMEMBERS key` 

返回集合中的所有成员
`SISMEMBER key member` 

判断 `member` 元素是否是集合 `key` 的成员(开发中：验证是否存在判断）
`SRANDMEMBER key [count]` 

返回集合中一个或多个随机数

### 删除语法

`SREM key member1 [member2]` 

移除集合中一个或多个成员
`SPOP key [count]` 

移除并返回集合中的一个随机元素
`SMOVE source destination member`
将 `member` 元素从 `source` 集合移动到 `destination` 集合

### 差集语法

`SDIFF key1 [key2]` 

返回给定所有集合的差集(左侧）
`SDIFFSTORE destination key1 [key2]` 

返回给定所有集合的差集并存储在 `destination` 中

### 交集语法

`SINTER key1 [key2]` 

返回给定所有集合的交集(共有数据）
`SINTERSTORE destination key1 [key2]` 

返回给定所有集合的交集并存储在 `destination` 中

### 并集语法

`SUNION key1 [key2]` 

返回所有给定集合的并集
`SUNIONSTORE destination key1 [key2]` 

所有给定集合的并集存储在 destination 集合中

### ZSET类型

**赋值语法**
`ZADD key score1 member1 [score2 member2]`
向有序集合添加一个或多个成员，或者更新已存在成员的分数

**取值语法**
`ZCARD key` 

获取有序集合的成员数
`ZCOUNT key min max` 

计算在有序集合中指定区间分数的成员数
`ZRANK key member` 

返回有序集合中指定成员的索引
`ZRANGE key start stop [WITHSCORES]`
通过索引区间返回有序集合成指定区间内的成员(低到高)
`ZREVRANGE key start stop [WITHSCORES]`
返回有序集中指定区间内的成员，通过索引，分数从高到底

**删除语法**
`del key` 

移除集合
`ZREM key member [member ...]` 

移除有序集合中的一个或多个成员
`ZREMRANGEBYRANK key start stop` 

移除有序集合中给定的排名区间的所有成员(第一名是0)(低到高排序)
`ZREMRANGEBYSCORE key min max` 

移除有序集合中给定的分数区间的所有成员

## HyperLogLog

HyperLogLog是Redis的高级数据结构，是统计基数的利器。

`PFADD key element [element …]`

将任意数量的元素添加到指定的 HyperLogLog 里面。

`PFCOUNT key [key …]`

当 `PFCOUNT]`命令作用于单个键时， 返回储存在给定键的 HyperLogLog 的近似基数， 如果键不存在， 那么返回 `0` 。

`PFMERGE destkey sourcekey [sourcekey …]`

将多个 HyperLogLog 合并（merge）为一个 HyperLogLog ， 合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合（observed set）的并集。

## Redis 的订阅

**订阅频道**
`SUBSCRIBE channel [channel ...]`

订阅给定的一个或多个频道的信息
`PSUBSCRIBE pattern [pattern ...]`

订阅一个或多个符合给定模式的频道。
**发布频道**
`PUBLISH channel message` 

将信息发送到指定的频道。
**退订频道**
`UNSUBSCRIBE [channel [channel ...]]` 

指退订给定的频道。
`PUNSUBSCRIBE [pattern [pattern ...]]`

退订所有给定模式的频道。

## Redis 的数据库

**FLUSHALL**

清空整个 Redis 服务器的数据(删除所有数据库的所有 key )。

**FLUSHDB**

清空当前数据库中的所有 key。

此命令从不失败。

**SELECT INDEX**

用于切换到第index个数据库。

## Redis的事务

1. Redis 事务可以一次执行多个命令（允许在一次单独的步骤中执行一组命令），并且带有以下两个重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

**Redis 不支持回滚（roll back）**

`DISCARD`
取消事务，放弃执行事务块内的所有命令。
`EXEC`
执行所有事务块内的命令。
`MULTI`
标记一个事务块的开始。
`UNWATCH`
Redis `Unwatch` 命令用于取消 `WATCH` 命令对所有 `key` 的监视。
如果在执行`WATCH`命令之后，`EXEC`命令或`DISCARD`命令先被执行的话，那就不需要再执行`UNWATCH`了

`WATCH key [key ...]`
监视一个(或多个) `key` ，如果在事务执行之前这个(或这些) `key` 被其他命令所改动，那么事务将被打断。

## Redis数据淘汰策略

`redis` 提供**6**种数据淘汰策略：

- **volatile-lru**：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
- **volatile-ttl**：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
- **volatile-random**：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
- **allkeys-lru**：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
- **allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰
- **no-enviction**（驱逐）：禁止驱逐数据

## Redis 的持久化

Redis有两种持久化的方式：快照（`RDB`文件）和追加式文件（`AOF`文件）

### RDB

**RDB持久化方式会在一个特定的间隔保存那个时间点的一个数据快照。**

**工作原理**

- Redis调用fork()，产生一个子进程。
- 子进程把数据写到一个临时的RDB文件。
- 当子进程写完新的RDB文件后，把旧的RDB文件替换掉。

**优点**

- RDB文件是一个很简洁的单文件，它保存了某个时间点的Redis数据，很适合用于做备份。你可以设定一个时间点对RDB文件进行归档，这样就能在需要的时候很轻易的把数据恢复到不同的版本。
- 基于上面所描述的特性，RDB很适合用于灾备。单文件很方便就能传输到远程的服务器上。
- RDB的性能很好，需要进行持久化时，主进程会fork一个子进程出来，然后把持久化的工作交给子进程，自己不会有相关的I/O操作。

**缺点**

- RDB容易造成数据的丢失。假设每5分钟保存一次快照，如果Redis因为某些原因不能正常工作，那么从上次产生快照到Redis出现问题这段时间的数据就会丢失了。
- RDB使用`fork()`产生子进程进行数据的持久化，如果数据比较大的话可能就会花费点时间，造成Redis停止服务几毫秒。如果数据量很大且CPU性能不是很好的时候，停止服务的时间甚至会到1秒。

### AOF

**每当Redis接受到会修改数据集的命令时，就会把命令追加到AOF文件里，当你重启Redis时，AOF里的命令会被重新执行一次，重建数据。**

**优点**

- 比RDB可靠。你可以制定不同的fsync策略：不进行fsync、每秒fsync一次和每次查询进行fsync。默认是每秒fsync一次。这意味着你最多丢失一秒钟的数据。
- AOF日志文件是一个纯追加的文件。就算是遇到突然停电的情况，也不会出现日志的定位或者损坏问题。甚至如果因为某些原因（例如磁盘满了）命令只写了一半到日志文件里，我们也可以用`redis-check-aof`这个工具很简单的进行修复。
- 当AOF文件太大时，Redis会自动在后台进行重写。重写很安全，因为重写是在一个新的文件上进行，同时Redis会继续往旧的文件追加数据。新文件上会写入能重建当前数据集的最小操作命令的集合。当新文件重写完，Redis会把新旧文件进行切换，然后开始把数据写到新文件上。
- AOF把操作命令以简单易懂的格式一条接一条的保存在文件里，很容易导出来用于恢复数据。例如我们不小心用`FLUSHALL`命令把所有数据刷掉了，只要文件没有被重写，我们可以把服务停掉，把最后那条命令删掉，然后重启服务，这样就能把被刷掉的数据恢复回来。

 **缺点**

- 在相同的数据集下，AOF文件的大小一般会比RDB文件大。
- 在某些fsync策略下，AOF的速度会比RDB慢。通常fsync设置为每秒一次就能获得比较高的性能，而在禁止fsync的情况下速度可以达到RDB的水平。
- 在过去曾经发现一些很罕见的BUG导致使用AOF重建的数据跟原数据不一致的问题。



## 缓存问题

### 缓存穿透

请求去查询一条压根儿数据库中根本就不存在的数据，也就是缓存和数据库都查询不到这条数据，但是请求每次都会打到数据库上面去。这种查询不存在数据的现象我们称为**缓存穿透**。

**穿透带来的问题**

如果有黑客会对你的系统进行攻击，拿一个不存在的id 去查询数据，会产生大量的请求到数据库去查询。可能会导致你的数据库由于压力过大而宕掉。

**解决方案**

1. 缓存空值

   之所以会发生穿透，就是因为缓存中没有存储这些空数据的key。从而导致每次查询都到数据库去了。那么我们就可以为这些key对应的值设置为null 丢到缓存里面去。后面再出现查询这个key 的请求的时候，直接返回null 。

2. 布隆过滤器

   这种方案可以加在第一种方案中，在缓存之前在加一层 BloomFilter ，在查询的时候先去 BloomFilter 去查询 key 是否存在，如果不存在就直接返回，存在再走查缓存 -> 查 DB。

### 缓存击穿

在平常高并发的系统中，大量的请求同时查询一个 key 时，此时这个key正好失效了，就会导致大量的请求都打到数据库上面去。这种现象我们称为**缓存击穿**。

**击穿带来的问题**

会造成某一时刻数据库请求量过大，压力剧增。

**解决方案**

上面的现象是多个线程同时去查询数据库的这条数据，那么我们可以在第一个查询数据的请求上使用一个 互斥锁来锁住它。其他的线程走到这一步拿不到锁就等着，等第一个线程查询到了数据，然后做缓存。后面的线程进来发现已经有缓存了，就直接走缓存。

### 缓存雪崩

缓存雪崩的情况是说，当某一时刻发生大规模的缓存失效的情况，比如你的缓存服务宕机了，会有大量的请求进来直接打到DB上面。

**缓存雪崩带来的问题**

DB 称不住，挂掉。

**解决方案**

1. 使用集群缓存，保证缓存服务的高可用

2. ehcache本地缓存 + Hystrix限流&降级,避免MySQL被打死
3. 开启Redis持久化机制，尽快恢复缓存集群

## 解决热点数据集中失效问题

我们在设置缓存的时候，一般会给缓存设置一个失效时间，过了这个时间，缓存就失效了。

对于一些热点的数据来说，当缓存失效以后会存在大量的请求过来，然后打到数据库去，从而可能导致数据库崩溃的情况。

**解决办法**

1. 设置不同的失效时间

为了避免这些热点的数据集中失效，那么我们在设置缓存过期时间的时候，我们让他们失效的时间错开。

比如在一个基础的时间上加上或者减去一个范围内的随机值。

2. 互斥锁

结合上面的击穿的情况，在第一个请求去查询数据库的时候对他加一个互斥锁，其余的查询请求都会被阻塞住，直到锁被释放，从而保护数据库。

但是也是由于它会阻塞其他的线程，此时系统吞吐量会下降。需要结合实际的业务去考虑是否要这么做。