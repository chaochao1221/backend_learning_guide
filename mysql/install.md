# 安装（Linux 环境）

### setp 1、安装 MYSQL

- sudo apt-get update
- sudo apt-get install mysql-server，安装过程中会提示输入 root 密码
- sudo apt-get install mysql-client
- sudo apt-get install libmysqlclient-dev

### setp 2、配置 MYSQL

- sudo mysql_secure_installation，根据自己的需要按 Y/N 来配置相应选项

### set 3、修改远程访问

- sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf，注释掉 bind-address = 127.0.0.1
- mysql -uroot -p，登录 mysql
- grant all privileges on _._ to ‘root'@'%' identified by ‘Your password' with grant option;
- 如果提示 Your password does not satisfy the current policy requirements，则表示输入的密码与默认的安全级别不符，可以输入 show variables like ‘validate_passwird%'查看密码安全级别，也可以输入 set global validate_password_policy=0 修改
- flush privileges; 然后执行 quit 命令退出 mysql
- service mysql restart; 重启
- 远程连接 mysql，连接不上

### step 3、查找远程连接不上原因

- 在服务器上输入 netstat -tnl，发现 3306 端口已经打开
- 本地 telnet 47.91.29.253 3306，报连接被拒
- 在服务器上输入 sudo ufw status，发现防火墙已经关闭（若没关闭，输入 sudo ufw disable 关闭）
- 进入阿里云服务器 ecs 选中该示例，点击本实例安全组规则，发现 3306 端口没被授权，添加 3306 授权，重启服务器即可远程访问
