---
name: redis-advanced
title: Redis 分布式应用
date: 2024-08-14
tags: 
- redis
- 集群
- 缓存
- 哈希插槽
categories: redis
---



# 分布式缓存

-- 基于Redis集群解决单机Redis存在的问题



单机的Redis存在四大问题：

![image-20210725144240631](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725144240631.png)





## 持久化

Redis有两种持久化方案：

- RDB持久化
- AOF持久化



### RDB 持久化

RDB全称Redis Database Backup file（Redis数据备份文件），也被叫做Redis数据快照（<mark>SNAPSHOT<mark>）。简单来说就是把内存中的所有数据都记录到磁盘中。当Redis实例故障重启后，从磁盘读取快照文件，恢复数据。快照文件称为RDB文件，默认是保存在当前运行目录。

执行时机

RDB持久化在四种情况下会执行：

- 执行save命令
- 执行bgsave命令
- Redis停机时
- 触发RDB条件时



**1）save命令**

执行下面的命令，可以立即执行一次RDB：

![image-20210725144536958](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725144536958.png)

save命令会导致主进程执行RDB，这个过程中其它所有命令都会被阻塞。只有在数据迁移时可能用到。

**2）bgsave命令**

下面的命令可以异步执行RDB：

![image-20210725144725943](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725144725943.png)

这个命令执行后会开启独立进程完成RDB，主进程可以持续处理用户请求，不受影响。

**3）停机时**

Redis停机时会执行一次save命令，实现RDB持久化。

**4）触发RDB条件**

Redis内部有触发RDB的机制，可以在redis.conf文件中找到，格式如下：

```properties
# 900秒内，如果至少有1个key发生变化，则自动执行bgsave ， 如果是save "" 则表示:禁用RDB
save 900 1  
save 300 10  
save 60 10000 
```

RDB的其它配置也可以在redis.conf文件中设置：

```properties
# 是否压缩 ,建议不开启，压缩也会消耗cpu，磁盘不值钱
rdbcompression yes
# RDB文件名称
dbfilename dump.rdb  
# 文件保存的路径目录
dir ./ 
```

#### RDB 原理

bgsave开始时会 **fork** 主进程得到子进程，子进程共享主进程的内存数据。完成fork后读取内存数据并写入 RDB 文件。

fork采用的是copy-on-write技术：写入时复制

- 当主进程执行读操作时，访问共享内存；
- 当主进程执行写操作时，则会拷贝一份数据，执行写操作。

![image-20210725151319695](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725151319695.png)

如果在RDB过程中，修改了所有的数据，那么redis占用内存将直接<mark>翻倍<mark>

#### 小结

##### RDB方式bgsave的基本流程？

- fork主进程得到一个子进程，共享内存空间
- 子进程读取内存数据并写入新的RDB文件
- 用新RDB文件替换旧的RDB文件

##### RDB会在什么时候执行？save 60 1000代表什么含义？

- 默认是服务停止时
- 代表60秒内至少执行1000次修改则触发RDB

##### RDB的缺点？

- RDB执行间隔时间长，两次RDB之间写入数据有丢失的风险
- fork子进程、压缩、写出RDB文件都比较耗时，仅仅修改save命令的第一个参数不会提高RDB执行的效率



### AOF持久化

#### AOF 原理

AOF全称为<mark>Append Only File<mark>（追加文件）。Redis处理的<mark>每一个写命令<mark>都会记录在AOF文件，可以看做是命令日志文件。(binlog in MySQL)。AOF是执行命令之后写入，binlog是执行之前写入

- 避免额外的检查开销，AOF 记录日志不会对命令进行语法检查；
- 在命令执行完之后再记录，不会阻塞当前的命令执行。
- 如果刚执行完命令 Redis 就宕机会导致对应的修改丢失；
- 可能会阻塞后续其他命令的执行（AOF 记录日志是在 Redis 主线程中进行的）。

![image-20210725151543640](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725151543640.png)



#### AOF 配置

AOF默认是关闭的，需要修改redis.conf配置文件来开启AOF：

```properties
# 是否开启AOF功能，默认是no
appendonly yes
# AOF文件的名称
appendfilename "appendonly.aof"
```

AOF的命令记录的频率也可以通过redis.conf文件来配：

```properties
# 表示每执行一次写命令，立即记录到AOF文件
appendfsync always 
# 写命令执行完先放入AOF缓冲区，然后表示每隔1秒将缓冲区数据写到AOF文件，是默认方案
appendfsync everysec 
# 写命令执行完先放入AOF缓冲区，由操作系统决定何时将缓冲区内容写回磁盘
appendfsync no
```

三种策略对比：

![image-20210725151654046](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725151654046.png)

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241117220311689.png" alt="image-20241117220311689" style="zoom: 50%;" />

#### AOF 步骤

**命令追加（append）**：所有的写命令会追加到 AOF 缓冲区中。

**文件写入（write）**：将 AOF 缓冲区的数据写入到 AOF 文件中。这一步需要调用`write`函数（系统调用），`write`将数据写入到了系统内核缓冲区之后直接返回了（延迟写）。注意！！！此时并没有同步到磁盘。写入系统内核缓冲区之后直接返回（仅仅是写到缓冲区），不会立即同步到硬盘。虽然提高了效率，但也带来了数据丢失的风险。同步硬盘操作通常依赖于系统调度机制，Linux 内核通常为 30s 同步一次，具体值取决于写出的数据量和 I/O 缓冲区的状态。

**文件同步（fsync）**：AOF 缓冲区根据对应的持久化方式（ `fsync` 策略）向硬盘做同步操作。这一步需要调用 `fsync` 函数（系统调用）， `fsync` 针对单个文件操作，对其进行强制硬盘同步，`fsync` 将阻塞直到写入磁盘完成后返回，保证了数据持久化。强制刷新系统内核缓冲区（同步到到磁盘），确保写磁盘操作结束才会返回。

**文件重写（rewrite）**：随着 AOF 文件越来越大，需要定期对 AOF 文件进行重写，达到压缩的目的。

**重启加载（load）**：当 Redis 重启时，可以加载 AOF 文件进行数据恢复。

![](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/aof-work-process.png)

从 Redis 7 开始，Redis 使用了 **Multi Part AOF** 机制。顾名思义，Multi Part AOF 就是将原来的单个 AOF 文件拆分成多个 AOF 文件。在 Multi Part AOF 中，AOF 文件被分为三种类型，分别为：

- BASE：表示基础 AOF 文件，它一般由子进程通过重写产生，该文件最多只有一个。
- INCR：表示增量 AOF 文件，它一般会在 AOFRW 开始执行时被创建，该文件可能存在多个。
- HISTORY：表示历史 AOF 文件，它由 BASE 和 INCR AOF 变化而来，每次 AOFRW 成功完成时，本次 AOFRW 之前对应的 BASE 和 INCR AOF 都将变为 HISTORY，HISTORY 类型的 AOF 会被 Redis 自动删除。

#### AOF 文件重写

因为是记录命令，AOF文件会比RDB文件大的多。而且AOF会记录对同一个key的多次写操作，但只有最后一次写操作才有意义。通过执行`bgrewriteaof`命令，可以让AOF文件执行重写功能，用最少的命令达到相同效果。

![image-20210725151729118](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725151729118.png)

如图，AOF原本有三个命令，但是`set num 123 和 set num 666`都是对num的操作，第二次会覆盖第一次的值，因此第一个命令记录下来没有意义。

所以重写命令后，AOF文件内容就是：`mset name jack num 666`

Redis也会在触发阈值时自动去重写AOF文件。阈值也可以在redis.conf中配置：

1. 增长百分比阈值
2. 文件体积大小阈值

```properties
# AOF文件比上次文件 增长超过多少百分比则触发重写
auto-aof-rewrite-percentage 100
# AOF文件体积最小多大以上才触发重写 
auto-aof-rewrite-min-size 64mb 
```

#### [AOF 校验机制了解吗？](#aof-校验机制了解吗)

AOF 校验机制是 Redis 在启动时对 AOF 文件进行检查，以判断文件是否完整，是否有损坏或者丢失的数据。这个机制的原理其实非常简单，就是通过使用一种叫做 **校验和（checksum）** 的数字来验证 AOF 文件。这个校验和是通过对整个 AOF 文件内容进行 CRC64 算法计算得出的数字。如果文件内容发生了变化，那么校验和也会随之改变。因此，Redis 在启动时会比较计算出的校验和与文件末尾保存的校验和（计算的时候会把最后一行保存校验和的内容给忽略点），从而判断 AOF 文件是否完整。如果发现文件有问题，Redis 就会拒绝启动并提供相应的错误信息。AOF 校验机制十分简单有效，可以提高 Redis 数据的可靠性。

------

著作权归JavaGuide(javaguide.cn)所有 基于MIT协议 原文链接：https://javaguide.cn/database/redis/redis-persistence.html

### RDB与AOF对比

RDB和AOF各有自己的优缺点，如果对数据安全性要求较高，在实际开发中往往会**结合**两者来使用。

- RDB 文件是以特定的二进制格式保存的，并且在 Redis 版本演进中有多个版本的 RDB，所以存在老版本的 Redis 服务不兼容新版本的 RDB 格式的问题。
- AOF 以一种易于理解和解析的格式包含所有操作的日志。你可以轻松地导出 AOF 文件进行分析，你也可以直接操作 AOF 文件来解决一些问题。比如，如果执行`FLUSHALL`命令意外地刷新了所有内容后，只要 AOF 文件没有被重写，删除最新命令并重启即可恢复之前的状态。

![image-20210725151940515](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725151940515.png)

Redis 7.0 版本之前，如果在重写期间有写入命令，AOF 可能会使用大量内存，重写期间到达的所有写入命令都会写入磁盘两次。

Redis 7.0 版本之后，AOF 重写机制得到了优化改进。具体方法是采用 base（全量数据）+inc（增量数据）独立文件存储的方式，彻底解决内存和 IO 资源的浪费，同时也支持对历史 AOF 文件的保存管理，结合 AOF 文件中的时间信息还可以实现 PITR 按时间点恢复（阿里云企业版 Tair 已支持），这进一步增强了 Redis 的数据可靠性，满足用户数据回档等需求。

[从Redis7.0发布看Redis的过去与未来 (qq.com)](https://mp.weixin.qq.com/s/RnoPPL7jiFSKkx3G4p57Pg) 



## Master/Slave 集群：高并发读

Redis 通过 Replica/Slave 从节点机制 完成主从复制与读写分离

### 搭建主从架构

主从复制：单节点Redis的并发能力是有上限的，要进一步提高Redis的并发能力，就需要搭建主从集群，实现读写分离。

![image-20210725152037611](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725152037611.png)

> 步骤

```sh
# 创建三个文件夹，将redis.conf复制过去
mkdir 7001 7002 7003
cp redis-6.2.6/redis.conf 7001
cp redis-6.2.6/redis.conf 7002
cp redis-6.2.6/redis.conf 7003

# 更改redis.conf中的port、dir
sed -i -e 's/6379/7001/g' -e 's/dir .\//dir \/root\/redissetup\/7001\//g' 7001/redis.conf
sed -i -e 's/6379/7002/g' -e 's/dir .\//dir \/root\/redissetup\/7002\//g' 7002/redis.conf
sed -i -e 's/6379/7003/g' -e 's/dir .\//dir \/root\/redissetup\/7003\//g' 7003/redis.conf
# 添加主从配置ip
sed -i '1a replica-announce-ip localhost' 7001/redis.conf
sed -i '1a replica-announce-ip localhost' 7002/redis.conf
sed -i '1a replica-announce-ip localhost' 7003/redis.conf
# 添加主从配置端口号 和 主节点的密码
sed -i '1i slaveof 127.0.0.1 7001' 7002/redis.conf
sed -i '2i masterauth 123321' 7002/redis.conf
sed -i '1i slaveof 127.0.0.1 7001' 7003/redis.conf
sed -i '2i masterauth 123321' 7003/redis.conf


# 启动redis-server
redis-server -p 7001/redis.conf
redis-server -p 7002/redis.conf
redis-server -p 7003/redis.conf

# 启动命令行
redis-cli -p 7001
redis-cli -p 7002
redis-cli -p 7003

INFO replication #显示主从信息
```



### 主从同步原理

#### 全量同步

主从第一次建立连接时，会执行**全量同步**，将master节点的所有数据都拷贝给slave节点，流程：

![image-20210725152222497](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725152222497.png)



这里有一个问题，master如何得知slave是第一次来连接呢？？

有几个概念，可以作为判断依据：

- **Replication Id**：简称replid，是数据集的标记，id一致则说明是同一数据集。每一个master都有唯一的replid，slave则会继承master节点的replid<mark>（表示主从关系）<mark> 
- **offset**：偏移量，随着记录在repl_backlog中的数据增多而逐渐增大。slave完成同步时也会记录当前同步的offset。如果slave的offset小于master的offset，说明slave数据落后于master，需要更新。<mark>（表示数据版本）<mark>

因此slave做数据同步，必须向master声明自己的replication id 和offset，master才可以判断到底需要同步哪些数据。



因为slave原本也是一个master，有自己的replid和offset，当第一次变成slave，与master建立连接时，发送的replid和offset是自己的replid和offset。

master判断发现slave发送来的replid与自己的不一致，说明这是一个全新的slave，就知道要做全量同步了。

master会将自己的replid和offset都发送给这个slave，slave保存这些信息。以后slave的replid就与master一致了。

因此，**master判断一个节点是否是第一次同步的依据，就是看replid是否一致**。

如图：AOF + RDB

![image-20210725152700914](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725152700914.png)

完整流程描述：

1. slave节点请求同步，发送replid + offset
2. master节点判断replid，发现不一致，则进行全量同步返回master的replid + offset
   - master将完整内存数据生成RDB（bgsave）发送RDB到slave
   - slave清空本地数据，加载master的RDB
   - master将RDB期间的命令记录在repl_backlog，并持续将repl_backlog的命令发送给slave

3. master节点判断replid，发现一致，并且offset较小
   - 去repl_backlog中取得offset之后的数据，将repl_backlog的命令发送给slave

4. slave执行接收到的命令，保持与master之间的同步

#### 增量同步

全量同步需要先做RDB，然后将RDB文件通过网络传输给slave，成本太高了。因此除了第一次做全量同步，其它大多数时候slave与master都是做**增量同步**

什么是增量同步？就是只更新slave与master存在差异的部分数据。如图：

![image-20210725153201086](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725153201086.png)

那么master怎么知道slave与自己的数据差异在哪里呢?



#### repl_backlog原理

master怎么知道slave与自己的数据差异在哪里呢?

这就要说到全量同步时的repl_backlog文件了。

这个文件是一个固定大小的数组，只不过数组是环形，也就是说**角标到达数组末尾后，会再次从0开始读写**，这样数组头部的数据就会被覆盖。

repl_backlog中会记录Redis处理过的命令日志及offset，包括master当前的offset，和slave已经拷贝到的offset：

![image-20210725153359022](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725153359022.png) 

slave与master的offset之间的差异，就是slave需要增量拷贝的数据了。

随着不断有数据写入，master的offset逐渐变大，slave也不断的拷贝，追赶master的offset：

![image-20210725153524190](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725153524190.png) 



直到数组被填满：

![image-20210725153715910](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725153715910.png) 

此时，如果有新的数据写入，就会覆盖数组中的旧数据。不过，旧的数据只要是绿色的，说明是已经被同步到slave的数据，即便被覆盖了也没什么影响。因为未同步的仅仅是红色部分。



但是，如果slave出现网络阻塞，导致master的offset远远超过了slave的offset：

![image-20210725153937031](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725153937031.png) 

slave : 5 master 110(10) 10-100错过的数据还在，错过了105-110(5-10)的数据



如果master继续写入新数据，其offset就会覆盖旧的数据，直到将slave现在的offset也覆盖：

![image-20210725154155984](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725154155984.png) 

棕色框中的红色部分，就是尚未同步，但是却已经被覆盖的数据。此时如果slave恢复，需要同步，却发现自己的offset都没有了，无法完成增量同步了。只能做全量同步。

![image-20210725154216392](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725154216392.png)





### 主从同步优化

主从同步可以保证主从数据的一致性，非常重要。

可以从以下几个方面来优化Redis主从就集群：

- 提高全量同步的性能

  - 在master中配置repl-diskless-sync yes启用无磁盘复制，避免全量同步时的磁盘IO。（磁盘慢，网络快）

  - Redis单节点上的内存占用不要太大，减少RDB导致的过多磁盘IO（RDB 小）


- 适当提高repl_backlog的大小，发现slave宕机时尽快实现故障恢复，尽可能避免全量同步(OFFSET_MAX)
- 限制一个master上的slave节点数量，如果实在是太多slave，则可以采用主-从-从链式结构，减少master压力（减少压力）

主从从架构图：

![image-20210725154405899](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725154405899.png)



### 小结

#### 简述全量同步和增量同步区别？

- 全量同步：master将完整内存数据生成RDB，发送RDB到slave。后续命令则记录在repl_backlog，逐个发送给slave。
- 增量同步：slave提交自己的offset到master，master获取repl_backlog中从offset之后的命令给slave

#### 什么时候执行全量同步？

- slave节点第一次连接master节点时
- slave节点断开时间太久，repl_backlog中的offset已经被覆盖时

#### 什么时候执行增量同步？

- slave节点断开又恢复，并且在repl_backlog中能找到offset时

## Sentinel 集群：保证M/S高可用

Redis提供了哨兵（Sentinel）机制来实现主从集群的自动故障恢复。

### 哨兵原理

#### 集群结构和作用

哨兵的结构如图：

![image-20210725154528072](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725154528072.png)

哨兵的作用如下：

- **监控**：Sentinel 会不断检查您的master和slave是否按预期工作 
- **自动故障恢复（转移）**：如果master故障，Sentinel会<mark>将一个slave提升为master<mark>。当故障实例恢复后也以新的master为主
- **通知**：Sentinel充当Redis<mark>客户端的服务发现来源<mark>，当集群发生故障转移时，会将最新信息推送给Redis的客户端

#### 集群监控状态原理：心跳机制

Sentinel基于<mark>心跳机制<mark>监测服务状态，每隔<mark>1秒<mark>向集群的每个实例<mark>发送ping命令<mark>：

•主观下线：如果某sentinel节点发现某实例未在规定时间响应，则认为该实例**主观下线**。

•客观下线：若超过指定数量（quorum）的sentinel都认为该实例主观下线，则该实例**客观下线**。quorum值最好超过Sentinel实例数量的一半。

![image-20210725154632354](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725154632354.png)

#### 集群故障恢复原理：选举机制

一旦发现master故障，sentinel需要在slave中选择一个作为新的master，选择依据是这样的：

- **与master断开时间**：首先会判断slave节点与master节点断开时间长短，如果超过指定值（down-after-milliseconds * 10）则会排除该slave节点
- **优先级**：然后判断slave节点的slave-priority值，越小优先级越高，如果是0则永不参与选举
- **offset**：如果slave-prority一样，则判断slave节点的offset值，越大说明数据越新，优先级越高
- **运行id**：最后是判断slave节点的运行id大小，越小优先级越高。每个 redis 节点启动时都有一个 40 字节随机字符串作为运行 id。



当选出一个新的master后，该如何实现切换呢？

流程如下：

- sentinel给备选的slave1节点发送`slaveof no one`命令，让该节点成为master
- sentinel给所有其它slave发送slaveof 127.0.0.1 7002(也就是slave1) 命令，让这些slave成为新master的从节点，开始从新的master上同步数据。
- 最后，sentinel将故障节点标记为slave，当故障节点恢复后会自动成为新的master的slave节点



![image-20210725154816841](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725154816841.png)

#### 小结

##### Sentinel的三个作用是什么？

- 监控
- 故障转移
- 通知

##### Sentinel如何判断一个redis实例是否健康？

- 每隔1秒发送一次ping命令，如果超过一定时间没有相向则认为是主观下线
- 如果大多数sentinel都认为实例主观下线，则判定服务下线

##### 故障转移步骤有哪些？

- 首先选定一个slave作为新的master，执行slaveof no one
- 然后让所有节点(包括故障节点)都执行slaveof 新master
- 修改故障节点配置，添加slaveof 新master

> 搭建哨兵集群：master宕机案例

![image-20241119171035000](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241119171035000.png)

![image-20241119171850683](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241119171850683.png)

从节点连不上主节点

![image-20241119172740595](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241119172740595.png)

哨兵选举一个slave作为新的master

![image-20241119172833310](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241119172833310.png)

被选中的slave变成master

### 客户端配置

在Sentinel集群监管下的Redis主从集群，其节点会因为自动故障转移而发生变化，Redis的客户端必须感知这种变化，及时更新连接信息。Spring的RedisTemplate底层利用lettuce实现了节点的感知和自动切换。

下面，我们通过一个测试来实现RedisTemplate集成哨兵机制。

#### 引入依赖

在项目的pom文件中引入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

#### 配置Redis哨兵集群地址

然后在配置文件application.yml中指定redis的sentinel相关信息：

```java
spring:
  redis:
    sentinel:
      master: mymaster
      nodes:
        - 127.0.0.1:27001//哨兵一号
        - 127.0.0.1:27002
        - 127.0.0.1:27003
```

哨兵的启动配置文件里已经有了主节点的一切信息，IP+PORT+quorum+name，只需要引入哨兵就能启用集群

#### 配置读写分离

在项目的启动类中，添加一个新的bean：

```java
@Bean
public LettuceClientConfigurationBuilderCustomizer clientConfigurationBuilderCustomizer(){
    return clientConfigurationBuilder -> clientConfigurationBuilder.readFrom(ReadFrom.REPLICA_PREFERRED);
}
```



这个bean中配置的就是读写策略，包括四种：

- MASTER：从主节点读取
- MASTER_PREFERRED：优先从master节点读取，master不可用才读取replica
- REPLICA：从slave（replica）节点读取
- <mark>REPLICA _PREFERRED<mark>：优先从slave（replica）节点读取，所有的slave都不可用才读取master



每个节点都会建立连接，读取时会按照读写分离的策略读。



## Cluster 集群：高并发写与海量数据存储

Redis Cluster 解决了哨兵机制的弊端：每个实例全量存储，木桶效应，提供动态的横向扩展，

### 纵向/横向扩展

纵向也可以多节点，纵/横的标准是：是否所有节点的存储内容都一样

究竟选择scale-up（纵向）还是scale-out（横向）架构,主要考虑以下因素：

|  成本  | Scale-up架构只有容量升级的成本，不会增加控制器或基础设施的开销。如果我们主要衡量每GB存储的单位价格，scale-up的扩展方式无疑更便宜一些 |
| :----: | :----------------------------------------------------------- |
|  容量  | 两种解决方案都可以满足容量需求，但scale-up架构也许会有些限制，主要取决于单个系统最大支持多少个磁盘数量和多大的容量 |
|  性能  | Scale-out架构在性能上具有扩展潜力，在多个存储控制器下，IOPS处理能力和吞吐带宽都可以聚合。虽然节点之间的通信会引发延迟，但那是部署时的细节问题 |
|  管理  | Scale-up架构本身就是以单一系统的方式来进行管理的。而Scale-out架构通常有聚合管理的能力，但每个厂商提供的产品可能会有所不同 |
| 复杂性 | Scale-up架构的存储相对简单，而scale-out架构的系统会更复杂一些，毕竟每个节点都需要管理 |
| 可用性 | 多个节点可以提供更好的可用性，假使有一个部件故障或失效，系统也不至于整体宕机。这一点与具体的实施方案也有关系 |

![Scale-UP 纵向扩展](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241119184050478.png)

![Scale-OUT 横向扩展](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241119184124945.png)

### 搭建分片集群

主从和哨兵可以解决高可用、高并发读的问题。但是依然有两个问题没有解决：

- <mark>海量数据存储问题<mark>

- <mark>高并发写的问题<mark>

使用横向扩展分片集群可以解决上述问题，如图:

![分片集群](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725155747294.png)



分片集群特征：

- 集群中有多个master，每个master保存不同数据

- 每个master都可以有多个slave节点（高可用）

- slave 不对外提供读服务，主要用来保障 master 的高可用，做数据的热备份，当 master 出现故障的时候替代它。

- master之间通过ping监测彼此健康状态（全员哨兵）

- 客户端请求可以访问集群任意节点，最终都会被自动路由转发到正确节点（自动路由）

本质就是多个主从集群的互相监控，不需要另外sentinel的监控，不过此时的slave

```properties
port 6379
# 开启集群功能
cluster-enabled yes
# 集群的配置文件名称，不需要我们创建，由redis自己维护
cluster-config-file /root/redissetup/6379/nodes.conf
# 节点心跳失败的超时时间
cluster-node-timeout 5000
# 持久化
dir /root/redissetup/6379

bind 0.0.0.0
# 后台运行
daemonize yes
# 主从ip地址
replica-announce-ip 127.0.0.1
# 保护模式
protected-mode no

databases 1
# 日志记录
logfile /root/redissetup/6379/run.log
```



```sh
printf '%s\n' 7001 7002 7003 8001 8002 8003 | xargs -I{} -t sed -i 's/6379/{}/g' {}/redis.conf
# 编辑端口号
printf '%s\n' 7001 7002 7003 8001 8002 8003 | xargs -I{} -t redis-server {}/redis.conf
# 开启服务器实例
```

```sh
redis-cli --cluster create --cluster-replicas 1 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:8001 127.0.0.1:8002 127.0.0.1:8003
# 开启分片集群功能
```

![image-20241119193312708](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241119193312708.png)

6/(1+1) = 3

！！！！！！！！

![image-20241119201022033](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241119201022033.png)

启动一定加 -c

![image-20241119203336515](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241119203336515.png)

查看节点状况

### 散列插槽（Hashslot）

#### slot原理

Redis Cluster 并没有使用一致性哈希，采用的是 哈希槽分区 ，每一个键值对都属于一个 hash slot（哈希槽） 。

Redis会把每一个master节点映射到0~16383共16384个插槽（hash slot）上，查看集群信息时就能看到：

![image-20210725155820320](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725155820320.png)



数据key不是与节点绑定，而是与插槽绑定。redis会根据key的有效部分计算插槽值，分两种情况：

- <mark>key中包含"{}"，且“{}”中至少包含1个字符，“{}”中的部分是有效部分<mark> 
- key中不包含“{}”，整个key都是有效部分

例如：key是num，那么就根据num计算，如果是{itcast}num，则根据itcast计算。计算方式是利用CRC16算法得到一个hash值，然后对16384取余，得到的结果就是slot值。

![image-20241119201236259](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241119201236259.png) 

如图，在7001这个节点执行set a 1时，对a做hash运算，对16384取余，得到的结果是15495，因此要存储到7003节点。

到了7003后，执行`get num`时，对num做hash运算，对16384取余，得到的结果是2765，因此需要切换到7001节点

当客户端发送命令请求的时候，需要

- 先根据 key 找到的对应的哈希槽，
- 然后再查询哈希槽和节点的映射关系，即可找到目标节点。 

#### 哈希槽数目为什么选择 16384？

`CRC16`算法产生的hash值有16bit，该算法可以产生2^16-=65536个值。换句话说，值是分布在0~65535之间。那作者在做`mod`运算的时候，为什么不`mod`65536，而选择`mod`16384？

哈希槽的数量选择 16384 而不是 65536 的主要原因：

- 哈希槽太大会导致心跳包（包括slots信息）太大，消耗太多带宽； 
- 哈希槽总数越少，对存储哈希槽信息的 bitmap 压缩效果越好； 
- Redis Cluster 的主节点通常不会扩展太多，16384 个哈希槽已经足够用了。

一致性哈希理论上可以有2^32^个节点，实际上不会有那么多，

正常的心跳包会携带一个节点的完整配置，它会以幂等的方式更新旧的配置，这意味着心跳包会附 带当前节点的负责的哈希槽的信息。假设哈希槽采用 16384 ,则占空间 2k(16384/8)。假设哈希槽 采用 65536， 则占空间 8k(65536/8)，这是令人难以接受的内存占用。

```c
typedef struct {
    // 省略部分字段
    // ......
    // 本节点负责的哈希槽信息,16384/8 个 char 数组，一共为16384bit
    unsigned char myslots[CLUSTER_SLOTS/8];
    // 集群的状态
    unsigned char state;
    // 消息的内容
    union clusterMsgData data;
} clusterMsg;
```

可以看到哈希槽是使用槽位数/8 使用一个char数组来表示，这里实际就是通过 bitmap 这种数据结构维护的哈希槽信息，每一个 bit 代表一个哈希槽，每个 bit 只能 存储 0/1 。如果该位为 1，表示这个哈希槽是属于这个节点。

虽说 Redis Cluster 可以扩展到 1000 个节点，但强烈不推荐这样做，应尽量避免集群中的节点过多。这 是因为 Redis Cluster 中的各个节点基于 Gossip 协议 来进行通信共享信息，当节点过多时，**Gossip** 协议的效率会显著下降，通信成本剧增。请注意，在小簇中，位图将很难压缩，因为当 N 很小时，位图将具有插槽/N 位设置，这是位设置的很大百分比。

#### 为什么不采用一致性哈希？

一致性哈希：为了防止节点和数据绑定，将整个哈希结果构造为一个2^32的域，服务器名称也在域中，根据key的哈希结果和服务器的位置关系判断应该访问哪个服务器。（狭义上）

为啥redis cluster 不设置 2的32次方个槽位呢？主要是考虑节点数在1000的规模左右，而使用 **Gossip** 去中心一致性协议，数据包不能太大，16K 个二进制位 2K字节已经很大了。

Redis 选择哈希插槽，是因为其设计目标（强一致性、高可用性）与哈希插槽的特性高度契合，而一致性哈希更适合对扩展性要求更高、允许最终一致性的场景（如缓存系统）。

- 一致性hash 哈希环顺时针映射 优先考虑的是： 如何实现 最少的节点数据发生数据迁移。

  一致性hash 哈希环上面，只有被干掉的节点顺时针方向最近的那一个节点涉及到数据迁移；其他间隔较远的节点，不涉及到数据迁移。迁移范围仅限环上相邻节点间的键。

- redis cluster 哈希槽静态映射 优先考虑的是： 如何 实现数据的均匀。

  槽位是逻辑单位，节点是物理单位，二者通过配置文件或集群协议关联。**映射关系由集群元数据控制**（如 Redis Cluster 的 Gossip 协议同步），客户端需缓存槽位分布表。

  redis cluster 各个节点都会参与数据迁移，优先保证各个redis节点承担同样的访问压力。

- 同时，redis cluster 哈希槽静态映射还有一个优点，<mark>手动迁移<mark>。

  redis cluster 可以自动分配，也可以根据节点的性能（比如Memory大小） 手动的调整slot的分配。

#### 如何判断某个key应该在哪个实例？

- 将16384个插槽分配到不同的实例
- 根据key的有效部分计算哈希值，对16384取余
- 余数作为插槽，寻找插槽所在实例即可

#### 如何将同一类数据固定的保存在同一个实例？

不同节点之间的切换比较耗时，所以同类的key尽量存在同一个节点上面

- 这一类数据使用相同的有效部分，例如key都以{typeId}为前缀

<mark>要清楚的一点：<mark>数据不存在hash插槽上，因此也不存在哈希冲突这一说，key映射到某个哈希插槽仅仅说明它属于XX节点，将来相同key也会去找相同的节点，仅此而已。

### 集群伸缩：插槽迁移

redis-cli --cluster提供了很多操作集群的命令，可以通过下面方式查看：

![image-20210725160138290](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725160138290.png)

比如，添加节点的命令：

![image-20210725160448139](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725160448139.png)



需求：向集群中添加一个新的master节点，并向其中存储 num = 10

- 启动一个新的redis实例，端口为7004
- 添加7004到之前的集群，并作为一个master节点
- 给7004节点分配插槽，使得num这个key可以存储到7004实例



这里需要两个新的功能：

- 添加一个节点到集群中
- 将部分插槽分配到新插槽

> 步骤

**创建新的redis实例**

创建一个文件夹：

```sh
mkdir 7004
```

拷贝配置文件：

```sh
cp redis.conf /7004
```

修改配置文件：

```sh
sed /s/6379/7004/g 7004/redis.conf
```

启动

```sh
redis-server 7004/redis.conf
```



**添加新节点到redis**

添加节点的语法如下：

![image-20210725160448139](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725160448139.png)



执行命令：

```sh
redis-cli --cluster add-node  127.0.0.1:7004 127.0.0.1:7001
```



通过命令查看集群状态：

```sh
redis-cli -p 7001 cluster nodes
```



如图，7004加入了集群，并且默认是一个master节点：

![image-20210725161007099](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725161007099.png)

但是，可以看到7004节点的插槽数量为0，因此没有任何数据可以存储到7004上



**转移插槽**

每个数据都有对应的插槽编号，可以根据编号查出数据

我们要将num存储到7004节点，因此需要先看看num的插槽是多少：

![image-20210725161241793](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725161241793.png)

如上图所示，num的插槽为2765.



我们可以将0~3000的插槽从7001转移到7004，命令格式如下：

![image-20210725161401925](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725161401925.png)



具体命令如下：

建立连接：

![image-20210725161506241](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725161506241.png)

得到下面的反馈：

![image-20210725161540841](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725161540841.png)

询问要移动多少个插槽，我们计划是3000个：新的问题来了：

![image-20210725161637152](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725161637152.png)

那个node来接收这些插槽？？显然是7004，那么7004节点的id是多少呢？

![image-20210725161731738](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725161731738.png)

复制这个id，然后拷贝到刚才的控制台后：

![image-20210725161817642](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725161817642.png)

这里询问，你的插槽是从哪里移动过来的？

- all：代表全部，也就是三个节点各转移一部分
- 具体的id：目标节点的id
- done：没有了

这里我们要从7001获取，因此填写7001的id：

![image-20210725162030478](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725162030478.png)

填完后，点击done，这样插槽转移就准备好了：

![image-20210725162101228](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725162101228.png)

确认要转移吗？输入yes：然后，通过命令查看结果：

![image-20210725162145497](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725162145497.png) 

可以看到： 

![image-20210725162224058](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725162224058.png)

目的达成。

#### 迁移流程：集群节点的数据结构

每个节点都有一个对应的clusterNode clusterState结构

![](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/无标题.jpg)

存储的时候，会根据key计算哈希，这是key到哈希槽位的映射，

迁移的时候，如果遍历整个数据库，分别计算哈希，效率低下，因此节点自身用一个跳表slots_to_key维护槽位到key的映射，score是槽位号，member是key值	

[Redis--集群Cluster（槽指派、重新分片） - 苏黎世湖畔 - 博客园 (cnblogs.com)](https://www.cnblogs.com/sulishihupan/p/14538864.html) 

假设将 `0-3000` 插槽从节点 A 迁移至节点 B：

1. 节点 A 标记为 `MIGRATING`，表示它即将释放这些插槽。节点 B 标记为 `IMPORTING`，表示它即将接管这些插槽。
2. 通知所有节点，让客户端感知到这些插槽的迁移状态，确保客户端请求能够动态路由到正确的节点。



### 故障转移：全员哨兵

clusterState结构中，有一个nodes，指向其他节点，因此能够实现全员哨兵

有了 Redis Cluster 之后，不需要专门部署 Sentinel 集群服务了。Redis Cluster 相当于是内置了 Sentinel 机制，Redis Cluster 内部的各个 Redis 节点通过 [Gossip 协议](https://zhuanlan.zhihu.com/p/463455831)互相探测健康状态，在故障时可 以自动切换。

集群初始状态是这样的：

![image-20210727161152065](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210727161152065.png)

其中7001、7002、7003都是master，我们计划让7002宕机。

#### 自动故障转移

当集群中有一个master宕机会发生什么呢？

直接停止一个redis实例，例如7002：

```sh
redis-cli -p 7002 shutdown
```

1）首先是该实例与其它实例失去连接

2）然后是疑似宕机：

![image-20210725162319490](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725162319490.png)

3）最后是确定下线，自动提升一个slave为新的master：

![image-20210725162408979](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725162408979.png)

4）当7002再次启动，就会变为一个slave节点了：

![image-20210727160803386](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210727160803386.png)

#### 手动故障转移

利用cluster failover命令可以手动让集群中的某个master宕机，切换到执行cluster failover命令的这个slave节点，实现无感知的数据迁移。其流程如下：

![image-20210725162441407](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210725162441407.png)

这种failover命令可以指定三种模式：

- 缺省：默认的流程，如图1~6歩
- force：省略了对offset的一致性校验
- takeover：直接执行第5歩，忽略数据一致性、忽略master状态和其它master的意见

**案例需求**：在7002这个slave节点执行手动故障转移，重新夺回master地位

步骤如下：

1）利用redis-cli连接7002这个节点

2）执行cluster failover命令

如图：

![image-20210727160037766](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210727160037766.png)



效果：

![image-20210727161152065](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210727161152065.png)



### 客户端访问分片集群

RedisTemplate底层同样基于lettuce实现了分片集群的支持，而使用的步骤与哨兵模式基本一致：

1）引入redis的starter依赖

2）配置分片集群地址

3）配置读写分离

与哨兵模式相比，其中只有分片集群的配置方式略有差异，如下：

```yaml
spring:
  redis:
    cluster:
      nodes:
        - 127.0.0.1:7001
        - 127.0.0.1:7002
        - 127.0.0.1:7003
        - 127.0.0.1:8001
        - 127.0.0.1:8002
        - 127.0.0.1:8003
```

#### 客户端访问时集群正在迁移数据怎么办？

![image-20241119212106075](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241119212106075.png)

##### ASK

1. 客户端发送请求命令，如果请求的 key 对应的哈希槽还在当前节点，就直接响应客户端的请求。如果客户端请求的 key 对应的哈希槽当前正在迁移至新的节点，就会返回`-ASK` 重定向错误，告知客户端要将请求发送到哈希槽被迁移到的目标节点。
2. 客户端收到`-ASK`重定向错误后，将会临时（一次性）重定向，自动向目标节点发送一条 ASKING 命令。也就是说，接收到 ASKING 命令的节点会强制执行一次请求，下次再来需要重新提前发送 ASKING 命令。
3. ASK 重定向并不会同步更新客户端缓存的哈希槽分配信息，也就是说，客户端对正在迁移的相同哈希槽的请求依然会发送到原节点而不是目标节点。

##### MOVED

1. 当客户端请求的 key 对应的哈希槽迁移完成，就会返回 -MOVED 重定向错误，告知客户端当前哈希槽是由哪个节点负责，客户端向目标节点发送请求并更新缓存的哈希槽分配信息。

##### 客户端缓存插槽与节点映射表

当 Redis 客户端连接到集群时，会通过 `CLUSTER SLOTS` 命令从 Redis 获取插槽与节点的对应关系。客户端将这张表缓存起来，用于后续操作中快速定位目标节点，而无需每次操作都向服务器查询。减少了每次请求都需要额外的网络交互来查询插槽分布，增加延迟。

缓存后，客户端可以直接根据键快速定位目标节点并发送请求。



## Sentinel vs Cluster

用 Redis 作为缓存或会话存储，数据量不大，Spring Boot 项目 + Jedis/Lettuce → 推荐用 **Sentinel**

如果是大型在线教育平台，Redis 承载排行榜、点赞数、搜索缓存等，数据量大、读写频繁 → 推荐用 **Cluster**

| 特性/维度              | Sentinel（哨兵）                                 | Cluster（分片集群）                                    |
| ---------------------- | ------------------------------------------------ | ------------------------------------------------------ |
| **高可用（故障转移）** | ✅ 支持自动故障转移                               | ✅ 支持自动故障转移（主从切换）                         |
| **水平扩展能力**       | ❌ 不支持，单实例容量受限                         | ✅ 原生支持分片，数据分散存储，可横向扩容               |
| **部署复杂度**         | ✅ 相对简单                                       | ❌ 更复杂（至少需要 6 个节点：3主3从）                  |
| **客户端支持**         | ✅ 普通客户端即可                                 | ❌ 客户端需支持 Cluster 协议（JedisCluster、Lettuce等） |
| **一致性控制**         | ✅ 使用主从结构，数据一致性较强（但仍为最终一致） | ❌ 存在异步复制，主从间可能数据丢失                     |
| **读写方式**           | 主写从读                                         | 自动分片读写                                           |
| **适用场景**           | 单机数据量不大、业务不复杂                       | 高并发、大数据量、分布式场景                           |

# 多级缓存

传统的缓存策略一般是请求到达Tomcat后，先查询Redis，如果未命中则查询数据库，如图：

![image-20210821075259137](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210821075259137.png)

存在下面的问题：

•请求要经过Tomcat处理，Tomcat的性能成为整个系统的瓶颈

•Redis缓存失效时，会对数据库产生冲击



多级缓存就是充分利用请求处理的每个环节，分别添加缓存，减轻Tomcat压力，提升服务性能：

- 浏览器访问静态资源时，优先读取浏览器本地缓存
- 访问非静态资源（ajax查询数据）时，访问服务端
- 请求到达Nginx后，优先读取Nginx本地缓存
- 如果Nginx本地缓存未命中，则去直接查询Redis（不经过Tomcat）
- 如果Redis查询未命中，则查询Tomcat
- 请求进入Tomcat后，优先查询JVM进程缓存
- 如果JVM进程缓存未命中，则查询数据库

![image-20210821075558137](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210821075558137.png)

在多级缓存架构中，Nginx内部需要编写本地缓存查询、Redis查询、Tomcat查询的业务逻辑，因此这样的nginx服务不再是一个**反向代理服务器**，而是一个编写**业务的Web服务器了**。

因此这样的业务Nginx服务也需要搭建集群来提高并发，再有专门的nginx服务来做反向代理，如图：

![image-20210821080511581](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210821080511581.png)



另外，我们的Tomcat服务将来也会部署为集群模式：

![image-20210821080954947](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210821080954947.png)

可见，多级缓存的关键有两个：

- 一个是在nginx中编写业务，实现nginx本地缓存、Redis、Tomcat的查询

- 另一个就是在Tomcat中实现JVM进程缓存

其中Nginx编程则会用到OpenResty框架结合Lua这样的语言。

## JVM进程缓存：Caffeine

Guava/Caffeine

为了演示多级缓存的案例，我们先准备一个商品查询的业务。

### 初识Caffeine

缓存在日常开发中启动至关重要的作用，由于是存储在内存中，数据的读取速度是非常快的，能大量减少对数据库的访问，减少数据库的压力。我们把缓存分为两类：

- 分布式缓存，例如Redis：
  - 优点：存储容量更大、可靠性更好、可以在集群间共享
  - 缺点：访问缓存有网络开销
  - 场景：缓存数据量较大、可靠性要求较高、需要在集群间共享
- 进程本地缓存，例如HashMap、GuavaCache：
  - 优点：读取本地内存，没有网络开销，速度更快
  - 缺点：存储容量有限、可靠性较低、无法共享
  - 场景：性能要求较高，缓存数据量较小

我们今天会利用Caffeine框架来实现JVM进程缓存。



**Caffeine**是一个基于Java8开发的，提供了近乎最佳命中率的高性能的本地缓存库。目前Spring内部的缓存使用的就是Caffeine。GitHub地址：https://github.com/ben-manes/caffeine

Caffeine的性能非常好，下图是官方给出的性能对比：

![image-20210821081826399](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210821081826399.png)

可以看到Caffeine的性能遥遥领先！

缓存使用的基本API：

```java
@Test
void testBasicOps() {
    // 构建cache对象
    Cache<String, String> cache = Caffeine.newBuilder().build();

    // 存数据
    cache.put("gf", "迪丽热巴");

    // 取数据
    String gf = cache.getIfPresent("gf");
    System.out.println("gf = " + gf);

    // 取数据，包含两个参数：
    // 参数一：缓存的key
    // 参数二：Lambda表达式，表达式参数就是缓存的key，方法体是查询数据库的逻辑
    // 优先根据key查询JVM缓存，如果未命中，则执行参数二的Lambda表达式
    String defaultGF = cache.get("defaultGF", key -> {
        // 根据key去数据库查询数据
        return "柳岩";
    });
    System.out.println("defaultGF = " + defaultGF);
}
```





Caffeine既然是缓存的一种，肯定需要有缓存的清除策略，不然的话内存总会有耗尽的时候。

Caffeine提供了三种缓存驱逐策略：

- **基于容量**：设置缓存的数量上限

  ```java
  // 创建缓存对象
  Cache<String, String> cache = Caffeine.newBuilder()
      .maximumSize(1) // 设置缓存大小上限为 1
      .build();
  ```

- **基于时间**：设置缓存的有效时间

  ```java
  // 创建缓存对象
  Cache<String, String> cache = Caffeine.newBuilder()
      // 设置缓存有效期为 10 秒，从最后一次写入开始计时 
      .expireAfterWrite(Duration.ofSeconds(10)) 
      .build();
  
  ```

- **基于引用**：设置缓存为软引用或弱引用，利用GC来回收缓存数据。性能较差，不建议使用。



> **注意**：在默认情况下，当一个缓存元素过期的时候，Caffeine不会自动立即将其清理和驱逐。而是在一次读或写操作后，或者在空闲时间完成对失效数据的驱逐。





### 实现JVM进程缓存

利用Caffeine实现下列需求：

- 给根据id查询商品的业务添加缓存，缓存未命中时查询数据库
- 给根据id查询商品库存的业务添加缓存，缓存未命中时查询数据库
- 缓存初始大小为100
- 缓存上限为10000

首先，我们需要定义两个Caffeine的缓存对象，分别保存商品、库存的缓存数据。

在item-service的`com.heima.item.config`包下定义`CaffeineConfig`类：

```java
package com.heima.item.config;

import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import com.heima.item.pojo.Item;
import com.heima.item.pojo.ItemStock;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CaffeineConfig {

    @Bean
    public Cache<Long, Item> itemCache(){
        return Caffeine.newBuilder()
                .initialCapacity(100)
                .maximumSize(10_000)
                .build();
    }

    @Bean
    public Cache<Long, ItemStock> stockCache(){
        return Caffeine.newBuilder()
                .initialCapacity(100)
                .maximumSize(10_000)
                .build();
    }
}
```



然后，修改item-service中的`com.heima.item.web`包下的ItemController类，添加缓存逻辑：

```java
@RestController
@RequestMapping("item")
public class ItemController {

    @Autowired
    private IItemService itemService;
    @Autowired
    private IItemStockService stockService;

    @Autowired
    private Cache<Long, Item> itemCache;
    @Autowired
    private Cache<Long, ItemStock> stockCache;
    
    // ...其它略
    
    @GetMapping("/{id}")
    public Item findById(@PathVariable("id") Long id) {
        return itemCache.get(id, key -> itemService.query()
                .ne("status", 3).eq("id", key)
                .one()
        );
    }

    @GetMapping("/stock/{id}")
    public ItemStock findStockById(@PathVariable("id") Long id) {
        return stockCache.get(id, key -> stockService.getById(key));
    }
}
```







# 缓存同步：Canal

大多数情况下，浏览器查询到的都是缓存数据，如果缓存数据与数据库数据存在较大差异，可能会产生比较严重的后果。

所以我们必须保证数据库数据、缓存数据的一致性，这就是缓存与数据库的同步。

## 数据同步策略

缓存数据同步的常见方式有三种：

**设置有效期**：给缓存设置有效期，到期后自动删除。再次查询时更新

- 优势：简单、方便
- 缺点：时效性差，缓存过期之前可能不一致
- 场景：更新频率较低，时效性要求低的业务

**同步双写**：在修改数据库的同时，直接修改缓存

- 优势：时效性强，缓存与数据库强一致
- 缺点：有代码侵入，耦合度高；
- 场景：对一致性、时效性要求较高的缓存数据

**异步通知：**修改数据库时发送事件通知，相关服务监听到通知后修改缓存数据

- 优势：低耦合，可以同时通知多个缓存服务
- 缺点：时效性一般，可能存在中间不一致状态
- 场景：时效性要求一般，有多个服务需要同步



而异步实现又可以基于MQ或者Canal来实现：

1）基于MQ的异步通知：

![image-20210821115552327](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210821115552327.png)

解读：

- 商品服务完成对数据的修改后，只需要发送一条消息到MQ中。
- 缓存服务监听MQ消息，然后完成对缓存的更新

依然有少量的代码侵入。



2）基于Canal的通知

![image-20210821115719363](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210821115719363.png)

解读：

- 商品服务完成商品修改后，业务直接结束，没有任何代码侵入
- Canal监听MySQL变化，当发现变化后，立即通知缓存服务
- 缓存服务接收到canal通知，更新缓存

代码零侵入





## Canal

**Canal [kə'næl]**，译意为水道/管道/沟渠，canal是阿里巴巴旗下的一款开源项目，基于Java开发。基于数据库增量日志解析，提供增量数据订阅&消费。GitHub的地址：https://github.com/alibaba/canal

Canal是基于mysql的主从同步来实现的，MySQL主从同步的原理如下：

![image-20210821115914748](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210821115914748.png)

- 1）MySQL master 将数据变更写入二进制日志( binary log），其中记录的数据叫做binary log events
- 2）MySQL slave 将 master 的 binary log events拷贝到它的中继日志(relay log)
- 3）MySQL slave 重放 relay log 中事件，将数据变更反映它自己的数据

------

- Canal就是把自己伪装成MySQL的一个slave节点，从而监听master的binary log变化。再把得到的变化信息通知给Canal的客户端，进而完成对其它数据库的同步。
- Canal用途很广，比如数据库实时备份、索引构建和实时维护(拆分异构索引、倒排索引等)、业务 cache 缓存刷新。
- Canal可以推送至非常多数据源，并支持推送到消息队列，方便多语言使用

![image-20210821115948395](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210821115948395.png)

## 踩坑

### 问题：Binlog解析错误 重复解析/DML解析为QUERY

这个问题主要由以下几种典型情况：

- INSERT/UPDATE/DELETE被解析为Query或DDL语句
- Binlog重复解析，即一个操作又有QUERY消息，又有对应的INSERT/UPDATE/DELETE消息。

原因分析：

1. binlog格式为非row模式，通过show variables like 'binlog_format'可以查看. 针对statement/mixed模式，DML语句都会是以SQL语句存在 [binlog 类型 | scatteredream's blog](https://scatteredream.github.io/2024/10/04/mysql-maintain/#类型) 
2. mysql5.6+之后，在binlog为row模式下，针对DML语句通过一个开关(binlog-rows-query-log-events=true, show variables里也可以看到该变量)，记录DML的原始SQL，对应binlog事件为RowsQueryLogEvent，同时也有对应的row记录. ps. canal可以通过properties设置来过滤：canal.instance.filter.query.dml = true

### 问题：Filter失效

Canal提供了filter可以过滤掉不需要监听的表（黑名单），或者指定需要监听的表（白名单）。

我们通常在canal-server端的conf/example/instance.properties文件中进行设置：

```python
# table regex
canal.instance.filter.regex=.*\\..*
# table black regex
canal.instance.filter.black.regex=
```

设置规则方式为：

```markdown
mysql 数据解析关注的表，Perl正则表达式.
多个正则之间以逗号(,)分隔，转义符需要双斜杠(\\) 
常见例子：
1.  所有表：.*   or  .*\\..*
2.  canal schema下所有表： canal\\..*
3.  canal下的以canal打头的表：canal\\.canal.*
4.  canal schema下的一张表：canal.test1
5.  多个规则组合使用：canal\\..*,mysql.test1,mysql.test2 (逗号分隔)
```

也可以在客户端与canal进行连接时，用客户端的`connector.subscribe("xxxxxxx");`来覆盖服务端初始化时的设置。

Canal官方可能是收到的filter设置不成功的反馈有点多了，在canal1.1.3+版本之后,会在日志里记录最后使用的filter条件，可以对比使用的filter看看是否和自己期望的是一致：

```lua
c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table filter : ^.*\..*$
c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table black filter :
```

可能原因一：客户端调用subscribe("xxx")，如果失效，首先看下自己在客户端是不是调用过`connector.subscribe("xxxxxxx");`覆盖了服务端初始化时的设置。

可能原因二：Binlog非ROW模式

> 过滤条件只针对row模式的数据有效(ps. mixed/statement因为不解析sql，所以无法准确提取tableName进行过滤)

我上面截图中那种收到两条消息的情况，第一条消息就是一个QURTY，并且没法确定表名，所以没法开启过滤。

### 消息堆积、滞后

[蛮三刀酱](https://www.cnblogs.com/rude3knife/p/13577743.html) 

> **一个可行的解决办法是，将消息拉取后，写入消息队列（如RabbitMQ/Kafka），用消息队列来堆积消息处理，来保证大量消息堆积后不会导致canal卡死，并且可以支持数据持久化。**
>
> **我自己对Canal这样做的的猜测：Canal应该想是让专业的工具做专业的事，Canal就只是一个读取Binlog的中间件，并不是专业的消息队列，消息应该让专业的消息队列来处理。**

## 监听 Canal

Canal提供了各种语言的客户端，当Canal监听到binlog变化时，会通知Canal的客户端。

![image-20210821120049024](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20210821120049024.png)

我们可以利用Canal提供的Java客户端，监听Canal通知消息。当收到变化的消息时，完成对缓存的更新。

[miaosha/miaosha-job/src/main/java/job/CanalClient.java](https://github.com/qqxx6661/miaosha/blob/25de14d951ecac92ca96b0b529762e584aa8f54e/miaosha-job/src/main/java/job/CanalClient.java#L15) 

# 最佳实践

**今日内容**

>* 批处理优化
>* 服务端优化
>* 集群最佳实践

## 批处理优化

### Pipeline

#### 我们的客户端与redis服务器是这样交互的

单个命令的执行流程

![image-20220521151459880](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20220521151459880.png)

N条命令的执行流程

![image-20220521151524621](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20220521151524621.png)

redis处理指令是很快的，主要花费的时候在于网络传输。于是乎很容易想到将多条指令批量的传输给redis

![image-20220521151902080](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20220521151902080.png)

#### MSet

Redis提供了很多Mxxx这样的命令，可以实现批量插入数据，例如：

- mset
- hmset

利用mset批量插入10万条数据

```java
@Test
void testMxx() {
    String[] arr = new String[2000];
    int j;
    long b = System.currentTimeMillis();
    for (int i = 1; i <= 100000; i++) {
        j = (i % 1000) << 1;
        arr[j] = "test:key_" + i;
        arr[j + 1] = "value_" + i;
        if (j == 0) {
            jedis.mset(arr);
        }
    }
    long e = System.currentTimeMillis();
    System.out.println("time: " + (e - b));
}
```

#### Pipeline

MSET虽然可以批处理，但是却只能操作部分数据类型，因此如果有对复杂数据类型的批处理需要，建议使用Pipeline

```java
@Test
void testPipeline() {
    // 创建管道
    Pipeline pipeline = jedis.pipelined();
    long b = System.currentTimeMillis();
    for (int i = 1; i <= 100000; i++) {
        // 放入命令到管道
        pipeline.set("test:key_" + i, "value_" + i);
        if (i % 1000 == 0) {
            // 每放入1000条命令，批量执行
            pipeline.sync();
        }
    }
    long e = System.currentTimeMillis();
    System.out.println("time: " + (e - b));
}
```

### 集群下的批处理

如MSET或Pipeline这样的批处理需要在一次请求中携带多条命令，而此时如果Redis是一个集群，那批处理命令的多个key必须落在一个插槽中，否则就会导致执行失败。大家可以想一想这样的要求其实很难实现，因为我们在批处理时，可能一次要插入很多条数据，这些数据很有可能不会都落在相同的节点上，这就会导致报错了

这个时候，我们可以找到4种解决方案

![1653126446641](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653126446641.png)

第一种方案：串行执行，所以这种方式没有什么意义，当然，执行起来就很简单了，缺点就是耗时过久。

第二种方案：串行slot，简单来说，就是执行前，客户端先计算一下对应的key的slot，一样slot的key就放到一个组里边，不同的，就放到不同的组里边，然后对每个组执行pipeline的批处理，他就能串行执行各个组的命令，这种做法比第一种方法耗时要少，但是缺点呢，相对来说复杂一点，所以这种方案还需要优化一下

第三种方案：并行slot，相较于第二种方案，在分组完成后串行执行，第三种方案，就变成了并行执行各个命令，所以他的耗时就非常短，但是实现呢，也更加复杂。

第四种：hash_tag，redis计算key的slot的时候，其实是根据key的有效部分来计算的，通过这种方式就能一次处理所有的key，这种方式耗时最短，实现也简单，但是如果通过操作key的有效部分，那么就会导致所有的key都落在一个节点上，产生数据倾斜的问题，所以我们推荐使用第三种方式。

#### 串行化执行代码实践

```java
public class JedisClusterTest {

    private JedisCluster jedisCluster;

    @BeforeEach
    void setUp() {
        // 配置连接池
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxTotal(8);
        poolConfig.setMaxIdle(8);
        poolConfig.setMinIdle(0);
        poolConfig.setMaxWaitMillis(1000);
        HashSet<HostAndPort> nodes = new HashSet<>();
        nodes.add(new HostAndPort("127.0.0.1", 7001));
        nodes.add(new HostAndPort("127.0.0.1", 7002));
        nodes.add(new HostAndPort("127.0.0.1", 7003));
        nodes.add(new HostAndPort("127.0.0.1", 8001));
        nodes.add(new HostAndPort("127.0.0.1", 8002));
        nodes.add(new HostAndPort("127.0.0.1", 8003));
        jedisCluster = new JedisCluster(nodes, poolConfig);
    }

    @Test
    void testMSet() {
        jedisCluster.mset("name", "Jack", "age", "21", "sex", "male");

    }

    @Test
    void testMSet2() {
        Map<String, String> map = new HashMap<>(3);
        map.put("name", "Jack");
        map.put("age", "21");
        map.put("sex", "Male");
        //对Map数据进行分组。根据相同的slot放在一个分组
        //key就是slot，value就是一个组
        Map<Integer, List<Map.Entry<String, String>>> result = map.entrySet()
                .stream()
                .collect(Collectors.groupingBy(
                        entry -> ClusterSlotHashUtil.calculateSlot(entry.getKey()))
                );
        //串行的去执行mset的逻辑
        for (List<Map.Entry<String, String>> list : result.values()) {
            String[] arr = new String[list.size() * 2];
            int j = 0;
            for (int i = 0; i < list.size(); i++) {
                j = i<<2;
                Map.Entry<String, String> e = list.get(0);
                arr[j] = e.getKey();
                arr[j + 1] = e.getValue();
            }
            jedisCluster.mset(arr);
        }
    }

    @AfterEach
    void tearDown() {
        if (jedisCluster != null) {
            jedisCluster.close();
        }
    }
}
```

#### Spring集群环境下批处理代码

```java
   @Test
    void testMSetInCluster() {
        Map<String, String> map = new HashMap<>(3);
        map.put("name", "Rose");
        map.put("age", "21");
        map.put("sex", "Female");
        stringRedisTemplate.opsForValue().multiSet(map);


        List<String> strings = stringRedisTemplate.opsForValue().multiGet(Arrays.asList("name", "age", "sex"));
        strings.forEach(System.out::println);

    }
```

**原理分析**

在RedisAdvancedClusterAsyncCommandsImpl 类中

首先根据slotHash算出来一个partitioned的map，map中的key就是slot，而他的value就是对应的对应相同slot的key对应的数据

通过 RedisFuture<String> mset = super.mset(op);进行异步的消息发送

```Java
@Override
public RedisFuture<String> mset(Map<K, V> map) {

    Map<Integer, List<K>> partitioned = SlotHash.partition(codec, map.keySet());

    if (partitioned.size() < 2) {
        return super.mset(map);
    }

    Map<Integer, RedisFuture<String>> executions = new HashMap<>();

    for (Map.Entry<Integer, List<K>> entry : partitioned.entrySet()) {

        Map<K, V> op = new HashMap<>();
        entry.getValue().forEach(k -> op.put(k, map.get(k)));

        RedisFuture<String> mset = super.mset(op);
        executions.put(entry.getKey(), mset);
    }

    return MultiNodeExecution.firstOfAsync(executions);
}
```

## 服务器端优化

### 持久化配置

Redis的持久化虽然可以保证数据安全，但也会带来很多额外的开销，因此持久化请遵循下列建议：

* 用来做缓存的Redis实例尽量不要开启持久化功能
* 建议关闭RDB持久化功能，使用AOF持久化
* 利用脚本定期在slave节点做RDB，实现数据备份
* 设置合理的rewrite阈值，避免频繁的bgrewrite
* 配置no-appendfsync-on-rewrite = yes，禁止在rewrite期间做aof，避免因AOF引起的阻塞
* 部署有关建议：
  * Redis实例的物理机要预留足够内存，应对fork和rewrite
  * 单个Redis实例内存上限不要太大，例如4G或8G。可以加快fork的速度、减少主从同步、数据迁移压力
  * 不要与CPU密集型应用部署在一起
  * 不要与高硬盘负载应用一起部署。例如：数据库、消息队列

### 慢查询优化

#### 什么是慢查询

并不是很慢的查询才是慢查询，而是：在Redis执行时耗时超过某个阈值的命令，称为慢查询。

慢查询的危害：由于Redis是单线程的，所以当客户端发出指令后，他们都会进入到redis底层的queue来执行，如果此时有一些慢查询的数据，就会导致大量请求阻塞，从而引起报错，所以我们需要解决慢查询问题。

![1653129590210](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653129590210.png)

慢查询的阈值可以通过配置指定：

slowlog-log-slower-than：慢查询阈值，单位是微秒。默认是10000，建议1000

慢查询会被放入慢查询日志中，日志的长度有上限，可以通过配置指定：

slowlog-max-len：慢查询日志（本质是一个队列）的长度。默认是128，建议1000

![1653130457771](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653130457771.png)

修改这两个配置可以使用：config set命令：

![1653130475979](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653130475979.png)

#### 如何查看慢查询

知道了以上内容之后，那么咱们如何去查看慢查询日志列表呢：

* slowlog len：查询慢查询日志长度
* slowlog get [n]：读取n条慢查询日志
* slowlog reset：清空慢查询列表

![1653130858066](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653130858066.png)

### 命令及安全配置

 安全可以说是服务器端一个非常重要的话题，如果安全出现了问题，那么一旦这个漏洞被一些坏人知道了之后，并且进行攻击，那么这就会给咱们的系统带来很多的损失，所以我们这节课就来解决这个问题。

Redis会绑定在0.0.0.0:6379，这样将会将Redis服务暴露到公网上，而Redis如果没有做身份认证，会出现严重的安全漏洞.
漏洞重现方式：https://cloud.tencent.com/developer/article/1039000

为什么会出现不需要密码也能够登录呢，主要是Redis考虑到每次登录都比较麻烦，所以Redis就有一种ssh免秘钥登录的方式，生成一对公钥和私钥，私钥放在本地，公钥放在redis端，当我们登录时服务器，再登录时候，他会去解析公钥和私钥，如果没有问题，则不需要利用redis的登录也能访问，这种做法本身也很常见，但是这里有一个前提，前提就是公钥必须保存在服务器上，才行，但是Redis的漏洞在于在不登录的情况下，也能把秘钥送到Linux服务器，从而产生漏洞

漏洞出现的核心的原因有以下几点：

* Redis未设置密码
* 利用了Redis的config set命令动态修改Redis配置
* 使用了Root账号权限启动Redis

所以：如何解决呢？我们可以采用如下几种方案

为了避免这样的漏洞，这里给出一些建议：

* Redis一定要设置密码
* 禁止线上使用下面命令：keys、flushall、flushdb、config set等命令。可以利用rename-command禁用。
* bind：限制网卡，禁止外网网卡访问
* 开启防火墙
* 不要使用Root账户启动Redis
* 尽量不是有默认的端口

### Redis内存划分和内存配置

当Redis内存不足时，可能导致Key频繁被删除、响应时间变长、QPS不稳定等问题。当内存使用率达到90%以上时就需要我们警惕，并快速定位到内存占用的原因。

**有关碎片问题分析**

Redis底层分配并不是这个key有多大，他就会分配多大，而是有他自己的分配策略，比如8,16,20等等，假定当前key只需要10个字节，此时分配8肯定不够，那么他就会分配16个字节，多出来的6个字节就不能被使用，这就是我们常说的 碎片问题

**进程内存问题分析：**

这片内存，通常我们都可以忽略不计

**缓冲区内存问题分析：**

一般包括客户端缓冲区、AOF缓冲区、复制缓冲区等。客户端缓冲区又包括输入缓冲区和输出缓冲区两种。这部分内存占用波动较大，所以这片内存也是我们需要重点分析的内存问题。

| **内存占用** | **说明**                                                     |
| ------------ | ------------------------------------------------------------ |
| 数据内存     | 是Redis最主要的部分，存储Redis的键值信息。主要问题是BigKey问题、内存碎片问题 |
| 进程内存     | Redis主进程本身运⾏肯定需要占⽤内存，如代码、常量池等等；这部分内存⼤约⼏兆，在⼤多数⽣产环境中与Redis数据占⽤的内存相⽐可以忽略。 |
| 缓冲区内存   | 一般包括客户端缓冲区、AOF缓冲区、复制缓冲区等。客户端缓冲区又包括输入缓冲区和输出缓冲区两种。这部分内存占用波动较大，不当使用BigKey，可能导致内存溢出。 |

于是我们就需要通过一些命令，可以查看到Redis目前的内存分配状态：

* info memory：查看内存分配的情况

![1653132073570](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653132073570.png)

* memory xxx：查看key的主要占用情况

![1653132098823](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653132098823.png)

接下来我们看到了这些配置，最关键的缓存区内存如何定位和解决呢？

内存缓冲区常见的有三种：

* 复制缓冲区：主从复制的repl_backlog_buf，如果太小可能导致频繁的全量复制，影响性能。通过replbacklog-size来设置，默认1mb
* AOF缓冲区：AOF刷盘之前的缓存区域，AOF执行rewrite的缓冲区。无法设置容量上限
* 客户端缓冲区：分为输入缓冲区和输出缓冲区，输入缓冲区最大1G且不能设置。输出缓冲区可以设置

以上复制缓冲区和AOF缓冲区 不会有问题，最关键就是客户端缓冲区的问题

客户端缓冲区：指的就是我们发送命令时，客户端用来缓存命令的一个缓冲区，也就是我们向redis输入数据的输入端缓冲区和redis向客户端返回数据的响应缓存区，输入缓冲区最大1G且不能设置，所以这一块我们根本不用担心，如果超过了这个空间，redis会直接断开，因为本来此时此刻就代表着redis处理不过来了，我们需要担心的就是输出端缓冲区

![1653132410073](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653132410073.png)

我们在使用redis过程中，处理大量的big value，那么会导致我们的输出结果过多，如果输出缓存区过大，会导致redis直接断开，而默认配置的情况下， 其实他是没有大小的，这就比较坑了，内存可能一下子被占满，会直接导致咱们的redis断开，所以解决方案有两个

1、设置一个大小

2、增加我们带宽的大小，避免我们出现大量数据从而直接超过了redis的承受能力



### 集群优化 集群 or 主从

集群虽然具备高可用特性，能实现自动故障恢复，但是如果使用不当，也会存在一些问题：

* 集群完整性问题
* 集群带宽问题
* 数据倾斜问题
* 客户端性能问题
* 命令的集群兼容性问题
* lua和事务问题

 **问题1、在Redis的默认配置中，如果发现任意一个插槽不可用，则整个集群都会停止对外服务：** 

大家可以设想一下，如果有几个slot不能使用，那么此时整个集群都不能用了，我们在开发中，其实最重要的是可用性，所以需要把如下配置修改成no，即有slot不能使用时，我们的redis集群还是可以对外提供服务

![1653132740637](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653132740637.png)

**问题2、集群带宽问题**

集群节点之间会不断的互相Ping来确定集群中其它节点的状态。每次Ping携带的信息至少包括：

* 插槽信息
* 集群状态信息

集群中节点越多，集群状态信息数据量也越大，10个节点的相关信息可能达到1kb，此时每次集群互通需要的带宽会非常高，这样会导致集群中大量的带宽都会被ping信息所占用，这是一个非常可怕的问题，所以我们需要去解决这样的问题

**解决途径：**

* 避免大集群，集群节点数不要太多，最好少于1000，如果业务庞大，则建立多个集群。
* 避免在单个物理机中运行太多Redis实例
* 配置合适的cluster-node-timeout值

**问题3、命令的集群兼容性问题**

有关这个问题咱们已经探讨过了，当我们使用批处理的命令时，redis要求我们的key必须落在相同的slot上，然后大量的key同时操作时，是无法完成的，所以客户端必须要对这样的数据进行处理，这些方案我们之前已经探讨过了，所以不再这个地方赘述了。

**问题4、lua和事务的问题**

lua和事务都是要保证原子性问题，如果你的key不在一个节点，那么是无法保证lua的执行和事务的特性的，所以在集群模式是没有办法执行lua和事务的



**那我们到底是集群还是主从**

单体Redis（主从Redis）已经能达到万级别的QPS，并且也具备很强的高可用特性。如果主从能满足业务需求的情况下，所以如果不是在万不得已的情况下，尽量不搭建Redis集群

