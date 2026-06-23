# Docker 学习笔记

这是一组面向新手的 Docker 入门文档，适合按顺序学习，也适合日常查命令。

## 文档目录

| 文件 | 内容 |
| --- | --- |
| [01-concepts.md](01-concepts.md) | Docker 是什么、镜像/容器/仓库、Dockerfile、Compose、安装检查 |
| [02-basic-commands.md](02-basic-commands.md) | 镜像命令、容器命令、常用参数、命令总览 |
| [03-volume-network.md](03-volume-network.md) | 数据卷、目录挂载、Docker 网络 |
| [04-dockerfile.md](04-dockerfile.md) | Dockerfile 写法、构建上下文、缓存、多阶段构建、常见错误 |
| [05-compose.md](05-compose.md) | Docker Compose 的服务、端口、环境变量、数据卷、网络和常用命令 |
| [06-cleanup-and-faq.md](06-cleanup-and-faq.md) | 清理命令、实践建议、常见问题 |
| [07-practice.md](07-practice.md) | Nginx、Redis、Compose 练习和学习顺序 |
| [08-registry.md](08-registry.md) | 公共镜像仓库、私有镜像仓库、镜像命名、tag、push、pull |

## 推荐学习顺序

1. 先读 [01-concepts.md](01-concepts.md)，理解镜像、容器、仓库的关系。
2. 再读 [02-basic-commands.md](02-basic-commands.md)，掌握最常用的命令。
3. 学习 [03-volume-network.md](03-volume-network.md)，理解数据如何持久化、容器如何通信。
4. 重点学习 [04-dockerfile.md](04-dockerfile.md)，理解镜像是怎样从 Dockerfile 构建出来的。
5. 重点学习 [05-compose.md](05-compose.md)，用一个文件启动多个互相关联的服务。
6. 学习 [08-registry.md](08-registry.md)，理解公共镜像仓库、私有镜像仓库和镜像推送流程。
7. 最后看 [06-cleanup-and-faq.md](06-cleanup-and-faq.md) 和 [07-practice.md](07-practice.md)，补齐实践细节。

## 高频命令速查

```bash
# 查看版本和环境信息
docker --version
docker info

# 镜像
docker pull nginx
docker images
docker build -t my-app .
docker tag my-app:latest registry.example.com/team/my-app:1.0.0
docker push registry.example.com/team/my-app:1.0.0
docker rmi nginx

# 容器
docker run -d --name my-nginx -p 8080:80 nginx
docker ps
docker ps -a
docker stop my-nginx
docker start my-nginx
docker restart my-nginx
docker rm my-nginx

# 日志和进入容器
docker logs -f my-nginx
docker exec -it my-nginx sh

# 数据卷和网络
docker volume ls
docker network ls

# Compose
docker compose up -d
docker compose ps
docker compose logs -f
docker compose down
```
