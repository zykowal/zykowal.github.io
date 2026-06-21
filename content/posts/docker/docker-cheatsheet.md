---
date: 2026-06-20
draft: false
title: 'Docker Cheatsheet'
tags: ["Cheatsheet"]
categories: ["Docker", "Cloud Native"]
series: []
summary: "Docker 常用命令与概念速查表"
---

## Docker Cheatsheet

### 什么是 Docker？Docker 解决了什么问题？

**Docker** 是一个容器化平台，允许你将应用及其依赖打包到一个轻量级、可移植的容器中。

**解决的问题：**
- **环境不一致**："在我机器上能跑" — 容器保证开发、测试、生产环境一致
- **依赖冲突**：每个容器有独立的文件系统、网络、进程空间
- **资源利用率低**：相比虚拟机，容器共享宿主机内核，启动快（毫秒级）、资源开销小
- **交付与部署**：Image 作为标准交付件，一次构建，随处运行

### Images vs Containers

| 概念 | 说明 |
|------|------|
| **Image（镜像）** | 只读模板，包含运行应用所需的代码、运行时、库、环境变量、配置文件 |
| **Container（容器）** | Image 的运行实例，可被创建、启动、停止、删除 |
| **Registry（仓库）** | 存储和分发 Image 的服务，默认是 [Docker Hub](https://hub.docker.com) |

- 一个 Image 可以启动多个 Container
- Image 是构建时的产物，Container 是运行时的实体

### Docker 命令变迁

| 旧命令 | 新命令（推荐） |
|--------|---------------|
| `docker run` | `docker container run` |
| `docker ps` | `docker container ls` |
| `docker rm` | `docker container rm` |
| `docker images` | `docker image ls` |
| `docker rmi` | `docker image rm` |
| `docker build` | `docker image build` |

### 容器生命周期

```bash
# 运行容器（前台）
docker container run -p 80:80 nginx

# 运行容器（后台 -d）
docker container run -p 80:80 -d nginx

# 指定名称和端口
docker container run --publish 80:80 --detach --name my-nginx nginx

# 交互式运行
docker container run -it --name my-ubuntu ubuntu bash

# 自动删除（运行结束后删除容器）
docker container run --rm -it ubuntu bash

# 列出运行中的容器
docker container ls

# 列出所有容器（包括已停止）
docker container ls -a

# 停止容器
docker container stop <container-id>

# 启动已停止的容器
docker container start <container-id>

# 重启容器
docker container restart <container-id>

# 查看容器进程
docker container top <container-id>

# 查看容器日志
docker container logs <container-id>
docker container logs -f <container-id>  # 实时追踪

# 进入运行中的容器
docker container exec -it <container-id> bash

# 删除容器
docker container rm <container-id>
docker container rm -f <container-id>   # 强制删除运行中的容器

# 查看容器资源使用
docker container stats
docker container stats <container-id>  # 查看指定容器

# 查看容器详细信息（IP、端口映射、挂载卷等）
docker container inspect <container-id>
docker container inspect -f '{{.NetworkSettings.IPAddress}}' <container-id>

# 清理所有已停止的容器
docker container prune
```

### 镜像管理

```bash
# 拉取镜像
docker image pull nginx
docker image pull nginx:1.25-alpine

# 查看本地镜像
docker image ls

# 查看镜像详细信息
docker image inspect <image-id>

# 构建镜像
docker image build -t my-app:1.0 .

# 标记镜像（打标签）
docker image tag my-app:1.0 <username>/my-app:1.0

# 推送镜像到仓库
docker image push <username>/my-app:1.0

# 删除镜像
docker image rm <image-id>

# 清理未使用的镜像
docker image prune
docker image prune -a  # 删除所有未使用的镜像
```

### Dockerfile 指令速查

```dockerfile
FROM    # 基础镜像，如 alpine:3.19
LABEL   # 元数据标签
ENV     # 环境变量
ARG     # 构建参数
WORKDIR # 工作目录
COPY    # 复制文件到镜像
ADD     # 复制文件（支持URL和解压）
RUN     # 构建时执行命令
CMD     # 容器启动时的默认命令（可被覆盖）
ENTRYPOINT # 容器入口命令（不可被覆盖）
EXPOSE  # 声明端口
VOLUME  # 挂载卷
USER    # 指定运行用户
HEALTHCHECK # 健康检查
```

### .dockerignore

`.dockerignore` 用于在构建镜像时排除不必要的文件，减少构建上下文体积、加快构建速度、避免敏感文件泄露。

```dockerfile
.git/
node_modules/
*.log
.env
.gitignore
Dockerfile
.gitkeep
__pycache__/
.venv/
.idea/
dist/
```

### 网络管理

```bash
# 列出网络
docker network ls

# 创建网络
docker network create my-network

# 运行容器并连接到指定网络
docker container run -d --name my-app --network my-network nginx

# 将已有容器连接到网络
docker network connect my-network my-container

# 断开连接
docker network disconnect my-network my-container

# 查看网络详情
docker network inspect my-network

# 删除网络
docker network rm my-network
```

### 数据卷（Volumes）

```bash
# 创建卷
docker volume create my-volume

# 挂载卷到容器
docker container run -d --name my-app -v my-volume:/app/data nginx

# 绑定挂载（宿主机目录）
docker container run -d --name my-app -v /host/path:/app/data nginx

# 查看卷
docker volume ls
docker volume inspect my-volume

# 删除卷
docker volume rm my-volume
docker volume prune  # 清理未使用的卷
```

### Docker Compose

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - frontend
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
    depends_on:
      - db
  db:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: secret

volumes:
  pgdata:

networks:
  frontend:
```

```bash
# 启动服务
docker-compose up
docker-compose up -d  # 后台运行

# 停止服务
docker-compose down
docker-compose down -v  # 同时删除卷

# 查看日志
docker-compose logs -f

# 重新构建并启动
docker-compose up -d --build

# 查看服务状态
docker-compose ps
```

### Docker Swarm

```bash
# 初始化 Swarm 集群（当前节点成为 Manager）
docker swarm init
docker swarm init --advertise-addr 192.168.1.100

# 查看 Swarm 集群状态
docker node ls

# 生成 Worker 加入令牌
docker swarm join-token worker

# 生成 Manager 加入令牌
docker swarm join-token manager

# 部署服务
docker service create --name my-web --replicas 3 -p 80:80 nginx

# 查看服务列表
docker service ls

# 查看服务详情
docker service inspect my-web

# 查看服务日志
docker service logs my-web

# 滚动更新服务
docker service update --image nginx:1.26-alpine --update-parallelism 2 my-web

# 扩缩容
docker service scale my-web=5

# 查看服务任务（容器分布）
docker service ps my-web

# 删除服务
docker service rm my-web

# 离开 Swarm 集群
docker swarm leave          # Worker 节点
docker swarm leave --force  # Manager 节点
docker swarm leave --force && docker swarm init  # 重置 Swarm

### Docker Context

`docker context` 用于管理多个 Docker 环境（本地、远程、云），方便在不同端点间切换。

```bash
# 查看当前使用的 context
docker context ls

# 创建远程 Docker context
docker context create remote-docker --docker "host=tcp://192.168.1.100:2375"

# 切换到指定 context
docker context use remote-docker

# 切换回默认 context
docker context use default

# 查看 context 详情
docker context inspect remote-docker

# 更新 context
docker context update remote-docker --docker "host=tcp://192.168.1.200:2375"

# 删除 context
docker context rm remote-docker
```

### 常用技巧

```bash
# 查看容器间网络通信
docker network inspect bridge

# 复制文件到容器
docker container cp <container-id>:/path/to/file ./local

# 从容器复制文件出来
docker container cp ./local <container-id>:/path/to/file

# 查看 Docker 系统信息
docker system df          # 查看磁盘使用
docker system info        # 查看系统信息
docker system prune       # 清理所有未使用的资源
docker system prune -a --volumes  # 彻底清理
```

### 常见问题

| 问题 | 命令 |
|------|------|
| 容器退出码非零 | `docker container logs <id>` 查看错误日志 |
| 端口冲突 | `docker container ls` 检查占用；换端口或停用冲突容器 |
| 镜像拉取慢 | 配置镜像加速器（如 阿里云、中科大） |
| 容器内时间不准 | 挂载 `/etc/localtime` 或设置 `TZ` 环境变量 |
| 磁盘空间不足 | `docker system prune -a` 清理未使用的镜像和容器 |
