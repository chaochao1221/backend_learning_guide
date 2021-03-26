# 哨兵模式

### 定义：
- Sentinel（哨兵）是 Redis 的高可用性（high availability）解决方案：由一个或多个 Sentinel 实例（instance）组成的 Sentinel 系统（system）可以监视任意多个主服务器，以及这些主服务器属下的所有从服务器，并在被监视的主服务器进入下线状态时，自动将下线主服务器属下的某个从服务器升级为新的主服务器，然后由新的主服务器代替已下线的主服务器继续处理命令请求。

### 启动并初始化 Sentinel

1. 启动 Sentinel 命令：redis-sentinel /path/to/your/sentinel.conf 或者 redis-server /path/to/your/sentinel.conf --sentinel
2. 启动 Sentinel 所执行的步骤：
    - **初始化服务器：**
       相当于初始化一个普通的Redis服务器。不过，因为Sentinel执行的工作和普通Redis服务器执行的工作不同，所以Sentinel的初始化过程和普通Redis服务器的初始化过程并不完全相同（例如，初始化Sentinel时就不会载入RDB文件或者AOF文件）。
    - **将普通Redis服务器使用的代码替换成Sentinel专用代码：**
        将一部分普通Redis服务器使用的代码替换成Sentinel专用代码。PING、SENTINEL、INFO、SUBSCRIBE、UNSUBSCRIBE、PSUBSCRIBE和PUNSUBSCRIBE这七个命令就是客户端可以对Sentinel执行的全部命令了。
    - **初始化Sentinel状态：**
        服务器会初始化一个sentinel.c/sentinelState结构，这个结构保存了服务器中所有和Sentinel功能有关的状态（服务器的一般状态仍然由redis.h/redisServer结构保存）。
    - **根据给定的配置文件，初始化Sentinel的监视主服务器列表：**
        Sentinel状态中的masters字典记录了所有被Sentinel监视的主服务器的相关信息，其中：
            - 字典的键是被监视主服务器的名字。
            - 而字典的值则是被监视主服务器对应的sentinel.c/sentinelRedisInstance结构。每个sentinelRedisInstance结构（实例结构）代表一个被Sentinel监视的Redis服务器实例（instance），这个实例可以是主服务器、从服务器，或者另外一个Sentinel。
    - **创建连向主服务器的网络连接：**
        创建连向被监视主服务器的网络连接，Sentinel将成为主服务器的客户端，它可以向主服务器发送命令，并从命令回复中获取相关的信息。
        对于每个被Sentinel监视的主服务器来说，Sentinel会创建两个连向主服务器的异步网络连接：
            1. 一个是命令连接，这个连接专门用于向主服务器发送命令，并接收命令回复。
            2. 另一个是订阅连接，这个连接专门用于订阅主服务器的__sentinel__:hello频道（在Redis目前的发布与订阅功能中，被发送的信息都
            不会保存在Redis服务器里面，如果在信息发送时，想要接收信息的客户端不在线或者断线，那么这个客户端就会丢失这条信息。因此，为了不
            丢失__sentinel__:hello频道的任何信息，Sentinel必须专门用一个订阅连接来接收该频道的信息）。
```

### 获取主服务器信息

```
1、Sentinel默认会以每十秒一次的频率，通过命令连接向被监视的主服务器发送INFO命令，并通过分析INFO命令的回复来获取主服务器的当前信息。
2、通过分析主服务器返回的INFO命令回复，Sentinel可以获取以下两方面的信息：
    1）一方面是关于主服务器本身的信息，包括run_id域记录的服务器运行ID，以及role域记录的服务器角色。
    2）另一方面是关于主服务器属下所有从服务器的信息。根据返回的从服务器IP地址和端口号，Sentinel无须用户提供从服务器的地址信息，就可以自动
    发现从服务器。
3、主服务器实例结构和从服务器实例结构之间的区别：
    1）主服务器实例结构的flags属性的值为SRI_MASTER，而从服务器实例结构的flags属性的值为SRI_SLAVE。
    2）主服务器实例结构的name属性的值是用户使用Sentinel配置文件设置的，而从服务器实例结构的name属性的值则是Sentinel根据从服务器的IP地址
    和端口号自动设置的。
```

### 获取从服务器信息

```
1、当Sentinel发现主服务器有新的从服务器出现时，Sentinel除了会为这个新的从服务器创建相应的实例结构之外，Sentinel还会创建连接到从服务器的
命令连接和订阅连接。
2、在创建命令连接之后，Sentinel在默认情况下，会以每十秒一次的频率通过命令连接向从服务器发送INFO命令，根据INFO命令的回复，Sentinel会提取
出以下信息：
    1）从服务器的运行ID run_id。
    2）从服务器的角色role。
    3）主服务器的IP地址master_host，以及主服务器的端口号master_port。
    4）主从服务器的连接状态master_link_status。
    5）从服务器的优先级slave_priority。
    6）从服务器的复制偏移量slave_repl_offset。
```

### 向主服务器和从服务器发送信息

```
1、在默认情况下，Sentinel会以每两秒一次的频率，通过命令连接向所有被监视的主服务器和从服务器发送以下格式的命令：
PUBLISH __sentinel__:hello "<s_ip>,<s_port>,<s_runid>,<s_epoch>,<m_name>,<m_ip>,<m_port>,<m_epoch>"。这条命令向服务器的
__sentinel__:hello频道发送了一条信息，信息的内容由多个参数组成：
    1）其中以s_开头的参数记录的是Sentinel本身的信息。
    2）而m_开头的参数记录的则是主服务器的信息。如果Sentinel正在监视的是主服务器，那么这些参数记录的就是主服务器的信息；如果Sentinel正在
    监视的是从服务器，那么这些参数记录的就是从服务器正在复制的主服务器的信息。
```

### 接收来自主服务器和从服务器的频道信息

```

```
