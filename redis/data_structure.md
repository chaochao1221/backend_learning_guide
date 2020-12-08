# 《Redis 数据结构》

## 一、字符串

- 定义：字符串（string）键是 Redis 最基本的键值对类型，这种类型的键值对会在数据库中把单独的一个键和单独的一个值关联起来，被关联的键和值既可以是普通的文字数据，也可以是图片、视频、音频、压缩文件等更为复杂的二进制数据。

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

2.  GET: 获取指定字符串键的值(GET key)，复杂度：O(1)。

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
