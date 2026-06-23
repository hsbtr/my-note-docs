# Docker Compose 入门

Docker Compose 用来管理多个容器。它把多个服务的镜像、端口、环境变量、数据卷、网络关系写在一个 `compose.yml` 或 `docker-compose.yml` 文件中，然后用一条命令启动。

如果只运行一个 Nginx，直接用 `docker run` 就够了。  
如果一个项目需要后端、数据库、缓存、消息队列，Compose 会更清晰。

## 为什么需要 Compose

假设一个后端项目需要：

- API 服务
- MySQL 数据库
- Redis 缓存

不用 Compose 时，可能要执行多条命令：

```bash
docker network create app-network
docker run -d --name mysql --network app-network -e MYSQL_ROOT_PASSWORD=123456 mysql:8
docker run -d --name redis --network app-network redis
docker run -d --name api --network app-network -p 3000:3000 my-api
```

服务一多，命令会越来越难维护。Compose 可以把这些配置集中到一个文件里：

```bash
docker compose up -d
```

停止时：

```bash
docker compose down
```

## Compose 的核心概念

### project

一个 Compose 文件启动的一组服务，称为一个项目。默认项目名通常来自当前目录名。

### service

`services` 下的每一项就是一个服务。一个服务通常对应一种容器角色，例如 `web`、`api`、`mysql`、`redis`。

```yaml
services:
  web:
    image: nginx
```

### container

容器是服务运行后的实例。一个服务可以启动一个或多个容器。

### network

Compose 默认会为项目创建一个网络。同一个 Compose 项目里的服务，可以通过服务名互相访问。

例如 `api` 访问 Redis，可以使用：

```text
redis:6379
```

而不是写 `localhost:6379`。

### volume

数据卷用于持久化数据。数据库容器尤其需要数据卷，否则容器删除后数据也容易丢失。

## 最小 Compose 示例

创建 `docker-compose.yml`：

```yaml
services:
  web:
    image: nginx:1.25
    ports:
      - "8080:80"
```

启动：

```bash
docker compose up -d
```

访问：

```text
http://localhost:8080
```

停止：

```bash
docker compose down
```

这个示例等价于：

```bash
docker run -d -p 8080:80 nginx:1.25
```

## Compose 文件结构

常见结构：

```yaml
services:
  app:
    image: my-app:1.0
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
    volumes:
      - app-data:/app/data
    depends_on:
      - redis

  redis:
    image: redis:7

volumes:
  app-data:
```

常见顶层字段：

| 字段 | 作用 |
| --- | --- |
| `services` | 定义要运行的服务 |
| `volumes` | 定义命名数据卷 |
| `networks` | 定义自定义网络 |

## services

`services` 是 Compose 文件最重要的部分。

```yaml
services:
  nginx:
    image: nginx:1.25
```

这里定义了一个名为 `nginx` 的服务。

服务名很重要，因为同一个 Compose 网络中的其他服务可以用服务名访问它。

## image 和 build

### 使用现成镜像

```yaml
services:
  redis:
    image: redis:7
```

适合 MySQL、Redis、Nginx 这类已有官方镜像的服务。

### 使用本地 Dockerfile 构建

```yaml
services:
  api:
    build: .
    image: my-api:1.0
    ports:
      - "3000:3000"
```

`build: .` 表示使用当前目录的 Dockerfile 构建镜像。

也可以指定 Dockerfile：

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile.prod
    image: my-api:prod
```

常见选择：

- 第三方服务：用 `image`
- 自己的项目：用 `build`
- 希望构建后有固定镜像名：同时写 `build` 和 `image`

## ports

`ports` 用于端口映射。

```yaml
ports:
  - "8080:80"
```

格式：

```text
主机端口:容器端口
```

表示访问主机 `8080` 端口，会转发到容器 `80` 端口。

注意：服务之间互相访问时，通常使用容器端口，不使用主机映射端口。

例如：

```yaml
services:
  api:
    environment:
      REDIS_URL: redis://redis:6379

  redis:
    image: redis:7
```

`api` 连接 Redis 使用的是 `redis:6379`，不是 `localhost:6379`。

## environment

`environment` 设置环境变量。

```yaml
services:
  mysql:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: app_db
```

也可以使用列表写法：

```yaml
environment:
  - MYSQL_ROOT_PASSWORD=123456
  - MYSQL_DATABASE=app_db
```

建议优先使用键值写法，更清晰。

## env_file

如果环境变量较多，可以放到 `.env` 文件中。

`.env` 示例：

```env
MYSQL_ROOT_PASSWORD=123456
MYSQL_DATABASE=app_db
```

Compose 示例：

```yaml
services:
  mysql:
    image: mysql:8
    env_file:
      - .env
```

注意：`.env` 里可能有密码、密钥等敏感信息，不要随便提交到公开仓库。

## volumes

`volumes` 用于挂载数据。

### 命名数据卷

```yaml
services:
  mysql:
    image: mysql:8
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

说明：

- `mysql-data` 是命名数据卷
- `/var/lib/mysql` 是容器内 MySQL 数据目录
- 容器删除后，数据卷仍然可以保留

### 目录挂载

```yaml
services:
  nginx:
    image: nginx:1.25
    volumes:
      - ./html:/usr/share/nginx/html
```

说明：

- `./html` 是当前目录下的本地目录
- `/usr/share/nginx/html` 是容器内目录

目录挂载适合开发调试，命名数据卷更适合数据库持久化。

## depends_on

`depends_on` 表示服务启动顺序。

```yaml
services:
  api:
    build: .
    depends_on:
      - mysql
      - redis

  mysql:
    image: mysql:8

  redis:
    image: redis:7
```

注意：`depends_on` 只能保证 Compose 先启动 `mysql` 和 `redis` 容器，不保证 MySQL 已经准备好接受连接。

如果应用启动时报数据库连接失败，需要应用自身支持重试，或者配合健康检查。

## restart

`restart` 设置容器异常退出后的重启策略。

```yaml
services:
  api:
    image: my-api:1.0
    restart: unless-stopped
```

常见值：

| 值 | 说明 |
| --- | --- |
| `no` | 不自动重启 |
| `always` | 总是自动重启 |
| `unless-stopped` | 除非手动停止，否则自动重启 |
| `on-failure` | 失败退出时重启 |

本地学习可以不写。长期运行的服务常用 `unless-stopped`。

## networks

Compose 默认会创建网络，多数入门场景不需要手动写 `networks`。

需要更清晰地声明网络时可以这样写：

```yaml
services:
  api:
    image: my-api:1.0
    networks:
      - app-network

  redis:
    image: redis:7
    networks:
      - app-network

networks:
  app-network:
```

同一个网络中的服务可以通过服务名互相访问。

## 完整示例：API + MySQL + Redis

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    image: demo-api:1.0
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
      MYSQL_HOST: mysql
      MYSQL_PORT: 3306
      MYSQL_DATABASE: app_db
      MYSQL_USER: root
      MYSQL_PASSWORD: 123456
      REDIS_HOST: redis
      REDIS_PORT: 6379
    depends_on:
      - mysql
      - redis
    restart: unless-stopped

  mysql:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: app_db
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    restart: unless-stopped

  redis:
    image: redis:7
    ports:
      - "6379:6379"
    restart: unless-stopped

volumes:
  mysql-data:
```

启动：

```bash
docker compose up -d
```

查看服务：

```bash
docker compose ps
```

查看日志：

```bash
docker compose logs -f
```

只查看某个服务日志：

```bash
docker compose logs -f api
```

停止：

```bash
docker compose down
```

## 常用命令

### 启动服务

```bash
docker compose up
```

后台启动：

```bash
docker compose up -d
```

如果包含 `build`，启动前重新构建：

```bash
docker compose up -d --build
```

### 查看服务状态

```bash
docker compose ps
```

### 查看日志

```bash
docker compose logs
```

实时查看：

```bash
docker compose logs -f
```

查看指定服务：

```bash
docker compose logs -f mysql
```

### 进入服务容器

```bash
docker compose exec api sh
```

如果容器有 `bash`：

```bash
docker compose exec api bash
```

### 执行一次性命令

```bash
docker compose run --rm api npm test
```

说明：

- `run` 会启动一个临时容器执行命令
- `--rm` 表示命令结束后删除临时容器

### 停止服务

停止并删除容器和默认网络：

```bash
docker compose down
```

停止但不删除容器：

```bash
docker compose stop
```

重新启动已停止服务：

```bash
docker compose start
```

### 删除数据卷

```bash
docker compose down -v
```

注意：`-v` 会删除 Compose 创建的数据卷。数据库数据也会被删除，执行前要确认。

## Compose 和 docker run 的关系

这个 Compose：

```yaml
services:
  nginx:
    image: nginx:1.25
    ports:
      - "8080:80"
```

大致等价于：

```bash
docker run -d --name 项目名-nginx-1 -p 8080:80 nginx:1.25
```

Compose 的优势不是做了 `docker run` 做不到的事，而是把多条命令变成可维护、可复用的配置文件。

## 常见问题

### 为什么服务之间不能用 localhost

在容器里，`localhost` 指的是当前容器自己，不是宿主机，也不是其他容器。

服务之间访问要用服务名：

```text
mysql:3306
redis:6379
```

### 修改 compose 文件后要做什么

通常重新执行：

```bash
docker compose up -d
```

如果 Dockerfile 或依赖变了：

```bash
docker compose up -d --build
```

### 数据库数据为什么还在

如果使用了命名数据卷，即使执行：

```bash
docker compose down
```

数据卷通常还会保留。

如果想连数据卷一起删除：

```bash
docker compose down -v
```

### container_name 要不要写

入门示例中写 `container_name` 看起来直观，但真实项目里不一定推荐。

不写时，Compose 会自动生成容器名，方便扩缩容和避免命名冲突。

建议：

- 学习和简单本地环境：可以写，便于识别
- 稍复杂项目：优先不写，让 Compose 管理名称

## 入门阶段推荐写法

1. 一个项目一个 Compose 文件。
2. 自己的服务用 `build`，第三方服务用 `image`。
3. 服务之间用服务名访问，不用 `localhost`。
4. 数据库使用命名数据卷保存数据。
5. 敏感配置放 `.env`，不要提交到公开仓库。
6. 先掌握 `up -d`、`ps`、`logs -f`、`exec`、`down`。

