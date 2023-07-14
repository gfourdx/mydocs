# Redis持久化

以redis7.0.11举例

---

### RDB

RDB也就是快照(SNAPSHOTTING)方式, 恢复时直接将RDB数据加载到内存中

在redis.conf中的相关配置参数为:

- save
- stop-writes-on-bgsave-error
- rdbcompression
- rdbchecksum
- sanitize-dump-payload
- dbfilename
- rdb-del-sync-files
- dir

其中最主要的是`save`参数

##### save

格式: **save** <seconds> <changes> [<seconds> <changes> ...]

配置保存数据到磁盘的策略

示例1: save 36000 1, 表示在3600秒内有一次修改操作就保存快照到磁盘

示例2: save 3600 1 300 100 60 10000, 表示在3600秒内有一次修改操作或300秒内有100次修改操作或60妙内有10000操作就保存快照到磁盘

示例3: save "", 表示禁用快照

关于手动触发持久化:

> 在redis-cli中可使用save或bgsave命令, 区别在于
>
> save是同步的, 使用当前进程进行持久化, 会阻塞当前进程, 即在持久化完成之前redis无法处理增删改查命令
>
> bgsave则是异步的, 即fork一个子进程进程持久化处理, 不会阻塞当前进程, 但会额外消耗内存
>
> 同时, bgsave利用了系统提供的写时复制(Copy-on-Write, COP)技术, 持久化开始时将当前数据写入RDB文件, 写入文件期间如果有新数据写入, 新数据也会写入RDB文件

##### stop-writes-on-bgsave-error

格式: stop-writes-on-bgsave-error yes|no

默认值是yes, 表示:

1、如果启用了RDB快照并且最新的后台保存(bgsave)失败, 则redis将停止接受写入, 是用户意识到数据没有正确的保存到磁盘上

2、如果后台保存过程再次开始工作，则Redis将自动允许再次写入

设置为no, 表示无论快照是否保存成功, redis都允许继续写入

##### rdbcompression

格式: rdbcompression yes|no

默认值是yes, 表示使用LZF压缩RDB数据, 提高持久化速度并减少磁盘占用, 但会增加CPU使用量

##### rdbchecksum

格式: rdbchecksum yes|no

默认值是yes, 表示持久化数据时使用CRC64计算数据的校验和, 数据恢复时读取持久化文件中的校验和并与实际数据进行比对, 以确保数据的准确性和完整性, 但会多消耗约10%的性能, 同时也会降低持久化速度

##### sanitize-dump-payload

格式: sanitize-dump-payload yes|no|clients

配置Redis使用RDB持久化时是否进行脱敏处理, 通常是将敏感信息替换为特定的占位符或进行加密处理, 以防止数据泄露

默认值是no且默认被注释掉(禁用)

设置为no时, Redis在进行RDB持久化时不会对数据进行脱敏处理, 持久化文件中的数据将保持原样

设置为yes时, Redis在进行RDB持久化时会对数据进行脱敏处理

设置为clients时，Redis会对RDB持久化中的数据进行脱敏处理，但只有来自外部客户端的请求数据会被脱敏，而内部的Redis命令和数据将保持原样。这样可以保护来自外部客户端的敏感信息，同时保持内部数据的可读性和完整性。

##### dbfilename

格式: rdbchecksum "string"

"string"表示生成的RDB文件名称, 默认值是dump.rdb

##### rdb-del-sync-files

格式: rdb-del-sync-files yes|no

默认值是no

在未启用持久性的实例中删除复制使用的RDB文件

简单说就是主从同步数据时使用RDB文件, 同步完成后是否删除RDB文件

该选项仅在同时禁用AOF和RDB持久化的实例中有效, 否则将完全被忽略

##### dir

格式: dir "path"

"path"表示存放RDB文件的目录, 默认值是/var/lib/redis

### AOF

aof即append-only file, 即将每条redis命令追加到appendfilename配置的文件中

并根据appendfsync方式同步到磁盘中, 恢复时读取AOF文件中的命令并逐条执行

在redis.conf中的相关配置参数为:

- appendonly
- appendfilename
- appenddirname
- appendfsync
- no-appendfsync-on-rewrite
- auto-aof-rewrite-percentage
- auto-aof-rewrite-min-size
- aof-load-truncated
- aof-use-rdb-preamble yes
- aof-timestamp-enabled

##### appendonly

格式: appendonly yes|no

默认值是no, yes表示启用aof持久化

##### appendfilename

同dbfilename, 设置aof文件名称, 默认值是appendonly.aof

从7.0版本开始使用一组文件进行持久化, 有两种可用的文件类型:

- 基本文件，它们是表示创建文件时数据集的完整状态的快照, 基本文件可以是RDB(二进制序列化)或AOF(文本命令)的形式。

- 增量文件，其中包含应用于上一个文件之后的数据集的其他命令

简单来说会生成3个文件:

- appendonly.aof.1.base.rdb, 实际上就是rdb文件

- appendonly.aof.1.incr.aof, 实际上就是aof文件

- appendonly.aof.manifest, 上述两个文件的说明

##### appenddirname

格式: appenddirname "string"

"string"表示存放aof文件的目录, 此目录事相对目录, 保存在dir配置的目录下

##### appendfsync

格式: appendfsync always|everysec|no

配置aof文件刷写磁盘的方式

默认值是everysec, 表示每秒刷写一次, 如果发生故障, 最多丢失1秒的数据

其他两个可选配置是always和no, always表示每次对redis的增删改命令都会写入aof并刷写到磁盘

此种方式最安全, 不会丢失数据但效率最慢

如果是no则表示交由系统处理, 速度最快但是最不安全

##### no-appendfsync-on-rewrite

格式: no-appendfsync-on-rewrite yes|no

默认值是no

这里涉及到aof文件重写机制, 例如在redis-cli中多次使用set对同一个key设置value或者incr对某个key的value自增

对于aof文件来说, 里面的命令是冗余的, 只要拿到最后一个值即可, 而重写机制能减少aof文件对磁盘的占用

当appendfsync配置为always或everysec时, 主进程会调用系统的fsync(), 会消耗大量的I/O, 可能导致阻塞太长时间

缓解办法是使用bgsave或bgrewriteaof, 这样就不会使用主进程调用fsync()而导致阻塞, 但依旧会消耗大量I/O

如果将此配置的值配置为yes, 那么当时会暂停对aof文件的同步(即刷写磁盘), 减少I/O, 但可能会导致数据丢失

因此如果磁盘I/O问题不是瓶颈, 建议保持默认配置, 即 no

##### auto-aof-rewrite-percentage

格式: auto-aof-rewrite-percentage percentage

percentage 是数字, 表示百分比, 默认值是100

当AOF日志大小增长到指定的百分比时, Redis能够自动重写隐式调用BGREWRITEAOF的日志文件

若设置为0则表示禁用aof重写

**注意, 需要配合auto-aof-rewrite-min-size参数一起使用**

##### auto-aof-rewrite-min-size

格式: auto-aof-rewrite-min-size size

size 是数字+单位, 表示文件大小, , 默认值是64mb

当aof文件大小到达指定值时进行重写, 后续再次重写则会根据auto-aof-rewrite-percentage配置的值计算, 满足条件后会再次重写

##### aof-load-truncated

格式: aof-load-truncated yes|no

默认值是yes

当重启redis后会自动加载aof文件, 如果发现aof文件末尾被截断(可以理解为aof文件内容格式不正确或损坏), 控制是否仍然加载正确的部分, 否则报错(配置为no时)

**注意, 如果aof文件从中间出错, 不是末尾, 即使配置为yes, redis重启时加载aof也会报错**

### aof-use-rdb-preamble

格式: aof-use-rdb-preamble yes|no

默认值yes, 表示混合持久化, 即同时使用RDB和AOF

具体就是将当前内存中的数据写入RDB文件(前面提到的appendonly.aof.1.base.rdb)

再次期间redis接收的操作将写入AOF文件(前面提到的appendonly.aof.1.incr.aof)

即加快的速度也保证了数据的安全

**注意: 开启混合持久化, 不需要开启RDB持久化, 即无需配置save参数**

##### aof-timestamp-enabled

格式: aof-timestamp-enabled yes|no

默认值是no, 如果改成yes, 则Redis支持在AOF中记录时间戳注释, 但可能出现不兼容情况

### RDB和AOF的对比

RDB的优势是磁盘空间占用小, 数据备份与恢复的速度快, 但可能会丢失大量数据

AOF的优势是数据量丢失较小, 但占用磁盘空间大, 恢复速度较慢

同时, AOF的启动优先级比RDB高

所以建议使用混合持久化