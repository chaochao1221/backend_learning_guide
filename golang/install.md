# 《Golang 安装(基于 Linux Ubuntu 18.04)》

### step1、 安装 Golang

```
1. apt-get update
2. apt install software-properties-common
3. add-apt-repository ppa:longsleep/golang-backports
4. apt-get install golang-1.13-go

5. mkdir -p go/src/demo
6. vim $HOME/.profile
   export GOROOT=/usr/lib/go-1.13
   export GOPATH=$HOME/go
   export PATH=$PATH:/usr/lib/go-1.13/bin
7. source $HOME/.profile
```

### step2、安装 Git

- **安装 Git 环境**

```
1. apt-get update
2. apt install git
3. cd go/src
4. git clone https://e.coding.net/demo.git
```

- **添加 coding 账户公钥(https://help.coding.net/docs/project-settings/features/ssh.html)**

```
1. ssh-keygen -m PEM -t rsa -b 4096 -C "your.email@example.com"，连续点击 Enter 键即可。
2. cat ~/.ssh/id_rsa.pub，复制全部内容。
3. 登录 CODING ，点击右上角【个人设置】，选择菜单【SSH 公钥】，点击【新增公钥】按钮。
4. 将第一步中复制的内容填写到【公钥内容】一栏，公钥名称按需填写即可。设定公钥有效期，可选择具体日期或设置永久有效。
5. 点击【添加】，然后输入账户密码即可成功添加公钥。
6. 完成后在命令行测试，首次建立链接会要求信任主机。命令 ssh -T git@e.coding.net。
```

### step3、安装 Supervisor

```
1. apt-get update
2. apt install supervisor
3. cd /etc/supervisor/conf.d
4. vim demo.conf
5. [program:demo]
    directory = /root/go/src/demo/
    command = /root/go/src/demo/demo
    autostart = true
    autorestart=true
    startsecs = 5
    redirect_stderr = true
    stdout_logfile=/root/go/log/http_demo.log
    stdout_logfile_maxbytes=1MB
    stdout_logfile_backups=10
    stdout_capture_maxbytes=1MB
    stderr_logfile=/root/go/log/error_demo.log
    stderr_logfile_maxbytes=1MB
    stderr_logfile_backups=10
    stderr_capture_maxbytes=1MB
6. supervisorctl update
```
