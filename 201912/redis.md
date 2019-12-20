# redis

### redis与memcache

1，redis的数据类型比较多，memcache只有字符串。

2，操作速度比较快——纯内存操作、单线程避免上下文切换操作、非阻塞异步io。

3，redis持久化，memcache只内存中。redis是memcache的高级版。

4，redis支持数据备份。主从模式-从从模式。

5，限制数据内存占用大小，超过则淘汰一定规则的key。

6, 操作原子性


#### 数据类型

`String/Hash/List/Set/zSet`, `HyperLogLog/Geo/Pub.Sub`, `Redis Module--BloomFilter,RedisSearch,Redis-ml`

#### 分布式锁

用`setnx`抢锁，设置`expire`，可以合成一条指令。

#### 找出N个固定开头的key

用`scan`无阻塞提取，但是会重复，最后做一次去重即可。

#### 异步队列

使用`list`数据结构，使用`rpush`生产消息，使用`lpop`消费消息。没有消息，需要`sleep`。但是可以使用`blpop`，如果么有消息会阻塞等待消息来，而不用`sleep`。使用`pub/sub`实现1:N消息队列，缺点就是消费者下线会丢失消息。如果使用延时队列：`sortedset`.

#### 如何实现持久化的

RDB——全量持久化（全量数据）与AOF(操作日志)——增量持久化.  AOF日志可以`sync`，都开`sync`不丢失数据，但是会影响性能。如果不全开启，则丢失的数据看时间。
