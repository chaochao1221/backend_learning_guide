# 容器

#### 1. 概念：容器是镜像的一个运行实例。与镜像不同的是，镜像是静态的只读文件，而容器带有运行时需要的可写文件层，同时容器中的应用进程处于运行状态。如果认为虚拟机是模拟运行的一整套操作系统和跑在上面的应用。那么 Docker 容器就是独立运行的一个（或一组）应用，以及它们必须的运行环境。

#### 2. docker [container] create
 - 使用该命令新建一个容器，新建的容器默认是停止状态
 - 由于容器是整个Docker技术栈的核心，create 命令和后续的 run 命令支持的选项都十分复杂，选项主要包括：与容器运行模式相关、与容器环境配置相关、与容器资源限制和安全保护相关等几大类。
 - eg. docker create -it ubuntu:latest

#### 3. docker [container] start
 - 启动一个已经创建的容器
 - eg. docker start ubuntu

#### 4. docker [container] run
 - 新建并启动容器。等价于先执行 docker [container] create命令，再执行 docker [container] start命令
 - 当利用 docker [container] run来创建并启动容器时，Docker 在后台运行的标准操作包括：检查本地是否存在指定的镜像，不存在就从公有仓库下载；利用镜像创建一个容器，并启动该容器；分配一个文件系统给容器，并在只读的镜像层外面挂载一层可读写层；从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去；从网桥的地址池配置一个ip地址给容器；执行用户指定的应用程序；执行完毕后容器自动终止。
 - eg. docker run -it ubuntu:18.04 /bin/bash，该命令启动一个 bash 终端，允许用户进行交互。

#### 5. docker [container] logs
 - 获取容器的输出信息
 - 支持的命令选项：-details：打印详细信息；-f,-follow：持续保持输出；-since string：输出从某个时间开始的日志；-tail string：输出最近的若干日志；-t,-timestamps：显示时间戳信息；-until string：输出某个时间之前的日志。
 - eg. docker logs -f 4688927d56b8

#### 6. docker [container] pause/unpause CONTAINER
 - 暂停或启动容器
 - eg. docker pause/unpause 4688927d56b8

#### 7. docker [container] stop [-t|--time[=10]][CONTAINER...]
 - 终止一个运行中的容器，该命令会首先向容器发送SIGTERM信号，等待一段时间后（默认10秒），再发送SIGKILL信号来终止容器。
 - eg. docker stop 4688927d56b8

#### 8. docker container prune
 - 清除所有处于停止状态的容器

#### 9. docker [container] kill
 - 直接发送SIGKILL信号来强制终止容器

#### 10. docker [container] restart
 - 先终止运行中的容器，然后再重新启动

#### 11. docker [container] attach [--detach-keys[=[]]][--no-stdin][--sig-proxy[=true]] CONTAINER
 - 进入容器。使用 attach命令有时不太方便，当多个窗口同时 attach 同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时，其他窗口也无法执行操作了。
 - eg. docker attach 4688927d56b8

#### 12. docker [container] exec [-d|--detach][--detach-keys[=[]]][-i|--interactive][--privileged][-t|--tty][-u|--user[=USER]] CONTAINER COMMAND [ARG...]
 - 进入容器。从 Docker 的1.3.0版本起，Docker 提供了一个更加方便的工具 exec 命令，可以在运营容器内直接执行任意操作。
 - 重要参数：-d,--detach：在容器中后台执行命令；--detach-keys=""：指定将容器切回后台的命令；-e,--env=[]：指定环境变量列表；-i,-interactive=ture|false：打开标准输入接收用户输入命令，默认是false；--privileged=true|false：是否给执行命令以高权限，默认false；-u,--user=""：执行命令的用户名或ID。
 - eg. docker exec -it 4688927d56b8 /bin/bash

#### 13. docker [container] rm [-f|--force][-l|--link][-v|--volumes] CONTAINER
 - 删除处于终止或退出状态的容器。
 - 选项：-f,--force=false：是否强行终止并删除一个运行中的容器；-l,--link=false：删除容器的连接，但保留容器；-v,--volumes=false：删除容器挂载的数据卷。
 - eg. docker rm 4688927d56b8

#### 14. docker [container] export [-o|--output[=""]] CONTAINER
 - 导出容器。导出一个已经创建的容器到一个文件，不管此时这个容器是否处于运行状态。
 - 选项：-o,--output=""：指定导出的tar文件名，也可以直接通过重定向来实现。
 - eg. docker export -o test_for_run.tar 4688927d56b8
 - eg. docker export 4688927d56b8 > test_for_run.tar

#### 15. docker [container] import [-c|--change[=[]]][-m|--message[=MESSAGE]] file|URL|-[REPOSITORY[:TAG]]
 - 导入容器变成一个镜像。
 - 选项：-c,--change=[]：导入的同时执行对容器进行修改的Dockerfile指令。
 - eg. docker import test_for_run.tar -test/ubuntu:v1.0
 - 实际上，即可以使用docker [image] load命令来导入镜像存储文件到本地镜像库，也可以使用docker [container] import命令来导入一个容器快照到本地镜像库。这两者的区别在于：容器快照文件将丢弃所有的历史数据和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积更大。此外，从容器快照文件导入时可以重新指定便签等元数据信息。

#### 16. docker container inspect [OPTIONS] CONTAINER
 - 查看容器的具体信息，内容包括容器id、创建时间、路径、状态、镜像、配置等。
 - eg. docker container inspect 4688927d56b8

#### 17. docker [container] top [OPTIONS] CONTAINER
 - 查看容器的进程信息，内容包括PID、用户、时间、命令等。
 - eg. docker top 4688927d56b8

#### 18. docker [container] stats [OPTIONS] CONTAINER
 - 查看统计信息，内容包括CPU、内存、存储、网络等。
 - 选项：-a,-all：输出所有容器统计信息，默认仅在运行中；-format string：格式化输出信息；-no-stream：不持续输出，默认会自动更新持续实时结果；-no-trunc：不截断输出信息。
 - eg. docker stats 4688927d56b8

#### 19. docker [container] cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
 - 复制文件。支持在容器和主机之间复制文件。
 - 选项：-a,-archive：打包模式，复制文件会带有原始的uid/gid信息；-L,-follow-link：跟随软连接，当原路径为软连接时，默认只复制链接信息。
 - eg. docker cp data test:/tmp/，将本地的路径data复制到test容器的/tmp 路径下。

#### 20. docker [container] diff CONTAINER
 - 查看容器内文件系统的变更。
 - eg. docker diff 4688927d56b8

#### 21. docker container port CONTAINER[PRIVATE_PORT[/PROTO]]
 - 查看端口的映射情况。
 - eg. docker container port 4688927d56b8

#### 22. docker [container] update [OPTIONS] CONTAINER
 - 更新容器运行时配置，主要是一些资源限制份额。
 - 选项：-blkio-weight uint16：更新块IO限制，10～1000，默认值为0，代表着无限制；-cpu-period int：限制CPU调度器CFS（Completely Fair Scheduler）使用时间，单位为微秒，最小1000；-cpu-quota int：限制CPU调度器CFS配额，单位为微秒，最小1000；-cpu-rt-period int：限制CPU调度器的实时周期，单位为微秒；-cpu-rt-runtime int：限制CPU调度器的实时运行时，单位为微秒；-c, -cpu-shares int：限制CPU使用份额；-cpus decimal：限制CPU个数；-cpuset-cpus string：允许使用的CPU核，如0-3,0,1；-cpuset-mems string：允许使用的内存块，如0-3,0,1；-kernel-memory bytes：限制使用的内核内存；-m, -memory bytes：限制使用的内存；-memory-reservation bytes：内存软限制；-memory-swap bytes：内存加上缓存区的限制，-1表示为对缓冲区无限制；-restart string：容器退出后的重启策略。
 - eg. docker update --cpu-quota 1000000 test
 - eg. docker update --cup-period 1000000 test