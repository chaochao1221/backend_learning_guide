# 《Redis 安装(基于 Linux)》

### step 1、安装 Redis

```
1. apt-get update	# 更新apt的资源列表
2. apt-get install redis-server	# 下载redis服务
```

### step 2、设置密码并允许远程访问

```
1. whereis redis	# 查看配置文件所在位置，eg: /etc/redis
2. vim /etc/redis/redis.conf	# 编辑配置文件
3. requirepass '写入你的密码'	# 设置密码
4. 注释(#)bind 127.0.0.1	# 允许远程访问
```

### step 3、启动服务端

```
1. redis-server --port 10086    # 启动服务(默认端口6379，可以指定任意端口)
```

### step 4、启动客户端

```
redis-cli -h 127.0.0.1	# 启动客户端
127.0.0.1:6379> auth 123456	# 输入密码
OK
127.0.0.1:6379> ping	# 测试客户端与服务端连接是否正常
PONG
127.0.0.1:6379> select 1	# 切换数据库(默认16个db)
OK
```
