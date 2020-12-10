# 《Set(集合)》

- ### **定义：Redis 的集合（set）键允许用户将任意多个各不相同的元素存储到集合中，这些元素既可以是文本数据，也可以是二进制数据。集合只会存储非重复元素，尝试将一个已存在的元素添加到集合将被忽略。并且集合是以无序方式存储元素。**

1. ** SADD：将元素添加到集合(SADD set element [element ...])。复杂度：O(N)，其中 N 为用户给定的元素数量。**

```
120.27.213.243:6379> SADD databases "Redis"
(integer) 1
120.27.213.243:6379> SADD databases "MongoDB" "CouchDB"
(integer) 2
120.27.213.243:6379> SADD databases "Redis"
(integer) 0 # 添加重复的元素 "Redis" 时将被忽略
```

2. ** SREM：从集合中移除元素(SREM set element [element ...])。复杂度：O(N)，其中 N 为用户给定的元素数量。**

```
120.27.213.243:6379> SREM databases "CouchDB"
(integer) 1
```

3. ** SMOVE：将元素从一个集合移动到另一个集合(SMOVE source traget element)。复杂度：O(1)**

```
120.27.213.243:6379> SMOVE databases nosql "Redis"  # 将集合 databases 的元素"Redis"移到集合 nosql 中
(integer) 1
```

4. ** SMEMBERS：获取集合包含的所有元素(SMEMBERS set)。复杂度：O(N)**

```
120.27.213.243:6379> SMEMBERS databases
1) "MongoDB"
```

5. ** SCARD：获取集合包含的元素数量(SCARD set)。复杂度：O(1)。**

```
120.27.213.243:6379> SCARD databases
(integer) 1
```

6. ** SISMEMBER：检查给定元素是否存在于集合(SISMEMBER set element)。复杂度：O(1)。**

```
120.27.213.243:6379> SISMEMBER databases "MongoDB"
(integer) 1 # 存在
120.27.213.243:6379> SISMEMBER databases "MySQL"
(integer) 0 # 不存在
```

7. ** SRANDMEMBER：随机获取集合中的元素(SRANDMEMBER set [count])。复杂度：O(N)，其中 N 为被返回的元素数量。**

```
120.27.213.243:6379> SRANDMEMBER databases 2
1) "Redis"
2) "MongoDB"
```

8. ** SPOP：随机地从集合中移除指定数量的元素(SPOP set [count])。复杂度：O(N)，其中 N 为被移除的元素数量。**

```
120.27.213.243:6379> SPOP databases 2
1) "MongoDB"
2) "CouchDB"
```

9. ** SINTER、SINTERSTORE：对集合执行交集计算(SINTER set [set ...]; SINTERSTORE destination_key set [set ...])。复杂度：O(N\*M)，其中 N 为给定集合的数量，而 M 则是所有给定集合当中，包含元素最少的那个集合的大小。**

```
120.27.213.243:6379> SADD sq a b c d
(integer) 4
120.27.213.243:6379> SADD sp c d e f
(integer) 4
120.27.213.243:6379> SINTER sq sp
1) "d"
2) "c"
120.27.213.243:6379> SINTERSTORE sq-inter-sp sq sp  # 把集合 sq 和 sp 的交集计算结果存储到集合 sq-inter-sp 中
(integer) 2
120.27.213.243:6379> SMEMBERS sq-inter-sp
1) "d"
2) "c"
```

10. ** SUNION、SUNIONSTORE：对集合执行并集计算(SUNION set [set ...]; SUNIONSTORE destination_key set [set ...])。复杂度：O(N)，其中 N 为所有给定集合包含的元素数量总和。**

```
120.27.213.243:6379> SUNION sq sp
1) "c"
2) "d"
3) "a"
4) "e"
5) "f"
6) "b"
120.27.213.243:6379> SUNIONSTORE sq-union-sp sq sp  # 把集合 sq 和 sp 的并集计算结果存储到集合 sq-union-sp 中
(integer) 6
120.27.213.243:6379> SMEMBERS sq-union-sp
1) "c"
2) "d"
3) "a"
4) "e"
5) "f"
6) "b"
```

11. ** SDIFF、SDIFFSTORE：对集合执行差集计算(SDIFF set [set ...]; SDIFFSTORE destination_key set [set ...])。复杂度：O(N)，其中 N 为所有给定集合包含的元素数量总和。**

```
120.27.213.243:6379> SDIFF sq sp    # SDIFF命令会按照用户给定集合的顺序，从左到右依次地对给定的集合执行差集计算。所以SDIFF后面集合的顺序不同，生成的结果也不一样
1) "b"
2) "a"
120.27.213.243:6379> SDIFF sp sq
1) "f"
2) "e"
120.27.213.243:6379> SDIFFSTORE sq-diff-sp sq sp
(integer) 2
120.27.213.243:6379> SMEMBERS sq-diff-sp
1) "b"
2) "a"
```
