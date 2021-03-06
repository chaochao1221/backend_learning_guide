# 事务操作

### 定义：
- Redis 通过 MULTI、EXEC、WATCH 等命令来实现事务（transaction）功能。事务提供了一种将多个命令请求打包，然后一次性、按顺序地执行多个命令的机制，并且在事务执行期间，服务器不会中断事务而改去执行其他客户端的命令请求，它会将事务中的所有命令都执行完毕，然后才去处理其他客户端的命令请求。

### 事务的实现步骤：

1. 开始事务：
    - MULTI 命令的执行标记着事务的开始。这个命令唯一做的就是， 将客户端的 REDIS_MULTI 选项打开， 让客户端从非事务状态切换到事务状态。
2. 命令入队：
    - 当一个客户端切换到事务状态之后，服务器会根据这个客户端发来的不同命令执行不同的操作。
    - 如果客户端发送的命令为EXEC、DISCARD、WATCH、MULTI四个命令的其中一个，那么服务器立即执行这个命令。
    - 与此相反，如果客户端发送的其他命令，那么服务器并不立即执行这个命令，而是将这个命令放入一个事务队列里面，然后向客户端返回QUEUED回复。
    - 事务队列是一个数组，每个数组项是都包含三个属性：要执行的命令（cmd）、命令的参数（argv）、参数的个数（argc）。
3. 执行事务。
    - 如果客户端正处于事务状态，那么当 EXEC 命令执行时，服务器根据客户端所保存的事务队列，以先进先出（FIFO）的方式执行事务队列中的命令：最先入队的命令最先执行，而最后入队的命令最后执行。当事务队列里的所有命令被执行完之后， EXEC 命令会将回复队列作为自己的执行结果返回给客户端， 客户端从事务状态返回到非事务状态，至此，事务执行完毕。

### MULTI 命令：

1. MULTI 命令的执行标记着事务的开始。Redis 的事务是不可嵌套的，当客户端已经处于事务状态，而客户端又再向服务器发送 MULTI 时，服务器只是简单地向客户端发送一个错误，然后继续等待其他命令的入队。MULTI 命令的发送不会造成整个事务失败， 也不会修改事务队列中已有的数据。

### WATCH 命令：

1. WATCH 命令是一个乐观锁（optimistic locking），它可以在 EXEC 命令执行之前，监视任意数量的数据库键，并在 EXEC 命令执行时，检查被监视的键是否至少有一个已经被修改过了，如果是的话，服务器将拒绝执行事务，并向客户端返回代表事务执行失败的空回复。
2. WATCH 只能在客户端进入事务状态之前执行，在事务状态下发送 WATCH 命令会引发一个错误，但它不会造成整个事务失败，也不会修改事务队列中已有的数据（和前面处理 MULTI 的情况一样）。

### DISCARD 命令：

1. DISCARD 命令用于取消一个事务，它清空客户端的整个事务队列，然后将客户端从事务状态调整回非事务状态，最后返回字符串 OK 给客户端，说明事务已被取消。

### WATCH 命令的实现：

1. 在每个代表数据库的 redis.h/redisDb 结构类型中，都保存了一个 watched_keys 字典，字典的键是这个数据库被监视的键，而字典的值则是一个链表，链表中保存了所有监视这个键的客户端。
2. 所有对数据库进行修改的命令，比如 SET、LPUSH、SADD、ZREM、DEL、FLUSHDB 等等，在执行之后都会调用 multi.c/touchWatchKey 函数对 watched_keys 字典进行检查，查看是否有客户端正在监视刚刚被命令修改过的数据库键，如果有的话，那么 touchWatchKey 函数会将监视被修改键的客户端的 REDIS_DIRTY_CAS 标识打开，表示该客户端的事务安全性已经被破坏。
3. 当客户端发送 EXEC 命令、触发事务执行时，服务器会对客户端的状态进行检查：
    - 如果客户端的 REDIS_DIRTY_CAS 选项已经被打开，那么说明被客户端监视的键至少有一个已经被修改了，事务的安全性已经被破坏。服务器会放弃执行这个事务，直接向客户端返回空回复，表示事务执行失败。
    - 如果 REDIS_DIRTY_CAS 选项没有被打开，那么说明所有监视键都安全，服务器正式执行事务。
4. 最后，当一个客户端结束它的事务时，无论事务是成功执行，还是失败，watched_keys 字典中和这个客户端相关的资料都会被清除。

### 事务的 ACID 性质：

1. 在Redis中，事务总是具有原子性（Atomicity）、一致性（Consistency）和隔离性（Isolation），并且当服务器运行在AOF持久化模式下，并且appendfsync选项的值为always时，事务也具有持久性（Durability）。
2. 原子性：对于Redis的事务功能来说，事务队列中的命令要么就全部都执行，要么就一个都不执行，因此，Redis的事务是具有原子性的（Redis的事务和传统的关系型数据库事务的最大区别在于，Redis不支持事务回滚机制（rollback），即使事务队列中的某个命令在执行期间出现了错误，整个事务也会继续执行下去，直到将事务队列中的所有命令都执行完毕为止）。
3. 一致性：分为三部分来讨论：入队错误、执行错误、Redis 进程被终结。
    - 入队错误：如果一个事务在入队命令的过程中，出现了命令不存在，或者命令的格式不正确等情况，那么Redis将拒绝执行这个事务。
    - 执行错误：如果命令在事务执行的过程中发生错误，比如说，对一个不同类型的 key 执行了错误的操作，那么 Redis 只会将错误包含在事务的结果中，这不会引起事务中断或整个失败，不会影响已执行事务命令的结果，也不会影响后面要执行的事务命令，所以这些出错命令不会对数据库做任何修改，也不会对事务的一致性产生任何影响。
    - 服务器停机：如果Redis服务器在执行事务的过程中停机，那么根据服务器所使用的持久化模式，可能有以下情况出现：1.如果服务器运行在无持久化的内存模式下，那么重启之后的数据库将是空白的，因此数据总是一致的。2.如果服务器运行在RDB模式下，那么在事务中途停机不会导致不一致性，因为服务器可以根据现有的RDB文件来恢复数据，从而将数据库还原到一个一致的状态。如果找不到可供使用的RDB文件，那么重启之后的数据库将是空白的，而空白数据库总是一致的。3.如果服务器运行在AOF模式下，那么在事务中途停机不会导致不一致性，因为服务器可以根据现有的AOF文件来恢复数据，从而将数据库还原到一个一致的状态。如果找不到可供使用的AOF文件，那么重启之后的数据库将是空白的，而空白数据库总是一致的。
4. 隔离行：因为Redis使用单线程的方式来执行事务（以及事务队列中的命令），并且服务器保证，在执行事务期间不会对事务进行中断，因此，Redis的事务总是以串行的方式运行的，并且事务也总是具有隔离性的。
5. 持久性：因为Redis的事务不过是简单地用队列包裹起了一组Redis命令，Redis并没有为事务提供任何额外的持久化功能，所以Redis事务的持久性由Redis所使用的持久化模式决定。（仅当服务器运行在AOF持久化模式下，并且appendfsync选项的值为always时，程序总会在执行命令之后调用同步（sync）函数，将命令数据真正地保存到硬盘里面，因此这种配置下的事务是具有持久性的。）

### 总结

1. 事务提供了一种将多个命令打包，然后一次性、有序地执行的机制。
2. 多个命令会被入队到事务队列中，然后按先进先出（FIFO）的顺序执行。
3. 事务在执行过程中不会被中断，当事务队列中的所有命令都被执行完毕之后，事务才会结束。
4. 带有WATCH命令的事务会将客户端和被监视的键在数据库的watched_keys字典中进行关联，当键被修改时，程序会将所有监视被修改键的客户端的REDIS_DIRTY_CAS标志打开。
5. 只有在客户端的REDIS_DIRTY_CAS标志未被打开时，服务器才会执行客户端提交的事务，否则的话，服务器将拒绝执行客户端提交的事务。
6. Redis的事务总是具有ACID中的原子性、一致性和隔离性，当服务器运行在AOF持久化模式下，并且appendfsync选项的值为always时，事务也具有耐久性。
