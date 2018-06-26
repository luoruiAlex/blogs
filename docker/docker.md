## 三大组件
- image，静态
- container为image的实例
- registry

## 使用
- **docker run** container-name
  - 客户端连到守护进程
  - 如果本地没有该镜像，则从registry拉取
  - 守护进程从镜像创建容器，执行COMMAND
  - 守护进程将容器的标准输出转发到客户端
- docker ps -a
  - docker ps -l 查看最后创建的一个容器
- docker images
- docker search image-keyword
- docker pull [repository/][image-name][:/tag]
  - repository默认为Docker Hub
  - tag默认为latest
- 创建容器 docker run -d --name tomcat001 -p 8081:8080 tomcat:7
  - -d指定容器在后台运行
  - -name指定容器的名称，不能重复
  - -p 用于将容器内的8080端口映射到本机的8081端口，本机端口不能重复
- docker logs -f container-name
  - -f表示实时输出最新日志
- 停止/启动容器 docker stop/start container-name
- 删除容器 docker rm container-name
  - -f表示强制删除，一般建议先停止容器
  - -v表示删除关联的数据卷
  
