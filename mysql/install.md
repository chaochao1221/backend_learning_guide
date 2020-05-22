# 安装（Linux 环境）

### setp 1、安装 MYSQL

1. sudo apt-get update
2. sudo apt-get install mysql-server（安装过程中可能会提示输入 root 密码）
3. sudo apt-get install mysql-client
4. sudo apt-get install libmysqlclient-dev

### setp 2、设置 root 密码（若第一步没有提示输入密码，需要自己设置）

1. cat /etc/mysql/debian.cnf（按照配置文件的帐号密码进行登录）
2. use mysql;（进入 mysql 库）
3. update user set authentication_string=PASSWORD("这里输入你要改的密码") where User='root';（设置 root 密码）
4. update user set plugin="mysql_native_password";（更新缓存密码）
5. flush privileges;（刷新操作权限）
6. exit;（退出重新登录）

### setp 3、配置 MYSQL（可以跳过）

1. sudo mysql_secure_installation，根据自己的需要按 Y/N 来配置相应选项

### set 4、修改远程访问

1. sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf，注释掉 bind-address = 127.0.0.1
2. mysql -uroot -p（登录 mysql）
3. grant all privileges on `*.*` to 'root'@'%' identified by 'Your password' with grant option;（设置远程连接权限）
4. flush privileges; 然后执行 quit 命令退出 mysql
5. service mysql restart;（重启服务）
6. 设置远程连接权限时如果提示 Your password does not satisfy the current policy requirements，则表示输入的密码与默认的安全级别不符，可以输入 show variables like 'validate_passwird%' 查看密码安全级别，也可以输入 set global validate_password_policy=0 修改

### step 5、查找远程连接不上原因

1. 在服务器上输入 netstat -tnl，发现 3306 端口已经打开
2. 本地 telnet 47.91.29.253 3306，报连接被拒
3. 在服务器上输入 sudo ufw status，发现防火墙已经关闭（若没关闭，输入 sudo ufw disable 关闭）
4. 进入阿里云服务器 ecs 选中该示例，点击本实例安全组规则，发现 3306 端口没被授权，添加 3306 授权，重启服务器即可远程访问
