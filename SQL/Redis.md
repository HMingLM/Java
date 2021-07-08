[toc]

# Redis

Redis命令不区分大小写   

> ps -ef|grep redis     

==用jmeter测试接口并发量==

## 1.五大数据类型

### 1.String


```
> EXPIRE key seconds         # 设置key的过期时间，单位是秒

> incr / decr           # 增/减 1

> incrby / decrby           # 设置增/减的步长

> getrange key start end      # end为-1时获取所有

> setrange key offset value

> setex key seconds value        # set with expire  设置有过期时间的值

> setnx key value           #set if not exist   不存在再设置【分布式锁中常用】

> mset key value [key value ..]     #批量设置值

> mget key [key ..]     #批量获取值

> msetnx key value [key value ..]     # msetnx 是原子性的，要么都成功，要么都失败

> getset        # 先get再set

> mset user:01:name admin user:01:age 18    # user:{id}:{field}
```


### 2.List


```
> LPUSH/RPUSH key value [value ...]

> Lpop/Rpop key 

> Llen key          # 获取长度

> LRANGE key start stop     # 获取指定范围中的元素

> Lindex key index      # 通过下标获取某一个值

> Lset key index value      # 更新指定下标的值

> Lrem key count value      # 移除list中指定个数的value

> Ltrim key start stop      # 截取并修改list

> rpoplpush resource destination    # 移除resource末尾元素并将其插到destination头部

> Linsert key before|after pivot value      #在list中指定元素的前/后插入
```


### 3.Set

Set中的元素无序不重复


```
> Sadd key member [member ...]

> Smembers key          # 获取所有元素

> Sismember key member      # 判断是否存在

> Scard key      # 获取个数

> Srem key member [member ..]       # 移除指定元素

> Sranmember key [count]    # 获取[指定个数的]随机元素

> Spop key          # 随机删除一个元素

> Smove resource destination member         # 将resource中指定元素移到destination中

> Sdiff key1 key2 [key ..]      # 差集

> Sinter key1 key2 [key ..]     # 交集  【共同好友，推荐好友（六度分隔理论）】

> Sunion key1 key2 [key ..]     # 并集
```


### 4.Hash

值是map集合 key-map


```
> hset key field value

> hget key field

> hmset key field value [field value ..]

> hmget key field [field ..]

> hgetall key       # 获取所有值

> hdel key field [field ..]

> hlen key      # 获取键值对个数

> hexist key field      # 判断hash中指定字段是否存在

> hkeys key         # 获取所有的field

> hvals key         # 获取所有的value

> hincrby key field increment       # 设置增量

> hsetnx key field value        # 不存在再设置

> hset user:01 name admin age 18
```

### 5.Zset

有序集合：排序/带权重

```
> Zadd key [NX|XX] [CH] [INCR] Sorce member [Sorce member ..]

> Zrange key start stop

> Zrangebyscore key min max [withscores]    # 小到大排序 正无穷【+inf】 负无穷【-inf】

> Zrevrangebyscore key start stop [withscopes]     # 大到小排序

> Zrem key member [menber ...]

> Zcard key     # 个数

> Zcount key min max        # 获取指定区间的元素个数

> Zremrange key start stop [withscopes]
```

## 2.三种特殊数据类型

### 1.geospatial【底层是Zset】

- **定位、附近的人、距离计算**
- 两级无法直接添加
- 一般通过Java程序直接一次性导入
- 存储的有效经度-180到180，有效纬度-85.05112878到85.05112878，超出范围将报错

```
> geoadd key longitude latitude member [longitude latitude member  ...]

> geopos key member [member ..]

> geodist key member1 member2 [unit]    # unit是距离单位【m、km、mi英里、ft英尺】

> georadius key longitude latitude radius unit [withdist] [withcoord] count 10     # 以给定的经纬度为中心，找出半径内的元素，可以限定个数（10）

> georadiusbymmeber key member radius unit [withcoord]      # 以给定元素为中心，找到半径内的元素【附近的人】

> geohash key member [member ..]        # 返回一个或多个位置元素的geohash表示【11位字符串】

# 用Zset的删除命令来操作geo
Zrem key member
```

### 2.Hyperloglog

- 是一种**数据结构**
- 它的内存是固定的【12K】，有一定的错误率（较小）

> 基数：不重复的元素的个数。个数概念的推广。

> 应用场景：网页的UV，即访问的用户量，一个用户多次访问算作 1 。【用set放用户id耗费内存，使用Hyperloglog较为合适】


```
> PFadd key element [element ..]

> PFcount key       # 统计基数

> PFmerge destkey sourcekey [sourcekey ..]      # 并集
```




### 3.Bitmap

- 是一种**数据结构**
- **位存储**。位图
- 适用于两个状态的信息场景。如**打卡/未打卡，登录/未登录**
- 用二进制位 0/1 来记录信息。如365天，需365bit

```
> setbit key offset value       # value为0/1

> getbit key offset

> bitcount key [start end]      # 统计
```


## 3.Redis事务

- 本质：一组命令的集合
- **一次性、顺序性、排他性**
  - 一个事务中所有命令都会被序列化，在事务执行时它们顺序执行
- **没有隔离级别的概念**
  - 一i那位所有命令在事务中没有直接被执行，只有发起执行命令时才执行
- **Redis单条命令是保证原子性的，但事务不保证原子性！**
  - 若是（'编译型异常'），如使用错误的命令，事务中所有命令都不会被执行，事务报错
  - 若是（'运行时异常'），事务中其他命令正常被执行，错误命令单独报错 



- 使用事务

- 开启事务（multi）
- 命令入队（..命令..）
- 执行事务（exec）

> 取消事务  （discard ）



## 4.Redis实现乐观锁

- 使用watch监控来实现乐观锁
  - 若事务过程中数据有修改，则监控到，导致事务失败
  - 其实就是在事务执行时检查watch的值，相当于mysql的version，检查没有发送变化则事务成功，发送变化则失败
- 监控在事务正常执行完毕后会自动取消
  - 若事务失败，需unwatch

```
> watch key [key ...]

> ....
```


## 5.Jedis | lettuce

- Jedis：采用直连，多个线程操作不安全，可以通过连接池避免。更像BIO模式
- lettuce：采用netty，实例可以在多个线程中进行共享，不存在线程不安全的情况。可以减少线程数量，更像NIO模式


> 存储对象类型需要序列化

自定义RedisTemplate 


## 6.持久化

- Redis是**内存**数据库，若不进行**持久化**，将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的**数据库状态也会消失**。
- 只做缓存，则不需要持久化

### 1.rdb

redis database

- 在**指定的时间间隔**内将**内存中的数据集快照**写入**磁盘**，即Snapshot快照，他恢复时是将快照文件直接读到内存里。

- Redis会单独创建一个**子进程**来进行持久化，会先将数据写入到一个**临时文件**中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件，成为**正式RDB文件**。
- 整个过程中，**主进程不进行任何IO操作**，这确保了极高的**性能**。
- 若需进行**大规模数据恢复**，且对于数据恢复的完整性不特别敏感，则使用**默认的RDB方式**比AOF方式更**高效**

RDB的**缺点**
- 进程操作需要一定时间，意外宕机会丢失上一次持久化后修改的数据。
- fork进程占用了一定的内存空间

> rdb保存的文件名默认是dump.rdb，配置文件中有持久化的相关配置

触发持久化、自动生成dump.rdb备份的时机：
- 满足配置文件中save规则
- 执行flushall命令
- 退出redis

如何恢复rdb文件
- 只需将rdb文件放在redis的启动目录即可，redis启动时会自动检查dump.rdb，恢复其中的数据
- 可通过config get dir查看需要存在的位置


### 2.AOF

append only file

- 以日志的形式来记录每个写操作，将Redis执行过的所有指令记录下来（不包括读操作）
- 只许追加文件，不许改写文件
- redis启动之初会读取该文件重新构建数据，就是将所有写指令重新执行一遍
- 不适合大数据量
- 默认不开启，需手动配置
- 如果aof文件有错误，则redis启动不起来，需要用redis提供的工具**redis-check-aof**修复

```
redis-check-aof --fix appendonly.aof
```

> AOF保存的是appendonly.aof文件


## 7.发布订阅

发布订阅（pub/sub）是一种**消息通信模式**

![avatar](D:/其他/图片/study/redis发布订阅.PNG)

> Redis是使用C实现的，通过分析Redis源码中pubsub.c文件，可以了解发布和订阅机制的底层实现

使用场景：
- 实时消息系统（聊天室）
- 订阅、关注



## 8.主从复制

### 概念

- 将一台Redis服务器的数据复制到其他Redis服务器
  - 前者为主节点（Master/Leader），后者为从节点（slave/follower）
  - **复制是单向的**，只能由主节点到从节点
  - Master以写为主，Slave以读为主（不能写）
- **默认情况下，每台Redis服务器都是主节点**
  - 一个主节点可以有零或多个从节点，但一个从节点只能有一个主节点

### 作用

- **数据冗余**：实现了数据的热备份，是持久化之外的一种数据冗余方式
- **故障恢复**：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复（服务的冗余）
- **负载均衡**：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务，即写数据时应用连接主节点，读数据时应用连接从节点，分担服务器负载
  - 在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量
- **高可用基石**：主从复制还是哨兵和集群实施的基础，是高可用的基础


> 单台Redis最大使用内存不应该超过20G

### 环境配置

只配置从库，不用配置主库

准备：复制多个配置文件供主机从机使用，修改对应信息
- 端口
- pid 名字
- log 文件名
- dump.rdb 名字

> 此时每个Redis服务都是master主机，此时来配置从机

从机配置：
```
slaveof host port
```

> 以上命令配置是暂时的（该从机服务重启后会变成主机），真实的主从配置应该在配置文件中，是永久的


### 原理

- slave成功连接到master后会发送一个sync同步命令
  - master接到命令后，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕后，master将传送整个数据文件到salve，完成一次完全同步
- 全量复制：salve服务在接收到数据库文件数据后，将其存盘并加载到内存中
- 增量复制：发生新的修改命令时，master继续将新的所有收集到的修改命令依次传给salve，完成同步

> 只要重新连接master，一次完全同步（全量复制）将被自动执行


**主机宕机后，手动配置从机为主机**

```
slaveof no one 
```

### 哨兵模式

Sentinel

后台监控主机是否故障，如果故障了，根据投票数**自动将从库转换为主库**

**原理：**哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例

![avatar](D:/其他/图片/study/哨兵模式.PNG)

哨兵的作用：
- 通过发送命令，让Redis服务器返回其运行状态，包括主机、从机
- 当哨兵监测到master宕机，会自动将slave转换成master，然后通过**发布订阅模式**通过其他从机，修改配置文件，让他们切换主机

> 哨兵之间还会进行监控，形成多哨兵模式

![avatar](D:/其他/图片/study/多哨兵模式.PNG)

1. **配置哨兵配置文件sentinel.conf**

```
sentinel monitor 被监控的服务名称 host port 1

# 后面的数字1，代表主机挂了slave投票，选举票数最多的从机
```

2. **启动哨兵服务**

```
> redis-sentinel kconfig/sentinel.conf
```

**优点**：
- 哨兵集群基于主从复制，拥有主从复制全部优点
- 主从切换，故障可以转移，系统可用性更好
- 主从模式的升级，手动到自动，更加健壮

**缺点**
- redis不好在线扩容的，集群一旦到达上限，在线扩容就十分麻烦
- 实现哨兵模式的配置其实很麻烦！（端口、工作目录、主机信息、确定为宕机等待的时间、故障转移的时间）


## 9.缓存穿透和雪崩

### 缓存穿透
- 用户想要查询一个数据，发现redis内存数据库没有，缓存没有命中，于是向持久层数据库查询，发现也没有，则本次查询失败。
  - 当用户很多时（秒杀），缓存都没有命中，都去请求持久层数据库，对其造成很大压力，这就是缓存穿透

解决方案：

- 布隆过滤器
  - 是一种数据结构，对所有能查询的参数以hash形式存储，在控制层先进行校验，不符合则丢弃，从而避免了对底层存储系统的查询压力
- 缓存空对象
  - 当存储层不命中，即使返回的空对象也将其缓存起来，同时设置过期时间，之后再访问该数据会从缓存中获取，保护了后端数据源
  - 该方法存在的问题：
    - ① 若空值能够被缓存，意味着需更多空间存储更多的键，其中可能很多空值的键
    - ② 即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响



### 缓存击穿（量太大，缓存过期）
- 指一个key非常热点，在不停扛着大并发，大并发集中访问该点，当这个key在失效（过期）的瞬间，持续的大并发就穿破缓存，直接请求数据库查询最新数据并回写缓存，导致数据库瞬间压力过大

解决方案：
- 设置热点数据永不过期
- 加互斥锁
  - 分布式锁，保证对于每个key同时只有一个线程去查询后端服务，其余等待。此方式将高并发的压力转移到了分布式锁


### 缓存雪崩

在某个时间段，缓存集中过期失效。

redis宕机

解决方案：
- redis高可用
- 限流降级
- 数据预热



![avatar](D:/其他/图片/study/缓存雪崩.PNG)