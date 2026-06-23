# 清理、实践建议与常见问题

## 清理命令

### 删除停止的容器

```bash
docker container prune
```

### 删除未使用的镜像

```bash
docker image prune
```

删除所有未使用镜像：

```bash
docker image prune -a
```

### 删除未使用的数据卷

```bash
docker volume prune
```

### 删除未使用的网络

```bash
docker network prune
```

### 一次性清理未使用资源

```bash
docker system prune
```

包含未使用镜像：

```bash
docker system prune -a
```

注意：清理命令可能删除暂时不用但仍然有价值的资源，执行前要仔细确认。


## 常见实践建议

### 不要把重要数据只放在容器内部

数据库、上传文件、缓存持久化文件等应使用数据卷或外部存储。

### 镜像标签不要总是依赖 latest

生产环境建议使用明确版本：

```bash
mysql:8.4
redis:7
nginx:1.25
```

这样可以避免镜像更新导致不可预期的问题。

### 为容器设置有意义的名称

推荐：

```bash
docker run -d --name project-api api-image
```

不推荐长期依赖随机容器名。

### 使用 .dockerignore

`.dockerignore` 可以避免把无关文件复制进镜像构建上下文。

示例：

```gitignore
node_modules
.git
.env
dist
logs
```

### 一个容器尽量只运行一个主要进程

例如：

- Nginx 一个容器
- MySQL 一个容器
- Redis 一个容器
- 后端服务一个容器

多个服务之间用 Docker Compose 编排。


## 常见问题

### 镜像和容器有什么区别

镜像是模板，容器是镜像运行后的实例。

删除容器不会自动删除镜像；删除镜像前需要确保没有容器依赖它。

### 为什么容器停止后数据没了

容器内部文件系统是临时的。要持久化数据，需要使用数据卷或目录挂载。

### docker run 和 docker start 有什么区别

`docker run` 会基于镜像创建一个新容器并启动。

`docker start` 会启动一个已经存在但停止的容器。

### docker exec 和 docker attach 有什么区别

`docker exec` 会在运行中的容器里开启一个新命令或新终端，常用且安全。

`docker attach` 会连接到容器主进程的输入输出，退出方式不当可能影响容器运行。

日常进入容器推荐：

```bash
docker exec -it 容器名 sh
```

### 端口映射方向怎么看

格式：

```bash
-p 主机端口:容器端口
```

示例：

```bash
-p 8080:80
```

表示访问主机的 `8080` 端口，会转发到容器的 `80` 端口。
