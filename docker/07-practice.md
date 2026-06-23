# 动手练习与学习顺序

## 学习练习

### 练习 1：运行 Nginx

```bash
docker run -d --name nginx-test -p 8080:80 nginx
```

浏览器访问：

```text
http://localhost:8080
```

停止并删除：

```bash
docker stop nginx-test
docker rm nginx-test
```

### 练习 2：运行 Redis

```bash
docker run -d --name redis-test -p 6379:6379 redis
```

进入 Redis：

```bash
docker exec -it redis-test redis-cli
```

测试：

```bash
set name docker
get name
```

### 练习 3：使用 Compose 启动多个服务

创建 `docker-compose.yml`：

```yaml
services:
  nginx:
    image: nginx
    ports:
      - "8080:80"

  redis:
    image: redis
```

启动：

```bash
docker compose up -d
```

停止：

```bash
docker compose down
```


## 推荐学习顺序

1. 理解镜像、容器、仓库的关系。
2. 掌握 `docker run`、`docker ps`、`docker stop`、`docker rm`。
3. 学会端口映射和数据卷挂载。
4. 学会写简单 Dockerfile。
5. 学会使用 Docker Compose 管理多个服务。
6. 再深入学习网络、构建优化、多阶段构建和生产部署。
