# RabbitMQ安装

#### 1. docker run -d --hostname my-rabbit --name some-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3

#### 2. root@my-rabbit:~# rabbitmq-plugins enable rabbitmq_management

#### 3. client url: amqp://127.0.0.1:5672/

#### 4. web url: http://localhost:15672
 - 用户名/密码: guest/guest