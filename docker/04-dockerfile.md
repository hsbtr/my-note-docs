# Dockerfile 入门

Dockerfile 是用来构建镜像的文本文件。它把“这个应用需要什么环境、复制哪些文件、安装哪些依赖、启动时执行什么命令”写成一组步骤。

如果说镜像是应用的安装包，那么 Dockerfile 就是生成这个安装包的配方。

## Dockerfile 的基本使用流程

常见流程是：

1. 在项目根目录创建 `Dockerfile`。
2. 编写构建步骤。
3. 使用 `docker build` 构建镜像。
4. 使用 `docker run` 运行镜像生成容器。

示例：

```bash
docker build -t my-app:1.0 .
docker run -d --name my-app -p 3000:3000 my-app:1.0
```

这里的 `.` 很重要，表示当前目录是构建上下文。

## 构建上下文是什么

执行：

```bash
docker build -t my-app .
```

最后的 `.` 表示把当前目录作为构建上下文发送给 Docker。Dockerfile 中的 `COPY`、`ADD` 只能访问构建上下文里的文件。

例如项目结构是：

```text
my-app/
  Dockerfile
  package.json
  src/
    index.js
```

在 `my-app` 目录执行：

```bash
docker build -t my-app .
```

Dockerfile 可以写：

```dockerfile
COPY package.json ./
COPY src ./src
```

如果你在上一级目录执行构建，或者把构建上下文写错，就可能遇到 `COPY failed`、`file not found` 之类的问题。

## 一个 Node.js 项目 Dockerfile 示例

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

这个 Dockerfile 做了几件事：

1. 使用 `node:20-alpine` 作为基础镜像。
2. 把容器内工作目录设置为 `/app`。
3. 先复制 `package.json` 和锁文件。
4. 执行 `npm install` 安装依赖。
5. 再复制项目其他文件。
6. 声明应用监听 `3000` 端口。
7. 容器启动时执行 `npm start`。

如果项目有 `package-lock.json`，生产构建中更推荐：

```dockerfile
RUN npm ci
```

`npm ci` 会严格按锁文件安装依赖，更适合可重复构建。

## 常用指令说明

### FROM

`FROM` 指定基础镜像。几乎所有 Dockerfile 都从 `FROM` 开始。

```dockerfile
FROM node:20-alpine
```

常见基础镜像：

- `ubuntu:22.04`
- `alpine:3.20`
- `node:20-alpine`
- `python:3.12-slim`
- `nginx:1.25`

入门阶段建议优先使用官方镜像，并指定明确版本，不要长期依赖 `latest`。

### WORKDIR

`WORKDIR` 设置容器内的工作目录。后续的 `COPY`、`RUN`、`CMD` 默认都在这个目录下执行。

```dockerfile
WORKDIR /app
```

推荐使用 `WORKDIR`，不要到处写：

```dockerfile
RUN cd /app && npm install
```

### COPY

`COPY` 把构建上下文中的文件复制到镜像中。

```dockerfile
COPY package*.json ./
COPY src ./src
```

格式：

```dockerfile
COPY 源路径 目标路径
```

源路径在主机的构建上下文中，目标路径在镜像内部。

### ADD

`ADD` 也可以复制文件，但它有额外能力：

- 可以自动解压本地压缩包
- 可以从 URL 下载文件

入门阶段建议优先使用 `COPY`。只有确实需要 `ADD` 的额外能力时再使用它。

### RUN

`RUN` 在构建镜像时执行命令，常用来安装依赖、编译代码、创建目录。

```dockerfile
RUN npm install
```

Debian/Ubuntu 镜像中安装软件示例：

```dockerfile
RUN apt-get update && apt-get install -y curl
```

Alpine 镜像中安装软件示例：

```dockerfile
RUN apk add --no-cache curl
```

注意：`RUN` 是构建镜像时执行，不是容器启动时执行。

### ENV

`ENV` 设置环境变量，构建镜像和运行容器时都可以使用。

```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
```

运行容器时也可以覆盖：

```bash
docker run -e PORT=8080 my-app
```

### ARG

`ARG` 设置构建参数，只在构建镜像时使用。

```dockerfile
ARG APP_VERSION=dev
ENV APP_VERSION=$APP_VERSION
```

构建时传入：

```bash
docker build --build-arg APP_VERSION=1.0.0 -t my-app:1.0.0 .
```

简单理解：

- `ARG`：构建时参数
- `ENV`：运行时环境变量

### EXPOSE

`EXPOSE` 声明容器内部应用监听的端口。

```dockerfile
EXPOSE 3000
```

注意：`EXPOSE` 只是声明，并不会自动把端口映射到宿主机。

真正让主机能访问容器端口，需要在运行容器时使用 `-p`：

```bash
docker run -p 3000:3000 my-app
```

### CMD

`CMD` 定义容器启动时默认执行的命令。

```dockerfile
CMD ["npm", "start"]
```

如果运行容器时手动指定命令，会覆盖 `CMD`：

```bash
docker run my-app npm run dev
```

### ENTRYPOINT

`ENTRYPOINT` 也用于定义容器启动命令，但它更像固定入口。

```dockerfile
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

入门阶段可以先这样理解：

- `CMD`：默认命令，容易被覆盖
- `ENTRYPOINT`：固定入口，适合工具型镜像

大多数普通业务服务，用 `CMD` 就够了。

### USER

`USER` 指定容器内以哪个用户运行后续命令和启动进程。

```dockerfile
USER node
```

不指定时，很多镜像默认使用 `root`。生产环境中通常建议使用非 root 用户运行应用。

## COPY 和 ADD 的区别

| 指令 | 主要用途 | 是否推荐日常使用 |
| --- | --- | --- |
| `COPY` | 复制本地文件或目录 | 推荐 |
| `ADD` | 复制文件、自动解压、下载 URL | 谨慎使用 |

推荐原则：

- 普通复制文件：用 `COPY`
- 需要自动解压压缩包：可以考虑 `ADD`
- 需要下载远程文件：更推荐用 `RUN curl ...`，过程更清晰

## CMD 和 ENTRYPOINT 的区别

### CMD 示例

```dockerfile
CMD ["node", "server.js"]
```

运行：

```bash
docker run my-app
```

实际执行：

```bash
node server.js
```

如果运行时传入新命令：

```bash
docker run my-app node worker.js
```

`CMD` 会被覆盖，实际执行 `node worker.js`。

### ENTRYPOINT 示例

```dockerfile
ENTRYPOINT ["redis-cli"]
```

运行：

```bash
docker run redis-tool -h redis-demo
```

实际执行效果类似：

```bash
redis-cli -h redis-demo
```

所以 `ENTRYPOINT` 更适合“这个镜像就是一个命令行工具”的场景。

## 为什么要先复制 package.json

很多 Node.js Dockerfile 会这样写：

```dockerfile
COPY package*.json ./
RUN npm install
COPY . .
```

而不是：

```dockerfile
COPY . .
RUN npm install
```

原因是 Docker 构建有缓存机制。只要某一层的输入没有变化，这一层就可以复用。

依赖文件不变时：

- `COPY package*.json ./` 可以复用
- `RUN npm install` 可以复用
- 只需要重新复制业务代码

如果一开始就 `COPY . .`，任何业务代码改动都可能导致依赖安装缓存失效，构建会变慢。

## .dockerignore

`.dockerignore` 用来排除不需要进入构建上下文的文件，作用类似 `.gitignore`。

Node.js 项目示例：

```gitignore
node_modules
npm-debug.log
.git
.env
dist
coverage
logs
```

为什么需要它：

- 减少构建上下文大小
- 避免把本地依赖复制进镜像
- 避免把 `.env`、日志、临时文件打进镜像
- 提升构建速度

注意：如果 Dockerfile 里需要复制某个文件，就不要在 `.dockerignore` 中把它排除掉。

## 前端静态项目多阶段构建示例

前端项目通常先用 Node 构建，再用 Nginx 提供静态文件。

```dockerfile
FROM node:20-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:1.25-alpine

COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

这个 Dockerfile 有两个阶段：

1. `build` 阶段：安装依赖并构建前端产物。
2. 最终阶段：只保留 Nginx 和构建后的静态文件。

好处：

- 最终镜像更小
- 不把 `node_modules` 和源码构建工具放进生产镜像
- 运行环境更简单

如果项目构建产物目录是 `build`，需要改成：

```dockerfile
COPY --from=build /app/build /usr/share/nginx/html
```

## Python 项目 Dockerfile 示例

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
```

如果是 Flask、FastAPI、Django 项目，启动命令会不同。例如 FastAPI 常见写法：

```dockerfile
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 构建镜像

在 Dockerfile 所在目录执行：

```bash
docker build -t my-app:1.0 .
```

参数说明：

- `-t my-app:1.0`：镜像名是 `my-app`，标签是 `1.0`
- `.`：当前目录作为构建上下文

指定 Dockerfile 文件名：

```bash
docker build -f Dockerfile.prod -t my-app:prod .
```

不使用缓存构建：

```bash
docker build --no-cache -t my-app:debug .
```

查看构建过程更详细输出：

```bash
docker build --progress=plain -t my-app .
```

## 运行镜像

```bash
docker run -d --name my-app -p 3000:3000 my-app:1.0
```

查看日志：

```bash
docker logs -f my-app
```

进入容器：

```bash
docker exec -it my-app sh
```

停止并删除：

```bash
docker stop my-app
docker rm my-app
```

## 常见错误

### COPY 文件找不到

常见原因：

- 构建命令最后的上下文路径写错
- 文件被 `.dockerignore` 排除了
- Dockerfile 中路径写错

排查：

```bash
docker build --progress=plain -t my-app .
```

### EXPOSE 了端口但浏览器访问不了

`EXPOSE` 不等于端口映射。运行容器时还需要：

```bash
docker run -p 8080:80 nginx
```

格式是：

```text
主机端口:容器端口
```

### 容器启动后马上退出

常见原因：

- `CMD` 命令执行完就结束了
- 应用启动失败
- 配置文件或环境变量缺失

排查：

```bash
docker ps -a
docker logs 容器名
```

### 镜像很大

常见原因：

- 基础镜像太大
- 构建产物、缓存、日志被复制进镜像
- 没有使用 `.dockerignore`
- 可以多阶段构建但没有使用

优化方向：

- 使用 `slim` 或 `alpine` 镜像
- 添加 `.dockerignore`
- 删除不需要的构建缓存
- 使用多阶段构建

## 入门阶段推荐写法

1. 基础镜像指定明确版本，例如 `node:20-alpine`。
2. 使用 `WORKDIR` 设置工作目录。
3. 先复制依赖描述文件，再安装依赖。
4. 再复制业务代码。
5. 使用 `.dockerignore` 排除无关文件。
6. 使用 `CMD` 定义默认启动命令。
7. 构建后用 `docker run` 和 `docker logs` 验证。

