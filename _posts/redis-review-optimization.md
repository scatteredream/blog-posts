---
name: redis-dpop
title: redis-review 项目优化
date: 2024-10-20
tags: 
- redis
- 分布式锁
- 缓存
- 布隆过滤器
- 乐观锁
- threadlocal
- mq
categories: 项目
---

# intro

配置环境： Nginx + SpringBoot + RabbitMQ + Redis

同城口碑主要采用的是 B2B2C（平台连接商家和消费者）的业务模式，是一个前后端分离的单体项目，但是为分布式和集群做了提前的适配，类似大众点评，实现了**登录认证**、**查看商家**、秒杀券抢购、**关注（共同关注）**、**发布点赞推送帖子**（实现了时间线timeline上的动态分页查询）、**签到**、附近商户等功能，业务可以帮助商家引流，增加曝光度。主要是为了学习redis的使用。也可以为用户提供查看提供附近消费场所 geo。主要是用来配合学习Redis在项目中的应用。

> **为什么选择RabbitMQ？**
>
> 消息堆积：一般就是nack太多或者是消费的太慢了，可以增加消费者数量，或者改成多线程消费。或者新开一个队列，将堆积的这些消息专门给消费者消费
>
> - 吞吐量虽然不如 kafka rocketMQ
> - 但是基于erlang语言进行开发，并发能力很好，时效性是μs级别
> - 基于主从架构实现的高可用，可靠性通过confirm、manual-ack、持久化实现
> - 与Redis消息队列相比支持持久化，性能更好
> - 订单处理：需要可靠的消息队列
>
> RocketMQ：复杂业务消息传递
>
> - 基于java
> - 高可用，0丢失
> - 事务消息、用户推送、分布式事务消息
>
> Kafka{Event Stream}：
>
> - 大数据，压缩数据
> - 负载均衡
> - 严格控制数据
> - 易用性不如MQ

> [HA](#ha)

<mark>登录功能</mark>：使用 redis token + 拦截器，<u>登录滑动窗口限流</u>。

[商户缓存](#cache)：进入主页，先从Redis中读出商户分类信息，若Redis中为空则向MySQL中读取，并写入Redis中。主页店铺分类信息为常用信息，应使用Redis避免频繁读取数据库，然后就是缓存的使用，延迟双删 + (canal+MQ 监控 binlog 异步重试删除)。对于redis挂掉的场景，就需要使用令牌桶限流数据库访问

[<mark>秒杀</mark>](#seckill)：异步下单，先运行Lua脚本：先判断库存够不够，再判断是否下过单，若未下过单，才能扣减redis库存和sadd成员，判定具备购买资格。「Lua执行过程中如果错误或是宕机，无法回滚，因此跟acid的原子性不一样」。

**订单预扣兜底**：可以lua脚本里面写一条预扣订单记录和zset补偿订单id表，定时任务扫描补偿表回滚。lua 脚本生成一个全局id作为订单id，存到补偿zset里。一开始订单状态肯定是pending。

**MQ**：MQ 处理成功则设置 status = success，失败则 status = failed 然后提前回滚库存相关，不管怎么样，成功或者失败都会把zset里的order_id给删掉，这样补偿zset里的id肯定都是pending状态了。（LUA2）rabbitmq使用镜像队列集群。

**定时任务**：订单状态肯定都是pending，那么把这些超时的设置 status = failed，然后都给回滚掉，最后删除zset里的order_id。（LUA3）

**前端**轮询订单接口的status。订单一开始是pending，超时兜底或者失败重试3次 failed，处理成功则 success。

- 一查到 failed/success 就删除预扣订单记录就返回结果，避免堆积（TTL也可以）。（LUA4）

[<mark>帖子点赞</mark>](#like)：用户浏览博客时，可以对博客进行点赞，点赞过的用户id，写入到Redis缓存中（zset：用户IDmember，点赞时间score）博客页并展示点赞次数和点赞列表头像，展示点赞列表时，注意点赞列表（id）按时间（score）排序，早的排在前面，SQL语句应拼接order By field (id = 1,2,3,4,5) 。

[<mark>帖子推送、时间线、分页查询</mark>](#timeline)：推模式，看看谁关注我了，发到这些关注关注者的收件箱（收件箱就是Redis里面的一个Zset，value有blogId，score就设置成当前的时间戳）SQL语句应拼接order By field (id = 1,2,3,4,5) 根据blogId进行查询。

- 推模式（写扩散）：写性能瓶颈。活跃粉丝用写扩散，不活跃的用读扩散。
- 拉模式（读扩散）：读取性能瓶颈，适合大V粉丝多。

[<mark>签到</mark>](#bitmap)：使用bitMap，打卡取1，为打卡取0，从第0位开始，n日的打卡数据在n-1位。把当月签到数据和1做与运算，得到最近一天是否打卡，为0则直接返回，为1则把签到数据右移一位和1做与运算，循环，直到与运算结果为0，循环次数为连续签到天数。

# 数据存储设计

## Redis 键值设计

> KEY：44B以内，embstr编码，内存连续。否则转换成raw，内存空间不连续，碎片化，访问性能不如前者

STRING 

- `sign:{userId}:202411` 2024年11月的userId的签到信息 （bitmap）
- `incr:order:2024:11:10` 2024年11月10号的订单序列号serialNumber，唯一id（timestamp<<32 | serialNumber）
- `seckill:stock:{voucherId}` voucherId的优惠券对应的库存
- `cache:shop:{shopId}` shopId的商铺信息
- `login:code:{phoneNumber}` 验证码信息

LIST 

- `cache:shoptype`（商铺类型信息JSON数组）

ZSET 

- `shop:geo:{shopId}` 商铺地理位置信息

- `blog:liked:{blogId}` blogId的博客点赞信息，包括userId 以及对应的score（unix时间）
- `feed:{userId}` userId 的收件箱，保存笔记时推送到所有粉丝（看看谁follow了我）包括了 blogId 以及对应的score 

SET

- `follows:{userId}` userId 关注的人
- `seckill:order:{voucherId}` 抢过voucherId的优惠券的人的集合

HASH 

- `login:token:{token}` 存储token对应的userDTO

------

• **热Key**：多级缓存 + 分片 + 异步更新。
• **大Key**：拆分 + 异步删除 + 设计规避。
• **分表**：合理分片规则 + 中间件 + 缓存加速。
• **监控**：实时监控Key大小和QPS，提前预防问题。



### 产生热/大key的原因

1. 将Redis用在并不适合其能力的场景，<u>造成Key的value过大</u>，如使用String类型的Key存放大体积二进制文件型数据（大Key）；
2. 业务上线前规划设计考虑不足<u>没有对Key中的成员进行合理的拆分</u>，造成个别Key中的成员数量过多（大Key）；
3. <u>没有对无效数据进行定期清理</u>，造成如HASH类型Key中的成员持续不断的增加（大Key）；
4. 使用LIST类型Key的业务消费侧代码故障，造成对应Key的成员只增不减（大Key）；
5. <u>预期外的访问量陡增</u>，如突然出现的爆款商品、访问量暴涨的热点新闻、直播间某大主播搞活动带来的大量刷屏点赞、游戏中某区域发生多个工会间的战斗涉及大量玩家等（热Key）；

#### 发现大key（--bigkeys & scan）

> 通常我们会将含有较大数据或含有大量成员、列表数的Key称之为大Key：
>
> - 一个<u>STRING</u>类型的Key，它的值为5MB（数据过大）
> - 一个<u>LIST</u>类型的Key，它的列表数量为20000个（列表数量过多）
> - 一个<u>ZSET</u>类型的Key，它的成员数量为10000个（成员数量过多）
> - 一个<u>HASH</u>格式的Key，它的成员数量虽然只有1000个但这些成员的value总大小为100MB（成员体积过大）数量不要超过1000

1. `redis-cli -a 密码 --bigkeys`：利用redis-cli提供的--bigkeys参数，可以<u>遍历分析</u>所有key，并返回Key的整体统计信息与每个数据的Top1的big key。缺点也很明显，结果不可定制化：bigkeys仅能分别输出Redis六种数据结构中的最大Key，如果你想只分析STRING类型或是找出全部成员数量超过10的HASH Key，那么bigkeys在此类需求场景下将无能为力。
2. 使用 `scan` 配合 `type` + `strlen` `hlen` 等命令：

```java
String cursor = ScanParams.SCAN_POINTER_START; // 初始为 "0"
ScanParams params = new ScanParams()
        .count(100)                // 每次扫描建议返回 100 个 key
        .match("user:*");          // 只匹配前缀为 "user:" 的 key
do {
    ScanResult<String> result = jedis.scan(cursor, params);
    List<String> keys = result.getResult();
    cursor = result.getCursor();
} while(!cursor.equals(ScanParams.SCAN_POINTER_START));
```

3. 自定义工具：监控进出Redis的网络数据，超出预警值时主动告警。阿里云Redis控制台中的CloudDBA
4. 第三方工具：Redis-Rdb-Tools 分析RDB快照文件，全面分析内存使用情况

#### 发现热key

> 某个Key接收到的访问次数显著高于其它Key，可以将其称之为热Key：
>
> - Redis实例QPS达10000，其中一个Key的QPS达到了7000（访问次数显著高于其它Key）
> - 对一个拥有上千个成员且总大小为1MB的<u>HASH</u> Key每秒发送大量的 `HGETALL`（带宽占用显著高于其它Key）
> - 对一个拥有数万个成员的<u>ZSET</u> Key每秒发送大量的 `ZRANGE`（CPU时间占用显著高于其它Key）

1. `redis-cli -a 密码 --hotkeys` Redis自4.0起提供了hotkeys参数来方便用户进行实例级的热Key分析功，该参数能够返回所有Key的被访问次数，它的缺点同样为不可定制化输出报告，大量的信息会使你在分析结果时复杂度较大，另外，使用该方案的前提条件是将redis-server的maxmemory-policy参数设置为LFU。
2. **通过业务层定位热Key**：指向Redis的每一次访问都来自业务层，因此我们可以通过在业务层增加相应的代码对Redis的访问进行记录并异步汇总分析。该方案的优势为能够准确并及时的分析出热Key的存在，缺点为业务代码复杂度的增加，降低性能。
3. `monitor`: 打印Redis中的所有请求，包括时间信息、Client信息、命令以及Key信息。缺点：资源占用，雪上加霜
4. 自定义工具：监控进出Redis的网络数据，超出预警值时主动告警。阿里云Redis控制台中的CloudDBA

### 大Key处理方案（拆分、删除、异步、压缩）

#### 拆分与分片（<u>水平</u>/垂直）

**垂直拆分**：按业务、功能将大key拆成小key，比如 HASH 的用户信息可以拆成基本信息和其他额外信息。

**水平拆分**：

- 哈希分桶：HASH的field、ZSET/SET的MEMBER
- ZSET可以按照SCORE范围划分
- LIST 有顺序要求，尽量直接按数量拆分，或者如果是日志信息可以按时间划分。

#### 拆分后的开销

- **查询逻辑调整**：拆分后需<u>合并</u>多个子 Key 的结果（如 `MGET`、`SUNION`）。
- **索引维护**：对按范围拆分的 Key（如 ZSet 按时间分片），需记录分片规则。
- **监控与清理**：定期检查拆分后的 Key 大小，避免二次膨胀。

> **典型案例参考**

- **电商购物车**：将包含 12000 个商品的 Hash 拆分为多个子 Hash，每个存储 1000 个商品。
- **社交好友列表**：按用户 ID 哈希分桶，每个子 List 存储 1000 个好友。
- **日志系统**：按日期分片存储 Set，并设置 TTL 自动清理。

#### 其他方式

1. **非阻塞清理**： 
   1. 使用`UNLINK`替代`DEL`，非阻塞删除大Key。

   2. 渐进式删除：通过脚本分批、定期删除Hash/Set失效的元素（如`HSCAN`遍历 `HDEL`删除）。
2. 异步处理：
   1. 使用 Redis 的异步 API（如 Redisson 的 `getAsync`）。
   2. 将耗时操作放入线程池或消息队列中处理。
3. **压缩与存储优化**： 
   1. 序列化优化：使用Protobuf或MessagePack替代JSON。

   2. 冷热分离：将大Key中低频数据存入MySQL/HBase，高频数据保留在Redis。

4. **设计规避**： 
   1. 避免单个Key存储超过1MB的数据。
   2. 使用HyperLogLog替代大Set统计UV，或使用TimeSeries存储时序数据。

### 热Key处理方案（多级缓存、读写分离、逻辑过期）

1. **多级缓存**
   • **本地缓存**：结合Guava Cache或Caffeine，将热key缓存在应用服务器本地，减少Redis压力。
   • **分布式缓存冗余**：复制热key到多个Redis实例（如`key:1`, `key:2`），通过随机访问分散压力。

2. **读写分离与分片** 
   • 使用Redis Cluster或Codis分片，分散热key压力。
   • 读写分离：通过从节点处理读请求，主节点处理写请求。

3. **缓存续期与互斥锁** 
   • **逻辑过期**：Value中存储过期时间，异步更新缓存，避免物理过期后大量请求穿透到DB。
   • **互斥锁（Mutex Lock）**：缓存失效时，仅允许一个线程重建数据，其他线程等待或返回降级结果。

4. **监控与识别**
   • 使用Redis监控工具（如`redis-cli --hotkeys`、`MONITOR`命令）或APM系统（如Prometheus）识别高频访问的key。
   • 业务侧预判热点（如秒杀商品ID、热门话题），提前介入优化。

### Key 数过多

（直接转hash/哈希映射到不同的hash桶）

一个是key本身的占用，再一个是集群模式服务端需要建立slot2key的映射，其指针也会占用空间

1. key 本身就有<mark>很强的相关性</mark>：相关的key合成一个hash
   1. user.zhangsan-id = 123;  user.zhangsan-age = 18; user.zhangsan-country = china;   
   2. 这三个key本身就具有很强的相关特性，转成Hash存储就可以

2. key 本身<mark>没有相关性</mark>：哈希桶
   1. 预估 key 的总数为 2亿，按照一个hash存储 100个field来算，需要 2亿 / 100 = 200W 个桶 (200W 个key占用的空间很少，2亿可能有将近 20G )
   2. 原先比如有三个key （userId） ：  123456789  , 987654321， 678912345
   3. 现在按照 2M 个 固定桶划分，先计算出桶的序号  hash(123456789)  % 2M
   4. `set (realKey, value) -> hset(bucketKey，realKey， value)`
   5. ``get (realKey)  -> hget(bucketKey， realKey)  `

- key1: hset (userid-bucket-1, 123456789 ,value)
- key2: hset (userid-bucket-2, 987654321, value)
- key3: hset (userid-bucket-2, 678912345, value)


### Redis分片与集群

1. **Redis Cluster** 
   • 自动分片（16384 slots），支持水平扩展和高可用。
   • 通过`CRC16(key) % 16384`计算slot，分散数据。

2. **客户端分片** 
   • 使用一致性哈希算法（如Jedis的`ShardedJedis`）在客户端路由请求。

3. **Proxy方案** 
   • 通过Twemproxy或Codis代理层管理分片，对业务透明。

## 数据库表设计

blog blog评论 关注的人(x在t时间关注了y) 用户表 

商铺 商铺类别

券 秒杀券 秒杀券订单

### 分布式唯一订单id——雪花算法

雪花算法：1位是占位符 中间41位是时间戳（支持69年） 接着10位是机器ID，最后 12位是序列id，

每次生成 id 需要 now 和 lastTimeStamp 做比较，如果超了就说明是另外的毫秒了序列id归零，如果没超说明还在同一毫秒内，序列id自增，last = now，可以看出一毫秒一台机器可以生成4096个不同的序列id。

**优点**：生成速度比较快、生成的 ID 有序递增、比较灵活（可以对 Snowflake 算法进行简单的改造比如加入业务 ID）

**缺点**：需要解决重复 ID 问题（ID 生成依赖时间，在获取时间的时候，可能会出现时间回拨的问题，也就是服务器上的时间突然倒退到之前的时间，进而导致会产生重复 ID）、依赖机器 ID 对分布式环境不友好（当需要自动启停或增减机器时，固定的机器 ID 可能不够灵活）。

```java
((now - twEpoch) << 22) | (datacenterId << 17) | (workerId << 12) | sequence;
```

#### UUID

**优点**：生成速度通常比较快、简单易用

**缺点**：<u>存储消耗空间大（32 个字符串，128 位）</u>、 <u>无序（非自增）</u>、不安全（基于 MAC 地址生成 UUID 的算法会造成 MAC 地址泄露)、没有具体业务含义、需要解决重复 ID 问题（当机器时间不对的情况下，可能导致会产生重复 ID）

### 水平分表（Sharding）

[几种水平分表方案与具体实践_水平分表的几种方式-CSDN博客](https://blog.csdn.net/TateBrwonJava/article/details/114527859) 

- 首先保证<u>全局唯一ID</u>与查询优化： 
  - 分布式ID生成（雪花算法、Redis自增ID）避免主键冲突。
  - 冗余字段或索引表：通过异步维护冗余字段（如商户ID+订单ID）支持多维度查询。

> 现在很多公司都是用的类似于 TiDB 这种分布式关系型数据库，不需要我们手动进行分库分表（数据库层面已经帮我们做了），也不需要解决手动分库分表引入的各种问题，直接一步到位，内置很多实用的功能（如无感扩容和缩容、冷热存储分离）！如果公司条件允许的话，个人也是比较推荐这种方式！
>
> 如果必须要手动分库分表的话，ShardingSphere 是首选！ShardingSphere 的功能完善，除了支持读写分离和分库分表，还提供分布式事务、数据库治理等功能。另外，ShardingSphere 的生态体系完善，社区活跃，文档完善，更新和发布比较频繁。

#### Sharding vs Partitioning

| Dimension    | Sharding             | Partitioning        |
| :----------- | :------------------- | :------------------ |
| 存储位置     | 不同机器             | 同一机器            |
| 可扩展性     | High (Scale Out)     | Limited (Scale UP)  |
| 可用性       | High                 | 跟单机类似          |
| 并发查询性能 | 取决于机器数目       | 取决于机器的CPU     |
| 查询时间     | 低，除非某台机器过载 | 中低，主要局限于CPU |

Partitioning 可以分为 水平 和 垂直，水平就是按行拆，垂直就是按列拆（主要是可以按照业务划分冷热数据列），而 Sharding 就是水平 Partitioning 的分布式版本。

![Image](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/6347a20e8598ff81de4fc4f7_QVjhYNKEHaJ3ZumpY0xicg19Tqoj0vHOVEHklTYHI65E9oG-t9sg0hZcokmdJI26_VDNkbnynbNvvF0Rl6Crx8cqq_Ffq129xsPbykCGqE8LLwPkKDaJ2zUQNJ0H55otwnNMi8IDEPE8ki4nPULl1RY6VLhh2DrsuuRs2fx0tNr9UyBOq0RAg0SLOg.png)

#### 分表带来的问题

1. join 无法进行，需要手动封装、聚合数据
2. 数据库自带的事务失效，要有分布式事务（ShardingSphere支持）
3. 分布式id
4. **跨库聚合查询**：分库分表会导致常规聚合查询操作，如 group by，order by 等变得异常复杂。这是因为这些操作需要在多个分片上进行数据汇总和排序，而不是在单个数据库上进行。为了实现这些操作，需要编写复杂的业务代码，或者使用中间件来协调分片间的通信和数据传输。这样会增加开发和维护的成本，以及影响查询的性能和可扩展性。

#### 规则分片策略

> 分片策略：包含分片键（将数据库表水平拆分的字段）以及分片算法

##### 分片键选择

减少单次查询涉及的分片数量，分割尽量均匀，值稳定也很重要，需要支持动态扩展避免重新分片。

1. <span id="fanwei">**主键**</span>：按照时间将订单水平分表，分成 `order2020_2023` `order2024_2027`，使用雪花算法id获取年份作为分片键，然后交由中间件进行路由查询。
2. **其他字段（以地区为例）**：同理，比如按照地区分表，每次查询的时候可以带上地区，交给中间件进行路由查询
3. 复合键
4. 不存在于数据库：可能来自上下文，需要在业务层强制路由
5. <span id="roundRobin">单纯拆分数据（不推荐）：直接拆成三个表，均匀</span>

##### 分片算法

- RoundRobin：id对表数取余，对应方法[4](#roundRobin)，数据很均匀，但是一旦节点变动，重新分片的开销也很大
  - hash
  - 一致性哈希，不过一般用于es、redis，主要适用于字段比较随机，不进行范围查询的数据，节点经常弹性扩展
- 范围法：按照字段处于的范围分，容易定位，对应方法 [1,2](#fanwei)

分片键缺失：全数据量查询

##### 非分表键

针对有些情况没有分表key的情况也是比较普遍的，比方说一个c端与b端之间的平台类项目，往往在选择分表key的时候会出现顾守不顾尾的情况。这个时候总要牺牲一端的用户体验。比方说针对userid对order表分表，那么c端用户查询确实是便利了，但是b端用户就很难找到这个userid，这个时候我们一般有几种处理方法：

- 改写sql走union查询：这种方式问题比较大，一方面是由于union效率比较低，做排序分页等查询问题也比较大，另一方面如果使用mybatis-plus之类的工具，使用拦截器插件进行开发分表功能的话，如果我们进行查询的时候参数拼接重写会有参数无法注入的问题。具体来说比方说select * from a where id = 1.那么分表之后的查询变成了(select * from a_1 where id = 1) union (select * from a_2 where id = 1);但在mp框架里会出现参数集为1，待重写阶段的sql为select * from a_1 where id = ?，这样的在一句sql里面出现了两个问号，但是没法填充。当然如果对mp源码理解比较深的话，可以把参数重写。即扩充一倍即可。
- 双写：另外搞一个主库或者主表，比较简单,每次插入更新的时候都做两次，主表的话加个事务问题不大，如果加个主库的话，则要注意保证一致性，可以考虑消息队列或者binlog之类的处理方法。
- 更改业务逻辑，这种方法一般不太可能，也有特殊情况。

#### 中间件

使用ShardingSphere（分库分表、分布式事务）、MyCat（代理层，SQL 路由、读写分离）等工具自动路由查询，侵入性较低。也可以使用 MP 拦截器自行开发分页插件。

### 其他策略（Redis缓存路由、热数据 | 垂直分表）

1. **结合Redis**：
   • 缓存分表路由信息（如`user_shard:1000 -> shard_2`），加速查询。
   • 热数据缓存：将分表后的热数据单独缓存，降低DB压力。
2. **垂直分表**  （一般不推荐）
   • 不要过度分表或分库，要从实际需求出发，可以是已经碰到的情况下，也可以是预估出不分表不行的情况
   • 所要分的表与其他表关联度不是很大，如果它经常被做连接查询，而且它的主键在别的表经常以外键形式存在，那么就不建议直接水平分表，可以先考虑垂直分表把数据量大的部分独立出去再做水平切分。将大表按列拆分（如用户基础信息表`user_base`与扩展信息表`user_extend`），减少单行数据量。

### 数据迁移

1. 停机，写脚本逐条迁移
2. 不停机（双写）：每条对老库的增删改都要写入新库（不存在则插入）自己需要写脚本让那些没有被改的数据同步到的新库，不断进行比对，直到相同为止。工具可以借助 canal

### 冷热分离

任务调度：可以利用 xxl-job 或者其他分布式任务调度平台定时去扫描数据库，找出满足冷数据条件的数据，然后批量地将其复制到冷库中，并从热库中删除。这种方法修改的代码非常少，非常适合按照时间区分冷热数据的场景。

监听 binlog ：将满足冷数据条件的数据从 binlog 中提取出来，然后复制到冷库中，并从热库中删除。这种方法可以不用修改代码，但不适合按照时间维度区分冷热数据的场景。

# 项目剖析

## Login (Redis Token)

1. 短信登录的短信怎么发送的?

   （使用hutool生成的随机六位数验证码）

2. 如何标识用户

   （主要是唯一的id主键，而登录靠的是也是唯一的手机号）

3. 项目的权限刷新什么意思

   （token具有ttl期，用户如果浏览网页就会刷新其ttl）

**原来**：tomcat 的 HttpSession 是集中的，如果要扩展的话同步是一个大问题，并且可能过多占用服务器的内存。

**改进**：<mark>Redis Token</mark>是分布式的，天生不存在同步问题。

- **Redis**：<u>使用 Hash 数据结构—— KEY: token:xxxxxxxx, VALUE: UserDTO的字段信息</u>
- **拦截器**：配置到需要拦截器的地方（个人中心等位置）根据请求头里的 token 查到对应的UserDTO，将其放入<mark>ThreadLocal</mark>，放行，同时也会刷新 token 有效期。<mark>核心方法：preHandle + afterCompletion</mark>
  - ThreadLocal 使用：`get()`,`set()` 在finally块调用`remove()` 用途 Spring 数据库连接、事务管理
  - 内存泄露：每个线程有个 Map，键是 tl（弱引用），值是对应的值（强引用），如果线程没有销毁，但是 tl 销毁了，就容易泄露内存。
- **问题**：访问别的页面也需要刷新，不然不浏览那些需要拦截器的页面就不会刷新token，莫名其妙就会过期
- **解决**：新加一个拦截器，负责查token、保存到tl、刷新，放行所有。原先的拦截器只负责从tl查询用户，tl 里没有就拒绝
- **其他缺点**：仍然依赖内存（redis是内存数据库） + redis接口调用会增加复杂性。需要注意的是，虽然这种方式支持分布式系统，但是根本上还是和 HttpSession 一样的**中心化**。

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653320764547.png" alt="优化拦截器" style="zoom:33%;" />

JWT：去中心化，服务端压力减小，便于分布式系统使用。基本信息可直接放在token中。功能权限较少可以直接放在token中，用bit位表示用户所具有的功能权限。但是缺点也很明显，签发以后就不能主动让其失效，拿到了就可以进行重放攻击。

### 滑动窗口登录限流

基于redis zset，member为请求的唯一uuid，score是请求对应的时间戳，windowSize是限流窗口的时间（12小时内），阈值limit（最多五次登录机会）

如果是要对同一个浏览器，zset对应的key可以使用token。如果仅针对同一用户，可以使用用户id做限流。

1. 请求到达时，删除 score< now - windowSize的元素。
2. 查看 ZSet 当前元素个数（即：当前时间窗口内的请求数）
3. 是否达到阈值limit？未达到则添加一个新元素（member=UUID，score=now）允许放行请求，达到拒绝

## <span id="cache">缓存</span>

> 为什么数据库会被打崩？
>
> - **慢查询开始堆积，雪崩效应**：争用CPU，锁竞争加剧。缓存命中率下降，跑去磁盘IO，最后请求会排队、超时，变得不可用。雪崩式效应
> - 数据库端的max连接数是有限的，每个都会占据tcp端口（fd数量限制） HikariCP 的连接池也是有限的默认 10 条 应用开始抛出与数据库连接相关的异常，例如 `CannotGetJdbcConnectionException` (Spring JDBC), `SQLException: Connection pool exhausted`, `Timeout waiting for idle object` (常见连接池如 HikariCP, Druid 的日志/错误)。瞬间涌入的海量查询会迅速消耗掉连接池中的所有可用连接。新的查询请求无法获取到连接，只能排队等待或直接失败。即使数据库本身还能处理请求，应用也无法将请求送达数据库了。
>
> 机器自身的原因：error请求
>
> - 关键是数据库的连接池有限，数据库比较慢，tcp连接不能及时释放，就有更多的请求来绑定新socket，BindException
> - acceptCount：最大请求队列长度
> - maxThreads: 最多同时处理多少个请求 1C2G=200  4C8G=800
> - maxConnections和accept-count的关系为：当连接数达到最大值maxConnections后，系统会继续接收连接，但不会超过acceptCount的值。
>
> ![在这里插入图片描述](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/20191027171725345.png)
>
> 



在我们实际的业务场景中，一定有很多需要做数据缓存的场景，比如售卖商品的页面，包括了许多并发访问量很大的数据，它们可以称作是是“热点”数据，这些数据有一个特点，**就是更新频率低，读取频率高**，这些数据应该尽量被缓存，从而减少请求打到数据库上的机会，减轻数据库的压力。

使用 Redis 作为商户信息缓存，避免大量请求直接打到数据库引发宕机，采取<mark>主动更新、旁路缓存</mark>的方法保证数据库与 Redis 之间的缓存一致性问题。使用<mark>布隆过滤器</mark>解决商户信息的缓存穿透问题，使用 **Redis 集群**等方式减
小缓存雪崩的概率，使用<mark>逻辑过期</mark>解决缓存击穿。

**内存更新**：

- 内存淘汰机制和TTL（对于低一致性场景）
- <mark>主动更新</mark>：编写业务逻辑，修改数据库的同时操作缓存，解决一致性问题。（对于高一致性场景）✅

**一致性解决方案**：

- **cache aside 旁路缓存** 编码使用客户端自行将数据写入cache ✅ 适合读多写少
- 读写穿透：封装了旁路缓存功能，其工作由cache层的服务自动完成，但是redis并不支持写入db
- 异步缓存写入：只操作cache，其他线程异步写到db。一般是写密集允许短暂的数据不一致

> 分布式系统里要么通过2PC或是Paxos协议保证一致性，要么就是拼命的降低并发时脏数据的概率
>
> 缓存系统适用的场景就是非强一致性的场景，所以它属于CAP中的AP，BASE理论。
>
> 异构数据库本来就没办法强一致，**只是尽可能减少时间窗口，达到最终一致性**。
>
> 还有别忘了设置过期时间，这是个兜底方案

**Cache Aside**：

- 更新缓存的优点是每次数据变化时都能**及时地更新缓存**，这样不容易出现查询未命中的情况，但这种操作的**消耗很大**，如果数据需要经过复杂的计算再写入缓存的话，频繁的更新缓存会影响到服务器的性能。如果是写入数据比较频繁的场景，可能会导致频繁的更新缓存却没有业务来读取该数据。*更新导致的无效写比较多（更新多次但只有最后一次被查询），并且线程安全问题难以解决：**要求数据库和缓存写入都要成功**，此时有可能A更新到v1，线程B突然插进来把缓存更新到v2，最后A把脏数据v1更新到缓存，如果缓存更新涉及多表的查询就更加难以维护。* 
- 选择删除cache，**cache删除和db更新放入一个事务**
- **删缓存、更新db的先后顺序**：读未命中、更新操作的互相穿插。读肯定大于更新
  - 选择先更新db后删除cache。导致不一致的情况：读未命中的过程中插进来一个更新操作，最后缓存里是旧数据。
  - *如果反过来，先删cache，更新db过程中进来一个查询的又把旧数据填充到缓存里，最后导致db和cache不一致。*
  - 最终选择：先删cache后更新db
- **既然是删除缓存的策略，可能会发生高并发请求打到数据库的情况。**
  - 参考 Hotspot Invalid，拿到互斥锁进行缓存重建，其他线程重试查询缓存。（当心死锁风险）
  - 参考 Cache Avalanche，对业务进行降级、限流处理，比如令牌桶

- **延迟双删：删完之后，隔段时间再尝试删一次。**
  - 休眠时间：覆盖这个并发的读请求回写的时间（回写窗口） + 几百毫秒
    - 第一次不删除：读取了一定的脏数据，但是也能替数据库挡住请求。
  - (读写分离：A 对主库写，B读从库的旧数据，因此只需要在一个主从同步的时间基础上加几百ms。) 不要说这个
  - 保证吞吐量：开个新线程进行异步的延迟双删。ScheduledExecutor.schedule()
- **如果缓存删除失败怎么办**？：异步监听删除，只是尝试删除缓存一次，canal监听binlog，遇到update就发消息给MQ，MQ负责删除以及重试。
  - MQ的解决办法：启动canal去订阅数据库的binlog，获得需要操作的数据。脱离业务代码，获得canal传来的信息，收到了发给MQ进删除缓存操作。[canal](https://scatteredream.github.io/2024/08/14/redis-cluster-canal/#%E7%BC%93%E5%AD%98%E5%90%8C%E6%AD%A5%EF%BC%9ACanal) 
    1. 更新数据库数据
    2. 数据库会将操作信息写入binlog日志当中
    3. 订阅程序提取出所需要的数据以及key
    4. 另起一段非业务代码，获得该信息
    5. 尝试删除缓存操作，发现删除失败
    6. 将这些信息发送至消息队列
    7. 重新从消息队列中获得该数据，重试操作。

**多重强一致性方案：**同步删除+ 异步监听+可靠消息删除，缓存时设置过期时间，查询时强制主库查；适合于强制要求数据一致性的情况。

1. **同步删除：**先更新数据库、再删除缓存；之后本链路禁止再查该数据，防止没来得及删缓存就又查到旧缓存数据。
2. **异步监听：**Canal等组件监听binlog发现有更新时就发可靠MQ删除缓存；二重保证删缓存成功；
3. (**延迟消息校验一致性：**Canal等组件监听binlog，发延迟MQ，延迟若干时间，校验缓存一致性；)
4. **缓存设置合理的TTL：**每次缓存时设置TTL；三重保证删缓存成功；
5. **强制Redis主库查：**以后查缓存时强制从缓存主库查；因为主从同步有延迟，同时不用担心主库压力大，因为分片集群机制。对一致性敏感的 key 强制读主，其余仍可读从，节省资源。

**如果删除失败以后读脏数据怎么办**？

- 写入标记+短ttl：删除失败之后写入一个特殊的key，短ttl，标记这个缓存可能不准确，然后在正常读缓存逻辑之前加上if判断即可，遇到特殊key就直读数据库，统计缓存失败率，可以据此熔断。

| **场景**                 | **解决方案**                    | **适用性**               |
| ------------------------ | ------------------------------- | ------------------------ |
| 短暂删除失败（网络抖动） | **标记待修复** + 读取时主动修复 | 高并发，允许短暂不一致   |
| 长期缓存服务不可用       | 熔断降级 + 直读数据库           | 强一致性要求场景         |
| 数据版本频繁变更         | **版本号校验** + 异步补偿任务   | 金融、订单等关键业务     |
| 灰度验证修复效果         | 流量染色 + 强制对比             | 需要验证缓存一致性的场景 |

<mark>缓存穿透和缓存击穿都实现在 **CacheClient** 中</mark>

**缓存穿透**：查询db里不存在的数据，导致db崩溃。

- 缓存空对象设置ttl：实现简单，维护方便。*造成短期的不一致，额外内存消耗。* 
  - 适合空key数目可控的场景，支持动态更新
- **布隆过滤器**：内存占用较少，没有多余key。*实现复杂，可能误判。*   本项目使用 Guava 实现。
  - 适合海量数据，且需要高效拦截绝对不存在的请求时（如用户注册校验）需要预先初始化 @PostConstruct 
  - 容量由bitmap大小与哈希函数数量决定，可能会满，满了以后可以扩容，可以业务层兜底（比如缓存空对象）。
  - 删除、扩容：不支持。
  - **计数布隆过滤器**(Counting Bloom Filter): 用空间复杂度换来删除功能，bit改为计数器，查询时所有位置的计数器都大于0才算存在。插入时在所有位置加1，删除时在所有位置减1，只要有一个归0就认定其不存在。
  - **布谷鸟过滤器**(Cuckoo Filter): **指纹**使用抗碰撞性强的哈希函数计算哈希值后截取低位获得。使用**桶数组**存储指纹，一个桶存四个指纹(连续,cpucache)。使用与当前指纹的哈希值异或的方式计算出另一个指纹副本的桶索引。
    - 插入时，第一个桶索引 i1= hash(x) 计算，第二个桶索引 i2 = i1 ⊕ hash(fingerprint(x))。桶只要有一个空位就可以插入，如果直接和指纹本身异或，在指纹跟桶数组大小差距悬殊时两个桶离得较近不利于均匀分布。如果没有空位就从其中挑一个指纹 f’ 迁出然后迁入，对于f’我们直接使用当前的桶索引和 f’ 进行异或就能计算出就能 f’ 备用桶的位置。
    - 查询时，看两个桶里面是否存在 x 的指纹，有指纹就算存在。删除时，从两个桶里删除匹配的指纹。
    - 降低误判率：增大指纹长度，减小桶的尺寸
    - 大多数情况下，选择2个哈希函数，桶的尺寸选择4，能够达到最佳或接近最佳的空间效率的假阳性率。
  - 动态数据集应该预防写满，或者使用多级过滤，例如使用 HashSet 过滤高频数据，然后用BF处理长尾数据
  - 可以使用功能更加强大的 RedisBloom

| 使用目的                                                 | 推荐结构           |
| -------------------------------------------------------- | ------------------ |
| 需要支持删除、插入频繁、对空间要求低的系统（如缓存服务） | **计数布隆过滤器** |
| 对空间敏感，插入元素不频繁且查询频繁的系统               | **布谷鸟过滤器**   |
| 追求查询性能、误判率低，且不怕插入时的复杂性             | **布谷鸟过滤器**   |

**缓存击穿**：一个被高并发访问并且缓存重建业务较复杂的key（或者是更新数据库的时候删除了缓存）突然失效了，无数的请求访问会在瞬间给数据库带来巨大的冲击。常见的解决方案有两种：

- **互斥锁 MUTEX**。线程 1 未命中，拿到锁去**同步地重建缓存**，线程 2 此时也紧随其后，拿不到锁就重复查询缓存。直到重建完成，线程 2 就能拿到重建的缓存值。优点是**一致性可以确保**，缺点是重建期间其他的线程会一直**同步递归调用**。
- 逻辑过期 LOGICAL EXPIRE。不存在未命中情况，发现过期就拿到锁然后提交一个重建缓存的任务（异步重建完之后finally释放锁），然后直接返回旧值。优点是**响应比较快**，是非阻塞的，缺点是一致性受影响。

**缓存雪崩**：同一时段大量的缓存key同时失效或redis宕机。

- 给不同的Key的TTL添加随机值
- 利用Redis集群提高服务的可用性
- 给缓存业务添加降级限流策略
- 给业务添加多级缓存

## 全局异常处理

### @RestControllerAdvice 和 @ExceptionHandler

对于更简单的异常处理，推荐使用 `@RestControllerAdvice`：所有方法默认返回 JSON，等价于自动添加 `@ResponseBody`。

```java
@RestControllerAdvice
public class GlobalApiExceptionHandler {

    // 自动返回 JSON，无需 @ResponseBody
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleUserNotFound(UserNotFoundException ex) {
        return new ErrorResponse(ex.getMessage(), 404);
    }

    // 更灵活的控制：自定义状态码和响应体
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidationError(ValidationException ex) {
        ErrorResponse error = new ErrorResponse(ex.getMessage(), 400);
        return ResponseEntity.badRequest().body(error);
    }
}
```

注意事项

- **执行顺序**：解析器按注册顺序执行。返回非空 `ModelAndView` 时终止处理。
- **默认解析器**：使用 `extendHandlerExceptionResolvers` 避免覆盖Spring默认解析器（如处理`@ExceptionHandler`的解析器）。
- **与Spring Boot整合**：在Spring Boot中，可结合 `@ControllerAdvice` 和 `ErrorController` 进行全局处理。

### Result 封装

```java
public class Result {
    private Boolean success;
    private String errorMsg;
    private Object data;
    private Long total;

    public static Result ok(){
        return new Result(true, null, null, null);
    }
    public static Result ok(Object data){
        return new Result(true, null, data, null);
    }
    public static Result ok(List<?> data, Long total){
        return new Result(true, null, data, total);
    }
    public static Result fail(String errorMsg){
        return new Result(false, errorMsg, null, null);
    }
}
```

## <span id="seckill">秒杀抢购</span>

• 使用乐观锁避免秒杀券超卖，同时提高并发。通过控制事务的粒度以及动态代理保证事务的有效性。使用 Lua 脚本解决秒杀券更新的原子性问题。 Redisson 分布式 锁解决集群环境下一人一单可能导致的并发安全问题。在 3000 并发测试中成功拦截 100%的重复请求

### 令牌桶限流

> 实际上缓存也可以有限流，如果缓存暂时失效了，就应该在数据库访问操作之前取令牌。

- **令牌生成机制**：系统按固定速率（如 100 个 / 秒）向桶中放入令牌，桶有最大容量（如 1000 个令牌），满后不再生成。
- **流量处理逻辑**：每个请求需要消耗一个令牌才能被处理，无令牌时请求被拒绝或排队。

| **维度**     | **漏桶算法**                       | **令牌桶算法**                        |
| ------------ | ---------------------------------- | ------------------------------------- |
| **控制目标** | 固定速率从桶中取出                 | 生成速率是 固定值，对消耗速率没有限制 |
| **突发处理** | 无法处理，超出部分直接丢弃         | 可利用预存令牌处理突发                |
| **桶的作用** | 存储待处理流量（溢出丢弃）         | 存储令牌（控制流量合法性）            |
| **适用场景** | 对速率严格限制的场景（如带宽限速） | 允许流量突发的场景（如 API 限流）     |

使用令牌桶限流防止突发流量：异步下单，先运行Lua脚本，判断库存够不够，判断是否下过单，若未下过单，才能扣减Redis库存，脚本运行成功，有购买资格。则生成一个全局Id作为订单id，生成订单信息，把订单保存到消息队列，redis 回滚。

> 令牌桶算法最初来源于计算机网络。在网络传输数据时，为了防止网络拥塞，需限制流出网络的流量，使流量以比较均匀的速度向外发送。令牌桶算法就实现了这个功能，可控制发送到网络上数据的数目，并允许突发数据的发送。大小固定的令牌桶可自行以恒定的速率源源不断地产生令牌。如果令牌不被消耗，或者被消耗的速度小于产生的速度，令牌就会不断地增多，直到把桶填满。后面再产生的令牌就会从桶中溢出。最后桶中可以保存的最大令牌数永远不会超过桶的大小。
>
> **阻塞式获取令牌**：请求进来后，若令牌桶里没有足够的令牌，就在这里阻塞住，等待令牌的发放。
>
> **非阻塞式获取令牌**：请求进来后，若令牌桶里没有足够的令牌，会尝试等待设置好的时间（这里写了1000ms），其会自动判断在1000ms后，这个请求能不能拿到令牌，如果不能拿到，直接返回抢购失败。如果timeout设置为0，则等于阻塞时获取令牌。

```lua
-- redis 使用hash来实现
-- 1. capacity 容量
-- 2. tokens 桶内现有令牌数量
-- 3. rate 添加速率
-- 4. timestamp 时间戳，用于懒加载
-- 调用redis指令TIME的结果 TIME
tokens = math.floor(capacity,(TIME-timestamp)*rate);
-- 然后减去tokens
```

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/kavrxpwgkb2oo_20240819_446c67dc76af4b589ca0bf818ddd07c5.webp" alt="img" style="zoom: 67%;" />

> 漏桶算法的“恒定出水速率”决定了它无法应对瞬时突发请求，一旦超出桶容量，流量就被丢弃。
>
> - **水桶模型**：将流量视为 “水”，流入一个固定容量的桶中，桶底部有一个恒定速率的 “漏洞” 向外流出。
> - **控制逻辑**：无论流入流量如何变化，流出速率始终保持恒定（等于漏洞的速率），桶满后多余流量会被丢弃。
>
> - **流入无缓冲空间**：假设业务突然产生大量流量（如电商大促瞬间的请求），漏桶仅能按固定速率处理，超出桶容量的流量会被直接丢弃，无法临时 “存储” 突发量。
>
> 跟消息队列有点类似。

<img src="https://ucc.alicdn.com/kavrxpwgkb2oo_20240819_40512d45f4894383bf56ced91e8a2f62.webp?x-oss-process=image%2Fresize%2Cw_1400%2Cm_lfit%2Fformat%2Cwebp" alt="img" style="zoom: 75%;" />

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250522214813369.png" alt="image-20250522214813369" style="zoom: 33%;" />

拓展阅读：

### 乐观锁 防止超卖 

**做完>0库存判断之后：**

- 原来：不检查直接扣【超卖】
- 改进1：扣减时的库存必须跟原来一样才成功。（类似于版本号）【成功率太低】
- 改进2：只要大于0就可以进行扣减。

### 一人一单（订单接口的幂等性）

1. 订单接口的幂等性是怎么做的？

   （意思就是post请求带着：一个用户id，一个优惠卷id。发送多次请求，如何保证只有一个成功，也就是一人一单，订单做成唯一id）还有一种方法是新加一个状态字段。

2. 库存扣减之后，还要去insert一个新的订单，你是如何保存这个的一致性的？`createOrder(order)`

   （使用spring自带的事务，将其放到一个事务里 `@Transactional`，注意事务有效性）

3. 抢优惠券没有及时处理怎么办?

   （通知用户已经抢了，延迟推送结果，轮询订单状态查询接口）

4. 抢优惠券处理完了如何通知用户？

   （前端轮询，状态变了自己会查到，也可以采用其他服务器推送模式，如SSE、WebSocket、第三方（firebase））

5. 秒杀场景下扣减库存太慢了怎么办？

   （数据库集群，分库分表，索引优化，Redis热key，大key，Redis缓存过小）

#### 幂等

1. **业务幂等令牌**：使用唯一的业务ID或请求ID
2. **乐观锁**：使用版本号或时间戳
3. **事务状态表**：记录已处理的事务ID
4. **分布式锁**：对于分布式系统

唯一主键是保障幂等性的有用工具之一，但对于复杂的业务场景，通常需要结合其他机制来实现完整的幂等性保障。

- 唯一id

- 乐观锁
- 防重token

#### 单机解决方案

- 原来：一人n单过来之后，一开始都还没有订单，问题还是一样。
- 改进1：变成同步方法以后锁粒度太大 `createVoucherOrder()`。
- 改进2：使用`this`作为锁缩小范围，`this`获取的是`VoucherOrderService`对象，只要获得了服务对象就要参与竞争，导致所有人都参与竞争，继续缩小粒度。
- 改进3：使用`userId.toString().intern()`作为锁对象锁住一人一单的判断逻辑，但是锁在事务内部会导致事务尚未提交锁就已经释放，所以应该在 `createVoucherOrder()` 外部进行加锁。
- 改进4：上方依然有问题，因为直接调用实例方法默认是使用`this.createVoucherOrder()`的，而Spring的事务基于 AOP 动态代理，因此需要显式获取代理对象。
- 改进5：外部的方法不要加事务，因为事务传播机制默认是加入已经存在的事务。这边锁释放了大事务还没提交所以没用。

#### 分布式锁

单机使用常量池的做法在集群环境下同步代码块会失效，因此需要使用redis作为公共的分布式锁进行改进

**原来**：setnx + ttl 防止死锁 

- 问题是如果在锁内部阻塞导致过期删除，别人抢来了锁开始执行，自己出来以后就会把别人的锁删掉。

**改进**：分布式锁的value为自己的线程id，释放逻辑里判断id是否和自己相同。

- 问题是如果发生比较极端的情况，判断逻辑里id相同，准备delete时可能发生fullgc导致超时释放，仍然会误删别人的锁，拿锁、比锁、删锁不是原子性的。因此引入LUA脚本。
- 自定义锁的局限性：实现比较简单，没有高级功能。

**REDISSON** 底层原理还是 setnx 加锁，lua 脚本解锁。但是解决了 setnx 的以下痛点：

1. 不可重入：会出现死锁，获取锁的线程不能进入相同的锁的代码块。
   - **Redisson 重入机制**：重入次数从0开始增加，使用hash结构存储线程id以及其重入次数。
2. 不可重试：尝试获取一次就返回没有重试机制。要么就是无效自旋过于耗费资源。
   - 获取失败会订阅（pub/sub）释放消息，再等一段时间，收到释放信号就重试，如果一直没获取到，超出最大等待时间之后返回失败。
3. 超时释放：业务执行时间长导致锁过期，虽然不会误删，但是有隐患。
   - **续约**：**watchdog** 线程，只要持有锁就会有定时更新ttl的任务，业务完成释放锁也会将定时任务取消。
4. 主从一致：主从同步出现问题可能出现死锁。
   - **multilock**：解决主从一致问题。必须所有的redis全获取成功才算获取到。

### RabbitMQ 异步秒杀

项目为什么要加个消息队列?

- Redis 效率比较高，而数据库效率不如redis快，tomcat中的程序，会进行串行操作，分成如下几步：查询优惠卷、**判断秒杀库存是否足够**、查询订单、**校验是否是一人一单**、扣减库存、创建订单。在这六步操作中，又有很多操作是要去操作数据库的，而且还是一个线程串行执行， 这样就会导致我们的程序执行的很慢，所以我们需要异步程序执行。
- 将耗时比较短的逻辑判断放入到redis中，比如是否库存足够，比如是否一人一单，这样的操作，只要这种逻辑可以完成，就意味着我们是一定可以下单完成的，我们只需要进行快速的逻辑判断，根本就不用等下单逻辑走完，我们直接给用户返回成功， 再在后台开一个线程，后台线程慢慢的去执行queue里边的消息。
- 为什么不使用线程池或者异步编排。如果访问的人很多，那么线程池中的线程可能一下子就被消耗完了，而且你使用上述方案，最大的特点在于，你觉得时效性会非常重要，但是你想想是吗？并不是，比如我只要确定他能做这件事，然后我后边慢慢做就可以了，我并不需要他一口气做完这件事，所以我们应当采用的是课程中，类似消息队列的方式来完成我们的需求，而不是使用线程池或者是异步编排的方式来完成这个需求。

#### Lua

**Lua 脚本**：把库存和一人一单做好判断，符合就直接返回成功。构建好订单发给rabbitMQ。

为了支持集群和维持原子性，lua脚本内操作的键必须在同一个哈希插槽内，因此需要使用前缀 + {voucherid}进行拼接，这样的两个键一定能在同一个哈西插槽内 HASHTAG。

一人一单的 set 存储太多用户会出现性能问题，生产环境里需要做出分片等优化，或者采用 zset

```lua
-- 0. 键列表
local stockPrefix = KEYS[1]

local orderPrefix = KEYS[2]
-- 1.参数列表
-- 1.1.优惠券id
local voucherId = ARGV[1]
-- 1.2.用户id
local userId = ARGV[2]
-- 1.3.订单id (为了使用消息队列将订单id传递到消息队列中)
--local orderId = ARGV[3]

-- 2.数据key
-- 2.1.库存key
local stockKey = stockPrefix .. "{" .. voucherId .."}"
-- 2.2.订单key
local orderKey = orderPrefix .. "{" .. voucherId .."}"

-- 3.脚本业务
--TODO 防止超卖 3.1.判断库存是否充足 get stockKey
if(tonumber(redis.call('get', stockKey)) <= 0) then
    -- 3.2.库存不足，返回1
    return 1
end
--TODO 一人一单 3.2.判断用户是否下单 SISMEMBER orderKey userId
if(redis.call('sismember', orderKey, userId) == 1) then
    -- 3.3.存在，说明是重复下单，返回2
    return 2
end
-- 3.4.扣库存 incrby stockKey -1
redis.call('incrby', stockKey, -1)
-- 3.5.下单（保存用户）sadd orderKey userId 没有orderKey则自动创建
redis.call('sadd', orderKey, userId)

-- 判定为有资格后面再创建订单，加到预扣订单id zset 跟订单hash里

return 0
```



#### 防止订单丢失（redis写预扣记录，加定时兜底补偿回滚）

> **预扣时记录事务状态**

1. 执行 Redis 预扣库存（如 `DECR`）时，**Lua脚本内同步写入一条预扣记录**：

```
HSET stock:lock:{order_id} item_id {item_id} quantity 1 status "pending" create_time {timestamp}
EXPIRE stock:lock:{order_id} 600  # 10分钟过期（防数据堆积）
```

2. 同时将 `order_id` 存入待补偿集合Zset（时间戳 value，订单id key）：

```
ZADD stock:recovery {timestamp} {order_id}
```

> **定时任务扫描补偿** 

- 启动定时任务（如每分钟），扫描 `stock:recovery` 中超时（如 >5min）的 `order_id`。
  - score < currentTime - 5分钟 的元素。

- 检查预扣订单状态（` hget stock:lock:{order_id} status `）：
  - LUA：检查状态：只能是 pending，然后 **回滚库存**（`INCR` 库存）并清理记录，并设置 status 为 failed，从 zset 里删除 orderid。
  - 只能是 pending。

#### MQ

MQ 这边本身支持 concurrency，不过用默认的 1 就好，我们手动配置好线程池的参数，然后提交给线程池。

##### 单消费者？

订单落库操作的核心瓶颈在数据库，每个消费者实例

- **优先选择单消费者 + 线程池模式：** 对于订单落库/回滚这类强事务、状态依赖、DB密集的操作，这是**更安全、更可控、更易于实现事务一致性和避免资源竞争**的方案。核心在于利用数据库连接池、本地事务、线程内顺序处理和基于订单ID的分区机制。
- **利用队列分片实现水平扩展：** 当单个消费者实例（即使有较大线程池）成为瓶颈时，通过创建多个分片队列并部署多个消费者实例（每个实例仍是单消费者+线程池模式）来水平扩展。关键在于**分片路由策略**和**确保同一订单的消息始终路由到同一队列并由同一消费者实例处理**。
- **务必实现幂等性：** 无论哪种模式，消息都可能重复传递。在落库/回滚逻辑中必须实现幂等（例如使用数据库唯一约束、乐观锁、状态机检查、记录消息ID等）。
- **完善的错误处理与监控：** 使用死信队列（DLX）处理反复失败的消息。监控 RabbitMQ 队列积压、消费者状态、线程池状态、数据库连接池状态和关键业务指标。



异步：时效性不是那么重要，因此使用 MQ 慢慢消化即可。

单机——轻量级异步任务编排：如并行调用多个本地接口、合并结果、链式处理。

- 高并发请求处理：通过线程池快速响应，避免阻塞主线程。
- 无需跨进程通信：所有操作在同一个 JVM 内完成。

**分布式——RabbitMQ**：

- 直连交换机 `hmdianping.direct` 负责接收和分发消息
- 队列 `direct.seckill.queue`  存储消息，是消息的“最终目的地”。
- 绑定关系：队列通过路由键 `direct.seckill` 绑定到交换机。分发消息的依据（地址）
- **参数注入**：
  - `Message`：原始消息对象（含元数据）。
  - `Channel`：RabbitMQ 通道，用于手动确认消息（ACK/NACK）。
  - `VoucherOrder`：消息体自动反序列化后的业务对象（需配置消息转换器，如 `Jackson2JsonMessageConverter`）。
- **手动ACK**：manual。业务处理成功调用 `basicAck`，告知 RabbitMQ 消息已被消费。
- **消息持久化**：发送者发消息时设置DeliveryMode为Persistent，同时队列创建时也要指定durable = true

- **生产者 confirm**：correlated。需要@PostConstruct配置回调函数，到达交换机和路由成功之后会触发回调。
- 重试机制：生产者发送消息的时候设置好消息头的 `retryCount = 0`，消费者检测 `retryCount <= 3`才消费。如果订单保存失败则抛异常，此时拒绝消息并重新入队，达到三次重试就将这条消息转发到延时队列中去。这样，配置延时队列的死信队列为我们的正常队列，等到消息的 ttl 到了就 Reject消息，消息就重新回到了“死信队列”，也就是消费者对应的对列中，确保了消息不丢失。

#### 数据库操作失败：回滚

• 使用乐观锁避免秒杀券超卖，同时提高并发。通过控制事务的粒度以及动态代理保证事务的有效性。使用 Lua 脚本解决秒杀券更新的原子性问题。 Redisson 分布式 锁解决集群环境下一人一单可能导致的并发安全问题。在 3000 并发测试中成功拦截 100%的重复请求

lua脚本对库存（string）执行了减1操作，对集合（set）进行了add操作

1. 指数退避重试机制（订单id唯一保证幂等性）
2. **补偿机制：最终一致性兜底**。如果重试到达一定次数就变成死信，死信监听器接受消息之后出发redis回滚，回滚的时候除了库存和购买者set，记得设置预扣订单状态为failed。在超时之前，提前完成回滚操作。

回滚的幂等性：可以加一个类似于分布式锁的操作，`SETNX rollback:<id> true` 成功才继续，如果 `rollback:<id>` 已存在，说明已经补偿过一次，就直接跳过。（TTL：回滚窗口+一段时间的安全期）

#### 请求堆积

MQ消息堆积：使用 CallerRunsPolicy()，一般就是nack太多或者是消费的太慢了，可以增加消费者数量，或者改成多线程消费，或者新开一个队列，将堆积的这些消息专门给消费者消费

| **方案**       | **实施难度** | **见效速度** | **适用阶段** | **成本** |
| -------------- | ------------ | ------------ | ------------ | -------- |
| 限流降级       | 低           | 立即         | 紧急处理     | 低       |
| 代码优化       | 中           | 1-3天        | 短期优化     | 低       |
| 数据库读写分离 | 高           | 1周          | 中期架构调整 | 中       |
| 微服务拆分     | 高           | 1-3个月      | 长期重构     | 高       |
| 消息队列削峰   | 中           | 3-7天        | 中期解耦     | 中       |

### 用户体感反馈：前端轮询接口查询订单状态

1. 活动开始/结束，token，参数非法
2. 判断资格，成功则返回“抢购成功，请耐心等待结果”避免用户重复点击。
3. 入队阶段：网络异常/MQ不可用，回滚 Redis，告知用户系统繁忙
4. 异步落库：成功可以推送一个站内信，失败就发送失败通知，同时后台记录好日志，做好异常落库工作。

那么问题来了，我们实现了上面的异步处理后，用户那边得到的结果是怎么样的呢？

用户点击了提交订单，收到了消息：您的订单已经提交成功。然后用户啥也没看见，也没有订单号，用户开始慌了，点到了自己的个人中心——已付款。发现居然没有订单！（因为可能还在队列中处理）

这样的话，用户可能马上就要开始投诉了！太不人性化了，我们不能只为了开发方便，舍弃了用户体验！

所以我们要改进一下，如何改进呢？其实很简单：

- 让前端在提交订单后，显示一个“排队中”（这里返回订单会包含orderid）。
- 同时，前端隔半秒请求一次 (检查用户和商品是否已经有订单) 的接口，如果得到订单已经处理完成的消息，页面跳转抢购成功。就是在后端单开一个查询接口，查询redis里预扣订单的状态：pending就返回pending，success/failed就返回remove的值。
  - 提交订单之后，设定状态为pending。
  - 下面两种状态最后都要删除 zset 的 order_id，表明已经处理完了，不要让定时任务对订单执行超时逻辑。
    - 如果mq监听器成功落库，则设定状态为success，删除 zset 的 order_id。
    - 如果mq监听器落库失败，设定状态为failed 然后回滚redis，删除 zset 的 order_id，将失败订单落库。


## <span id="like">点赞、排行榜</span>

**使用 ZSet 与 MySQL 的自定义排序实现点赞及排行榜，使用 Set 作为共同关注的解决方案。** 

使用Zset存储每个blog的点赞者id。 保证按照点赞时间（score）排序，并且点赞人不会重复。

Set 存储 用户关注的人，set 之间可以有intersect交集操作。

```java
//  idStr 是前5个点赞的用户的 id，由于MySQL查询的结果顺序不定，因此需要 手动使用 ORDER BY FIELD 指定 ID 字段 按照 idStr 的顺序
List<UserDTO> userDTOS = userService.query()
        .in("id", ids).last("ORDER BY FIELD(id," + idStr + ")").list()
        .stream()
        .map(user -> BeanUtil.copyProperties(user, UserDTO.class))
        .collect(Collectors.toList());
```

## <span id="timeline">分页查询 时间线</span>

**实现了基于推模式的关注内容推送，用基于时间戳的分页查询实现时间线功能。**  

优点：时效快，不用临时拉取

缺点：内存压力大，假设一个大V写信息，很多人关注他， 就会写很多分数据到粉丝那边去

我们需要记录每次操作的最后一条，然后从这个位置开始去读取数据

举个例子：我们从t1时刻开始，拿第一页数据，拿到了10~6，然后记录下当前最后一次拿取的记录，就是6，t2时刻发布了新的记录，此时这个11放到最顶上，但是不会影响我们之前记录的6，此时t3时刻来拿第二页，第二页这个时候拿数据，还是从6后一点的5去拿，就拿到了5-1的记录。我们这个地方可以采用sortedSet来做，可以进行范围查询，并且还可以记录当前获取数据时间戳最小值，就可以实现滚动分页了。

```java
public Result queryBlogOfFollow(Long max, Integer offset) {
    // 1.获取当前用户
    Long userId = UserHolder.getUser().getId();
    // 2.查询收件箱 ZREVRANGEBYSCORE key Max Min LIMIT offset count
    String key = FEED_KEY + userId;
    Set<ZSetOperations.TypedTuple<String>> typedTuples = stringRedisTemplate.opsForZSet()
            .reverseRangeByScoreWithScores(key, 0, max, offset, 2);
    // 3.非空判断
    if (typedTuples == null || typedTuples.isEmpty()) {
        return Result.ok();
    }
    // 4.解析数据：blogId、minTime（时间戳）、offset
    List<Long> ids = new ArrayList<>(typedTuples.size());
    long minTime = 0; // 2
    int os = 1; // 2
    for (ZSetOperations.TypedTuple<String> tuple : typedTuples) { // 5 4 4 2 2
        // 4.1.获取id
        ids.add(Long.valueOf(tuple.getValue()));
        // 4.2.获取分数(时间戳）
        long time = tuple.getScore().longValue();
        if(time == minTime){
            os++;
        }else{
            minTime = time;
            os = 1;
        }
    }
    os = minTime == max ? os : os + offset;
    // 5.根据id查询blog
    String idStr = StrUtil.join(",", ids);
    List<Blog> blogs = query().in("id", ids).last("ORDER BY FIELD(id," + idStr + ")").list();

    for (Blog blog : blogs) {
        setBlogUserInfo(blog);
        setBlogIsLike(blog);
    }
    // 6.封装并返回
    ScrollResult r = new ScrollResult();
    r.setList(blogs);
    r.setOffset(os);
    r.setMinTime(minTime);

    return Result.ok(r);
}
public class ScrollResult {
    private List<?> list;
    private Long minTime;
    private Integer offset;
    //minTime 上次查询的最小值 offset从上次查询最小值偏移的个数(上次最小值有多少个)
}
```

## <span id="bitmap">使用 Bitmap 签到</span>

```java
// 签到实际上就是将天数所在的位置 SETBIT，重点是记录连续签到天数
public Result signCount() {
    String key = USER_SIGN_KEY + userId + keySuffix;
    // 4.获取今天是本月的第几天
    int dayOfMonth = now.getDayOfMonth();
    List<Long> result = stringRedisTemplate.opsForValue().bitField(key,
            BitFieldSubCommands.create()
            .get(BitFieldSubCommands.BitFieldType.unsigned(dayOfMonth)).valueAt(0));
    if (result == null || result.isEmpty()) {return Result.ok(0);}
    Long num = result.getFirst();
    if (num == null || num == 0) {return Result.ok(0);}
    // 6.循环遍历
    int count = 0;
    while (true) {
        // 6.1.让这个数字与1做与运算，得到数字的最后一个bit位  // 判断这个bit位是否为0
        if ((num & 1) == 0) {
            break; // 如果为0，说明未签到，结束
        }else {
            // 如果不为0，说明已签到，计数器+1
            count++;
        }
        // 把数字右移一位，抛弃最后一个bit位，继续下一个bit位
        num >>>= 1;
    }
    return Result.ok(count);
}
```

# <span id="ha">高可用</span>

三大法宝：缓存预热、拆分服务、横向扩展机器。

挂了，重启服务之前拦截流量，并且再次做好缓存的预热工作。

## 流量削峰

Cache/Cache预热、MQ削峰、限流（RateLimiter）、熔断降级、静态资源CDN分发（ServerPush）、请求异步化/批量化。

## Redis 集群

在 Redis Cluster 中，**每个节点都承担了一部分类似于 Sentinel 的职责**：监控其他节点、检测故障、参与主从切换，因此在高可用机制上，**Redis Cluster 实现了去中心化的哨兵机制**。

🔹 **Redis Sentinel：集中式哨兵 + 单主多从** 

🔸 **Redis Cluster：去中心化哨兵 + 多主多从 + 分片 ** 

### Cluster（人人有哨兵当）

| 类型    | 说明                                             |
| ------- | ------------------------------------------------ |
| Master  | 负责处理写入和读取请求，持有部分哈希槽的数据     |
| Replica | 复制某个 Master 节点的数据，用于故障转移和高可用 |

所有节点通过 Gossip 协议周期性通信，维护一份**集群拓扑图（cluster state）**。

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725155747294.png" alt="分片集群" style="zoom:80%;" />

至少 6 个。3 个 Master 对应所属 3 个 Replica 节点。每台机器部署 **2 个 Redis 实例**（一个主、一个从）推荐使用 `redis-cli --cluster create` 命令时加上 `--replicas 1` 自动搭建。Replica 节点也会参与维护集群的拓扑，将某个节点PFAIL 的信息 上报给 Master ，但是只有 Master 才能参与投票某个节点是 FAIL。

| 功能/问题                     | Redis Cluster 使用的算法或机制                               |
| ----------------------------- | ------------------------------------------------------------ |
| Key 分片                      | 基于 CRC16 的 Hash Slot 分片机制（16384 个槽位）             |
| 节点状态传播/监控             | 所有节点之间都参与随机心跳检测：Gossip                       |
| 节点失败检测                  | 单节点超时: `PFAIL`  超过半数master认为 PFAIL: `FAIL`        |
| 主从复制 + 自动选主(failover) | Replicas 发起选举请求，先拿到半数master选票的成为新master，更新拓扑。 |
| 客户端请求路由                | MOVED / ASK 重定向机制，更新客户端这边的请求分片缓存。       |
| 数据一致性                    | 最终一致性（AP），不保证强一致                               |
| 数据重分片                    | 可以从 redis-cli 动态迁移所属插槽                            |

### Sentinel

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725154632354.png" alt="image-20210725154632354" style="zoom:80%;" />

| 方面       | Sentinel         | Cluster               |
| ---------- | ---------------- | --------------------- |
| 分片支持   | ❌（单主）        | ✅（自动分片）         |
| 数据一致性 | 弱一致性         | 最终一致性            |
| 高可用     | ✅                | ✅                     |
| 故障恢复   | 自动主从切换     | 自动主从切换+槽位迁移 |
| 扩容方式   | 手动添加主从节点 | 支持自动 resharding   |

## 限流、降级、缓存优化

### 限流策略（控制请求流量）

通过限制单位时间内的请求量，保护数据库不被突发流量压垮：

1. **令牌桶算法（Token Bucket）**
   - **原理**：以固定速率向桶中添加令牌，请求需获取令牌才能执行，否则被拒绝或等待。
   - **适用场景**：允许突发流量（如短时间内消耗积攒的令牌）。
   - **实现组件**：
     - Guava `RateLimiter`（单机）
     - Sentinel（分布式，支持QPS/并发数限流）
2. **滑动窗口算法（Sliding Window）**
   - **原理**：将时间分割为小窗口，统计最近N个窗口的请求量，避免固定窗口的临界突发问题。
   - **实现**：Redis + Lua脚本（分布式场景），通过`ZREMRANGEBYSCORE`删除旧时间戳，`ZCARD`统计当前请求数。
3. **漏桶算法（Leaky Bucket）**
   - **原理**：以恒定速率处理请求，超出桶容量的请求被丢弃。
   - **与令牌桶区别**：漏桶**平滑输出**，令牌桶**允许突发**。
4. **分布式限流**
   - **Redis + Lua**：原子操作计数，避免单点限流不均。
   - **Sentinel集群流控**：通过Token Server统一分配令牌。

### 降级策略（有损保核心）

当系统压力过大时，暂时牺牲非核心功能，确保主链路可用：

1. **读服务降级**
   - **返回兜底数据**：缓存失效时返回旧缓存或默认值（如库存显示“有货”）。
   - **强制读缓存**：关闭DB查询，仅允许读缓存（需提前预热热点数据）。
2. **写服务降级**
   - **异步写队列**：将写请求转入消息队列（如Kafka），异步消费（如订单创建）。
   - **关闭非核心写功能**：如电商大促时禁用评论、积分服务。
3. **多级降级开关**
   - **页面层降级**：前端隐藏非核心功能入口（如隐藏积分抵扣按钮）。
   - **应用层降级**：通过配置中心（如ZooKeeper、Nacos）动态切换降级策略。

### 缓存层优化（减少穿透）

1. **互斥锁（Mutex Lock）**
   - 缓存失效时，仅允许一个线程查询数据库，其他线程等待或读旧缓存。
   - **实现**：Redis `SETNX` 或分布式锁（Redisson）。
2. **缓存预热与更新**
   - **预热**：系统启动时加载热点数据到缓存。
   - **双缓存策略**：
     - 设置主备缓存（主缓存过期后，备份缓存仍可短暂提供服务）。
     - 示例：缓存A过期时间30分钟，缓存B过期时间60分钟。
3. **过期时间随机化**
   - 避免大量缓存同时失效：`过期时间 = 基础时间 + 随机值`。

## Redis 集群定时任务

### 分片？

> 秒杀的核心特征：
>
> - 商品数量有限（比如100件、1000件）
> - 短时间内请求量极大（可能达到几十万、上百万 QPS）
> - 并发冲突激烈，目标是“快、准、抢”

秒杀业务下，其实对单个商品的分片是个伪需求，因为商品绝对是限量的，库存少、访问量高，因此分片不是首要问题。秒杀本身库存确实很少，数据量小，Redis 完全放得下，Redis 单节点的性能（10万QPS级别）就足以支撑。

如果有成百上千的商品，那就按照商品 id 进行分片就行了。

### 键值设计

| Key 类型 | 用途                 | Key 名称（下划线是动态变化的，{} 保证在同一个节点里） |
| -------- | -------------------- | ----------------------------------------------------- |
| String   | 商品库存数量         | seckill:stock:{**<u>product_id</u>**}                 |
| Set      | 购买此商品的用户id   | seckill:buyers:{**<u>product_id</u>**}                |
| Hash     | 订单信息以及状态     | seckill:order:{**<u>product_id</u>**}:<u>order_id</u> |
| ZSet     | 存预扣订单id和时间戳 | seckill:recovery:{**<u>product_id</u>**}              |
| Set      | 秒杀商品id           | seckill:products                                      |

### Redis 定时 Lua 脚本

> Lua 脚本(recovery.lua)

```lua
-- Lua脚本保证原子性（单节点）
-- KEYS[1]: 商品id集合key "seckill:products"
-- KEYS[2]: 预扣zset的前缀 "seckill:recovery:"
-- KEYS[3]: 订单hash前缀  "seckill:order:"
-- KEYS[4]: 商品库存前缀   "seckill:stock:"
-- KEYS[5]: 购买者集合前缀 "seckill:buyers:"

-- ARGV[1]: timestamp

local now = tonumber(ARGV[1])

-- 获取所有商品ID
local product_ids = redis.call('SMEMBERS', KEYS[1])

local expired_total = 0;

for i, product_id in ipairs(product_ids) do
    -- 处理商品最多
    local recoveryKey = KEYS[2] .. '{' .. product_id .. '}'

    -- 5分钟订单过期
    local expired = redis.call('ZRANGEBYSCORE', 
        recoveryKey, 
        '-inf', 
        now - 5*60*1000
    )
    local count = #expired;
    expired_total = expired_total + count;
    -- 过期回滚逻辑
    for _, orderId in ipairs(expired) do
        local orderKey = KEYS[3] .. '{' .. product_id .. '}' .. orderId
        local status = redis.call('HGET', orderKey, 'status')

        -- 只处理未完成的订单
        if status == 'pending' then
            local buyerId = redis.call('HGET', orderKey, 'buyer_id')

            -- 更新订单状态
            redis.call('HSET', orderKey, 'status', 'failed')

            -- 库存回滚
            redis.call('INCR', KEYS[4] .. '{' .. product_id .. '}')

            -- 移除购买者
            redis.call('SREM', KEYS[5] .. '{' .. product_id .. '}', buyerId)
        end

        -- 从zset移除
        redis.call('ZREM', recoveryKey, orderId)
    end
    
end

return expired_total
```

> XXL-Job 

```xml
<!-- 引入依赖 -->
<!-- Spring Data Redis Lettuce -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!-- XXL-JOB Executor -->
<dependency>
  <groupId>com.xuxueli</groupId>
  <artifactId>xxl-job-core</artifactId>
  <version>2.3.0</version>
</dependency>

```



## RabbitMQ 高可用

[rabbitmq高可用集群搭建-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2448992)

[如何保证消息队列（RabbitMQ）的高可用？ - 非学无以广才 - SegmentFault 思否](https://segmentfault.com/a/1190000023008259)

RabbitMQ有三种模式：单机模式、普通集群模式、镜像集群模式。

### 单机模式

单机模式就是处于一种demo模式、一般就是自己本地搞一个玩玩，生产环境没有那个哥们会使用单机模式的。

### Classic Queue 

#### 普通集群模式（无高可用）

![RabbitMQ普通集群.jpg](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/bVbItWg)
如上图是普通集群模式
1、RabbitMQ在多台服务器启动实例、每台服务器一个实例、当你创建queue时、queue（元数据+具体数据）只会落在一台RabbitMQ实例上、但是集群中每个实例都会同步queue的元数据（元数据：真实数据的描述如具体位置等)。
2、当用户消费时如果连接的是另外一个实例，当前实例会根据同步的元数据找到具体的数据所在的实例从其上把具体数据拉过来消费。

##### 优缺点

这种方式的缺点很明显，没有做到所谓的分布式、只是一个普通的集群。这种方式在消费数据时要么随机选择一个实例拉去数据、要么固定连接那个queue所在的实例来拉取数据，前者导致一次实例见拉取数据的开销、而后在会导致单实例性能的瓶颈。

而且如果存放数据的queue的实例宕机了、会导致其它实例无法从该实例来拉取数据了，如果你开启了RabbitMQ的持久化功能，消息不一定会丢失，但是得等待这个实例重启后才能继续从该queue拉取数据。

所以总的来说这事就比较尴尬了，就完全没有所谓的高可用一说了，这个方案主要的目的是**提高吞吐量**的，就是说让集群中的多个节点来服务某个queue的读写操作。

##### 使用

- 多台机器（如 node1、node2、node3）
- 每台安装 RabbitMQ，Erlang cookie 一致（`/var/lib/rabbitmq/.erlang.cookie`）
- 主机名互通 + 开放端口（5672、15672、25672）

```bash
# 所有节点配置 hostnames
sudo hostnamectl set-hostname node1
# 启动 node1 为独立节点
sudo systemctl start rabbitmq-server

# node2、node3 加入集群
# 停止 rabbitmq app
rabbitmqctl stop_app
# 加入集群
rabbitmqctl join_cluster rabbit@node1
# 启动
rabbitmqctl start_app

# 设置磁盘节点（持久元数据）或内存节点 (可选)
rabbitmqctl set_cluster_name my-cluster
```

#### 镜像集群模式

> 建立在普通集群基础上，**通过策略（POLICY）配置实现某些队列的多副本同步**，具备**高可用性**。

![RabbitMQ镜像集群.jpg](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/bVbIt3o)

##### 优缺点

这种集群模式真正达到了RabbitMQ高可用性，和普通集群不一样的是你创建的queue不管是元数据还是里面的具体消息都会存在于所有的实例上。每次写消息时都会把消息同步到每个节点的queue中去。

这种方式的优点在于，你任何一个节点宕机了、都没事儿，别的节点都还可以正常使用。

**低吞吐**：性能开销，每条消息都要复制到所有 mirror 节点，且同步 ack。

**性能瓶颈点**：

- 多副本同步（默认是同步模式）
- 网络延迟和磁盘写入同步放大

这种模式没有扩展性可言，如果你某个queue的负载很高，你加机器，新增的机器也包含了这个queue的所有数据，并没有办法线性扩展你的queue.

##### 使用

这里在多说下如何开启镜像集群模式？其实开启很简单在RabbitMQ的管理控制台，新增一条策略、这个策略就是开启开启镜像集群模式策略、指定的时候可以指定数据同步到所有的节点，也可以要求同步到指定的节点数量，之后你在创建queue时使用这个策略、就会在动降数据同步到其它节点上去了。

![RabbitMQ开启镜像集群.jpg](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/bVbIt6Q)

> 额外配置步骤

```bash
# 所有队列镜像到所有节点
rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'

# 或者镜像到 2 个节点
rabbitmqctl set_policy ha-two "^" '{"ha-mode":"exactly","ha-params":2,"ha-sync-mode":"automatic"}'
```

`ha-mode`：

- `all`: 所有节点
- `exactly`: 指定数量
- `nodes`: 指定节点列表

`ha-sync-mode`：

- `automatic`：自动同步（推荐）
- `manual`：需要手动触发

### Quorum（强一致高可用）

#### 优缺点

**推荐 quorum 队列**，维护简单、强一致。**Raft 一致性协议**保证数据一致，自动选主，推荐用于**持久队列**。

**性能瓶颈点**：

- 每次写入都要经过 Raft 协议投票（多数副本响应）
- 磁盘写入压力更高（写放大现象）

**优势**：在高并发或节点失效情况下，**延迟更稳定，不易脑裂**。

#### 使用

同上，先将节点加入集群。

然后声明队列为 quorum 队列。

```java
Map<String, Object> args = new HashMap<>();
args.put("x-queue-type", "quorum");
channel.queueDeclare("my-quorum-queue", true, false, false, args);

// 或者在控制台生命也可以
rabbitmqadmin declare queue name=my-quorum-queue durable=true arguments='{"x-queue-type":"quorum"}'
```

> 无需设置 policy。
>
> 副本数量默认是奇数个（3个节点，2副本才有效）。
>
> 自动高可用、自动 leader 选举。

### Stream（高吞吐）

实时日志、事件流、重放需求`stream` ✅

> **RabbitMQ Stream 是官方推出的用于高吞吐量、日志流式、事件驱动架构的队列类型**，支持类 Kafka 的功能。

- 单个节点可支撑**百万级消息/秒**
- 支持**消息批量拉取、零拷贝传输**
- 自带**消费位置游标 offset**，可从任意 offset 重播（像 Kafka 一样）
- 持久化存储，**文件页缓存结构**设计，写入吞吐极高

| 问题点                                              | 详细说明                                                     |
| --------------------------------------------------- | ------------------------------------------------------------ |
| ❌ **没有强一致性机制**                              | Stream 队列目前**不使用 Raft 或类似协议**，多副本复制是**异步的、最终一致**，在节点宕机或网络抖动下可能存在短暂数据差异或消息丢失风险。 |
| ❌ **无主从控制、不可强制顺序消费**                  | 消费者是 Pull 模式，自主决定 offset，**无法全局强制顺序消费**，对于订单这类“按订单处理”的逻辑不适合。 |
| ❌ **ack 模型为 offset-based，不是 per-message ack** | Stream 使用 Kafka 式的 offset ack，不支持 RabbitMQ 的 fine-grained per-message ack，这意味着某个消息处理失败无法精确重试，只能靠 offset 逻辑回溯，**处理失败的补偿逻辑复杂**。 |
| ⚠️ **消费语义是 at-least-once**（可能重复）          | Stream 消费语义是默认 at-least-once，需要业务实现幂等，不适合订单“**只能处理一次**”的场景。 |
| ❌ **没有死信机制、延迟队列机制不完善**              | 与 classic/quorum 队列不同，目前 Stream 不直接支持 DLX、TTL、延迟消息等机制，需要外部管理。 |
| ⚠️ **客户端 SDK 限制**                               | Stream 使用的是 RabbitMQ 的 **专用客户端**（非 AMQP 0-9-1），不兼容已有 RabbitMQ 生态（如 Spring AMQP、RabbitTemplate），上手成本较高。 |

## RabbitMQ 吞吐

### 多 Quorum 队列 + Sharding（推荐）

> QUORUM 队列
>
> - 每个队列 = 一个独立的 Raft 副本组
> - 保证**消息写入的强一致性**
>   - 写入前必须得到多数副本的确认（`N/2 + 1`）
>   - 保障即使挂掉节点也**不会丢消息**
> - 但每个队列只有一个 leader 能处理写入请求
> - 单个 quorum 队列 = 单 Leader 复制 + 单写通道
> - 即使你加了很多 RabbitMQ 节点，只要这个队列的 Leader 没变，写操作都还在一台机器上处理 →**无法横向扩容**。

分片：

- 创建多个 quorum 队列（如 `order-queue-0`, `order-queue-1`, …）
- 每个队列在集群中独立选主，分布在不同节点。每个队列都可以写入、复制、持久化，吞吐量接近线性提升。
- 各个 leader **并行处理写入、复制、确认** ➜ 实现高吞吐

| 队列名         | 节点分布 | 被选为 leader 的节点 |
| -------------- | -------- | -------------------- |
| order-events-0 | A、B、C  | A                    |
| order-events-1 | A、B、C  | B                    |
| order-events-2 | A、B、C  | C                    |

> 按照 `orderId` 或 `userId` 进行 hash 分片，分别写入多个 quorum 队列并发处理。

```
order-events-0
order-events-1
...
order-events-N
```

发送时根据 `orderId % N` 路由到不同队列，每个队列独立存在、独立复制、独立 leader，大幅提升吞吐。

```java
String queueName = "order-events-" + (orderId % SHARD_COUNT);
channel.basicPublish("", queueName, persistentProps, orderBytes);
```

### 生产端优化（批量发送 + confirm）

- 使用 publisher confirm（`channel.confirmSelect()`）
- 开启异步 confirm（通过 `ConfirmListener` 异步回调）
- 发送时批量处理，减少网络/磁盘压力

### 消费端优化（多线程 + 批量 ack）

- 使用线程池并发消费不同的 Quorum 队列
- 消费逻辑异步化，确保处理速度跟得上发送速度
- 使用 `channel.basicAck(tag, true)` 批量 ack

### 将只读订单归档到 Stream Queue（冷热分离）

对于超过一定时间的订单（如 > 5 分钟），可归档到 **Stream 队列**：

- 优点：Stream 吞吐超高、支持 offset 重播
- 用途：可供日志审计、BI 分析、客服查询
- Quorum 队列只保存“活跃订单”，减轻主队列压力





# 分布式锁≠流量削峰

不，光开了分布式锁并不会减少外部打过来的请求数——它只是保证「同一时刻只有一个请求能拿到锁去真正执行业务」，但所有其他请求还是会去 Redis 调用一次 `tryLock()`，只不过拿不到锁而已。所以，分布式锁并不能从源头上降请求量或 Redis 的流量。

如果你想真正减轻 Redis 压力、提前拦截大批量的并发请求，可以考虑在业务层或网关层做以下几种优化：

1. **本地预热+内存拦截**
   - 把库存、限流配额等关键信息缓存在应用内存（比如 `ConcurrentHashMap`、Guava Cache、Caffeine）里，先在本地判断 “库存是否用完” 或 “是否超过限流阈值”，
   - 只有本地判断通过的请求才继续走远程 Redis 锁或库存扣减，未通过的直接快速返回。
   - 这样在高并发时，大部分请求都被本地拦截掉，Redis 压力会大幅下降。
2. **网关限流/熔断**
   - 在 Nginx 或 API 网关（如 Kong、Zuul、Spring Cloud Gateway）层面，给热点接口打限流，使用漏桶、令牌桶算法，把峰值流量削峰；
   - 超出限流阈值的请求直接在网关被拒绝，不会转到业务后端。
3. **消息队列异步排队**
   - 接口层接到请求后，迅速把请求放到 MQ（RabbitMQ、Kafka、RocketMQ），立即返回“排队中”给用户；
   - 后端消费者单线程或限并发地从队列里拉取消息、串行执行业务、更新库存、入库；
   - 这样除了一条队列写入，后续几乎零并发冲突，也不会给 Redis 带来峰值压力。
4. **Lua 脚本原子化合并操作**
   - 把库存检查、扣减、锁定、消息入队等逻辑写到一段 Redis Lua 脚本里，一次网络 RTT 就能完成多步校验和更新，虽然还是落到 Redis，但网络交互次数最低。
   - 适合逻辑较简单的场景。
5. **多级限流**
   - 前端做点击防抖/禁用按钮，避免用户重复快速点击；
   - 服务端在 Controller 层用注解（如 `@RateLimiter`）做粗粒度限流；
   - 再到业务层做精确限流、库存预判。

- **分布式锁能防止并发冲突**，但它并不是流量削峰的手段，**并不会减少对 Redis 的调用次数**；
- 如果想从源头上削峰，必须借助 **本地缓存预判、网关限流、消息队列排队** 等手段，才能真正降低对 Redis（或后端服务）的并发压力。

# CAP

**CAP 理论**，也称为 **Brewer 定理**，是分布式系统设计中的一个基本原则。它指出，在一个分布式存储系统中，**一致性（Consistency）**、**可用性（Availability）**、**分区容错性（Partition Tolerance）** 三者不可兼得，只能同时满足其中的两项。下面分点说明。

- **CAP 不告诉你选哪两个**，而是帮助你在设计时**明确取舍**：
  - 如果业务对**强一致性**要求高（如支付、库存扣减），在分区时宁可牺牲可用性 → 选 **CP**。
  - 如果业务对**可用性/延迟**要求高（如社交、日志收集），在分区时宁可牺牲一致性 → 选 **AP**。
- 在**真实业务中**，常结合**最终一致性**、**多副本延迟补偿**、**幂等重试**等手段，综合平衡一致性与可用性。

了解 CAP 理论，有助于你在做分布式架构选型时，**根据业务特性、容忍度和运维成本**做出最合适的权衡。

------

## 三个核心

1. **一致性（Consistency）**
    所有节点在同一时间看到的数据是一致的。一次写入操作完成后，所有后续的读取都能立刻返回最新的写入值。
2. **可用性（Availability）**
    每个请求都会在有限时间内得到非错误响应（不保证是最新数据，但保证服务不宕机、总有回应）。
3. **分区容错性（Partition Tolerance）**
    系统能够容忍网络分区（节点或网络断开）而继续提供服务。即使部分节点之间无法通信，系统仍能保持一致性或可用性之一。

项目当中，CP：分布式锁判定一人一单，强一致性    AP：最终一致性，redis预扣。redis集群。

------

## “不可兼得” 的含义

CAP 告诉我们：“当系统发生网络分区时（P发生），你必须在**一致性**（C）和**可用性**（A）之间做选择，**不能同时满足**。”

在真实的分布式环境中，**网络分区故障（网络抖动、链路中断）几乎不可避免**。因此，**要构建一个健壮的分布式系统，必须选择在网络分区时牺牲“一致性”或“可用性”中的一项**：

- **CP 系统（Consistency + Partition Tolerance）**：在网络分区时，放弃可用性，保证一致性。常见于金融交易系统。
- **AP 系统（Availability + Partition Tolerance）**：在网络分区时，放弃一致性，保证可用性。常见于社交媒体、缓存系统。
- **CA 系统（Consistency + Availability）**：保证一致性和可用性，但无法容忍网络分区。严格来说，分布式环境中无法真正做到 CA，因为分区总会发生。

| 模型   | 分区发生时的策略                                             | 场景举例                             |
| ------ | ------------------------------------------------------------ | ------------------------------------ |
| **CP** | 停止提供服务或延迟响应，直到网络恢复并保证数据一致           | ZooKeeper、HBase（强一致读写）       |
| **AP** | 继续提供服务，但有可能读到老数据，待网络恢复后数据再最终一致 | Cassandra、Redis Cluster（异步复制） |
| **CA** | 不适用于跨网络数据中心的分布式系统；只在单机或无网络分区场景满足 | 传统关系型数据库（单实例）           |

------

## 实际应用

- **Cassandra（AP）**
  - 选用最终一致性模型，允许在分区时继续读写，性能和可用性高，但短期内可能读不到最新写入。
- **MongoDB（CP 可调）**
  - 默认主从复制，主节点写入保证一致性；分区时次节点不可写（牺牲可用性），保证读写一致。
- **Redis Sentinel/ZK（CP）**
  - 在主备切换时，会阻止客户端写入，直到选举出新主节点，保证强一致。

# [线程池调优](https://scatteredream.github.io/2024/11/30/juc-in-one/#connectpool)



# 压力测试

首先设置为10000个并发请求的情况下，运行JMeter，结果首先出现了大量的报错，10000个请求中98%的请求都直接失败了。SpringBoot内置的Tomcat最大并发数默认值为200，对于10000的并发，单机服务实在是力不从心。当然，你可以修改这里的并发数设置（`server.tomcat.max_threads`），但是你的小机器仍然可能会扛不住。

```properties
server.tomcat.max-threads=10000
server.tomcat.max-connections=10000
```

不使用缓存的情况下，吞吐量为668个请求每秒：

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1599136383601-image.png)

使用缓存的情况下，吞吐量为2177个请求每秒：

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1599136392397-image.png)

------

在1c4g1m带宽的云数据库上，**设置商品数量5000个，同时并发访问10000次**。数据库直接被打挂：

Caused by: java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30073ms. 

表明 HikariCP 连接池在 30073 毫秒（约 30 秒）内未能获取到可用的数据库连接，从而导致请求超时。

连接池配置不合理/数据库负载过高 [解决方法](https://blog.csdn.net/qq_32495261/article/details/117689255)

![image](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/5718317-1ce7a79e5b9c9e93.png)


我们改成1000个线程并发，商品库存为500个，**使用常规的非异步下单接口**，吞吐量37.3：

![image](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/5718317-b90a65d09f68c8f7.png)

对比1000个线程并发，**使用异步订单接口**：吞吐量 600

![image](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/5718317-f5f703025a2fe0f0.png)

RabbitMQ：

![image](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/5718317-440730e106c71864.png)

![image](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/5718317-440730e106c71864.png)



库存耗尽的时候的日志快照：

![image-20250606134815284](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250606134815284.png)
