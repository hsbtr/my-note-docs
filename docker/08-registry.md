# 公共镜像仓库与私有镜像仓库

镜像仓库用于保存、分发 Docker 镜像。你可以从仓库拉取别人已经做好的镜像，也可以把自己构建好的镜像推送到仓库，供其他机器或团队成员使用。

这里的“库”指镜像仓库，不是代码里的公共库、工具库或依赖库。

## 为什么需要镜像仓库

如果只在本机学习，可以直接：

```bash
docker build -t my-app .
docker run my-app
```

但一旦需要把镜像给别人或服务器使用，就需要仓库：

1. 本地构建镜像。
2. 给镜像打上仓库地址和版本标签。
3. 推送到镜像仓库。
4. 其他机器从仓库拉取镜像并运行。

流程类似：

```text
本地 Dockerfile -> 构建镜像 -> 推送到仓库 -> 服务器拉取镜像 -> 运行容器
```

## 公共镜像仓库

公共镜像仓库中的镜像通常可以被任何人拉取。最常见的是 Docker Hub。

常见公共镜像：

- `nginx`
- `redis`
- `mysql`
- `postgres`
- `node`
- `python`
- `ubuntu`

拉取镜像：

```bash
docker pull nginx:1.25
```

运行镜像：

```bash
docker run -d --name my-nginx -p 8080:80 nginx:1.25
```

公共镜像的优点：

- 获取方便
- 官方镜像维护较好
- 文档和示例多

需要注意：

- 不要随便使用来源不明的镜像
- 生产环境不要长期依赖 `latest`
- 选择镜像时看清楚镜像来源、版本和说明

## 私有镜像仓库

私有镜像仓库用于保存不希望公开的镜像，例如公司内部应用、业务服务镜像。

常见私有仓库类型：

- Docker Hub 私有仓库
- Harbor
- 云厂商镜像仓库
- 自建 Docker Registry

私有仓库通常需要登录后才能拉取或推送镜像。

登录：

```bash
docker login registry.example.com
```

退出登录：

```bash
docker logout registry.example.com
```

拉取私有镜像：

```bash
docker pull registry.example.com/team/api:1.0.0
```

运行私有镜像：

```bash
docker run -d --name api -p 3000:3000 registry.example.com/team/api:1.0.0
```

## 镜像命名规则

完整镜像名一般长这样：

```text
仓库地址/命名空间/镜像名:标签
```

示例：

```text
registry.example.com/team/api:1.0.0
```

含义：

| 部分 | 示例 | 说明 |
| --- | --- | --- |
| 仓库地址 | `registry.example.com` | 镜像仓库服务地址 |
| 命名空间 | `team` | 组织、用户或项目名 |
| 镜像名 | `api` | 具体镜像名称 |
| 标签 | `1.0.0` | 镜像版本 |

Docker Hub 官方镜像可以省略很多内容。

例如：

```bash
docker pull nginx:1.25
```

完整理解可以近似看成：

```text
docker.io/library/nginx:1.25
```

其中：

- `docker.io` 是 Docker Hub 地址
- `library` 是官方镜像命名空间
- `nginx` 是镜像名
- `1.25` 是标签

## tag 的作用

标签用于区分镜像版本。

常见标签：

```text
latest
1.0.0
1.0
dev
staging
prod
2026-05-21
```

查看本地镜像：

```bash
docker images
```

给镜像打标签：

```bash
docker tag my-app:latest registry.example.com/team/my-app:1.0.0
```

这不会复制一份完整镜像，只是给同一个镜像增加一个新名字。

## 推送镜像到仓库

完整流程：

```bash
# 1. 构建本地镜像
docker build -t my-app:1.0.0 .

# 2. 登录私有仓库
docker login registry.example.com

# 3. 打上仓库地址标签
docker tag my-app:1.0.0 registry.example.com/team/my-app:1.0.0

# 4. 推送镜像
docker push registry.example.com/team/my-app:1.0.0
```

其他机器拉取：

```bash
docker pull registry.example.com/team/my-app:1.0.0
```

运行：

```bash
docker run -d --name my-app -p 3000:3000 registry.example.com/team/my-app:1.0.0
```

## 推送到 Docker Hub 示例

假设 Docker Hub 用户名是 `alice`，镜像名是 `my-app`。

登录：

```bash
docker login
```

构建镜像：

```bash
docker build -t my-app:1.0.0 .
```

打标签：

```bash
docker tag my-app:1.0.0 alice/my-app:1.0.0
```

推送：

```bash
docker push alice/my-app:1.0.0
```

拉取：

```bash
docker pull alice/my-app:1.0.0
```

如果仓库是私有的，拉取前也需要登录。

## 自建本地 Registry 示例

学习时可以用官方 `registry` 镜像在本机启动一个简单仓库。

启动本地仓库：

```bash
docker run -d \
  --name local-registry \
  -p 5000:5000 \
  registry:2
```

构建镜像：

```bash
docker build -t my-app:1.0.0 .
```

打标签：

```bash
docker tag my-app:1.0.0 localhost:5000/my-app:1.0.0
```

推送：

```bash
docker push localhost:5000/my-app:1.0.0
```

拉取：

```bash
docker pull localhost:5000/my-app:1.0.0
```

停止并删除本地仓库：

```bash
docker stop local-registry
docker rm local-registry
```

注意：这个示例适合本地学习。正式环境需要配置权限、存储、HTTPS、备份等。

## 公共仓库和私有仓库的区别

| 对比项 | 公共仓库 | 私有仓库 |
| --- | --- | --- |
| 访问权限 | 通常任何人可拉取 | 需要授权 |
| 适合内容 | 通用软件、开源镜像 | 公司内部应用、业务镜像 |
| 常见示例 | Docker Hub 官方镜像 | Harbor、云厂商镜像仓库 |
| 安全要求 | 关注来源可信度 | 关注账号、权限、网络和审计 |

## 版本标签建议

入门阶段容易直接使用：

```text
latest
```

但生产环境更推荐明确版本：

```text
1.0.0
1.0.1
2.0.0
```

可以同时维护多个标签：

```bash
docker tag my-app:1.0.0 registry.example.com/team/my-app:1.0.0
docker tag my-app:1.0.0 registry.example.com/team/my-app:stable
docker push registry.example.com/team/my-app:1.0.0
docker push registry.example.com/team/my-app:stable
```

建议：

- 开发环境可以用 `dev`
- 测试环境可以用 `test` 或 `staging`
- 生产环境使用明确版本号
- 不要把不同内容反复推到同一个生产版本标签

## 常见问题

### pull、push、tag 分别做什么

| 命令 | 作用 |
| --- | --- |
| `docker pull` | 从仓库拉取镜像 |
| `docker push` | 推送镜像到仓库 |
| `docker tag` | 给本地镜像增加新名字或新标签 |

### 为什么 push 失败

常见原因：

- 没有 `docker login`
- 没有目标仓库权限
- 镜像名没有带正确仓库地址
- 目标仓库不存在或命名空间写错
- 网络无法访问仓库

排查顺序：

```bash
docker images
docker login registry.example.com
docker push registry.example.com/team/my-app:1.0.0
```

### 为什么 pull 私有镜像失败

常见原因：

- 没有登录
- 当前账号没有权限
- 镜像名称或标签写错
- 私有仓库证书或网络配置有问题

可以先确认镜像名：

```bash
docker pull registry.example.com/team/my-app:1.0.0
```

### 镜像仓库是不是代码仓库

不是。

代码仓库保存源码，例如 Git 仓库。  
镜像仓库保存构建后的镜像，例如 `my-app:1.0.0`。

常见流程是：

```text
代码仓库 -> CI/CD 构建镜像 -> 镜像仓库 -> 服务器部署
```

## 入门阶段推荐掌握

1. 能看懂镜像名：`registry/namespace/name:tag`。
2. 能从公共仓库拉取镜像：`docker pull nginx:1.25`。
3. 能给本地镜像打标签：`docker tag ...`。
4. 能登录私有仓库：`docker login registry.example.com`。
5. 能推送镜像：`docker push registry.example.com/team/app:1.0.0`。
6. 生产环境使用明确版本标签，不长期依赖 `latest`。

