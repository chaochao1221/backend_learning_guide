### setp 1、安装 Redis

1. sudo apt-get update
2. sudo apt-get install redis-server

### setp 2、设置密码

1. whereis redis（查看配置文件所在位置，eg. /etc/redis）
2. vim /etc/redis/redis.conf
3. requirepass '写入你的密码'

### setp 3、允许远程访问

1. whereis redis（查看配置文件所在位置，eg. /etc/redis）
2. vim /etc/redis/redis.conf
3. 注释掉 bind 127.0.0.1

### setp 4、启动/重启/停止命令

1. cd /etc/init.d/
2. ./redis-server start/stop/restart
