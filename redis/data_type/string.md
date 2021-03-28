# 字符串

### 定义：
- REDIS_STRING （字符串）是 Redis 使用得最为广泛的数据类型， 它除了是 SET、GET 等命令的操作对象之外，数据库中的所有键，以及执行命令时提供给 Redis 的参数，都是用这种类型保存的。

### 基本命令

1. SET: 为字符串键设置值(SET key value)，复杂度：O(1)。

```
127.0.0.1:6379[1]> SET number 10086	# 设置键 number 的值为 10086
OK
127.0.0.1:6379[1]> SET password 123456 NX	# 对尚未有值的 password 键进行设置，成功
OK
127.0.0.1:6379[1]> SET password 99999 NX	# 对已经有值的 password 键进行设置，失败
(nil)
127.0.0.1:6379[1]> SET username chaochao XX	# 对尚未有值的 username 键进行设置，失败
(nil)
127.0.0.1:6379[1]> SET password 88888 XX	# 对已经有值的  password 键进行设置，成功
OK
```

2. GET: 获取指定字符串键的值(GET key)，复杂度：O(1)。

```
127.0.0.1:6379[1]> keys *   # 查询当前db中的所有键
1) "number"
2) "password"
127.0.0.1:6379[1]> GET number   # 获取 number 键对应的值
"10086"
127.0.0.1:6379[1]> GET username # 获取不存在的 username 键对应的值时，返回一个空值
(nil)
```

3. GETSET：获取旧值并设置新值(GETSET key new_value)，复杂度：O(1)。

```
127.0.0.1:6379[1]> GETSET number 6379   # 获取 number 键的旧值 10086 并为它设置新值 6379
"10086"
127.0.0.1:6379[1]> GETSET username chaochao # 因为 username 键不存在，因此获取的旧值为空，并为它设置新值 chaochao
(nil)
```

4. MSET：一次为多个字符串键设置值(MSET key value [key value ...])

```
127.0.0.1:6379[1]> MSET a 1 b 2 c 3 # 为 a,b,c 三个键分别设置值 1,2,3
OK
```

5. MGET：一次获取多个字符串键的值(MGET key [key ...])

```
127.0.0.1:6379[1]> MGET a b c   # 获取 a,b,c 键的值为 1,2,3
1) "1"
2) "2"
3) "3"
```

6. MSETNX：只在键不存在的情况下，一次为多个字符串键设置值(MSETNX key value [key value ...])

```
127.0.0.1:6379[1]> MSETNX b 2 c 3 d 4   # 因为键 b,c 已经存在，所以设置失败
(integer) 0
127.0.0.1:6379[1]> MSETNX d 4 e 5   # 键 d,e 都不存在，设置成功
(integer) 1
```

7. STRLEN：获取字符串值的字节长度(STRLEN key)

```
127.0.0.1:6379[1]> STRLEN username
(integer) 8
```

8. GETRANGE：获取字符串值指定索引范围上的内容(GETRANGE key start end)，字符串的正数索引从第一个 0 开始，负数索引从最后一个-1 开始

```
127.0.0.1:6379[1]> GETRANGE username 0 4
"chaoc"
127.0.0.1:6379[1]> GETRANGE username -4 -1
"chao"
```

9. SETRANGE：对字符串值的指定索引范围进行设置(SETRANGE key index substitute)

```
127.0.0.1:6379[1]> SETRANGE username 4 dc   # 将 username 键的值从索引4开始的内容替换为 dc，当用户给定的新内容比被替换的内容更长时，SETRANGE命令就会自动扩展被修改的字符串值，从而确保新内容可以顺利写入
(integer) 8
```

10. APPEND：追加新内容到值的末尾(APPEND key suffix)

```
127.0.0.1:6379> APPEND username " good"
(integer) 13    # 追加操作执行完毕后，值的长度
127.0.0.1:6379> APPEND k1 a # 当键 k1 不存在时，相当于执行了 SET 操作
(integer) 1
```

11. INCRBY、DECRBY：对整数值执行加法操作和减法操作(INCRBY key increment)

```
127.0.0.1:6379> INCRBY number 10
(integer) 11
127.0.0.1:6379> DECRBY number 2
(integer) 9
127.0.0.1:6379> SET pi 3.14
OK
127.0.0.1:6379> INCRBY pi 100
(error) ERR value is not an integer or out of range # 当字符串键的值不是整数时，对键执行INCRBY命令或是DECRBY命令将返回一个错误
127.0.0.1:6379> INCRBY k2 2 # 当键 k2 不存在时，先将键 k2 的值初始化为0，然后再执行加上2的操作
(integer) 2
```

12. INCR、DECR：对整数值执行加 1 操作和减 1 操作(INCR key)

```
127.0.0.1:6379> INCR number # 对整数 number 执行加1
(integer) 10
```

13. INCRBYFLOAT：对数字值执行浮点数加法操作(INCRBYFLOAT key increment)

```
127.0.0.1:6379> INCRBYFLOAT pi 2.55
"5.69"
127.0.0.1:6379> INCRBYFLOAT pi -1
"4.69"
```
