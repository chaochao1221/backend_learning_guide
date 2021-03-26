# 发布与订阅

### 定义：Redis 的发布与订阅功能由 PUBLISH、SUBSCRIBE、PSUBSCRIBE 等命令组成。通过执行 SUBSCRIBE 命令，客户端可以订阅一个或多个频道，从而成为这些频道的订阅者（subscriber）：每当有其他客户端向被订阅的频道发送消息（message）时，频道的所有订阅者都会收到这条消息。除了订阅频道之外，客户端还可以通过执行 PSUBSCRIBE 命令订阅一个或多个模式，从而成为这些模式的订阅者：每当有其他客户端向某个频道发送消息时，消息不仅会被发送给这个频道的所有订阅者，它还会被发送给所有与这个频道相匹配的模式的订阅者。

### 频道的订阅（SUBSCRIBE）

```
1、当一个客户端执行SUBSCRIBE命令订阅某个或某些频道的时候，这个客户端与被订阅频道之间就建立起了一种订阅关系。
2、Redis将所有频道的订阅关系都保存在服务器状态的pubsub_channels字典里面，这个字典的键是某个被订阅的频道，而键的值则是一个链表，链表里面记录了所有订阅这个频道的客户端：
    1）如果频道已经有其他订阅者，那么它在pubsub_channels字典中必然有相应的订阅者链表，程序唯一要做的就是将客户端添加到订阅者链表的末尾。
    2）如果频道还未有任何订阅者，那么它必然不存在于pubsub_channels字典，程序首先要在pubsub_channels字典中为频道创建一个键，并将这个键的值设置为空链表，然后再将客户端添加到链表，成为链表的第一个元素。
```

### 频道的退订（UNSUBSCRIBE）

```
1、UNSUBSCRIBE命令的行为和SUBSCRIBE命令的行为正好相反，当一个客户端退订某个或某些频道的时候，服务器将从pubsub_channels中解除客户端与被退订频道之间的关联：
    1）程序会根据被退订频道的名字，在pubsub_channels字典中找到频道对应的订阅者链表，然后从订阅者链表中删除退订客户端的信息。
    2）如果删除退订客户端之后，频道的订阅者链表变成了空链表，那么说明这个频道已经没有任何订阅者了，程序将从pubsub_channels字典中删除频道对应的键。
```

### 模式的订阅（PSUBSCRIBE）

```
1、服务器将所有模式的订阅关系都保存在服务器状态的pubsub_patterns属性里面。pubsub_patterns属性是一个链表，链表中的每个节点都包含着一个pubsubPattern结构，这个结构的pattern属性记录了被订阅的模式，而client属性则记录了订阅模式的客户端。
2、每当客户端执行PSUBSCRIBE命令订阅某个或某些模式的时候，服务器会对每个被订阅的模式执行以下两个操作：
    1）新建一个pubsubPattern结构，将结构的pattern属性设置为被订阅的模式，client属性设置为订阅模式的客户端。
    2）将pubsubPattern结构添加到pubsub_patterns链表的表尾。
```

### 模式的退订（PUNSUBSCRIBE）

```
1、模式的退订命令PUNSUBSCRIBE是PSUBSCRIBE命令的反操作：当一个客户端退订某个或某些模式的时候，服务器将在pubsub_patterns链表中查找并删除那些pattern属性为被退订模式，并且client属性为执行退订命令的客户端的pubsubPattern结构。
```

### 发送消息（PUBLISH）

```
1、当一个Redis客户端执行PUBLISH＜channel＞＜message＞命令将消息message发送给频道channel的时候，服务器需要执行以下两个动作：
    1）将消息message发送给channel频道的所有订阅者。
    2）如果有一个或多个模式pattern与频道channel相匹配，那么将消息message发送给pattern模式的订阅者。
```

### 查看订阅信息（PUBSUB）

```
1、PUBSUB命令是Redis 2.8新增加的命令之一，客户端可以通过这个命令来查看频道或者模式的相关信息，比如某个频道目前有多少订阅者，又或者某个模式目前有多少订阅者，诸如此类。
2、PUBSUB CHANNELS[pattern]：该命令用于返回服务器当前被订阅的频道，其中pattern参数是可选的：
    1）如果不给定pattern参数，那么命令返回服务器当前被订阅的所有频道。
    2）如果给定pattern参数，那么命令返回服务器当前被订阅的频道中那些与pattern模式相匹配的频道。
    这个子命令是通过遍历服务器pubsub_channels字典的所有键（每个键都是一个被订阅的频道），然后记录并返回所有符合条件的频道来实现的。
3、PUBSUB NUMSUB[channel-1 channel-2...channel-n]：该命令接受任意多个频道作为输入参数，并返回这些频道的订阅者数量。
    1）这个子命令是通过在pubsub_channels字典中找到频道对应的订阅者链表，然后返回订阅者链表的长度来实现的（订阅者链表的长度就是频道订阅者的数量）。
4、PUBSUB NUMPAT：这个子命令是通过返回pubsub_patterns链表的长度来实现的，因为这个链表的长度就是服务器被订阅模式的数量。
```
