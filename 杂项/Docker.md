`Docker`三要素：
1. 镜像(`Image`): 其实就是一个配置好的操作环境  
2. 容器(`Container`)：可以理解为将镜像安装下来的使用环境  
3. 仓库(`Repository`)、注册服务器(`Registry`)

## 基本使用
安装镜像
```bash
docker search name   查看仓库里name有的镜像  如： docker search  ubuntu
docker pull name     将镜像从仓库下载到本地

docker save images-nameS -o xxx.tar  将镜像打包成tar  也可以  docker images-nameS > xxx.tar 
docker load -i xxx.tar 从压缩包导入镜像  也可以 docker load < xxx.tar
```

查看镜像、运行容器
```bash
docker images            查看本地镜像
docker run image-name    运行容器  

docker run -it image-name   当前终端运行容器(输出容器) -i 以交互模式运行容器 -t 为容器重新分配伪终端
docker run -it image-name [/bin/bash | sh] 当前终端运行容器(能使用容器的终端)
docker run -itd image-name 后台运行容器

docker run -itd --rm image-name 容器关闭后自动删除容器
docker run -itd -p 3306:3306 image-name -p 将宿主机的端口与容器绑定 -P 3306 宿主机随机端口 
docker run -itd --name container-name image-name  --name 自定义容器名称
docker run -itd -e xxx:xxx image-name -e 设置环境变量

docker exec -it container-name [/bin/bash | sh]  进入容器(exit 退出，容器不会停止)

docker logs -f container-id/container-name 查看容器运行日志 (--tail 限制前几行)
```

容器保存成镜像
```bash
docker ps 查看正在运行的容器 
docker ps -a 查看所有容器
docker commit -m "explanation information" -a "user-message" container-id user-name/Repository-name:tag-message  将容器打包成镜像(不推荐，建议使用 dockerfile)
如： docker commit -m "ubuntu before" -a "begin" 01218ac847cd begin/ubuntu:begin
```

删除容器、镜像
```bash
docker rm container-id/container-name  删除容器
docker rmi image-id/image-name 删除镜像
docker image prune  自动删除不必要文件

docker image prune -af   -a 镜像也删除  -f 不必弹出确认信息

docker tag container-name newName 重令名镜像(将镜像归入仓库)
docker rename old_name new_name   为已有的容器命名
```

容器重新启动
```bash
docker start container_name/container_id        后台启动容器
docker stop container_name/container_id         停止容器运行
docker restart container_name/container_id      重启容器
```

# 数据存储
将容器文件传入宿主机 `docker cp <容器ID>:/文件路径/文件名 /宿主机目标路径`

数据卷  
一个可供一个或多个容器使用的特殊目录，可以在容器之间共享和重用，对其的修改会立马生效，默认会一直存在即使容器被删除  
数据卷基本操作
```bash
docker volume create vol_name  创建数据卷
docker volume rm vol_name      删除数据卷
docker volume ls               查看所有数据卷
docker volume inspect vol_name 查看数据卷信息
docker volume prune            清理没有被挂载的数据卷
```

在使用`docker run`命令时，使用`--mount`来挂载数据卷 如：
```bash
 docker run -d -P \
    --name web \
    # -v my-vol:/usr/share/nginx/html \
    --mount source=my-vol,target=/usr/share/nginx/html \
    nginx:alpine
```

使用`--mount`将本地主机文件作为数据卷 如:
```bash
docker run --rm -it \
   # -v $HOME/.bash_history:/root/.bash_history \
   --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
   ubuntu:18.04 \
```

数据卷挂载(推荐方法)
```bash
docker run -v /宿主机目录:/容器内目录 image-name   # 宿主机目录没有会自动创建 
# 数据覆盖优先级： 宿主机 > 容器   当挂载的目录没文件，会复制容器目录的文件;有文件则使用挂载目录的文件
```

# 网络
docker 网络有三种类型(driver): bridge、host、null

>查看docker网络 `docker network ls` 
 创建自定义bridge `docker network create -d bridge bridge_name`
>`docker run -itd --network bridge_name -p 8080:8080 images` 指定网络创建容器
![[docker三种网络模式对比.png]]

# 最佳实践
```bash
docker run -d -p 3306:3306 \
    -v ./mysql:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=password \
    --name mysql \
    mysql:latest
   
docker run -p 6379:6379 --name redis -v ./redis/redis.conf:/etc/redis/redis.conf \
     -v ./redis/data:/data -d redis redis-server /etc/redis/redis.conf \
     --appendonly yes

docker run -d -p 15672:15672 -p 5672:5672 \
-e RABBITMQ_DEFAULT_VHOST=/  \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
--name rabbitmq \
rabbitmq:management
```

# Dockerfile
用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明
> 已经集成在`docker`，不需要额外下载

| 命令         | 说明                                       |
| ---------- | ---------------------------------------- |
| FROM       | 指定基础镜像，必须作为第一条命令                         |
| MAINTAINER | 添加描述、作者                                  |
| ENV        | 设置环境变量                                   |
| RUN        | 构建容器时执行的命令                               |
| CMD        | 构建容器后执行的命令                               |
| ADD        | 将本地文件添加到容器中,tar类型文件会自动解压缩(wget 这种网络类型不会) |
| COPY       | 类似ADD，但不会自动解压缩                           |
| WORKDIR    | 设置工作目录，类似cd命令                            |
| ARG        | 指定转递构建运行时的变量                             |
| VOLUMN     | 用于指定持久化目录                                |
| EXPOSE     | 指定与外界交互端口（规范性操作，可有可无）                    |
| USER       | 指定容器运行时的用户                               |
## 最佳实践
Dockerfile
```bash
FROM mysql:latest   
ENV MYSQL_ROOT_PASSWORD password   
EXPOSE 3306
```

构建
```bash
# Dockerfile 同级目录
docker build 构建容器
docker build --rm -f -t  # -t 设置镜像名称及标签  name:tag
```

# Docker compose
用于定义和运行多容器 Docker 应用程序的工具。使用YML文件来配置应用程序需要的所有服务

默认使用文件名`docker-compose.yml`，也可以使用 `-f` 参数指定具体文件  
文件包含4个一级key:`services`、`networks`、`volumes`

运行命令
```bash
# 文件名是docker-compose.yml,且文件在当前目录
docker compose up  直接运行输出运行日志
docker compose down 关闭并删除容器
docker compose ps
docker compose logs
docker compose build 构建或重新构建服务
docker compose start | stop | restart

docker compose up -f xxx.xxx up -d 后台运行(xxx.xxx是docker-compose文件)
```
## 最佳实践
单`docker-compose.yml`文件
```bash
services:  
  mysql:  
    image: mysql:latest  
    container_name: mysql  
    restart: always  
    environment:  
      MYSQL_ROOT_PASSWORD: password  
    ports:  
      - "3306:3306"  
    volumes:  
      - ./mysql:/var/lib/mysql  
  redis:  
    image: redis:latest  
    container_name: redis  
    restart: always  
    ports:  
      - "6379:6379"  
    volumes:  
      - ./redis:/data  
  rabbitmq:  
    image: rabbitmq:management  
    container_name: rabbitmq  
    restart: always  
    environment:  
      RABBITMQ_DEFAULT_VHOST: /  
      RABBITMQ_DEFAULT_USER: admin  
      RABBITMQ_DEFAULT_PASS: admin  
    ports:  
      - "15672:15672"  
      - "5672:5672"
```

与`Dockerfile`一起使用
```bash
services:  
  mysql:  
    image: mysql:8.0.36  
    container_name: swapstay_mysql  
    environment:  
      MYSQL_DATABASE: swapstay  
      MYSQL_USER: swapstay  
      MYSQL_PASSWORD: password  
      MYSQL_ALLOW_EMPTY_PASSWORD: true  
      TZ: Asia/Shanghai  
    ports:  
      - "3307:3306"  
  redis:  
    image: redis:7.2.4  
    container_name: swapstay_redis  
    ports:  
      - "6379:6379"  
  rabbitmq:  
  # 在 build 中添加 dockerfile 字段， context 是其所在位置
    build:  
      context: .  
      dockerfile: Dockerfile.rabbitmq  
    container_name: swapstay_rabbitmq  
    environment:  
      RABBITMQ_DEFAULT_USER: swapstay  
      RABBITMQ_DEFAULT_PASS: password  
      RABBITMQ_DEFAULT_VHOST: swapstay  
    ports:  
      - "5672:5672"  
      - "15672:15672"  
  minio:  
    image: bitnami/minio:2024.4.18  
    container_name: swapstay_minio  
    environment:  
      MINIO_ROOT_USER: minio  
      MINIO_ROOT_PASSWORD: "correct horse battery staple"  
    ports:  
      - "9000:9000"  
      - "9001:9001"
```