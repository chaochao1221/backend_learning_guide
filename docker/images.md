# 镜像

#### 1. 概念：Docker运行容器前需要在本地存在对应的镜像，如果镜像不存在，Docker会尝试先从默认镜像库中下载（默认使用 Docker Hub公共注册服务器中的仓库），用户也可以通过配置，使用自定义的镜像仓库。

#### 2. docker [image] pull NAME[:TAG]
 - 使用该命令从Docker Hub镜像源来下载镜像。
 - NAME是镜像仓库名称（用来区分镜像），TAG是镜像的标签（往往用来表示版本信息。如果不显示指定该标签，则默认会选择latest标签，这会下载仓库中最新版本的镜像）。
 - eg. docker pull ubuntu: 18.04

#### 3. docker images 或 docker image ls
 - 使用改命令可以列出本地主机上已有镜像的基本信息。
 - 返回信息中的'TAG'用来区分发行版本。'SIZE'表示镜像大小信息，该信息只是表示了镜像逻辑体积的大小，实际上相同的镜像层本地只会存储一份，物理上占用的存储空间会小于各镜像逻辑体积之和。

#### 4. docker tag NAME[:TAG] NEWNAME[:TAG]
 - 使用该命令为本地镜像添加新的标签。
 - eg. docker tag ubuntu:latest myubuntu:latest
 - 该命令添加的标签实际上起到了类似链接的作用，ubuntu:latest镜像的ID和myubuntu:latest完全一致，他们实际上指向了同一个文件，只是别名不同而已。

#### 5. docker [image] inspect NAME[:TAG]
 - 使用该命令可以获取该镜像的详细信息，包括制造者、适应架构、各层的数字摘要等。
 - eg. docker inspect ubuntu:latest

#### 6. docker history NAME[:TAG]
 - 因为镜像由多个层组成，使用改命令可以列出各层的创建信息。
 - eg. docker history ubuntu:latest

#### 7. docker search [option] keyword
 - 使用该命令可以搜索 Docker Hub 官方仓库中的镜像。
 - 命令选项包括：-f,--filter filter：过滤输出内容。--format string：格式化输出内容。--limit int：限制输出结果个数，默认25个。--no-trunc：不截断输出结果。
 - eg. 搜索官方提供带nginx关键字的镜像：docker search -f=is-official=true nginx
 - eg. 搜索搜所有收藏数超过4的关键词包括tensorflow的镜像：docker search --filter=stars=4 tensorflow
 - 返回信息中包含镜像名称、描述、收藏数、是否官方创建、是否自动创建等信息。

#### 8. docker rmi IMAGE[IMAGE...] 或 docker image rm IMAGE[IMAGE...]
 - 使用该命令可以删除镜像，期中IMAGE可以为标签或ID
 - 命令选项包括：-f,-froce：强制删除镜像，即使有容器依赖它。-no-prune：不要清理未带标签的父镜像。
 - eg. docker rmi myubuntu:latest

#### 9. docker image prune
 - 使用改命令可以清理系统中遗留的一些临时的镜像文件，以及一些没有被使用的镜像文件。
 - 命令选项包括：-a,-all：删除所有无用镜像，不光是临时文件。-filter filter：只清理符合给定过滤器的镜像文件。-f,-force：强制删除镜像，而不进行提示确认。

#### 10. docker [container] commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
 - 基于已有容器创建镜像
 - 命令选项包括：-a,--author=""：作者信息。-c,--change=[]：提交的时候执行 Dockerfile 指令，包括CMD|ENTRYPOINT|ENV|EXPOSE|LABEL|ONBUILD|USER|VOLUME|WORKDIR等。-m,--message=""：提交消息。-p,--pause=true：提交时暂停容器运行。
 - eg. docker commit -m "add a new file" -a "dc" 8fa4587a0b07 mygotenberg:6

#### 11. docker [image] import [OPTIONS] file|URL|-[REPOSITORY[:TAG]]
 - 基于本地模板导入创建镜像
 - eg. cat ubuntu-18.04-x86_64-minimal.tar.gz | docker import -utuntu:18.04

#### 12. Dockerfile

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

#### 13. docker [image] save
 - 使用改命令可以导出镜像到本地
 - 命令选项：-o、-output string参数，导出镜像到指定的文件中。
 - eg. docker save -o ubuntu_18.04.tar ubuntu:18.04

#### 14. docker [image] load
 - 使用该命令可以将导出的tar文件导入到本地镜像库
 - 命令选项：-i、-input string，从指定文件中读入镜像内容。
 - eg. docker load -i ubuntu_18.04.tar。从文件ubuntu_18.04.tar导入镜像到本地镜像列表。
 - eg. docker load < ubuntu_18.04.tar。这将导入镜像及其相关的元数据信息（包括标签等）。导入成功后，可以使用docker images 查看，与原镜像一致。

#### 15. docker [image] push NAME[:TAG] | [REGISTRY_HOST[:REGISTRY_PORT]/]NAME[:TAG]
 - 使用该命令可以上传镜像到仓库，默认上传到Docker Hub官方仓库。
 - 使用该命令之前，需要保证本地已经登录Docker Hub帐号， 如果尚未登录，需先使用docker login进行登录。