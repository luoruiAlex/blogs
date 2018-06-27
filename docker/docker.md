## 三大组件
- image，为Docker容器运行的静态模板
  - Docker使用unionFS来将一系列的层组合为单个镜像
  - 从基础镜像通过一系列指令构建，每条指令会在镜像中创建一个新的层
  - 指令
    - 放在Dockerfile文本文件中
    - 包括 运行一个命令、添加文件或者文件夹、创建环境变量、从该镜像启动容器时运行哪个程序
- container为image的实例，通过run启动
  - 拉取镜像
  - 创建容器
  - 分配一个文件系统并将其挂载为一个可读写层
  - 分配网桥/网卡
  - 分配一个IP地址
  - 执行指定的进程
  - 捕获并提供应用的输出
- registry

## 分层文件系统
- linux至少有２个文件系统，boot file system(只和内核版本有关，启动后卸载释放) root file system(与linux发行版本有关，启动后由只读模式改为读写模式)
- ＡUFS unionFS是指将不同物理位置的目录合并mount到同一个目录中
- Docker 中，基础镜像中的 roofs 会一直保持只读模式，Docker 会利用 union mount 来在这个 rootfs 上增加更多的只读文件系统，最后它们看起来就像一个文件系统即容器的 rootfs
- 在一个Linux 系统之中
  - 所有 Docker 容器都共享主机系统的 bootfs 即 Linux 内核
  - 每个容器有自己的 rootfs，它来自不同的 Linux 发行版的基础镜像，包括 Ubuntu，Debian 和 SUSE 等
  - 所有基于一种基础镜像的容器都共享这种 rootfs
- Docker 采用 AFUS 分层文件系统时，文件系统的改动都是发生在最上面的容器层

## Volume
- Data Volume 本质上是 Docker Host 文件系统中的目录或文件，能够直接被 mount 到容器的文件系统中
- Data Volume 是目录或文件，而非没有格式化的磁盘（块设备）
- 镜像是分层存储，容器也是，为容器运行时读写准备的存储层为容器存储层
- 容器存储层声明周期和容器一样
- 最佳实践：容器存储层应无状态化，文件写入应使用数据卷(Volumn)或者绑定宿主目录
- docker volume create my-vol
- docker volume ls
- docker volume inspect my-vol
- docker run 
  - -v 本地主机目录:容器目录\[:ro, consistent, delegated, cached, z, Z] 用于独立的容器，本地目录必须是绝对路径，如果本地目录不存在会创建
  - --mount type=bind,source=/src/webapp,target=/opt/webapp\[,readonly] 用于swarm容器，17.06之后也可用于独立容器，如果可以本地目录如果不存在则报错

## Dockerfile
- 第一条指令必须为 FROM，比如FROM redis FROM scratch
- RUN 

## 使用
- **docker run** container-name[:version] [COMMAND]
  - 客户端连到守护进程
  - 如果本地没有该镜像，则从registry拉取
  - 守护进程从镜像创建容器，执行COMMAND
  - 守护进程将容器的标准输出转发到客户端
- docker ps -a
  - docker ps -l 查看最后创建的一个容器
- docker images
- docker search image-keyword
- docker pull [[user-name/]registry[:port]/][image-name][:/tag]
  - Docker Hub的user-name默认为library，即官方镜像
  - registry默认为Docker Hub
  - tag默认为latest
- 创建容器 docker run -d --name tomcat001 -p 8081:8080 tomcat:7
  - -it 进入交互式终端
  - -d指定容器在后台运行
  - -name指定容器的名称，不能重复
  - -p 用于将容器内的8080端口映射到本机的8081端口，本机端口不能重复
  - -P 随机映射49000~49900 的端口到内部容器开放的网络端口
- docker logs -f container-name
  - -f表示实时输出最新日志
- 停止/启动容器 docker stop/start container-name
- 删除容器 docker rm container-name
  - -f表示强制删除，一般建议先停止容器
  - -v表示删除关联的数据卷
  
## 制作自己的镜像
- 1.docker pull ubuntu:16.04 这里不从srcatch镜像而是从系统镜像开始
- mkdir baseos 该目录作为镜像的上下文
- cd baseos
- touch Dockerfile
```
# Base os image
FROM ubuntu:16.04
MAINTAINER your_name <your_email_address>
LABEL Description="This image is the base os images."  Version="1.0"
# reconfig timezone
RUN echo "Asia/Shanghai" > /etc/timezone \
    && dpkg-reconfigure -f noninteractive tzdata
```
- docker build -t example.com/baseos:1.0 .

