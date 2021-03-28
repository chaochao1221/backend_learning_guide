# 列表

### 定义：

- Redis 的列表（list）是一种线性的有序结构，可以按照元素被推入列表中的顺序来存储元素，这些元素既可以是文字数据，又可以是二进制数据，并且列表中的元素可以重复出现。

### 编码及选择

1. 列表对象的编码可以是 ziplist/REDIS_ENCODING_ZIPLIST 或者 linkedlist/REDIS_ENCODING_LINKEDLIST。
2. 创建新列表时 Redis 默认使用 ziplist 编码， 当以下任意一个条件被满足时， 列表会被转换成 linkedlist 编码：
   - 试图往列表新添加一个字符串值，且这个字符串的长度超过 server.list_max_ziplist_value （默认值为 64 ）。
   - ziplist 包含的节点超过 server.list_max_ziplist_entries （默认值为 512 ）。

### 底层实现

1. ziplist 编码的列表对象使用压缩列表作为底层实现，每个压缩列表节点（entry）保存了一个列表元素。。
2. linkedlist 编码的列表对象使用双端链表作为底层实现，每个双端链表节点（node）都保存了一个字符串对象，而每个字符串对象都保存了一个列表元素。

### 基本命令

1. LPUSH：将元素推入列表左端(LPUSH list item [item item ...])

```
127.0.0.1:6379> LPUSH todo "buy some milk" "watch tv" "finish homework" # LPUSH命令将按照元素给定的顺序，从左到右依次将所有给定元素推入列表左端
(integer) 3
```

2. RPUSH：将元素推入列表右端(RPUSH list item [item item ...])

```
127.0.0.1:6379> RPUSH todo1 "buy some milk" "watch tv" "finish homework"    # RPUSH命令将按照元素给定的顺序，从左到右依次将所有给定元素推入列表右端
(integer) 3
```

3. LPUSHX、RPUSHX：只对已存在的列表执行推入操作(LPUSHX list item)

```
127.0.0.1:6379> LPUSHX todo3 "hello"
(integer) 0 # 列表 todo3 不存在，没有推入任何元素
```

4. LPOP：弹出列表最左端的元素(LPOP list)

```
127.0.0.1:6379> LPOP todo
"finish homework"
127.0.0.1:6379> LPOP todo4
(nil)   # 列表 todo4 不存在，没有弹出任何元素
```

5. RPOP：弹出列表最右端的元素(RPOP list)

```
127.0.0.1:6379> RPOP todo1
"finish homework"
```

6. RPOPLPUSH：将右端弹出的元素推入左端(RPOPLPUSH source traget)

```
127.0.0.1:6379> RPOPLPUSH todo todo1    # 将列表 todo 的右端元素弹出，然后将其推入 todo1 的左端
"buy some milk"
```

7. LLEN：获取列表的长度(LLEN list)

```
127.0.0.1:6379> LLEN todo
(integer) 1
```

8. LINDEX：获取指定索引上的元素(LINDEX list index)

```
127.0.0.1:6379> LINDEX todo1 3
"buy some milk"
127.0.0.1:6379> LINDEX todo1 -1
"watch tv"
```

9. LRANGE：获取指定索引范围上的元素(LRANGE list start end)

```
127.0.0.1:6379> LRANGE todo1 0 -1
1) "buy some milk"
2) "-1"
3) "0"
4) "buy some milk"
5) "watch tv"
```

10. LSET：为指定索引设置新元素(LSET list index new_element)

```
127.0.0.1:6379> LSET todo1 1 "hello"
OK
```

11. LINSERT：将元素插入列表(LINSERT list BEFORE|AFTER target_element new_element)

```
127.0.0.1:6379> LINSERT todo1 AFTER "0" "1"
(integer) 6
127.0.0.1:6379> LINSERT todo1 AFTER "buy some milk" "2" # 存在两个相同目标元素 "buy some milk" 时，新元素 "2" 只插在第一个 "buy some milk" 后面
(integer) 7
127.0.0.1:6379> LINSERT todo1 AFTER "not-exists-element" "3"    # 目标元素不存在时，插入失败
(integer) -1
```

12. LTRIM：修剪列表(LTRIM list start end)

```
127.0.0.1:6379> LTRIM todo1 1 6
OK
127.0.0.1:6379> LTRIM todo1 -5 -1
OK
```

13. LREM：从列表中移除指定元素(LREM list count element)

```
如果count参数的值【等于】0，那么LREM命令将移除列表中包含的【所有】指定元素。
如果count参数的值【大于】0那么LREM命令将从列表的【左端】开始向右进行检查，并移除最先发现的【count】个指定元素。
如果count参数的值【小于】0，那么LREM命令将从列表的【右端】开始向左进行检查，并移除最先发现的【abs(count)】个指定元素（abs(count)即count的绝对值）。
127.0.0.1:6379> LRANGE sample1 0 -1
1) "a"
2) "b"
3) "b"
4) "a"
5) "c"
6) "c"
7) "a"
127.0.0.1:6379> LREM sample1 0 "a"
(integer) 3
127.0.0.1:6379> LRANGE sample1 0 -1
1) "b"
2) "b"
3) "c"
4) "c"
```

14. BLPOP：阻塞式左端弹出操作(BLPOP list [list ...] timeout)

```
BLPOP命令会按照从左到右的顺序依次检查用户给定的列表，并对最先遇到的非空列表执行左端元素弹出操作。
如果BLPOP命令在检查了用户给定的所有列表之后都没有发现可以执行弹出操作的非空列表，那么它将阻塞执行该命令的客户端并开始等待，直到某个给定列表变为非空，又或者等待时间超出给定时限为止。
127.0.0.1:6379> BLPOP todo1 5   # 弹出返回一个数组，数组的一个元素记录了执行弹出操作的列表，第二个元素则是被弹出元素的本身
1) "todo1"
2) "hello"
127.0.0.1:6379> BLPOP todo5 5   # 弹出一个空列表是，等待5秒超时后返回空值
(nil)
(5.19s)
```

15. BRPOP：阻塞式左端弹出操作(BRPOP list [list ...] timeout)

```
BRPOP命令会按照从右到左的顺序依次检查用户给定的列表，并对最先遇到的非空列表执行右端元素弹出操作。
如果BRPOP命令在检查了用户给定的所有列表之后都没有发现可以执行弹出操作的非空列表，那么它将阻塞执行该命令的客户端并开始等待，直到某个给定列表变为非空，又或者等待时间超出给定时限为止。
127.0.0.1:6379> BRPOP todo1 5   # 弹出返回一个数组，数组的一个元素记录了执行弹出操作的列表，第二个元素则是被弹出元素的本身
1) "todo1"
2) "hello"
127.0.0.1:6379> BRPOP todo5 5   # 弹出一个空列表是，等待5秒超时后返回空值
(nil)
(5.19s)
```

16. BRPOPLPUSH：阻塞式弹出并推入操作(BRPOPLPUSH source traget timeout)

```
如果源列表非空，那么BRPOPLPUSH命令的行为就和RPOPLPUSH命令的行为一样，BRPOPLPUSH命令会弹出位于源列表最右端的元素，并将该元素推入目标列表的左端，最后向客户端返回被推入的元素。
如果源列表为空，那么BRPOPLPUSH命令将阻塞执行该命令的客户端，然后在给定的时限内等待可弹出的元素出现，或者等待时间超过给定时限为止。
127.0.0.1:6379> BRPOPLPUSH todo1 todo2 5
"watch tv"
127.0.0.1:6379> BRPOPLPUSH todo3 todo2 5    # 源列表 todo3 不存在，阻塞5秒直至超时
(nil)
(5.19s)
```
