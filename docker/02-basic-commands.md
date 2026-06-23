# 镜像与容器常用命令

## 镜像常用命令

### 查看本地镜像

```bash
docker images
```

或：

```bash
docker image ls
```

### 拉取镜像

```bash
docker pull nginx
```

拉取指定版本：

```bash
docker pull nginx:1.25
```

如果不写版本，默认使用 `latest`：

```bash
docker pull nginx:latest
```

### 搜索镜像

```bash
docker search nginx
```

### 删除镜像

```bash
docker rmi nginx
```

删除指定镜像 ID：

```bash
docker rmi 镜像ID
```

如果镜像正在被容器使用，需要先删除相关容器。

### 给镜像打标签

```bash
docker tag 原镜像名:标签 新镜像名:标签
```

示例：

```bash
docker tag my-app:latest my-app:v1.0
```

### 构建镜像

在 Dockerfile 所在目录执行：

```bash
docker build -t my-app:latest .
```

参数说明：

- `-t`：指定镜像名称和标签
- `.`：表示使用当前目录作为构建上下文


## 容器常用命令

### 创建并运行容器

```bash
docker run nginx
```

后台运行：

```bash
docker run -d nginx
```

指定容器名称：

```bash
docker run -d --name my-nginx nginx
```

### 端口映射

将主机的 `8080` 端口映射到容器的 `80` 端口：

```bash
docker run -d --name my-nginx -p 8080:80 nginx
```

访问：

```text
http://localhost:8080
```

### 查看正在运行的容器

```bash
docker ps
```

查看所有容器，包括已停止的：

```bash
docker ps -a
```

### 停止容器

```bash
docker stop my-nginx
```

也可以使用容器 ID：

```bash
docker stop 容器ID
```

### 启动已停止的容器

```bash
docker start my-nginx
```

### 重启容器

```bash
docker restart my-nginx
```

### 删除容器

删除已停止的容器：

```bash
docker rm my-nginx
```

强制删除正在运行的容器：

```bash
docker rm -f my-nginx
```

### 查看容器日志

```bash
docker logs my-nginx
```

实时查看日志：

```bash
docker logs -f my-nginx
```

查看最近 100 行：

```bash
docker logs --tail 100 my-nginx
```

### 进入容器

```bash
docker exec -it my-nginx bash
```

如果容器中没有 `bash`，可以使用 `sh`：

```bash
docker exec -it my-nginx sh
```

### 查看容器详细信息

```bash
docker inspect my-nginx
```

### 查看容器资源占用

```bash
docker stats
```


## 常用参数速查

| 参数 | 说明 | 示例 |
| --- | --- | --- |
| `-d` | 后台运行容器 | `docker run -d nginx` |
| `--name` | 指定容器名称 | `docker run --name my-nginx nginx` |
| `-p` | 端口映射 | `-p 8080:80` |
| `-v` | 挂载数据卷或目录 | `-v data:/var/lib/mysql` |
| `-e` | 设置环境变量 | `-e MYSQL_ROOT_PASSWORD=123456` |
| `-it` | 交互式终端 | `docker exec -it app sh` |
| `--rm` | 容器退出后自动删除 | `docker run --rm alpine echo hello` |
| `--network` | 指定网络 | `--network my-network` |


## 常用命令总览

```bash
# 查看版本
docker --version

# 查看 Docker 信息
docker info

# 拉取镜像
docker pull nginx

# 查看镜像
docker images

# 构建镜像
docker build -t my-app .

# 运行容器
docker run -d --name my-nginx -p 8080:80 nginx

# 查看运行中的容器
docker ps

# 查看所有容器
docker ps -a

# 停止容器
docker stop my-nginx

# 启动容器
docker start my-nginx

# 重启容器
docker restart my-nginx

# 删除容器
docker rm my-nginx

# 删除镜像
docker rmi nginx

# 查看日志
docker logs -f my-nginx

# 进入容器
docker exec -it my-nginx sh

# 查看数据卷
docker volume ls

# 查看网络
docker network ls

# Compose 启动
docker compose up -d

# Compose 停止
docker compose down
```
