# Redis 持久化机制: RDB 和 AOF

## Redis Persistence

### Why do we need Persistence?

Redis 是基于内存的数据库，服务一旦宕机，内存中的数据将全部丢失。通常来说可以通过数据库来恢复这些数据，但这会给数据库带来非常大的读压力，并且这个过程将非常缓慢，并导致程序响应慢。因此数据的持久化就显得至关重要。

### Persistence Options

目前 [Redis Documentation](https://redis.io/docs/management/persistence/) 上对持久化的支持有以下几种方案：

1. RDB (Redis Database): RDB persistence performs point-in-time snapshots of your dataset at specified intervals.
2. AOF (Append Only File): AOF persistence logs every write operation received by the server. These operations can then be replayed again at server startup, reconstructing the original dataset. Commands are logged using the same format as the Redis protocol itself.
3. RDB + AOF: You can also combine both AOF and RDB in the same instance.

## RDB

#### 常见问题

一般来说 Redis 实例占用的内存都比较大，那么在使用 bgsave 将内存数据持久化的操作将会持续较长的时间，如果在持久化的过程中，Redis 服务收到了数据写操作请求，那么这样的写操作会被持久化么？如何保证持久化前后的数据一致性？


Copy-on-Write and its memory usage

Redis' snapshot backup leverage the CoW semantics, which is provided by modern operating system to resolve the issue that when forking processes, the memory of the parent process is copied to the child process thus doubles the memory footprint. In CoW, the forked child process will share the original memory space of the parent process. It only copies the memory page when either process modifies that memory page. Here is an illustration of the memory space before data modification and after data modification:


RDB advantages
RDB is a very compact single-file point-in-time representation of your Redis data. RDB files are perfect for backups. For instance you may want to archive your RDB files every hour for the latest 24 hours, and to save an RDB snapshot every day for 30 days. This allows you to easily restore different versions of the data set in case of disasters.
RDB is very good for disaster recovery, being a single compact file that can be transferred to far data centers, or onto Amazon S3 (possibly encrypted).
RDB maximizes Redis performances since the only work the Redis parent process needs to do in order to persist is forking a child that will do all the rest. The parent process will never perform disk I/O or alike.
RDB allows faster restarts with big datasets compared to AOF.
On replicas, RDB supports partial resynchronizations after restarts and failovers.
RDB disadvantages
RDB is NOT good if you need to minimize the chance of data loss in case Redis stops working (for example after a power outage). You can configure different save points where an RDB is produced (for instance after at least five minutes and 100 writes against the data set, you can have multiple save points). However you'll usually create an RDB snapshot every five minutes or more, so in case of Redis stopping working without a correct shutdown for any reason you should be prepared to lose the latest minutes of data.
RDB needs to fork() often in order to persist on disk using a child process. fork() can be time consuming if the dataset is big, and may result in Redis stopping serving clients for some milliseconds or even for one second if the dataset is very big and the CPU performance is not great. AOF also needs to fork() but less frequently and you can tune how often you want to rewrite your logs without any trade-off on durability.

## ADB


AOF advantages
Using AOF Redis is much more durable: you can have different fsync policies: no fsync at all, fsync every second, fsync at every query. With the default policy of fsync every second, write performance is still great. fsync is performed using a background thread and the main thread will try hard to perform writes when no fsync is in progress, so you can only lose one second worth of writes.
The AOF log is an append-only log, so there are no seeks, nor corruption problems if there is a power outage. Even if the log ends with a half-written command for some reason (disk full or other reasons) the redis-check-aof tool is able to fix it easily.
Redis is able to automatically rewrite the AOF in background when it gets too big. The rewrite is completely safe as while Redis continues appending to the old file, a completely new one is produced with the minimal set of operations needed to create the current data set, and once this second file is ready Redis switches the two and starts appending to the new one.
AOF contains a log of all the operations one after the other in an easy to understand and parse format. You can even easily export an AOF file. For instance even if you've accidentally flushed everything using the FLUSHALL command, as long as no rewrite of the log was performed in the meantime, you can still save your data set just by stopping the server, removing the latest command, and restarting Redis again.
AOF disadvantages
AOF files are usually bigger than the equivalent RDB files for the same dataset.
AOF can be slower than RDB depending on the exact fsync policy. In general with fsync set to every second performance is still very high, and with fsync disabled it should be exactly as fast as RDB even under high load. Still RDB is able to provide more guarantees about the maximum latency even in the case of a huge write load.
Redis < 7.0


AOF can use a lot of memory if there are writes to the database during a rewrite (these are buffered in memory and written to the new AOF at the end).
All write commands that arrive during rewrite are written to disk twice.
Redis could freeze writing and fsyncing these write commands to the new AOF file at the end of the rewrite.