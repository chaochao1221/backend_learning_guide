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
        - 一个是命令连接，这个连接专门用于向主服务器发送命令，并接收命令回复。
        - 另一个是订阅连接，这个连接专门用于订阅主服务器的__sentinel__:hello频道（在Redis目前的发布与订阅功能中，被发送的信息都不会保存在Redis服务器里面，如果在信息发送时，想要接收信息的客户端不在线或者断线，那么这个客户端就会丢失这条信息。因此，为了不丢失__sentinel__:hello频道的任何信息，Sentinel必须专门用一个订阅连接来接收该频道的信息）。

### 获取主服务器信息

1. Sentinel默认会以每十秒一次的频率，通过命令连接向被监视的主服务器发送INFO命令，并通过分析INFO命令的回复来获取主服务器的当前信息。
2. 通过分析主服务器返回的INFO命令回复，Sentinel可以获取以下两方面的信息：
    - 一方面是关于主服务器本身的信息，包括run_id域记录的服务器运行ID，以及role域记录的服务器角色。
    - 另一方面是关于主服务器属下所有从服务器的信息。根据返回的从服务器IP地址和端口号，Sentinel无须用户提供从服务器的地址信息，就可以自动发现从服务器。
3. 主服务器实例结构和从服务器实例结构之间的区别：
    - 主服务器实例结构的flags属性的值为SRI_MASTER，而从服务器实例结构的flags属性的值为SRI_SLAVE。
    - 主服务器实例结构的name属性的值是用户使用Sentinel配置文件设置的，而从服务器实例结构的name属性的值则是Sentinel根据从服务器的IP地址和端口号自动设置的。

### 获取从服务器信息

1. 当Sentinel发现主服务器有新的从服务器出现时，Sentinel除了会为这个新的从服务器创建相应的实例结构之外，Sentinel还会创建连接到从服务器的命令连接和订阅连接。
2. 在创建命令连接之后，Sentinel在默认情况下，会以每十秒一次的频率通过命令连接向从服务器发送INFO命令，根据INFO命令的回复，Sentinel会提取出以下信息：
    - 从服务器的运行ID run_id。
    - 从服务器的角色role。
    - 主服务器的IP地址master_host，以及主服务器的端口号master_port。
    - 主从服务器的连接状态master_link_status。
    - 从服务器的优先级slave_priority。
    - 从服务器的复制偏移量slave_repl_offset。

### 向主服务器和从服务器发送信息

1. 在默认情况下，Sentinel会以每两秒一次的频率，通过命令连接向所有被监视的主服务器和从服务器发送以下格式的命令：PUBLISH __sentinel__:hello "<s_ip>,<s_port>,<s_runid>,<s_epoch>,<m_name>,<m_ip>,<m_port>,<m_epoch>"。这条命令向服务器的__sentinel__:hello频道发送了一条信息，信息的内容由多个参数组成：
    - 其中以s_开头的参数记录的是Sentinel本身的信息。
    - 而m_开头的参数记录的则是主服务器的信息。如果Sentinel正在监视的是主服务器，那么这些参数记录的就是主服务器的信息；如果Sentinel正在监视的是从服务器，那么这些参数记录的就是从服务器正在复制的主服务器的信息。

### 接收来自主服务器和从服务器的频道信息
1. 发送命令：当Sentinel与一个主服务器或者从服务器建立起订阅连接之后，Sentinel就会通过订阅连接，向服务器发送以下命令：SUBSCRIBE __sentinel__:hello。对于每个与Sentinel连接的服务器，Sentinel既通过命令连接向服务器的__sentinel__:hello频道发送信息，又通过订阅连接从服务器的__sentinel__:hello频道接收信息。
2. 更新sentinels字典：
    - Sentinel为主服务器创建的实例结构中的sentinels字典保存了除Sentinel本身之外，所有同样监视这个主服务器的其他Sentinel的资料。当一个Sentinel接收到其他Sentinel发来的信息时，目标Sentinel会从信息中分析并提取出以下两方面参数：
        - 与Sentinel有关的参数：源Sentinel的IP地址、端口号、运行ID和配置纪元。
        - 与主服务器有关的参数：源Sentinel正在监视的主服务器的名字、IP地址、端口号和配置纪元。
    - 根据信息中提取出的主服务器参数，目标Sentinel会在自己的Sentinel状态的masters字典中查找相应的主服务器实例结构，然后根据提取出的Sentinel参数，检查主服务器实例结构的sentinels字典中，源Sentinel的实例结构是否存在：
        - 如果源Sentinel的实例结构已经存在，那么对源Sentinel的实例结构进行更新。
        - 如果源Sentinel的实例结构不存在，那么说明源Sentinel是刚刚开始监视主服务器的新Sentinel，目标Sentinel会为源Sentinel创建一个新的实例结构，并将这个结构添加到sentinels字典里面。
3. 创建连向其他Sentinel的命令连接：
    - 当Sentinel通过频道信息发现一个新的Sentinel时，它不仅会为新Sentinel在sentinels字典中创建相应的实例结构，还会创建一个连向新Sentinel的命令连接，而新Sentinel也同样会创建连向这个Sentinel的命令连接，最终监视同一主服务器的多个Sentinel将形成相互连接的网络：Sentinel A有连向Sentinel B的命令连接，而Sentinel B也有连向Sentinel A的命令连接。
    - Sentinel在连接主服务器或者从服务器时，会同时创建命令连接和订阅连接，但是在连接其他Sentinel时，却只会创建命令连接。这是因为Sentinel需要通过接收主服务器或者从服务器发来的频道信息来发现未知的新Sentinel，所以才需要建立订阅连接，而相互已知的Sentinel只要使用命令连接来进行通信就足够了。

### 检测主观下线状态

1. Sentinel会以每秒一次的频率向所有与它创建了命令连接的实例（包括主服务器、从服务器、其他Sentinel在内）发送PING命令，并通过实例返回的PING命令回复来判断实例是否在线。
2. 实例对PING命令的回复可以分为以下两种情况：
    - 有效回复：实例返回+PONG、-LOADING、-MASTERDOWN三种回复的其中一种。
    - 无效回复：实例返回除+PONG、-LOADING、-MASTERDOWN三种回复之外的其他回复，或者在指定时限内没有返回任何回复。
3. Sentinel配置文件中的down-after-milliseconds选项指定了Sentinel判断实例进入主观下线所需的时间长度：如果一个实例在down-after-milliseconds毫秒内，连续向Sentinel返回无效回复，那么Sentinel会修改这个实例所对应的实例结构，在结构的flags属性中打开SRI_S_DOWN标识，以此来表示这个实例已经进入主观下线状态。对于监视同一个主服务器的多个Sentinel来说，这些Sentinel所设置的down-after-milliseconds选项的值也可能不同，因此，当一个Sentinel将主服务器判断为主观下线时，其他Sentinel可能仍然会认为主服务器处于在线状态。

### 检查客观下线状态

1. 当Sentinel将一个主服务器判断为主观下线之后，为了确认这个主服务器是否真的下线了，它会向同样监视这一主服务器的其他Sentinel进行询问，看它们是否也认为主服务器已经进入了下线状态（可以是主观下线或者客观下线）。当Sentinel从其他Sentinel那里接收到足够数量的已下线判断之后，Sentinel就会将从服务器判定为客观下线，并对主服务器执行故障转移操作。
2. 发送SENTINEL is-master-down-by-addr命令：SENTINEL is-master-down-by-addr <ip> <port> <current_epoch> <runid>，命令询问其他Sentinel是否同意主服务器已下线。
3. 接收SENTINEL is-master-down-by-addr命令：当一个Sentinel（目标Sentinel）接收到另一个Sentinel（源Sentinel）发来的SENTINEL is-master-down-by命令时，目标Sentinel会分析并取出命令请求中包含的各个参数，并根据其中的主服务器IP和端口号，检查主服务器是否已下线，然后向源Sentinel返回一条包含三个参数的Multi Bulk回复作为SENTINEL is-master-down-by命令的回复：
    - <down_state>：返回目标 sentinel 对主服务器的检查结果，1代表主服务器已下线，0代表主服务器未下线。
    - <leader_runid>：可以是*符号或者目标 sentinel 的局部领头的 sentinel 运行id，*号代表命令仅仅用于监测主服务器的下线状态，而局部领头 sentinel 的运行id则用于选举领头 sentinel。
    - <leader_epoch>：目标 sentinel 的局部领头 sentinel 的配置纪元，用于选举领头 sentinel，仅在 leader_runid 的值不为*时有用，如果 leader_runid 为*时，那么 leader_epoch 总为0。
4. 根据其他Sentinel发回的SENTINEL is-master-down-by-addr命令回复，Sentinel将统计其他Sentinel同意主服务器已下线的数量，当这一数量达到配置指定的判断客观下线所需的数量时，Sentinel会将主服务器实例结构flags属性的SRI_O_DOWN标识打开，表示主服务器已经进入客观下线状态。

### 选举领头 Sentinel

1. 当一个主服务器被判断为客观下线时，监视这个下线主服务器的各个Sentinel会进行协商，选举出一个领头Sentinel，并由领头Sentinel对下线主服务器执行故障转移操作。
2. 选举领头Sentinel的规则和方法比较复杂，所以举例说明：
    - 假设现在有三个Sentinel正在监视同一个主服务器，并且这三个Sentinel之前已经通过SENTINEL is-master-down-by-addr命令确认主服务器进入了客观下线状态。
    - 那么为了选出领头Sentinel，三个Sentinel将再次向其他Sentinel发送SENTINELis-master-down-by-addr命令，和检测客观下线状态时发送的SENTINEL is-master-down-by-addr命令不同，Sentinel这次发送的命令会带有Sentinel自己的运行ID，例如：SENTINEL is-master-down-by-addr 127.0.0.1 6379 0 e955b4c85598ef5b5f055bc7ebfd5e828dbed4fa。
    - 如果接收到这个命令的Sentinel还没有设置局部领头Sentinel的话，它就会将运行ID为e955b4c85598ef5b5f055bc7ebfd5e828dbed4fa的Sentinel设置为自己的局部领头Sentinel，并返回类似以下的命令回复：
        - 1）1
        - 2）e955b4c85598ef5b5f055bc7ebfd5e828dbed4fa
        - 3）0
    - 然后接收到命令回复的Sentinel就可以根据这一回复，统计出有多少个Sentinel将自己设置成了局部领头Sentinel。
    - 根据命令请求发送的先后顺序不同，可能会有某个Sentinel的SENTINEL is-master-down-by-addr命令比起其他Sentinel发送的相同命令都更快到达，并最终胜出领头Sentinel的选举，然后这个领头Sentinel就可以开始对主服务器执行故障转移操作了。

### 故障转移

1. 在选举产生出领头Sentinel之后，领头Sentinel将对已下线的主服务器执行故障转移操作，该操作包含以下三个步骤：
    - 在已下线主服务器属下的所有从服务器里面，挑选出一个从服务器，并将其转换为主服务器。
    - 让已下线主服务器属下的所有从服务器改为复制新的主服务器。
    - 将已下线主服务器设置为新的主服务器的从服务器，当这个旧的主服务器重新上线时，它就会成为新的主服务器的从服务器。
2. 选出新的主服务器：
    - 删除列表中所有处于下线或者断线状态的从服务器，这可以保证列表中剩余的从服务器都是正常在线的。
    - 删除列表中所有最近五秒内没有回复过领头Sentinel的INFO命令的从服务器，这可以保证列表中剩余的从服务器都是最近成功进行过通信的。
    - 删除所有与已下线主服务器连接断开超过down-after-milliseconds*10毫秒的从服务器：down-after-milliseconds选项指定了判断主服务器下线所需的时间，而删除断开时长超过down-after-milliseconds*10毫秒的从服务器，则可以保证列表中剩余的从服务器都没有过早地与主服务器断开连接，换句话说，列表中剩余的从服务器保存的数据都是比较新的。
    - 之后，领头Sentinel将根据从服务器的优先级，对列表中剩余的从服务器进行排序，并选出其中优先级最高的从服务器。
    - 如果有多个具有相同最高优先级的从服务器，那么领头Sentinel将按照从服务器的复制偏移量，对具有相同最高优先级的所有从服务器进行排序，并选出其中偏移量最大的从服务器（复制偏移量最大的从服务器就是保存着最新数据的从服务器）。
    - 最后，如果有多个优先级最高、复制偏移量最大的从服务器，那么领头Sentinel将按照运行ID对这些从服务器进行排序，并选出其中运行ID最小的从服务器。
    - 领头Sentinel向被选中的从服务器发送SLAVEOF no one命令。在发送SLAVEOF no one命令之后，领头Sentinel会以每秒一次的频率（平时是每十秒一次），向被升级的从服务器发送INFO命令，并观察命令回复中的角色（role）信息，当被升级服务器的role从原来的slave变为master时，领头Sentinel就知道被选中的从服务器已经顺利升级为主服务器了。
3. 修改从服务器的复制目标：
    - 领头Sentinel向已下线主服务器的从服务器发送SLAVEOF命令，让它们复制新的主服务器。
4. 将旧的主服务器变为从服务器：
    - 因为旧的主服务器已经下线，所以这种设置是保存在旧的主服务器对应的实例结构里面的，当旧的主服务器重新上线时，Sentinel就会向它发送SLAVEOF命令，让它成为新的主服务器的从服务器。

### 总结

1. Sentinel只是一个运行在特殊模式下的Redis服务器，它使用了和普通模式不同的命令表，所以Sentinel模式能够使用的命令和普通Redis服务器能够使用的命令不同。
2. Sentinel会读入用户指定的配置文件，为每个要被监视的主服务器创建相应的实例结构，并创建连向主服务器的命令连接和订阅连接，其中命令连接用于向主服务器发送命令请求，而订阅连接则用于接收指定频道的消息。
3. Sentinel通过向主服务器发送INFO命令来获得主服务器属下所有从服务器的地址信息，并为这些从服务器创建相应的实例结构，以及连向这些从服务器的命令连接和订阅连接。
4. 在一般情况下，Sentinel以每十秒一次的频率向被监视的主服务器和从服务器发送INFO命令，当主服务器处于下线状态，或者Sentinel正在对主服务器进行故障转移操作时，Sentinel向从服务器发送INFO命令的频率会改为每秒一次。
5. 对于监视同一个主服务器和从服务器的多个Sentinel来说，它们会以每两秒一次的频率，通过向被监视服务器的__sentinel__:hello频道发送消息来向其他Sentinel宣告自己的存在。
6. 每个Sentinel也会从__sentinel__:hello频道中接收其他Sentinel发来的信息，并根据这些信息为其他Sentinel创建相应的实例结构，以及命令连接。
7. Sentinel只会与主服务器和从服务器创建命令连接和订阅连接，Sentinel与Sentinel之间则只创建命令连接。
8. Sentinel以每秒一次的频率向实例（包括主服务器、从服务器、其他Sentinel）发送PING命令，并根据实例对PING命令的回复来判断实例是否在线，当一个实例在指定的时长中连续向Sentinel发送无效回复时，Sentinel会将这个实例判断为主观下线。
9. 当Sentinel将一个主服务器判断为主观下线时，它会向同样监视这个主服务器的其他Sentinel进行询问，看它们是否同意这个主服务器已经进入主观下线状态。
10. 当Sentinel收集到足够多的主观下线投票之后，它会将主服务器判断为客观下线，并发起一次针对主服务器的故障转移操作。

