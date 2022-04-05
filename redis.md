# 一，redis概述

+ Redis是一个开源的key-value存储系统
+ 和Memcached类似，它支持的value类型相对更多，包括String，List，Set，Zset（有序set），和hash（哈希类型）

+ 累些数据类型都支持 push/pop，add/remove，及取交集并集和差集以及更丰富的操作
+ 支持各种不同方式的排序
+ 数据**缓存在内存**中
+ **周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件**
+ 并且在此基础上实现了master-slave（主从）同步
+ **单线程+多路io复用**

## 1.1 安装

+ 官网redis.io，下载传linux，解压，make，make install，默认在/usr/local/bin
+ 修改redis.conf   daemonize no 改为 yes (为了后台运行)
+ 启动命令：`./redis-server redis.conf`

## 1.2 配置

+ **Redis CONFIG 命令格式如下：**

```bash
redis 127.0.0.1:6379> CONFIG GET CONFIG_SETTING_NAME
redis 127.0.0.1:6379> CONFIG GET *
redis 127.0.0.1:6379> CONFIG GET loglevel
```

from `https://www.runoob.com/`

**修改redis密码：**

**配置文件redis.conf中 requirepass后加密码，eg:  requirepass yuqianredis**，重启后生效，auth password认证密码

检查redis是否正常：ping-pong

| 编号 | 配置项                                                       | 作用                                                         |
| ---- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 1    | `daemonize no`                                               | Redis 默认不是以守护进程的方式运行，可以通过该配置项修改，使用 yes 启用守护进程（Windows 不支持守护线程的配置为 no ） |
| 2    | `pidfile /var/run/redis.pid`                                 | 当 Redis 以守护进程方式运行时，Redis 默认会把 pid 写入 /var/run/redis.pid 文件，可以通过 pidfile 指定 |
| 3    | `port 6379`                                                  | 指定 Redis 监听端口，默认端口为 6379，作者在自己的一篇博文中解释了为什么选用 6379 作为默认端口，因为 6379 在手机按键上 MERZ 对应的号码，而 MERZ 取自意大利歌女 Alessia Merz 的名字 |
| 4    | `bind 127.0.0.1`                                             | 绑定的主机地址                                               |
| 5    | `timeout 300`                                                | 当客户端闲置多长秒后关闭连接，如果指定为 0 ，表示关闭该功能  |
| 6    | `loglevel notice`                                            | 指定日志记录级别，Redis 总共支持四个级别：debug、verbose、notice、warning，默认为 notice |
| 7    | `logfile stdout`                                             | 日志记录方式，默认为标准输出，如果配置 Redis 为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给 /dev/null |
| 8    | `databases 16`                                               | 设置数据库的数量，默认数据库为0，可以使用SELECT 命令在连接上指定数据库id |
| 9    | `save <seconds> <changes>`Redis 默认配置文件中提供了三个条件：**save 900 1****save 300 10****save 60 10000**分别表示 900 秒（15 分钟）内有 1 个更改，300 秒（5 分钟）内有 10 个更改以及 60 秒内有 10000 个更改。 | 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合 |
| 10   | `rdbcompression yes`                                         | 指定存储至本地数据库时是否压缩数据，默认为 yes，Redis 采用 LZF 压缩，如果为了节省 CPU 时间，可以关闭该选项，但会导致数据库文件变的巨大 |
| 11   | `dbfilename dump.rdb`                                        | 指定本地数据库文件名，默认值为 dump.rdb                      |
| 12   | `dir ./`                                                     | 指定本地数据库存放目录                                       |
| 13   | `slaveof <masterip> <masterport>`                            | 设置当本机为 slave 服务时，设置 master 服务的 IP 地址及端口，在 Redis 启动时，它会自动从 master 进行数据同步 |
| 14   | `masterauth <master-password>`                               | 当 master 服务设置了密码保护时，slave 服务连接 master 的密码 |
| 15   | `requirepass foobared`                                       | 设置 Redis 连接密码，如果配置了连接密码，客户端在连接 Redis 时需要通过 AUTH <password> 命令提供密码，默认关闭 |
| 16   | ` maxclients 128`                                            | 设置同一时间最大客户端连接数，默认无限制，Redis 可以同时打开的客户端连接数为 Redis 进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis 会关闭新的连接并向客户端返回 max number of clients reached 错误信息 |
| 17   | `maxmemory <bytes>`                                          | 指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会先尝试清除已到期或即将到期的 Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis 新的 vm 机制，会把 Key 存放内存，Value 会存放在 swap 区 |
| 18   | `appendonly no`                                              | 指定是否在每次更新操作后进行日志记录，Redis 在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis 本身同步数据文件是按上面 save 条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为 no |
| 19   | `appendfilename appendonly.aof`                              | 指定更新日志文件名，默认为 appendonly.aof                    |
| 20   | `appendfsync everysec`                                       | 指定更新日志条件，共有 3 个可选值：**no**：表示等操作系统进行数据缓存同步到磁盘（快）**always**：表示每次更新操作后手动调用 fsync() 将数据写到磁盘（慢，安全）**everysec**：表示每秒同步一次（折中，默认值） |
| 21   | `vm-enabled no`                                              | 指定是否启用虚拟内存机制，默认值为 no，简单的介绍一下，VM 机制将数据分页存放，由 Redis 将访问量较少的页即冷数据 swap 到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析 Redis 的 VM 机制） |
| 22   | `vm-swap-file /tmp/redis.swap`                               | 虚拟内存文件路径，默认值为 /tmp/redis.swap，不可多个 Redis 实例共享 |
| 23   | `vm-max-memory 0`                                            | 将所有大于 vm-max-memory 的数据存入虚拟内存，无论 vm-max-memory 设置多小，所有索引数据都是内存存储的(Redis 的索引数据 就是 keys)，也就是说，当 vm-max-memory 设置为 0 的时候，其实是所有 value 都存在于磁盘。默认值为 0 |
| 24   | `vm-page-size 32`                                            | Redis swap 文件分成了很多的 page，一个对象可以保存在多个 page 上面，但一个 page 上不能被多个对象共享，vm-page-size 是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page 大小最好设置为 32 或者 64bytes；如果存储很大大对象，则可以使用更大的 page，如果不确定，就使用默认值 |
| 25   | `vm-pages 134217728`                                         | 设置 swap 文件中的 page 数量，由于页表（一种表示页面空闲或使用的 bitmap）是在放在内存中的，，在磁盘上每 8 个 pages 将消耗 1byte 的内存。 |
| 26   | `vm-max-threads 4`                                           | 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4 |
| 27   | `glueoutputbuf yes`                                          | 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启 |
| 28   | `hash-max-zipmap-entries 64 hash-max-zipmap-value 512`       | 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法 |
| 29   | `activerehashing yes`                                        | 指定是否激活重置哈希，默认为开启（后面在介绍 Redis 的哈希算法时具体介绍） |
| 30   | `include /path/to/local.conf`                                | 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件 |

# 二，常用五大数据类型

## 2.1 redis key

| 命令          | 作用                                                         |
| ------------- | ------------------------------------------------------------ |
| keys *        | 查看当前库中所有key                                          |
| exists key    | 判断某个key是否存在                                          |
| type key      | 查看key类型                                                  |
| del key       | 删除指定的key数据                                            |
| unlink key    | 根据value选择非阻塞删除，仅将keys从keyspace元数据删除，真正的删除会在后续异步操作 |
| expire key 10 | 为key设置过期时间10s                                         |
| ttl key       | 查看还有多少秒过期 -1永不过期，-2已经过期                    |
| select        | 切换数据库（默认1-15号库）                                   |
| dbsize        | 查看key的数量                                                |
| flushdb       | 清空当前库                                                   |
| flushall      | 通杀全部库                                                   |

## 2.2 Redis String

+ String是**二进制安全**的，可以包含任意数据，比如jpg图片或者序列化对象
+ 一个value最多可以是512MB

**常用命令**

| 命令                                | 作用                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| `set <key> <value>`                 | NX：当数据库key不存在时，将key-value添加数据库<br/>XX: 与nx相反<br/>EX: 设置超时秒数<br/>PX：key的超时毫秒数，与EX互斥 |
| `setex <key> <过期时间> <value>`    | 这是键值对的同时设置时间                                     |
| `setnx <key> <value>`               | 只有在key不存在的时候，设置key的值                           |
| `getset <key> <value>`              | 得到原来的值，并用新的kv替换原来的kv                         |
| `get key`                           | 查询对应键值                                                 |
| `append <key> <value>`              | 把value追加到原来的末尾                                      |
| `strlen <key>`                      | 获得值的长度                                                 |
| `incr <key>`                        | 把key中存的数字+1                                            |
| decr <key>                          | 把key中存的数字 -1                                           |
| `incrby /decrby <key> 步长`         | 把key中存的数字增减，自定义步长                              |
| `mset <key> <value><key> <value>`   | 设置多个key-value                                            |
| `mget <key> <key> <key> <key>`      | 得到多个value                                                |
| `msetnx <key> <value><key> <value>` | 只有所有的key都不存在才会成功                                |
| `getrange<key><起始位置><结束位置>` | 获得值的范围，类似于java中的substring                        |
| `setrange<key><起始位置><value>`    | 从起始位置开始覆盖字符串                                     |

**原子操作**：

不会被线程调度机制打断的操作

**数据结构**

底层数据结构是动态字符串

## 2.3 Redis List

**简介**

**单键多值**

Redis列表是简单的字符串列表，按照插入的顺序排序，你可以添加一个元素到列表头部或尾部。

它的底层是双向**链表**，对两端的操作性能很高，通过索引下标的操作中间的节点性能较差

| 命令                                            | 作用                                   |
| ----------------------------------------------- | -------------------------------------- |
| `lpush/rpush kvvvvv`                            | 从左边/右边插入一个或者多个值          |
| `lpop/rpop k`                                   | 从左边/右边吐出一个值（打印并删除）    |
| `rpoplpush<k1><k2>`                             | 从k1右边吐出一个值插入到k2左边         |
| `lrange <key><start><end>`                      | 取值 0表示左边第一个，-1表示右边第一个 |
| `lindex <key> <index>`                          | 通过下标和key取出元素                  |
| `llen<key>`                                     | 长度                                   |
| `linsert <key> before/after <value> <newvalue>` | 在value前或者后添加一个新的值          |
| `lrem <key> <n> <value>`                        | 从左边删除n个value                     |
| `lset <key><index><value>`                      | 将列表key下标为index的值替换为value    |

## 2.4 Redis Set

简介：自动排重

| 命令                                 | 作用                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| `sadd<key><value1><value2>.....`     | 将一个或者多个member元素加入集合key中，已经存在的member元素将被忽略 |
| `smembers<key>`                      | 取出所有值                                                   |
| `sismember <key><value>`             | 判断集合<key>是否有该value的值                               |
| `scard<key>`                         | 返回集合元素的个数                                           |
| `srem<key><value1><value2>`          | 删除集合中的某个元素                                         |
| `spop <key>`                         | 随机从该集合吐出一个值                                       |
| `srandmember<key><n>`                | 随机从该集合取出n个值。不会从集合中删除                      |
| `smove <source><destination><value>` | value把集合中一个值从一个集合移动到另一个集合                |
| `sinter<key1><key2>`                 | 返回两个集合的交集                                           |
| `sunion<key1><key2>`                 | 返回两个集合的并集                                           |
| `sdiff<k1><k2>`                      | 返回两个集合的差集（只存在于k1中，不存在k2中）               |

## 2.5 Redis Hash

Redis hash是一个键值对集合

Redis Hash 是一个String类型的field和value的映射表，hash特别适合用于存储对象。类似java里面的Map<String , Object>

相当于 key ----(field,value)

| 命令                                          | 作用                                                       |
| --------------------------------------------- | ---------------------------------------------------------- |
| `hset <key><field><value>`                    | 给key集合中的 field健赋值value                             |
| `hget<key1><field>`                           | 从key1集合中的field取出value                               |
| `hmset<key1><field1><value1><field2><value2>` | 一次赋值多个                                               |
| `hexists<key><field>`                         | 查看key中field是否存在                                     |
| `hkeys<key>`                                  | 列出所有field                                              |
| `hvals<key>`                                  | 列出所有value                                              |
| `hincrby <key><field><increment>`             |                                                            |
| `hsetnx<key><field><value>`                   | 将hash表key中的field的值设置为value，当且仅当域field不存在 |

Hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtale

## 2.6 Redis Zset(sorted set)

Zset和set不同的地方是有序集合每个成员都关联了一个评分（score），这个评分被用来按照从低到高的方式排序集合中的成员。**集合的成员是唯一的，但是评分可以是重复的**

因为元素是有序的，所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素

访问有序集合的中间元素也是非常快的，因此你能够使用有序集合作为一个没有重复成员的智能列表

| 代码                                                         | 作用                                                 |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| `zadd <key> <score1> <value1><score2> <value2>.....`         | 将一个或多个member元素及其score值加入到有序集key当中 |
| `zrange <key> <start>< end>（withscores）`                   | 取出值(并且显示评分)                                 |
| `zrangebyscore key <minscore><maxscore>(withscores)（limit 开始 大小）` | 取出在score范围内的value（并显示score）              |
| `zrevrangebyscore key <minscore><maxscore>(withscores)（limit 开始 大小）` | 降序取出在score范围内的value（并显示score）          |
| `zincrby <key> <increment><value>`                           | 为元素的score加上增量                                |
| `zrem<key><value>`                                           | 删除集合下，指定值的元素                             |
| `zcount<key><min><max>`                                      | 统计该集合下，分数区间内的元素个数                   |
| `zrank <key><value>`                                         | 返回该值在集合中的排名，从0开始                      |

# 三，Redis配置文件

## 3.1 配置单位

只支持bytes，大小写不敏感

```bash
# Redis configuration file example.
#
# Note that in order to read the configuration file, Redis must be
# started with the file path as first argument:
#
# ./redis-server /path/to/redis.conf

# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.
```



## 3.2 includes 包含

可以有其他文件

````bash
################################## INCLUDES ###################################

# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis servers but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#
# Note that option "include" won't be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel. Since Redis always uses the last processed
# line as value of a configuration directive, you'd better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
#
# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
#
# include /path/to/local.conf
# include /path/to/other.conf
````

## 3.3 network

```bash
################################## NETWORK #####################################

# By default, if no "bind" configuration directive is specified, Redis listens
# for connections from all available network interfaces on the host machine.
# It is possible to listen to just one or multiple selected interfaces using
# the "bind" configuration directive, followed by one or more IP addresses.
# Each address can be prefixed by "-", which means that redis will not fail to
# start if the address is not available. Being not available only refers to
# addresses that does not correspond to any network interfece. Addresses that
# are already in use will always fail, and unsupported protocols will always BE
# silently skipped.
#
# Examples:
#
# bind 192.168.1.100 10.0.0.1     # listens on two specific IPv4 addresses
# bind 127.0.0.1 ::1              # listens on loopback IPv4 and IPv6
# bind * -::*                     # like the default, all available interfaces
#
# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
# internet, binding to all the interfaces is dangerous and will expose the
# instance to everybody on the internet. So by default we uncomment the
# following bind directive, that will force Redis to listen only on the
# IPv4 and IPv6 (if available) loopback interface addresses (this means Redis
# will only be able to accept client connections from the same host that it is
# running on).
#
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT OUT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
bind 127.0.0.1 -::1 
(表示只能本地连接)

# Protected mode is a layer of security protection, in order to avoid that
# Redis instances left open on the internet are accessed and exploited.
#
# When protected mode is on and if:
#
# 1) The server is not binding explicitly to a set of addresses using the
#    "bind" directive.
# 2) No password is configured.
#
# The server only accepts connections from clients connecting from the
# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
# sockets.
#
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured, nor a specific set of interfaces
# are explicitly listed using the "bind" directive.
protected-mode yes
（开启保护模式，只能本地访问）


如果想远程访问：注释bind，把保护模式改为no


# Accept connections on the specified port, default is 6379 (IANA #815344).
# If port 0 is specified Redis will not listen on a TCP socket.
port 6379
端口号默认
# TCP listen() backlog.
#
# In high requests-per-second environments you need a high backlog in order
# to avoid slow clients connection issues. Note that the Linux kernel
# will silently truncate it to the value of /proc/sys/net/core/somaxconn so
# make sure to raise both the value of somaxconn and tcp_max_syn_backlog
# in order to get the desired effect.
tcp-backlog 511
backlog是一个连接队列，backlog队列总和=未完成三次握手队列+已经完成三次握手队列
# Unix socket.
#
# Specify the path for the Unix socket that will be used to listen for
# incoming connections. There is no default, so Redis will not listen
# on a unix socket when not specified.
#
# unixsocket /run/redis.sock
# unixsocketperm 700

# Close the connection after a client is idle for N seconds (0 to disable)
timeout 0
一个空闲的客户端维持多久会关闭，0表示关闭该功能。即永不关闭
# TCP keepalive.
#
# If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
# of communication. This is useful for two reasons:
#
# 1) Detect dead peers.
# 2) Force network equipment in the middle to consider the connection to be
#    alive.
#
# On Linux, the specified value (in seconds) is the period used to send ACKs.
# Note that to close the connection the double of the time is needed.
# On other kernels the period depends on the kernel configuration.
#
# A reasonable value for this option is 300 seconds, which is the new
# Redis default starting with Redis 3.2.1.
tcp-keepalive 300
检查心跳时间，每隔300检查一次
```

## 3.4 general

```bash
################################# GENERAL #####################################

# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
# When Redis is supervised by upstart or systemd, this parameter has no impact.
daemonize yes
后台启动
# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#                        requires "expect stop" in your upstart job config
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#                        on startup, and updating Redis status on a regular
#                        basis.
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous pings back to your supervisor.
#
# The default is "no". To run under upstart/systemd, you can simply uncomment
# the line below:
#
# supervised auto

# If a pid file is specified, Redis writes it where specified at startup
# and removes it at exit.
#
# When the server runs non daemonized, no pid file is created if none is
# specified in the configuration. When the server is daemonized, the pid file
# is used even if not specified, defaulting to "/var/run/redis.pid".
#
# Creating a pid file is best effort: if Redis is not able to create it
# nothing bad happens, the server will start and run normally.
#
# Note that on modern Linux systems "/run/redis.pid" is more conforming
# and should be used instead.
pidfile /var/run/redis_6379.pid
保存进程号
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice
日志级别
# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile ""
设置日志的输出文件路径
# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# syslog-enabled no

# Specify the syslog identity.
# syslog-ident redis

# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
# syslog-facility local0

# To disable the built in crash log, which will possibly produce cleaner core
# dumps when they are needed, uncomment the following:
#
# crash-log-enabled no

# To disable the fast memory check that's run as part of the crash log, which
# will possibly let redis terminate sooner, uncomment the following:
#
# crash-memcheck-enabled no

# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16
默认使用16号数据库
# By default Redis shows an ASCII art logo only when started to log to the
# standard output and if the standard output is a TTY and syslog logging is
# disabled. Basically this means that normally a logo is displayed only in
# interactive sessions.
#
# However it is possible to force the pre-4.0 behavior and always show a
# ASCII art logo in startup logs by setting the following option to yes.
always-show-logo no

# By default, Redis modifies the process title (as seen in 'top' and 'ps') to
# provide some runtime information. It is possible to disable this and leave
# the process name as executed by setting the following to no.
set-proc-title yes

# When changing the process title, Redis uses the following template to construct
# the modified title.
#
# Template variables are specified in curly brackets. The following variables are
# supported:
#
# {title}           Name of process as executed if parent, or type of child process.
# {listen-addr}     Bind address or '*' followed by TCP or TLS port listening on, or
#                   Unix socket if only that's available.
# {server-mode}     Special mode, i.e. "[sentinel]" or "[cluster]".
# {port}            TCP port listening on, or 0.
# {tls-port}        TLS port listening on, or 0.
# {unixsocket}      Unix domain socket listening on, or "".
# {config-file}     Name of configuration file used.
#
proc-title-template "{title} {listen-addr} {server-mode}"
```

## 3.5 security

```bash
################################## SECURITY ###################################

# Warning: since Redis is pretty fast, an outside user can try up to
# 1 million passwords per second against a modern box. This means that you
# should use very strong passwords, otherwise they will be very easy to break.
# Note that because the password is really a shared secret between the client
# and the server, and should not be memorized by any human, the password
# can be easily a long string from /dev/urandom or whatever, so by using a
# long and unguessable password no brute force attack will be possible.

# Redis ACL users are defined in the following format:
#
#   user <username> ... acl rules ...
#
# For example:
#
#   user worker +@list +@connection ~jobs:* on >ffa9203c493aa99
#
# The special username "default" is used for new connections. If this user
# has the "nopass" rule, then new connections will be immediately authenticated
# as the "default" user without the need of any password provided via the
# AUTH command. Otherwise if the "default" user is not flagged with "nopass"
# the connections will start in not authenticated state, and will require
# AUTH (or the HELLO command AUTH option) in order to be authenticated and
# start to work.
#
# The ACL rules that describe what a user can do are the following:
#
#  on           Enable the user: it is possible to authenticate as this user.
#  off          Disable the user: it's no longer possible to authenticate
#               with this user, however the already authenticated connections
#               will still work.
#  skip-sanitize-payload    RESTORE dump-payload sanitation is skipped.
#  sanitize-payload         RESTORE dump-payload is sanitized (default).
#  +<command>   Allow the execution of that command
#  -<command>   Disallow the execution of that command
#  +@<category> Allow the execution of all the commands in such category
#               with valid categories are like @admin, @set, @sortedset, ...
#               and so forth, see the full list in the server.c file where
#               the Redis command table is described and defined.
#               The special category @all means all the commands, but currently
#               present in the server, and that will be loaded in the future
#               via modules.
#  +<command>|subcommand    Allow a specific subcommand of an otherwise
#                           disabled command. Note that this form is not
#                           allowed as negative like -DEBUG|SEGFAULT, but
#                           only additive starting with "+".
#  allcommands  Alias for +@all. Note that it implies the ability to execute
#               all the future commands loaded via the modules system.
#  nocommands   Alias for -@all.
#  ~<pattern>   Add a pattern of keys that can be mentioned as part of
#               commands. For instance ~* allows all the keys. The pattern
#               is a glob-style pattern like the one of KEYS.
#               It is possible to specify multiple patterns.
#  allkeys      Alias for ~*
#  resetkeys    Flush the list of allowed keys patterns.
#  &<pattern>   Add a glob-style pattern of Pub/Sub channels that can be
#               accessed by the user. It is possible to specify multiple channel
#               patterns.
#  allchannels  Alias for &*
#  resetchannels            Flush the list of allowed channel patterns.
#  ><password>  Add this password to the list of valid password for the user.
#               For example >mypass will add "mypass" to the list.
#               This directive clears the "nopass" flag (see later).
#  <<password>  Remove this password from the list of valid passwords.
#  nopass       All the set passwords of the user are removed, and the user
#               is flagged as requiring no password: it means that every
#               password will work against this user. If this directive is
#               used for the default user, every new connection will be
#               immediately authenticated with the default user without
#               any explicit AUTH command required. Note that the "resetpass"
#               directive will clear this condition.
#  resetpass    Flush the list of allowed passwords. Moreover removes the
#               "nopass" status. After "resetpass" the user has no associated
#               passwords and there is no way to authenticate without adding
#               some password (or setting it as "nopass" later).
#  reset        Performs the following actions: resetpass, resetkeys, off,
#               -@all. The user returns to the same state it has immediately
#               after its creation.
#
# ACL rules can be specified in any order: for instance you can start with
# passwords, then flags, or key patterns. However note that the additive
# and subtractive rules will CHANGE MEANING depending on the ordering.
# For instance see the following example:
#
#   user alice on +@all -DEBUG ~* >somepassword
#
# This will allow "alice" to use all the commands with the exception of the
# DEBUG command, since +@all added all the commands to the set of the commands
# alice can use, and later DEBUG was removed. However if we invert the order
# of two ACL rules the result will be different:
#
#   user alice on -DEBUG +@all ~* >somepassword
#
# Now DEBUG was removed when alice had yet no commands in the set of allowed
# commands, later all the commands are added, so the user will be able to
# execute everything.
#
# Basically ACL rules are processed left-to-right.
#
# For more information about ACL configuration please refer to
# the Redis web site at https://redis.io/topics/acl

# ACL LOG
#
# The ACL Log tracks failed commands and authentication events associated
# with ACLs. The ACL Log is useful to troubleshoot failed commands blocked 
# by ACLs. The ACL Log is stored in memory. You can reclaim memory with 
# ACL LOG RESET. Define the maximum entry length of the ACL Log below.
acllog-max-len 128

# Using an external ACL file
#
# Instead of configuring users here in this file, it is possible to use
# a stand-alone file just listing users. The two methods cannot be mixed:
# if you configure users here and at the same time you activate the external
# ACL file, the server will refuse to start.
#
# The format of the external ACL user file is exactly the same as the
# format that is used inside redis.conf to describe users.
#
# aclfile /etc/redis/users.acl

# IMPORTANT NOTE: starting with Redis 6 "requirepass" is just a compatibility
# layer on top of the new ACL system. The option effect will be just setting
# the password for the default user. Clients will still authenticate using
# AUTH <password> as usually, or more explicitly with AUTH default <password>
# if they follow the new protocol: both will work.
#
# The requirepass is not compatable with aclfile option and the ACL LOAD
# command, these will cause requirepass to be ignored.
#
# requirepass foobared

# New users are initialized with restrictive permissions by default, via the
# equivalent of this ACL rule 'off resetkeys -@all'. Starting with Redis 6.2, it
# is possible to manage access to Pub/Sub channels with ACL rules as well. The
# default Pub/Sub channels permission if new users is controlled by the 
# acl-pubsub-default configuration directive, which accepts one of these values:
#
# allchannels: grants access to all Pub/Sub channels
# resetchannels: revokes access to all Pub/Sub channels
#
# To ensure backward compatibility while upgrading Redis 6.0, acl-pubsub-default
# defaults to the 'allchannels' permission.
#
# Future compatibility note: it is very likely that in a future version of Redis
# the directive's default of 'allchannels' will be changed to 'resetchannels' in
# order to provide better out-of-the-box Pub/Sub security. Therefore, it is
# recommended that you explicitly define Pub/Sub permissions for all users
# rather then rely on implicit default values. Once you've set explicit
# Pub/Sub for all existing users, you should uncomment the following line.
#
# acl-pubsub-default resetchannels

# Command renaming (DEPRECATED).
#
# ------------------------------------------------------------------------
# WARNING: avoid using this option if possible. Instead use ACLs to remove
# commands from the default user, and put them only in some admin user you
# create for administrative purposes.
# ------------------------------------------------------------------------
#
# It is possible to change the name of dangerous commands in a shared
# environment. For instance the CONFIG command may be renamed into something
# hard to guess so that it will still be available for internal-use tools
# but not available for general clients.
#
# Example:
#
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#
# It is also possible to completely kill a command by renaming it into
# an empty string:
#
# rename-command CONFIG ""
#
# Please note that changing the name of commands that are logged into the
# AOF file or transmitted to replicas may cause problems.


```



## 3.6 limits



# 四，发布和订阅

## 4.1 什么是发布和订阅

Redis发布订阅（push/sub）是一种通信模式：发送者（pub）发送消息，订阅者（sub）接收消息。

Redis客户端可以订阅**任意数量**的频道

subscribe channel1

publish channel1 hello

# 五，Redis6新的数据类型

## 5.1 Bitmaps

主要进行位操作

+ Bitmaps本身不是一种数据类型，实际上它是字符串（key-value），但是它可以对字符串的位进行操作
+ Bitmaps单独提供了一套命令，所以在Redis种使用Bitmaps和使用字符串的方法不太相同。可以把Bitmaps想象成一个以位为单位的数组，数组的每个单元只能存储0和1，数组的下标在Bitmaps中叫做偏移量。

​		key 0101001010...

| 命令                                       | 作用                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| `setbit <key> <offset> <value>`            | 设置某个偏移量的值（会返回0）                                |
| `getbit<key><offset>`                      | 取出偏移量                                                   |
| `bitcount<key>[start end]`                 | 取出范围内值为1的数量                                        |
| `bitop and(or/not/xor) <destkey> [key...]` | 复合操作，and（交集）or（并集）not（非集） xor（异或）并将结果存到destkey |



## 5.2 HyperLogLog

**简介**

Redis 在 2.8.9 版本添加了 HyperLogLog 结构。

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

| 命令                                   | 描述                                      |
| -------------------------------------- | ----------------------------------------- |
| `pfadd key element[elements]`          | 添加指定元素到HyperLogLog中               |
| `pfcount key[key....]`                 | 返回给定HyperLogLog的基数估算值           |
| `pfmerge destkey sourcekey[sourcekey]` | 将多个 HyperLogLog 合并为一个 HyperLogLog |



## 5.3 Geospatial

就是地理位置，该元素是二维坐标，在地图上就是经纬度。redis基于该类型，提供了经纬度设置，查询，范围查询，距离查询，经纬度hash等常见操作

| 命令                                                         | 功能                               |
| ------------------------------------------------------------ | ---------------------------------- |
| `geoadd <key> <longitude><latitude><member>....`             | 添加地理位置                       |
| `geopos <key> member.....`                                   | 取出位置                           |
| `geodist <key><member1><member2> [m/km/ft/mi]`               | 两个位置的直线距离，可以自定义单位 |
| `georedius  <key> <longitude><latitude> <redius> [m/km/ft/mi]` | 找出某一半径内的元素               |

# 六，Jedis操作redis

## 6.1 Jedis测试

**引入依赖**

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>4.2.1</version>
</dependency>
```

**编写测试类**

```java
@Test
    public void testjedis(){
        String host = "144.24.72.***";
        int port = 6379;
        String pass = "yuqianredis";
        Jedis jedis = new Jedis(host,port);
        jedis.auth(pass);
        String ping = jedis.ping();
        System.out.println(ping);
    }
```

**key**

```java
Set<String> keys = jedis.keys("*");
```

**String**

```java
jedis.set("String1","helloJedis");
System.out.println("String1中存的是"+jedis.get("String1"));
```

**List**

```java
jedis.lpush("list2","jedis1","jedis2","jedis3");
List<String> list2 = jedis.lrange("list2", 0, -1);
```

....方法名和命令都差不多。。。

## 6.2 Springboot整合Jedis

0.0

# 七，事务_锁机制，秒杀

## 7.1 Redis事务定义

Redis事务是一个单独的隔离操作，事务中的所有命令都会序列化，按顺序执行。事务在执行过程中，不会被其他客户端发送来的命令打断

Redis事务的主要作用就是**串联多个命令**防止别的命令插队

## 7.2 Multi，Exec，Discard

输入Multi命令开始，输入的命令都会都会依次进入到队列命令中，但是不会执行，知道输入Exec后，Redis会将之前的命令一次执行

组队过程中可以使用discard来放弃组队

**组队过程中**：**如果某个命令出错**，**整个队列都会取消**

**执行过程中：如果某个命令出错，会跳过执行出错的命令继续执行**

## 7.3 事务冲突问题

### **悲观锁**：

每次拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block。传统的关系型数据库里面就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等。都是在操作之前先上锁

### **乐观锁**

每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有更新数据，可以使用版本号等机制。**乐观锁适用于多读的应用类型，这样可以提高吞吐量**Redis就是利用这种Check-And-Set的机制实现事务的

### WATCH Key[keys ....]

**在执行multi之气那，先执行watch key1[key2]，可以监视一个或多个key，如果在事务执行之前这些key被其他命令改动，那么事务会被打断**

unwatch取消对所有key的监视，如果在执行watch命令之后，exec命令或discard命令先被执行了的话，那么就不需要再执行unwatch了

## 7.4 Redis事务的三特性

**单独的隔离操作**

+ 事务中的所有命令都会序列化，按请求顺序的执行。事务在执行过程中，不会被其他客户端发来的命令请求所打断

**没有隔离级别的概念**

+ 队列中的命令没有提交之前都不会被实际执行，因为事务提交前任何指令都不会被实际执行

**不保证原子性**

+ 事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚

## 7.5 Redis秒杀案例

1. 解决计数器和人员记录的事务操作

   

![](https://raw.githubusercontent.com/yqimg/img/main/20220405133634.png)

