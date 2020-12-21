# NoSQL简介

1. 单机版本的MSQL

网站访问量不大的时候，用单个数据库完全可以应付

存储瓶颈

- 数据量的总大小一个机器放不下的时候
- 数据的索引一个机器放不下
- 访问能量一个实例不能承受



<img src="resource\单机.png" style="zoom: 80%;" />

2. Memcached缓存+MySQL+垂直拆分

   随着访问量增加开始出现性能问题，通过文件缓存来缓解数据库的压力。

<img src="resource\缓存.png" style="zoom:80%;" />

3. MySQL主从分离

   由于数据库写入压力大，Memcached只能缓解数据库读取压力，读写集中在一个数据库上压力太大，使用读写分离提高扩展性

<img src="resource\主从分离.png" style="zoom:80%;" />

4. 分表分库+水平拆分+mysql集群

   随着数据写的压力瓶颈出现，开始使用集群解决

   <img src="resource\集群.png" style="zoom:80%;" />

5. MySQL性能瓶颈

6. 现状

   <img src="Redis\resource\现状.png" style="zoom:80%;" />

# NoSQL数据库的分类

<img src="resource\4种NoSQL数据库.png" style="zoom:150%;" />

# 分布式数据库理论

## CAP

- 一致性（Consistency）：一致性说的是客户端的每次读操作，不管访问哪个节点，要么读到的都是同一份最新写入的数据，要么读取失败。

- 可用性（Availability）：可用性说的是任何来自客户端的请求，不管访问哪个非故障节点，都能得到响应数据，但不保证是同一份最新数据。这个指标强调的是服务可用，但不保证数据正确

- 分区容错性（Partition Tolerance）：当节点间出现任意数量的消息丢失或高延迟的时候，系统仍然在继续工作。也就是说，分布式系统在告诉访问本系统的客户端：不管我的内部出现什么样的数据同步问题，我会一直运行。这个指标，强调的是集群对分区故障的容错能力。

<img src="resource\cap.png" style="zoom:80%;" />

- 当选择了一致性（C）的时候，一定会读到最新的数据，不会读到旧数据，但如果因为消息丢失、延迟过高发生了网络分区，那么这个时候，当集群节点接收到来自客户端的读请求时，为了不破坏一致性，可能会因为无法响应最新数据，而返回出错信息。
- 当选择了可用性（A）的时候，系统将始终处理客户端的查询，返回特定信息，如果发生了网络分区，一些节点将无法返回最新的特定信息，它们将返回自己当前的相对新的信息。

在不存在网络分区的情况下，也就是分布式系统正常运行时（这也是系统在绝大部分时候所处的状态），就是说在不需要 P 时，C 和 A 能够同时保证。只有当发生分区故障的时候，也就是说需要 P 时，才会在 C 和 A 之间做出选择。而且如果读操作会读到旧数据，影响到了系统运行或业务运行（也就是说会有负面的影响），推荐选择 C，否则选 A。

## CAP中追求一致性ACID

实现了ACID就是实现了事物，

分布式系统中使用两阶段提交协议来保证ACID

## CAP中追求可用性BASE

- 基本可用（Basically Available）
  1. 流量削峰：12306火车票在不同时间开售
  2. 延迟响应：买票排队等待一段时间才能买
  3. 体验降级：用小图片，低清晰度的图片代替原来的图片
  4. 过载保护：等待队列满了就剔除请求
- 最终一致性（Eventually consistent）
  1. 读时修复：在读取数据时，检测数据的不一致，进行修复
  2. 写时修复：在写入数据，检测数据的不一致时
  3. 异步修复：
- 软状态（Soft state）：一种过渡状态

# Redis

[命令参考大全]: http://redisdoc.com/index.html

redis的作用

<img src="resource\redis作用.png" style="zoom:60%;" />

## 常用命令

| 命令                                      | 描述                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| DEL key                                   | 该命令用于在 key 存在时删除 key。                            |
| DUMP key                                  | 序列化给定 key ，并返回被序列化的值。                        |
| EXISTS key                                | 检查给定 key 是否存在。                                      |
| EXPIRE key seconds                        | 为给定 key 设置过期时间，以秒计。                            |
| EXPIREAT key timestamp                    | EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。 |
| PEXPIRE key milliseconds                  | 设置 key 的过期时间以毫秒计。                                |
| PEXPIREAT key milliseconds-timestamp      | 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计           |
| KEYS pattern                              | 查找所有符合给定模式( pattern)的 key 。                      |
| MOVE key db                               | 将当前数据库的 key 移动到给定的数据库 db 当中。              |
| PERSIST key                               | 移除 key 的过期时间，key 将持久保持。                        |
| PTTL key                                  | 以毫秒为单位返回 key 的剩余的过期时间。                      |
| TTL key                                   | 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。 |
| RANDOMKEY                                 | 从当前数据库中随机返回一个 key 。                            |
| RENAME key newkey                         | 修改 key 的名称                                              |
| RENAMENX key newkey                       | 仅当 newkey 不存在时，将 key 改名为 newkey 。                |
| SCAN cursor [MATCH pattern] [COUNT count] | 迭代数据库中的数据库键。                                     |
| TYPE key                                  | 返回 key 所储存的值的类型。                                  |

## Redis常见数据结构

### String(字符串)

- string是redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。
- string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。
- string类型是Redis最基本的数据类型，一个redis中字符串value最多可以是512M

### Hash(哈希，类似java里的Map)

- Redis hash 是一个键值对集合。
- Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。
- 类似Java里面的Map<String,Object>

### List(列表)

- Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）。
- 它的底层实际是个链表

### Set(集合)

- Redis的Set是string类型的无序集合。它是通过HashTable实现实现的

### Zset(sorted set:有序集合)

- Redis zset 和 set 一样也是string类型元素的集合，且不允许重复的成员。
- 不同的是每个元素都会关联一个double类型的分数。
- redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的，但分数(score)却可以重复。

## Redis配置文件解析

| 序号 | 配置项                                                       | 说明                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | `daemonize no`                                               | Redis 默认不是以守护进程的方式运行，可以通过该配置项修改，使用 yes 启用守护进程（Windows 不支持守护线程的配置为 no ） |
| 2    | `pidfile /var/run/redis.pid`                                 | 当 Redis 以守护进程方式运行时，Redis 默认会把 pid 写入 /var/run/redis.pid 文件，可以通过 pidfile 指定 |
| 3    | `port 6379`                                                  | 指定 Redis 监听端口，默认端口为 6379，作者在自己的一篇博文中解释了为什么选用 6379 作为默认端口，因为 6379 在手机按键上 MERZ 对应的号码，而 MERZ 取自意大利歌女 Alessia Merz 的名字 |
| 4    | `bind 127.0.0.1`                                             | 绑定的主机地址                                               |
| 5    | `timeout 300`                                                | 当客户端闲置多长秒后关闭连接，如果指定为 0 ，表示关闭该功能  |
| 6    | `loglevel notice`                                            | 指定日志记录级别，Redis 总共支持四个级别：debug、verbose、notice、warning，默认为 notice |
| 7    | `logfile stdout`                                             | 日志记录方式，默认为标准输出，如果配置 Redis 为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给 /dev/null |
| 8    | `databases 16`                                               | 设置数据库的数量，默认数据库为0，可以使用SELECT 命令在连接上指定数据库id |
| 9    | `save <seconds> <changes>` Redis 默认配置文件中提供了三个条件： `save 900 1` `save 300 10` `save 60 10000` | 分别表示 900 秒（15 分钟）内有 1 个更改，300 秒（5 分钟）内有 10 个更改以及 60 秒内有 10000 个更改。 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合 |
| 10   | `rdbcompression yes`                                         | 指定存储至本地数据库时是否压缩数据，默认为 yes，Redis 采用 LZF 压缩，如果为了节省 CPU 时间，可以关闭该选项，但会导致数据库文件变的巨大 |
| 11   | `dbfilename dump.rdb`                                        | 指定本地数据库文件名，默认值为 dump.rdb                      |
| 12   | `dir ./`                                                     | 指定本地数据库存放目录                                       |
| 13   | `slaveof <masterip> <masterport>`                            | 设置当本机为 slave 服务时，设置 master 服务的 IP 地址及端口，在 Redis 启动时，它会自动从 master 进行数据同步 |
| 14   | `masterauth <master-password>`                               | 当 master 服务设置了密码保护时，slav 服务连接 master 的密码  |
| 15   | `requirepass foobared`                                       | 设置 Redis 连接密码，如果配置了连接密码，客户端在连接 Redis 时需要通过 AUTH 命令提供密码，默认关闭 |
| 16   | `maxclients 128`                                             | 设置同一时间最大客户端连接数，默认无限制，Redis 可以同时打开的客户端连接数为 Redis 进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis 会关闭新的连接并向客户端返回 max number of clients reached 错误信息 |
| 17   | `maxmemory <bytes>`                                          | 指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会先尝试清除已到期或即将到期的 Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis 新的 vm 机制，会把 Key 存放内存，Value 会存放在 swap 区 |
| 18   | `appendonly no`                                              | 指定是否在每次更新操作后进行日志记录，Redis 在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis 本身同步数据文件是按上面 save 条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为 no |
| 19   | `appendfilename appendonly.aof`                              | 指定更新日志文件名，默认为 appendonly.aof                    |
| 20   | `appendfsync everysec`                                       | 指定更新日志条件，共有 3 个可选值： no：表示等操作系统进行数据缓存同步到磁盘（快） always：表示每次更新操作后手动调用 fsync() 将数据写到磁盘（慢，安全） everysec：表示每秒同步一次（折中，默认值） |
| 21   | `vm-enabled no`                                              | 指定是否启用虚拟内存机制，默认值为 no，简单的介绍一下，VM 机制将数据分页存放，由 Redis 将访问量较少的页即冷数据 swap 到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析 Redis 的 VM 机制） |
| 22   | `vm-swap-file /tmp/redis.swap`                               | 虚拟内存文件路径，默认值为 /tmp/redis.swap，不可多个 Redis 实例共享 |
| 23   | `vm-max-memory 0`                                            | 将所有大于 vm-max-memory 的数据存入虚拟内存，无论 vm-max-memory 设置多小，所有索引数据都是内存存储的(Redis 的索引数据 就是 keys)，也就是说，当 vm-max-memory 设置为 0 的时候，其实是所有 value 都存在于磁盘。默认值为 0 |
| 24   | `vm-page-size 32`                                            | Redis swap 文件分成了很多的 page，一个对象可以保存在多个 page 上面，但一个 page 上不能被多个对象共享，vm-page-size 是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page 大小最好设置为 32 或者 64bytes；如果存储很大大对象，则可以使用更大的 page，如果不确定，就使用默认值 |
| 25   | `vm-pages 134217728`                                         | 设置 swap 文件中的 page 数量，由于页表（一种表示页面空闲或使用的 bitmap）是在放在内存中的，，在磁盘上每 8 个 pages 将消耗 1byte 的内存。 |
| 26   | `vm-max-threads 4`                                           | 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4 |
| 27   | `glueoutputbuf yes`                                          | 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启 |
| 28   | `hash-max-zipmap-entries 64` `hash-max-zipmap-value 512`     | 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法 |
| 29   | `activerehashing yes`                                        | 指定是否激活重置哈希，默认为开启（后面在介绍 Redis 的哈希算法时具体介绍） |
| 30   | `include /path/to/local.conf`                                | 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件 |

## 持久化

### RDB（Redis Database）

- 在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里

- Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到 一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。

#### 优势与劣势

- 优势
  - 适合大规模的数据恢复
  - 对数据完整性和一致性要求不高
- 劣势
  - 在一定间隔时间做一次备份，所以如果redis意外down掉的话，就 会丢失最后一次快照后的所有修改
  - Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑

### AOF(Append only File)

- 以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)， 只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

#### 优势与劣势

- 优势
  - 每修改同步：appendfsync always 同步持久化 每次发生数据变更会被立即记录到磁盘 性能较差但数据完整性比较好
  - 每秒同步：appendfsync everysec 异步操作，每秒记录 如果一秒内宕机，有数据丢失
  - 不同步：appendfsync no 从不同步
- 劣势
  - 相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb
  - Aof运行效率要慢于rdb,每秒同步策略效率较好，不同步效率和rdb相同