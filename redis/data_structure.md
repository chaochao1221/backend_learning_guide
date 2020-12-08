# 《Redis 数据结构》

## 一、字符串

- **定义：字符串（string）键是 Redis 最基本的键值对类型，这种类型的键值对会在数据库中把单独的一个键和单独的一个值关联起来，被关联的键和值既可以是普通的文字数据，也可以是图片、视频、音频、压缩文件等更为复杂的二进制数据。**

**1. SET: 为字符串键设置值(SET key value)，复杂度：O(1)。**

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

**2. GET: 获取指定字符串键的值(GET key)，复杂度：O(1)。**

```
127.0.0.1:6379[1]> keys *   # 查询当前db中的所有键
1) "number"
2) "password"
127.0.0.1:6379[1]> GET number   # 获取 number 键对应的值
"10086"
127.0.0.1:6379[1]> GET username # 获取不存在的 username 键对应的值时，返回一个空值
(nil)
```

**3. GETSET：获取旧值并设置新值(GETSET key new_value)，复杂度：O(1)。**

```
127.0.0.1:6379[1]> GETSET number 6379   # 获取 number 键的旧值 10086 并为它设置新值 6379
"10086"
127.0.0.1:6379[1]> GETSET username chaochao # 因为 username 键不存在，因此获取的旧值为空，并为它设置新值 chaochao
(nil)
```

**4. MSET：一次为多个字符串键设置值(MSET key value [key value ...])**

```
127.0.0.1:6379[1]> MSET a 1 b 2 c 3 # 为 a,b,c 三个键分别设置值 1,2,3
OK
```

**5. MGET：一次获取多个字符串键的值(MGET key [key ...])**

```
127.0.0.1:6379[1]> MGET a b c   # 获取 a,b,c 键的值为 1,2,3
1) "1"
2) "2"
3) "3"
```

**6. MSETNX：只在键不存在的情况下，一次为多个字符串键设置值(MSETNX key value [key value ...])**

```
127.0.0.1:6379[1]> MSETNX b 2 c 3 d 4   # 因为键 b,c 已经存在，所以设置失败
(integer) 0
127.0.0.1:6379[1]> MSETNX d 4 e 5   # 键 d,e 都不存在，设置成功
(integer) 1
```

**7. STRLEN：获取字符串值的字节长度(STRLEN key)**

```
127.0.0.1:6379[1]> STRLEN username
(integer) 8
```
