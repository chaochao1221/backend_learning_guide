# 仓库

#### 1. 仓库概念：仓库是集中存放镜像的地方，又分公共仓库和私有仓库。有时容易把仓库和注册服务器（Registry）混淆。实际上注册服务器是存放仓库的具体服务器，一个注册服务器上可以有多个仓库，而每个仓库下面可以有多个镜像。从这方面来说，仓库可以被认为是一个具体的项目或目录。例如仓库地址 private-docker.com/ubuntu，private-docker.com是注册服务器地址，ubuntu是仓库名。

#### 2. 公共镜像市场概念：Docker Hub是Docker提供的最大的公共镜像库，地址为https://hub.docker.com。大部分对镜像的需求，都可以通过在 Docker Hub 中直接下载镜像来实现。

#### 3. docker login
 - 输入用户名、密码和邮箱来完成注册和登录。注册完成以后，本地用户目录下会自动创建 ~/.docker/config.json文件，保存用户的认证信息。
 - 用户 docker login 后，可以通过 docker search，docker [image] pull命令来下载镜像到本地。

#### 4. 自动创建
 - 自动创建是Docker Hub提供的自动化服务，可以自动跟随项目代码的变更而重新构建镜像。
 - 配置自动创建步骤：1）创建并登录Docker Hub，以及目标网站如Github。2）在目标网站中允许Docker Hub访问服务。3）在Docker Hub中配置一个"自动创建"类型的项目。4）选取一个目标网站中的项目（需要含DockerFile）和分支。5）指定DockerFile的位置，并提交创建。