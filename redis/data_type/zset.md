# 有序集合

### 定义：

- Redis 的有序集合（sorted set）同时具有“有序”和“集合”两种性质，这种数据结构中的每个元素都由一个成员和一个与成员相关联的分值组成，其中成员以字符串方式存储，而分值则以 64 位双精度浮点数格式存储。

### 编码及选择

1. 有序集合的编码可以是 ziplist/REDIS_ENCODING_ZIPLIST 或者 skiplist/REDIS_ENCODING_SKIPLIST。
2. 当有序集合对象可以同时满足以下两个条件时，对象使用 ziplist 编码（否则使用 skiplist 编码）：
   - 有序集合保存的元素数量小于 server.zset_max_ziplist_entries （默认值为 128 个）。
   - 有序集合保存的所有元素成员的长度都小于 server.zset_max_ziplist_value（默认为 64 字节）。

### 底层实现

1. ziplist 编码的压缩列表对象使用压缩列表作为底层实现。
   - 每个集合元素使用两个紧挨在一起的压缩列表节点来保存，第一个节点保存元素的成员（member），而第二个元素则保存元素的分值（score）。
   - 多个元素之间按 score 值从小到大排序， 如果两个元素的 score 相同，那么按字典序对 member 进行对比，决定那个元素排在前面，那个元素排在后面。
   - 虽然元素是按 score 域有序排序的，但对 ziplist 的节点指针只能线性地移动，所以在 ziplist 编码的有序集中，查找某个给定元素的复杂度为 O(N) 。
   - 每次执行添加/删除/更新操作都需要执行一次查找元素的操作，因此这些函数的复杂度都不低于 O(N) ，至于这些操作的实际复杂度，取决于它们底层所执行的 ziplist 操作。
2. skiplist 编码的有序集合对象使用 zset 结构作为底层实现。zset 同时使用字典和跳跃表两个数据结构来保存有序集元素。
   - zset 结构中的 zsl 跳跃表按分值从小到大保存了所有集合元素，每个跳跃表节点都保存了一个集合元素：跳跃表节点的 object 属性保存了元素的成员，而跳跃表节点的 score 属性则保存了元素的分值。通过这个跳跃表，程序可以对有序集合进行范围型操作，比如 ZRANK、ZRANGE 等命令就是基于跳跃表 API 来实现的。
   - 除此之外，zset 结构中的 dict 字典为有序集合创建了一个从成员到分值的映射，字典中的每个键值对都保存了一个集合元素：字典的键保存了元素的成员，而字典的值则保存了元素的分值。通过这个字典，程序可以用 O（1）复杂度查找给定成员的分值，ZSCORE 命令就是根据这一特性实现的，而很多其他有序集合命令都在实现的内部用到了这一特性。
   - 有序集合每个元素的成员都是一个字符串对象，而每个元素的分值都是一个 double 类型的浮点数。值得一提的是，虽然 zset 结构同时使用跳跃表和字典来保存有序集合元素，但这两种数据结构都会通过指针来共享相同元素的成员和分值，所以同时使用跳跃表和字典来保存集合元素不会产生任何重复成员或者分值，也不会因此而浪费额外的内存。
3. 为什么 skiplist 编码的有序集合需要同时使用跳跃表和字典来实现：
   - 通过使用字典结构， 并将 member 作为键， score 作为值， 有序集可以在 O(1) 复杂度内：
       - 检查给定 member 是否存在于有序集（被很多底层函数使用）。
       - 取出 member 对应的 score 值（实现 ZSCORE 命令）。
   - 另一方面， 通过使用跳跃表， 可以让有序集支持以下两种操作：
       - 在 O(logN) 期望时间、 O(N) 最坏时间内根据 score 对 member 进行定位（被很多底层函数使用）。
       - 范围性查找和处理操作，这是（高效地）实现 ZRANGE 、 ZRANK 和 ZINTERSTORE 等命令的关键。
   - 通过同时使用字典和跳跃表，有序集可以高效地实现按成员查找和按顺序查找两种操作。

### 基本命令

1. ZADD：添加或更新成员(ZADD sorted_set [XX|NX][ch] [INCR] score member [score member])。复杂度：O(M\*log(N))，其中 M 为给定成员的数量，而 N 则为有序集合包含的成员数量。

```
带有 XX 选项的 ZADD 命令只会对有序集合已有的成员进行更新，而不会向有序集合添加任何新成员。
带有 NX 选项的 ZADD 命令只会向有序集合添加新成员，而不会对已有的成员进行任何更新。
带有 CH 选项的 ZADD 命令会返回被修改（changed）成员的数量作为返回值。
120.27.213.243:6379> ZADD salary 3500 "peter" 4000 "jack" 2000 "tom" 5500 "mary"
(integer) 4
```

2. ZREM：移除指定的成员(ZREM sorted_set member [member ...])。复杂度：O(M\*log(N))，其中 M 为给定成员的数量，N 为有序集合包含的成员数量。

```
120.27.213.243:6379> ZREM salary "peter"
(integer) 1
```

3. ZSCORE：获取成员的分值(ZSCORE sorted_set member)。复杂度：O(1)。

```
120.27.213.243:6379> ZSCORE salary "jack"
"4000"
```

4. ZINCRBY：对成员的分值执行自增或自减操作(ZINCRBY sorted_set increment member)。复杂度：O(log (N))，其中 N 为有序集合包含的成员数量。

```
120.27.213.243:6379> ZINCRBY salary 1000 "jack"     # 自增
"5000"
120.27.213.243:6379> ZINCRBY salary -2000 "jack"    # 自减
"3000"
120.27.213.243:6379> ZINCRBY salary -2000 "lily"    # ZINCRBY命令将把"lily"成员添加到salary有序集合中，并把给定的增量-2000设置为"lily"成员的分值，效果相当于执行ZADD salary -2000 "lily"
"-2000"
```

5. ZCARD：获取有序集合的大小(ZCARD sorted_set)。复杂度：O(1)。

```
120.27.213.243:6379> ZCARD salary
(integer) 4
```

6. ZRANK、ZREVRANK：获取成员在有序集合中的排名(ZRANK sorted_set member; ZREVRANK sorted_set member)。复杂度：O(log (N))，其中 N 为有序集合包含的成员数量。

```
排序的第一名值为0。
其中ZRANK命令返回的是成员的升序排列排名，即成员在按照分值从小到大进行排列时的排名。
而ZREVRANK命令返回的则是成员的降序排列排名，即成员在按照分值从大到小进行排列时的排名。
120.27.213.243:6379> ZRANK salary "jack"
(integer) 2
120.27.213.243:6379> ZREVRANK salary "jack"
(integer) 1
```

7. ZRANGE、ZREVRANGE：获取指定索引范围内的成员(ZRANGE sorted_set start end [WITHSCORES]; ZREVRANGE sorted_set start end [WITHSCORES])。复杂度：O(log (N) + M)，其中 N 为有序集合包含的成员数量，而 M 则为命令返回的成员数量。

```
其中ZRANGE命令用于获取按照分值大小实施升序排列的成员。
而ZREVRANGE命令则用于获取按照分值大小实施降序排列的成员。
如果用户想要在获取这些成员的同时也获取与之相关联的分值，那么给定可选的WITHSCORES选项。
120.27.213.243:6379> ZRANGE salary 0 -1
1) "lily"
2) "tom"
3) "jack"
4) "mary"
120.27.213.243:6379> ZREVRANGE salary 0 -1
1) "mary"
2) "jack"
3) "tom"
4) "lily"
120.27.213.243:6379> ZREVRANGE salary 0 -1 WITHSCORES
1) "mary"
2) "5500"
3) "jack"
4) "3000"
5) "tom"
6) "2000"
7) "lily"
8) "-2000"
```

8. ZRANGEBYSCORE、ZREVRANGEBYSCORE：获取指定分值范围内的成员(ZRANGEBYSCORE sorted_set min max [WITHSCORES][limit offset count]; ZREVRANGEBYSCORE sorted_set max min [WITHSCORES][limit offset count])。复杂度：O(log (N) +M)，其中 N 为有序集合包含的成员数量，而 M 则为命令返回的成员数量。

```
通过使用ZRANGEBYSCORE命令或者ZREVRANGEBYSCORE命令，用户可以以升序排列或者降序排列的方式获取有序集合中分值介于指定范围内的成员。
如果用户想要在获取指定范围内的成员同时也获取与之相关联的分值，可以通过在执行时给定可选的 WITHSCORES 选项来同时获取成员及其分值。
若果只需要范围内的其中一部分成员，那么可以使用可选的 LIMIT 选项来限制命令返回的成员数量。
120.27.213.243:6379> ZRANGEBYSCORE salary 2000 4000 # 升序
1) "tom"
2) "jack"
120.27.213.243:6379> ZREVRANGEBYSCORE salary 6000 3000 WITHSCORES   # 降序并获取分值
1) "mary"
2) "5500"
3) "jack"
4) "3000"
120.27.213.243:6379> ZREVRANGEBYSCORE salary 6000 3000 LIMIT 0 1    # 降序并限制返回第一成员数据
1) "mary"
120.27.213.243:6379> ZRANGEBYSCORE salary (3000 (6000 WITHSCORES    # 开区间，不包含3000和6000
1) "mary"
2) "5500"
120.27.213.243:6379> ZRANGEBYSCORE salary -inf (3000 WITHSCORES     # 无穷小到3000
1) "lily"
2) "-2000"
3) "tom"
4) "2000"
120.27.213.243:6379> ZRANGEBYSCORE salary (3000 +inf WITHSCORES     # 3000到无穷大
1) "mary"
2) "5500"
```

9. ZCOUNT：统计指定分值范围内的成员数量(SCOUNT sorted_set min max)。复杂度：O(log (N))，其中 N 为有序集合包含的成员数量。

```
120.27.213.243:6379> ZCOUNT salary 3000 5000
(integer) 1
120.27.213.243:6379> ZCOUNT salary (2000 +inf
(integer) 2
```

10. ZREMRANGEBYRANK：移除指定排名范围内的成员(ZREMRANGEBYRANK sorted_set start end)。复杂度：O(log (N) + M)，其中 N 为有序集合包含的成员数量，M 为被移除的成员数量。

```
ZREMRANGEBYRANK命令可以从升序排列的有序集合中移除位于指定排名范围内的成员，然后返回被移除成员的数量。
120.27.213.243:6379> ZREMRANGEBYRANK salary 0 2
(integer) 3
```

11. ZREMRANGEBYSCORE：移除指定分值范围内的成员(ZREMRANGEBYSCORE sorted_set min max)。复杂度：O(log (N) + M)，其中 N 为有序集合包含的成员数量，M 为被移除成员的数量。

```
120.27.213.243:6379> ZREMRANGEBYSCORE salary 5000 6000
(integer) 1
```

12. ZUNIONSTORE、ZINTERSTORE：有序集合的并集运算和交集运算(ZUNIONSTORE destination numbers sorted_set [sorted_set ...]; ZINTERSTORE destination numbers sorted_set [sorted_set ...])

