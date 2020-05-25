Gitbook 部署（ubuntu）

### 一、安装

1. apt-get update

2. apt-get install npm

3. ln -s /usr/bin/nodejs /usr/bin/node

4. npm install gitbook-cli -g

### 二、部署

1. 创建文件夹：mkdir mygitbook && cd mygitbook
2. 初始化：gitbook init
3. 启动服务器：gitbook serve --lrport 35288 --port 4001
4. 执行 gitbook serve 命令后，会先编译 gitbook build，如果没有问题会打开一个 web 服务器，默认监听 4000 端口，浏览器打开http://localhost:4000即可访问

### 三、常用命令

1. 查看版本：gitbook -V

2. 更新 gitbook：npm update gitbook-cli -g

3. 卸载 gitbook：sudo npm uninstall gitbook-cli -g

4. 查看安装目录：which gitbook

5. 编译：gitbook build --gitbook=2.6.7

6. 指定启动端口：gitbook --port 4001 serve

### 四、常见问题

1. 如果遇到：Error loading version latest: Error: Cannot find module 'internal/util/types'：

- 安装 node 管理：sudo npm install -g n
- 降低版本，更新 npm：sudo n 6

2. 如果遇到：Error: Error loading plugins: katex,splitter,toggle-chapters. Run 'gitbook install' to install plugins from NPM：

- 执行 gitbook install 安装新的插件
