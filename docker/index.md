# Docker命令指南

## 镜像

##### 1. 概念：Docker运行容器前需要在本地存在对应的镜像，如果镜像不存在，Docker会尝试先从默认镜像库中下载（默认使用 Docker Hub公共注册服务器中的仓库），用户也可以通过配置，使用自定义的镜像仓库。

##### 2. docker [image] pull NAME[:TAG]
 - 使用该命令从Docker Hub镜像源来下载镜像。
 - NAME是镜像仓库名称（用来区分镜像），TAG是镜像的标签（往往用来表示版本信息。如果不显示指定该标签，则默认会选择latest标签，这会下载仓库中最新版本的镜像）。
 - eg. docker pull ubuntu: 18.04

##### 3. docker images 或 docker image ls
 - 使用改命令可以列出本地主机上已有镜像的基本信息。
 - 返回信息中的'TAG'用来区分发行版本。'SIZE'表示镜像大小信息，该信息只是表示了镜像逻辑体积的大小，实际上相同的镜像层本地只会存储一份，物理上占用的存储空间会小于各镜像逻辑体积之和。

##### . docker tag NAME[:TAG] NEWNAME[:TAG]
 - 使用该命令为本地镜像添加新的标签。
 - eg. docker tag ubuntu:latest myubuntu:latest
 - 该命令添加的标签实际上起到了类似链接的作用，ubuntu:latest镜像的ID和myubuntu:latest完全一致，他们实际上指向了同一个文件，只是别名不同而已。

##### 5. docker [image] inspect NAME[:TAG]
 - 使用该命令可以获取该镜像的详细信息，包括制造者、适应架构、各层的数字摘要等。
 - eg. docker inspect ubuntu:latest

##### 6. docker history NAME[:TAG]
 - 因为镜像由多个层组成，使用改命令可以列出各层的创建信息。
 - eg. docker history ubuntu:latest

##### 7. docker search [option] keyword
 - 使用该命令可以搜索 Docker Hub 官方仓库中的镜像。
 - 命令选项包括：-f,--filter filter：过滤输出内容。--format string：格式化输出内容。--limit int：限制输出结果个数，默认25个。--no-trunc：不截断输出结果。
 - eg. 搜索官方提供带nginx关键字的镜像：docker search -f=is-official=true nginx
 - eg. 搜索搜所有收藏数超过4的关键词包括tensorflow的镜像：docker search --filter=stars=4 tensorflow
 - 返回信息中包含镜像名称、描述、收藏数、是否官方创建、是否自动创建等信息。

##### 8. docker rmi IMAGE[IMAGE...] 或 docker image rm IMAGE[IMAGE...]
 - 使用该命令可以删除镜像，期中IMAGE可以为标签或ID
 - 命令选项包括：-f,-froce：强制删除镜像，即使有容器依赖它。-no-prune：不要清理未带标签的父镜像。
 - eg. docker rmi myubuntu:latest

##### 9. docker image prune
 - 使用改命令可以清理系统中遗留的一些临时的镜像文件，以及一些没有被使用的镜像文件。
 - 命令选项包括：-a,-all：删除所有无用镜像，不光是临时文件。-filter filter：只清理符合给定过滤器的镜像文件。-f,-force：强制删除镜像，而不进行提示确认。

##### 10. docker [container] commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
 - 基于已有容器创建镜像
 - 命令选项包括：-a,--author=""：作者信息。-c,--change=[]：提交的时候执行 Dockerfile 指令，包括CMD|ENTRYPOINT|ENV|EXPOSE|LABEL|ONBUILD|USER|VOLUME|WORKDIR等。-m,--message=""：提交消息。-p,--pause=true：提交时暂停容器运行。
 - eg. docker commit -m "add a new file" -a "dc" 8fa4587a0b07 mygotenberg:6

##### 11. docker [image] import [OPTIONS] file|URL|-[REPOSITORY[:TAG]]
 - 基于本地模板导入创建镜像
 - eg. cat ubuntu-18.04-x86_64-minimal.tar.gz | docker import -utuntu:18.04

##### 12. Dockerfile

 ```
 FROM debian:stretch-slim
 LABEL version="1.0" maintainer="docker user <docker_user@github>"
 RUN apt-get update && \
 		apt-get install -y python3 && \
 		apt-get clean && \
 		rm-rf /var/lib/apt/lists/*
 ```
 - 基于Dockerfile创建镜像是最常见方式。Dockerfile是一个文本文件，利用给定的指令描述基于某个父镜像创建新镜像的过程。
 - 创建镜像的过程可以使用docker [image] build指令，编译成功后本地将多出一个python:3镜像
 - eg. docker build -t python:3

##### 13. docker [image] save
 - 使用改命令可以导出镜像到本地
 - 命令选项：-o、-output string参数，导出镜像到指定的文件中。
 - eg. docker save -o ubuntu_18.04.tar ubuntu:18.04

##### 14. docker [image] load
 - 使用该命令可以将导出的tar文件导入到本地镜像库
 - 命令选项：-i、-input string，从指定文件中读入镜像内容。
 - eg. docker load -i ubuntu_18.04.tar。从文件ubuntu_18.04.tar导入镜像到本地镜像列表。
 - eg. docker load < ubuntu_18.04.tar。这将导入镜像及其相关的元数据信息（包括标签等）。导入成功后，可以使用docker images 查看，与原镜像一致。
 
##### 15. docker [image] push NAME[:TAG] | [REGISTRY_HOST[:REGISTRY_PORT]/]NAME[:TAG]
 - 使用该命令可以上传镜像到仓库，默认上传到Docker Hub官方仓库。
 - 使用该命令之前，需要保证本地已经登录Docker Hub帐号， 如果尚未登录，需先使用docker login进行登录。

## 容器

##### 1. 概念：容器是镜像的一个运行实例。与镜像不同的是，镜像是静态的只读文件，而容器带有运行时需要的可写文件层，同时容器中的应用进程处于运行状态。如果认为虚拟机是模拟运行的一整套操作系统和跑在上面的应用。那么 Docker 容器就是独立运行的一个（或一组）应用，以及它们必须的运行环境。

##### 2. docker [container] create
 - 使用该命令新建一个容器，新建的容器默认是停止状态
 - 由于容器是整个Docker技术栈的核心，create 命令和后续的 run 命令支持的选项都十分复杂，选项主要包括：与容器运行模式相关、与容器环境配置相关、与容器资源限制和安全保护相关等几大类。
 - eg. docker create -it ubuntu:latest

##### 3. docker [container] start
 - 启动一个已经创建的容器
 - eg. docker start ubuntu

##### 4. docker [container] run
 - 新建并启动容器。等价于先执行 docker [container] create命令，再执行 docker [container] start命令
 - 当利用 docker [container] run来创建并启动容器时，Docker 在后台运行的标准操作包括：检查本地是否存在指定的镜像，不存在就从公有仓库下载；利用镜像创建一个容器，并启动该容器；分配一个文件系统给容器，并在只读的镜像层外面挂载一层可读写层；从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去；从网桥的地址池配置一个ip地址给容器；执行用户指定的应用程序；执行完毕后容器自动终止。
 - eg. docker run -it ubuntu:18.04 /bin/bash，该命令启动一个 bash 终端，允许用户进行交互。

##### 5. docker [container] logs
 - 获取容器的输出信息
 - 支持的命令选项：-details：打印详细信息；-f,-follow：持续保持输出；-since string：输出从某个时间开始的日志；-tail string：输出最近的若干日志；-t,-timestamps：显示时间戳信息；-until string：输出某个时间之前的日志。
 - eg. docker logs -f 4688927d56b8

##### 6. docker [container] pause/unpause CONTAINER
 - 暂停或启动容器
 - eg. docker pause/unpause 4688927d56b8

##### 7. docker [container] stop [-t|--time[=10]][CONTAINER...]
 - 终止一个运行中的容器，该命令会首先向容器发送SIGTERM信号，等待一段时间后（默认10秒），再发送SIGKILL信号来终止容器。
 - eg. docker stop 4688927d56b8

##### 8. docker container prune
 - 清除所有处于停止状态的容器

##### 9. docker [container] kill
 - 直接发送SIGKILL信号来强制终止容器

##### 10. docker [container] restart
 - 先终止运行中的容器，然后再重新启动
 
##### 11. docker [container] attach [--detach-keys[=[]]][--no-stdin][--sig-proxy[=true]] CONTAINER
 - 进入容器。使用 attach命令有时不太方便，当多个窗口同时 attach 同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时，其他窗口也无法执行操作了。
 - eg. docker attach 4688927d56b8

##### 12. docker [container] exec [-d|--detach][--detach-keys[=[]]][-i|--interactive][--privileged][-t|--tty][-u|--user[=USER]] CONTAINER COMMAND [ARG...]
 - 进入容器。从 Docker 的1.3.0版本起，Docker 提供了一个更加方便的工具 exec 命令，可以在运营容器内直接执行任意操作。
 - 重要参数：-d,--detach：在容器中后台执行命令；--detach-keys=""：指定将容器切回后台的命令；-e,--env=[]：指定环境变量列表；-i,-interactive=ture|false：打开标准输入接收用户输入命令，默认是false；--privileged=true|false：是否给执行命令以高权限，默认false；-u,--user=""：执行命令的用户名或ID。
 - eg. docker exec -it 4688927d56b8 /bin/bash

##### 13. docker [container] rm [-f|--force][-l|--link][-v|--volumes] CONTAINER
 - 删除处于终止或退出状态的容器。
 - 选项：-f,--force=false：是否强行终止并删除一个运行中的容器；-l,--link=false：删除容器的连接，但保留容器；-v,--volumes=false：删除容器挂载的数据卷。
 - eg. docker rm 4688927d56b8

##### 14. docker [container] export [-o|--output[=""]] CONTAINER
 - 