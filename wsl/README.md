# WSL 开发环境入门

WSL（Windows Subsystem for Linux）可以在 Windows 上直接运行 Linux 发行版、命令行工具和开发工具链。本文以首次安装 Ubuntu 为例，目标是把 WSL 配置成日常开发环境。

## 适用场景

- 在 Windows 上使用 Linux 命令、Shell、包管理器和开发工具链。
- 使用 VS Code 连接 WSL，在 Linux 环境中编辑、运行、调试项目。
- 避免双系统或完整虚拟机带来的切换成本。
- 让项目依赖、脚本和部署环境更接近 Linux 服务器。

## 安装前准备

1. 使用 Windows 10 或 Windows 11，并确保系统已更新到较新版本。
2. 推荐安装 Windows Terminal，便于管理 PowerShell、CMD、Ubuntu 等多个终端配置。
3. 推荐安装 VS Code，并准备后续安装 `WSL` 扩展。
4. 安装命令建议在“以管理员身份运行”的 PowerShell 或 Windows Terminal 中执行。

## 安装 WSL 和 Ubuntu

首次安装时，打开管理员 PowerShell，执行：

```powershell
wsl --install
```

该命令通常会完成以下工作：

- 启用 WSL 和虚拟机平台相关 Windows 组件。
- 安装或更新 WSL 所需的 Linux 内核。
- 将 WSL 2 设置为默认版本。
- 安装默认 Linux 发行版，通常是 Ubuntu。

如果想明确指定 Ubuntu，可以执行：

```powershell
wsl --install -d Ubuntu
```

安装过程中如果提示重启，请先重启 Windows。重启后打开 Ubuntu，按提示创建 Linux 用户名和密码。

> 注意：这里创建的是 Ubuntu 内部用户，不一定要和 Windows 用户名、密码一致。

## 常用 WSL 管理命令

在 PowerShell 或 CMD 中执行以下命令：

```powershell
# 查看已安装的发行版和 WSL 版本
wsl -l -v

# 查看可在线安装的发行版
wsl --list --online

# 启动默认发行版
wsl

# 启动指定发行版
wsl -d Ubuntu

# 关闭所有 WSL 发行版
wsl --shutdown

# 更新 WSL
wsl --update

# 查看 WSL 状态
wsl --status
```

如果发现 Ubuntu 不是 WSL 2，可以执行：

```powershell
wsl --set-version Ubuntu 2
```

## 初始化 Ubuntu 开发环境

进入 Ubuntu 后，先更新软件包：

```bash
sudo apt update
sudo apt upgrade -y
```

安装常用开发工具：

```bash
sudo apt install -y build-essential curl wget git unzip zip ca-certificates
```

配置 Git 基本信息：

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
git config --global core.autocrlf input
git config --global core.quotepath false
```

建议把项目代码放在 Ubuntu 文件系统中，例如：

```bash
mkdir -p ~/code
cd ~/code
```

不要长期在 `/mnt/c/...` 下进行高频开发，因为跨 Windows 文件系统访问通常更慢，也更容易遇到权限、换行符或文件监听问题。

## 使用 VS Code 连接 WSL

1. 在 Windows 中安装 VS Code。
2. 在 VS Code 扩展市场安装 `WSL` 扩展。
3. 打开 Ubuntu，进入项目目录：

```bash
cd ~/code/your-project
code .
```

第一次执行 `code .` 时，VS Code 会在 WSL 中安装服务端组件。之后即可在 Windows 的 VS Code 界面中使用 WSL 内的终端、Git、Node、Python、Go 等工具链。

## 安装常见语言运行时

### Node.js

推荐在 WSL 内使用 nvm 管理 Node.js：

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
source ~/.bashrc
nvm install --lts
nvm use --lts
node -v
npm -v
```

### Python

Ubuntu 通常自带 Python，可以补充安装虚拟环境工具：

```bash
sudo apt install -y python3 python3-pip python3-venv
python3 --version
```

创建项目虚拟环境：

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### Docker

如果使用 Docker Desktop，可以在 Docker Desktop 设置中启用 WSL 集成，然后在 Ubuntu 中使用 Docker 命令。

检查方式：

```bash
docker --version
docker compose version
```

## Windows 和 WSL 文件互访

在 WSL 中访问 Windows C 盘：

```bash
cd /mnt/c/Users
```

在 Windows 文件资源管理器中访问 WSL 文件：

```text
\\wsl$\Ubuntu\home\你的用户名
```

也可以在 Ubuntu 目录中执行：

```bash
explorer.exe .
```

## 日常开发建议

- 项目代码优先放在 `~/code` 等 WSL 内部目录。
- 依赖、编译、测试命令尽量都在 WSL 终端中执行。
- Windows 侧主要负责浏览器、编辑器、聊天工具等图形界面应用。
- Git、Node、Python、Go、Rust 等开发工具建议安装在 WSL 内，而不是混用 Windows 和 WSL 两套环境。
- 如果终端或文件监听异常，可以先尝试执行 `wsl --shutdown` 后重新进入 Ubuntu。

## 常见问题

### 忘记 Ubuntu 密码怎么办？

可以在 PowerShell 中以 root 用户进入发行版：

```powershell
wsl -d Ubuntu -u root
```

然后在 Ubuntu 中重置指定用户密码：

```bash
passwd 你的用户名
```

### 如何确认当前是否在 WSL 中？

```bash
uname -a
cat /proc/version
```

输出中通常会包含 `Microsoft` 或 `WSL` 字样。

### 如何让 Ubuntu 成为默认发行版？

```powershell
wsl --set-default Ubuntu
```

## 参考资料

- [Microsoft Learn: Install WSL](https://learn.microsoft.com/windows/wsl/install)
- [Microsoft Learn: Set up a WSL development environment](https://learn.microsoft.com/windows/wsl/setup/environment)
- [Microsoft Learn: Basic commands for WSL](https://learn.microsoft.com/windows/wsl/basic-commands)
- [Microsoft Learn: Get started using VS Code with WSL](https://learn.microsoft.com/windows/wsl/tutorials/wsl-vscode)
