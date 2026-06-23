# 数据卷与网络

## 数据卷 Volume

容器删除后，容器内部的数据通常也会丢失。为了持久化数据，可以使用数据卷。

### 使用命名数据卷

```bash
docker volume create mysql-data
```

查看数据卷：

```bash
docker volume ls
```

运行 MySQL 并挂载数据卷：

```bash
docker run -d \
  --name mysql-demo \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v mysql-data:/var/lib/mysql \
  mysql:8
```

说明：

- `-v mysql-data:/var/lib/mysql`：将数据卷挂载到容器中的 MySQL 数据目录
- 容器删除后，`mysql-data` 数据卷仍然存在

### 使用目录挂载

```bash
docker run -d \
  --name my-nginx \
  -p 8080:80 \
  -v /host/path/html:/usr/share/nginx/html \
  nginx
```

说明：

- `/host/path/html`：主机目录
- `/usr/share/nginx/html`：容器目录


## 网络 Network

Docker 容器之间可以通过 Docker 网络通信。

### 查看网络

```bash
docker network ls
```

### 创建网络

```bash
docker network create my-network
```

### 容器加入网络

```bash
docker run -d --name redis-demo --network my-network redis
```

同一个自定义网络中的容器，可以通过容器名互相访问。

例如后端容器连接 Redis 时，可以使用：

```text
redis-demo:6379
```
