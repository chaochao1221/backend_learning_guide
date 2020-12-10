# 《Zset(有序集合)》

- **定义：Redis 的有序集合（sorted set）同时具有“有序”和“集合”两种性质，这种数据结构中的每个元素都由一个成员和一个与成员相关联的分值组成，其中成员以字符串方式存储，而分值则以 64 位双精度浮点数格式存储。**

1. **ZADD：添加或更新成员(ZADD sorted_set [XX|NX][ch] [INCR] score member [score member])。复杂度：O(M\*log(N))，其中 M 为给定成员的数量，而 N 则为有序集合包含的成员数量。**

```
带有 XX 选项的 ZADD 命令只会对有序集合已有的成员进行更新，而不会向有序集合添加任何新成员。
带有 NX 选项的 ZADD 命令只会向有序集合添加新成员，而不会对已有的成员进行任何更新。
带有 CH 选项的 ZADD 命令会返回被修改（changed）成员的数量作为返回值。
120.27.213.243:6379> ZADD salary 3500 "peter" 4000 "jack" 2000 "tom" 5500 "mary"
(integer) 4
```

2. **ZREM：移除指定的成员(ZREM sorted_set member [member ...])。复杂度：O(M\*log(N))，其中 M 为给定成员的数量，N 为有序集合包含的成员数量。**

```
120.27.213.243:6379> ZREM salary "peter"
(integer) 1
```

3. **ZSCORE：获取成员的分值(ZSCORE sorted_set member)。复杂度：O(1)。**

```
120.27.213.243:6379> ZSCORE salary "jack"
"4000"
```

4. **ZINCRBY：对成员的分值执行自增或自减操作(ZINCRBY sorted_set increment member)。复杂度：O(log (N))，其中 N 为有序集合包含的成员数量。**

```
120.27.213.243:6379> ZINCRBY salary 1000 "jack"     # 自增
"5000"
120.27.213.243:6379> ZINCRBY salary -2000 "jack"    # 自减
"3000"
120.27.213.243:6379> ZINCRBY salary -2000 "lily"    # ZINCRBY命令将把"lily"成员添加到salary有序集合中，并把给定的增量-2000设置为"lily"成员的分值，效果相当于执行ZADD salary -2000 "lily"
"-2000"
```

5. **ZCARD：获取有序集合的大小(ZCARD sorted_set)。复杂度：O(1)。**

```
120.27.213.243:6379> ZCARD salary
(integer) 4
```

6. **ZRANK、ZREVRANK：获取成员在有序集合中的排名(ZRANK sorted_set member; ZREVRANK sorted_set member)。复杂度：O(log (N))，其中 N 为有序集合包含的成员数量。**

```
排序的第一名值为0。
其中ZRANK命令返回的是成员的升序排列排名，即成员在按照分值从小到大进行排列时的排名。
而ZREVRANK命令返回的则是成员的降序排列排名，即成员在按照分值从大到小进行排列时的排名。
120.27.213.243:6379> ZRANK salary "jack"
(integer) 2
120.27.213.243:6379> ZREVRANK salary "jack"
(integer) 1
```

7. **ZRANGE、ZREVRANGE：获取指定索引范围内的成员(ZRANGE sorted_set start end [WITHSCORES]; ZREVRANGE sorted_set start end [WITHSCORES])。复杂度：O(log (N) + M)，其中 N 为有序集合包含的成员数量，而 M 则为命令返回的成员数量。**

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

8. **ZRANGEBYSCORE、ZREVRANGEBYSCORE：获取指定分值范围内的成员(ZRANGEBYSCORE sorted_set min max [WITHSCORES][limit offset count]; ZREVRANGEBYSCORE sorted_set max min [WITHSCORES][limit offset count])。复杂度：O(log (N) +M)，其中 N 为有序集合包含的成员数量，而 M 则为命令返回的成员数量。**

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

9. **ZCOUNT：统计指定分值范围内的成员数量(SCOUNT sorted_set min max)。复杂度：O(log (N))，其中 N 为有序集合包含的成员数量。**

```
120.27.213.243:6379> ZCOUNT salary 3000 5000
(integer) 1
120.27.213.243:6379> ZCOUNT salary (2000 +inf
(integer) 2
```

10. **ZREMRANGEBYRANK：移除指定排名范围内的成员(ZREMRANGEBYRANK sorted_set start end)。复杂度：O(log (N) + M)，其中 N 为有序集合包含的成员数量，M 为被移除的成员数量**

```
ZREMRANGEBYRANK命令可以从升序排列的有序集合中移除位于指定排名范围内的成员，然后返回被移除成员的数量。
120.27.213.243:6379> ZREMRANGEBYRANK salary 0 2
(integer) 3
```

11. **ZREMRANGEBYSCORE：移除指定分值范围内的成员(ZREMRANGEBYSCORE sorted_set min max)。复杂度：O(log (N) + M)，其中 N 为有序集合包含的成员数量，M 为被移除成员的数量。**

```
120.27.213.243:6379> ZREMRANGEBYSCORE salary 5000 6000
(integer) 1
```

12. **ZUNIONSTORE、ZINTERSTORE：有序集合的并集运算和交集运算(ZUNIONSTORE destination numbers sorted_set [sorted_set ...]; ZINTERSTORE destination numbers sorted_set [sorted_set ...])**
