## 定义和运行多个Docker容器的应用
- 运行用户通过一个单独的docker-compose.yml模板文件来定义一组关联的应用容器为一个项目
- 服务(Service)：一个应用的容器，可包含多个运行相同镜像的容器
- 项目(Porject)：由一组关联的应用容器组成的一个完整业务单元，在docker-compose.yml中定义
