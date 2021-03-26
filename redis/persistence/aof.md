# AOF 持久化

### 定义：AOF 以协议文本的方式，将所有对数据库进行过写入的命令（及其参数）记录到 AOF 文件，以此达到记录数据库状态的目的（为了处理的方便， AOF 文件使用网络通讯协议的格式来保存这些命令）。

### AOF 持久化的实现步骤：

```
1、命令传播：Redis 将执行完的命令、命令的参数、命令的参数个数等信息发送到 AOF 程序中。
2、缓存追加：AOF 程序根据接收到的写命令数据，会以协议格式将被执行的写命令追加到服务器状态的aof_buf缓冲区的末尾。
3、文件写入和保存：将aof_buf缓冲区中的内容被写入到 AOF 文件末尾，如果设定的 AOF 保存条件被满足的话， fsync 函数或者 fdatasync 函数会被调用，将写入的内容真正地保存到磁盘中。
```

### 文件写入和保存：

```
每当服务器常规任务函数被执行、 或者事件处理器被执行时， aof.c/flushAppendOnlyFile 函数都会被调用， 这个函数执行以下两个工作：
1、WRITE：根据条件，将 aof_buf 中的缓存写入到 AOF 文件。
2、SAVE：根据条件，调用 fsync 或 fdatasync 函数，将 AOF 文件保存到磁盘中。
```

### 保存模式：

```
1、AOF_FSYNC_NO ：不保存。
2、AOF_FSYNC_EVERYSEC ：每一秒钟保存一次。
3、AOF_FSYNC_ALWAYS ：每执行一个命令保存一次。

不保存
在这种模式下， 每次调用 flushAppendOnlyFile 函数， WRITE 都会被执行， 但 SAVE 会被略过。
在这种模式下， SAVE 只会在以下任意一种情况中被执行：
Redis 被关闭、AOF 功能被关闭、系统的写缓存被刷新（可能是缓存已经被写满，或者定期保存操作被执行）
写入和保存都由主进程执行，两个操作都会阻塞主进程。

每一秒钟保存一次
在这种模式中， SAVE 原则上每隔一秒钟就会执行一次。
写入操作由主进程执行，阻塞主进程。保存操作由子线程执行，不直接阻塞主进程，但保存操作完成的快慢会影响写入操作的阻塞时长。

每执行一个命令保存一次
在这种模式下，每次执行完一个命令之后， WRITE 和 SAVE 都会被执行。
写入和保存都由主进程执行，两个操作都会阻塞主进程。
```

### AOF 文件的载入和数据还原步骤：

```
1、创建一个不带网络连接的伪客户端（fake client）：因为Redis的命令只能在客户端上下文中执行，而载入AOF文件时所使用的命令直接来源于AOF文件而不是网络连接，所以服务器使用了一个没有网络连接的伪客户端来执行AOF文件保存的写命令，伪客户端执行命令的效果和带网络连接的客户端执行命令的效果完全一样。
2、从AOF文件中分析并读取出一条写命令。
3、使用伪客户端执行被读出的写命令。
4、一直执行步骤2和步骤3，直到AOF文件中的所有写命令都被处理完毕为止。
```

### AOF 重写：

```
1、简介：为了解决AOF文件体积膨胀的问题，Redis提供了AOF文件重写（rewrite）功能。通过该功能，Redis服务器可以创建一个新的AOF文件来替代现有的AOF文件，新旧两个AOF文件所保存的数据库状态相同，但新AOF文件不会包含任何浪费空间的冗余命令，所以新AOF文件的体积通常会比旧AOF文件的体积要小得多。

2、原理：首先从数据库中读取键现在的值，然后用一条命令去记录键值对，代替之前记录这个键值对的多条命令，这就是AOF重写功能的实现原理（AOF文件重写并不需要对现有的AOF文件进行任何读取、分析或者写入操作，这个功能是通过读取服务器当前的数据库状态来实现的。）。

2、AOF 后台重写（BGREWRITEAOF）实现：

Redis 不希望 AOF 重写造成服务器无法处理请求， 所以 Redis 决定将 AOF 重写程序放到（后台）子进程里执行， 这样处理的最大好处是：
1）子进程进行 AOF 重写期间，主进程可以继续处理命令请求。
2）子进程带有主进程的数据副本，使用子进程而不是线程，可以在避免锁的情况下，保证数据的安全性。

使用子进程也有一个问题需要解决： 因为子进程在进行 AOF 重写期间， 主进程还需要继续处理命令， 而新的命令可能对现有的数据进行修改， 这会让当前数据库的数据和重写后的 AOF 文件中的数据不一致。
为了解决这个问题， Redis 增加了一个 AOF 重写缓冲区， 这个缓冲区在 fork 出子进程之后开始启用， Redis 主进程在接到新的写命令之后， 除了会将这个写命令的协议内容追加到现有的 AOF 文件之外， 还会追加到这个缓冲区中。
换言之， 当子进程在执行 AOF 重写时， 主进程需要执行以下三个工作：
1）处理命令请求。
2）将写命令追加到现有的 AOF 文件中。
3）将写命令追加到 AOF 重写缓冲区中。

当子进程完成 AOF 重写之后， 它会向父进程发送一个完成信号， 父进程在接到完成信号之后， 会调用一个信号处理函数， 并完成以下工作：
1）将 AOF 重写缓存中的内容全部写入到新 AOF 文件中。
2）对新的 AOF 文件进行改名，覆盖原有的 AOF 文件。
当步骤 1 执行完毕之后， 现有 AOF 文件、新 AOF 文件和数据库三者的状态就完全一致了。
当步骤 2 执行完毕之后， 程序就完成了新旧两个 AOF 文件的交替。
这个信号处理函数执行完毕之后， 主进程就可以继续像往常一样接受命令请求了。 在整个 AOF 后台重写过程中， 只有最后的写入缓存和改名操作会造成主进程阻塞， 在其他时候， AOF 后台重写都不会对主进程造成阻塞， 这将 AOF 重写对性能造成的影响降到了最低。
```