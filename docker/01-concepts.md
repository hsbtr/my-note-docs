# Docker 基础概念与安装检查

## Docker 是什么

Docker 是一种容器化技术，用来把应用程序及其运行环境一起打包、分发和运行。

传统部署中，应用依赖系统环境、语言版本、配置文件、第三方库等。如果不同机器环境不一致，就容易出现“我电脑上能跑，服务器上跑不了”的问题。

Docker 的核心价值是：

- 环境一致：开发、测试、生产环境尽量保持一致。
- 部署方便：应用和依赖被打包成镜像，到哪里都能运行。
- 启动快速：容器通常比虚拟机更轻量。
- 隔离性好：不同应用可以运行在不同容器中，互不干扰。


## Docker 的核心概念

### 镜像 Image

镜像是一个只读模板，里面包含运行应用所需的代码、依赖、系统工具和配置。

可以把镜像理解成“安装包”或“应用运行环境快照”。

常见镜像示例：

- `nginx`
- `mysql`
- `redis`
- `ubuntu`
- `node`
- `python`

### 容器 Container

容器是镜像运行起来之后的实例。

镜像和容器的关系类似：

- 镜像：类、模板、安装包
- 容器：对象、实例、正在运行的程序

一个镜像可以启动多个容器。

### 仓库 Repository

仓库用来存储和分发镜像。

常见镜像仓库：

- Docker Hub：官方公共镜像仓库
- 私有仓库：公司或个人内部使用

常见操作：

- 从仓库拉取镜像：`docker pull nginx`
- 将镜像推送到仓库：`docker push username/image-name`

### Dockerfile

Dockerfile 是构建镜像的脚本文件。

它描述了镜像应该基于什么系统、安装哪些依赖、复制哪些文件、启动什么命令。

### Docker Compose

Docker Compose 用来管理多个容器。

例如一个 Web 项目可能需要：

- 后端服务
- MySQL 数据库
- Redis 缓存
- Nginx 代理

这些服务可以写在一个 `docker-compose.yml` 文件中，一条命令启动。


## 安装后检查

查看 Docker 版本：

```bash
docker --version
```

查看 Docker 详细信息：

```bash
docker info
```

运行官方测试容器：

```bash
docker run hello-world
```

如果能看到成功提示，说明 Docker 可以正常使用。
