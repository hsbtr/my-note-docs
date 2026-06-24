# WSL Ubuntu 开发环境手册

WSL（Windows Subsystem for Linux）适合把 Windows 电脑配置成接近 Linux 服务器的开发环境：Windows 负责桌面应用、浏览器和编辑器界面，Ubuntu 负责 Git、Shell、语言运行时、依赖安装、构建和测试。

本文以首次发行版使用 Ubuntu 为主线，重点记录安装位置、导入导出、常用管理命令、Git、网络、环境变量、文件系统和开发工具配置。

## 快速目录

- 先装起来：安装前准备、安装 Ubuntu、指定安装位置。
- 长期维护：发行版管理、导出导入、备份恢复、资源限制。
- 开发环境：Ubuntu 初始化、Git、VS Code、语言运行时、Docker。
- Windows 协作：文件互访、网络、代理、环境变量、命令互调。
- 排障入口：默认用户、换行、PATH、DNS、释放资源。

## 推荐使用方式

- 项目代码放在 Ubuntu 文件系统内，例如 `~/code`，不要长期放在 `/mnt/c/...` 下做高频开发。
- Git、Node.js、Python、Go、Rust、Java、包管理器和构建命令优先安装并运行在 WSL 内。
- Windows 侧主要运行 VS Code、浏览器、Office、聊天工具、数据库 GUI 等图形界面应用。
- 需要容器时优先使用 Docker Desktop + WSL Integration，或者在 WSL 内独立安装 Docker Engine。
- 重要发行版定期 `wsl --export` 备份，迁移机器时再 `wsl --import`。

## 安装前准备

建议先安装：

- Windows Terminal。
- VS Code。
- VS Code 扩展：`WSL`。
- Git for Windows（主要用于 Windows 侧命令和 Git Credential Manager）。

检查 WSL 状态：

```powershell
wsl --version
wsl --status
wsl --help
```

查看可在线安装的发行版：

```powershell
wsl --list --online
```

## 安装 Ubuntu

默认安装 WSL 和 Ubuntu：

```powershell
wsl --install -d Ubuntu
```

如果命令提示需要重启，重启 Windows 后再打开 Ubuntu，按提示创建 Linux 用户名和密码。

> Ubuntu 内的用户名和密码只属于该 Linux 发行版，不需要和 Windows 用户一致。

安装完成后检查：

```powershell
wsl -l -v
```

如果 Ubuntu 不是 WSL 2：

```powershell
wsl --set-version Ubuntu 2
```

设置 Ubuntu 为默认发行版：

```powershell
wsl --set-default Ubuntu
```

## 指定安装位置

### 新安装时指定位置

新版 WSL 支持在安装时指定位置，例如放到 `D:\WSL\Ubuntu`：

```powershell
wsl --install -d Ubuntu --location D:\WSL\Ubuntu
```

如果当前系统的 `wsl --install --help` 中没有 `--location`，说明 WSL 版本较旧。先更新：

```powershell
wsl --install --help
wsl --update
wsl --shutdown
```

仍不支持时，可以先正常安装，再用导出和导入完成迁移。

### 已有发行版迁移位置

已经安装好的发行版不能直接修改安装目录，常用做法是先导出，再注销原发行版，最后导入到新目录。

假设当前发行版叫 `Ubuntu`，目标位置为 `D:\WSL\Ubuntu`，备份文件为 `D:\WSL\backup\ubuntu.tar`：

```powershell
wsl --shutdown
New-Item -ItemType Directory -Force D:\WSL\backup
New-Item -ItemType Directory -Force D:\WSL\Ubuntu
wsl --export Ubuntu D:\WSL\backup\ubuntu.tar
Get-Item D:\WSL\backup\ubuntu.tar
wsl --unregister Ubuntu
wsl --import Ubuntu D:\WSL\Ubuntu D:\WSL\backup\ubuntu.tar --version 2
```

`wsl --unregister Ubuntu` 会删除原发行版，请确认 `ubuntu.tar` 已经导出成功后再执行。

导入后默认用户可能变成 `root`。可以在 Ubuntu 内创建或确认用户：

```bash
id
ls /home
```

然后在 `/etc/wsl.conf` 中指定默认用户：

```ini
[user]
default=你的用户名
```

修改后在 PowerShell 中重启 WSL：

```powershell
wsl --shutdown
wsl -d Ubuntu
```

## 发行版管理命令

这些命令在 PowerShell 或 CMD 中执行：

```powershell
# 查看已安装发行版和 WSL 版本
wsl -l -v

# 查看在线可安装发行版
wsl --list --online

# 启动默认发行版
wsl

# 启动指定发行版
wsl -d Ubuntu

# 以 root 用户进入
wsl -d Ubuntu -u root

# 执行单条 Linux 命令
wsl -d Ubuntu -- uname -a

# 关闭某个发行版
wsl --terminate Ubuntu

# 关闭所有 WSL 发行版和 WSL 虚拟机
wsl --shutdown

# 更新 WSL
wsl --update

# 查看 WSL 组件版本
wsl --version

# 查看 WSL 状态
wsl --status

# 设置以后新发行版默认使用 WSL 2
wsl --set-default-version 2

# 导出发行版
wsl --export Ubuntu D:\WSL\backup\ubuntu.tar

# 导出为 VHDX，适合 WSL 2 大环境备份
wsl --export Ubuntu D:\WSL\backup\ubuntu.vhdx --vhd

# 从 tar 导入发行版
wsl --import Ubuntu-Dev D:\WSL\Ubuntu-Dev D:\WSL\backup\ubuntu.tar --version 2

# 从 VHDX 导入发行版
wsl --import Ubuntu-Dev D:\WSL\Ubuntu-Dev D:\WSL\backup\ubuntu.vhdx --vhd --version 2

# 直接登记已有 ext4.vhdx
wsl --import-in-place Ubuntu-Dev D:\WSL\Ubuntu-Dev\ext4.vhdx

# 删除发行版，危险操作
wsl --unregister Ubuntu-Dev
```

建议给不同用途的发行版取清晰名字，例如：

- `Ubuntu`：主力开发环境。
- `Ubuntu-Test`：临时实验环境。
- `Ubuntu-Clean`：干净模板环境。

## Ubuntu 基础初始化

进入 Ubuntu 后先更新系统：

```bash
sudo apt update
sudo apt upgrade -y
```

安装基础开发工具：

```bash
sudo apt install -y \
  build-essential \
  ca-certificates \
  curl \
  wget \
  git \
  gnupg \
  lsb-release \
  unzip \
  zip \
  jq \
  tree \
  htop \
  ripgrep \
  fd-find \
  net-tools \
  iproute2 \
  dnsutils
```

Ubuntu 中 `fd-find` 安装后的命令通常叫 `fdfind`。如果习惯使用 `fd`，可以加一个软链接：

```bash
mkdir -p ~/.local/bin
ln -sf "$(command -v fdfind)" ~/.local/bin/fd
```

创建开发目录：

```bash
mkdir -p ~/code ~/tools ~/tmp
```

常用 Shell 配置可以放在 `~/.bashrc` 或 `~/.zshrc`：

```bash
export EDITOR=code
export LANG=C.UTF-8
export LC_CTYPE=C.UTF-8
alias ll='ls -alF'
alias gs='git status --short'
```

## Git 配置

### 基础配置

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
git config --global init.defaultBranch main
git config --global core.autocrlf input
git config --global core.eol lf
git config --global core.quotepath false
git config --global pull.rebase false
git config --global fetch.prune true
```

说明：

- `core.autocrlf input`：提交时把 CRLF 转成 LF，检出时不强制改回 CRLF，适合 Linux 开发环境。
- `core.eol lf`：让文本文件在 WSL 内优先使用 LF。
- `core.quotepath false`：中文文件名在 Git 输出中更容易阅读。
- `fetch.prune true`：拉取远程分支信息时清理已经删除的远程分支引用。

建议仓库内补充 `.gitattributes`，把换行规则写进项目：

```gitattributes
* text=auto eol=lf
*.bat text eol=crlf
*.cmd text eol=crlf
*.ps1 text eol=crlf
```

### 凭据管理

如果使用 HTTPS 克隆 GitHub、GitLab 或 Azure DevOps，推荐复用 Git for Windows 自带的 Git Credential Manager。

先确认 Windows 侧已安装 Git Credential Manager：

```powershell
git credential-manager --version
```

在 WSL 中检查是否能调用：

```bash
git credential-manager --version
```

如果 WSL 内不能直接调用，可以指定 Windows 侧的 GCM：

```bash
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"
```

如果 Git for Windows 安装路径不同，先在 PowerShell 中查找：

```powershell
where git-credential-manager
```

### SSH 配置

如果使用 SSH 克隆仓库，在 WSL 内生成密钥：

```bash
ssh-keygen -t ed25519 -C "你的邮箱"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

把公钥添加到代码托管平台后测试：

```bash
ssh -T git@github.com
```

建议在 `~/.ssh/config` 中配置常用主机：

```sshconfig
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519
  AddKeysToAgent yes
```

不要把 Windows 用户目录下的私钥直接复制到公开仓库或同步盘。密钥权限异常时执行：

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

### 多账号配置

可以按目录设置不同身份，例如个人项目在 `~/code/personal`，公司项目在 `~/code/work`：

```bash
git config --global includeIf.gitdir:~/code/work/.path ~/.gitconfig-work
git config --global includeIf.gitdir:~/code/personal/.path ~/.gitconfig-personal
```

`~/.gitconfig-work` 示例：

```ini
[user]
  name = 公司姓名
  email = name@company.com
```

## 文件系统边界

WSL 访问 Windows 文件：

```bash
cd /mnt/c/Users
```

Windows 文件资源管理器访问 WSL：

```text
\\wsl$\Ubuntu\home\你的用户名
```

从 WSL 打开当前目录的资源管理器：

```bash
/mnt/c/Windows/explorer.exe .
```

开发建议：

- 代码、依赖、构建产物放在 `~/code`、`~/.cache`、`~/.npm`、`~/.cargo` 等 Linux 文件系统内。
- 只在 `/mnt/c` 下读取或复制文件，不建议把大型前端、Java、Python 项目长期放在 Windows 文件系统内开发。
- 避免 Windows 和 WSL 同时修改同一批依赖目录，例如 `node_modules`、`.venv`、`target`、`dist`。
- 需要从 Windows 工具打开项目时，用 VS Code 的 WSL 模式打开，而不是直接打开 `\\wsl$` 下的目录做重型文件操作。

## VS Code 连接 WSL

在 Ubuntu 项目目录中执行：

```bash
cd ~/code/your-project
code .
```

VS Code 左下角应显示连接到 WSL。此时：

- 终端运行在 Ubuntu 内。
- Git 使用 Ubuntu 内的 Git。
- Node、Python、Go 等扩展应安装在 WSL 远程环境中。
- 项目路径应是 `/home/你的用户名/code/...`，不是 `C:\...`。

如果 `code .` 不可用，先确认 Windows 已安装 VS Code 和 `WSL` 扩展，然后在 Windows 中重开 VS Code 或重启 WSL：

```powershell
wsl --shutdown
```

如果已经在 `/etc/wsl.conf` 中设置 `appendWindowsPath=false`，`code` 命令可能不会自动出现在 WSL 的 PATH 里。可以在 Windows PowerShell 中查到 VS Code 命令路径：

```powershell
where code
```

再把对应的 `bin` 目录按需加入 WSL 的 `~/.bashrc`，例如：

```bash
export PATH="$PATH:/mnt/c/Users/你的Windows用户名/AppData/Local/Programs/Microsoft VS Code/bin"
```

## 语言运行时

### Node.js

推荐使用 nvm：

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
source ~/.bashrc
nvm install --lts
nvm alias default 'lts/*'
node -v
npm -v
```

建议项目内使用 `packageManager`、`.nvmrc` 或 Volta/asdf 固定版本，避免不同机器版本漂移。

### Python

```bash
sudo apt install -y python3 python3-pip python3-venv pipx
python3 --version
```

项目虚拟环境：

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -U pip
```

### Java

```bash
sudo apt install -y openjdk-21-jdk
java -version
```

### Go

Ubuntu apt 源里的 Go 版本可能偏旧。需要新版本时建议按官方二进制包或 asdf 管理。

```bash
sudo apt install -y golang-go
go version
```

## Docker

### Docker Desktop + WSL Integration

推荐方式：

1. Windows 安装 Docker Desktop。
2. Docker Desktop 设置中启用 WSL integration。
3. 勾选 `Ubuntu`。
4. 在 Ubuntu 中验证：

```bash
docker --version
docker compose version
docker run --rm hello-world
```

项目文件建议放在 WSL 内，再执行 `docker compose up`，性能通常更好。

### WSL 内独立 Docker Engine

如果不使用 Docker Desktop，也可以在 Ubuntu 内安装 Docker Engine。注意这会让容器运行完全依赖该发行版，网络、开机启动、资源限制和公司安全策略需要单独处理。

## WSL 配置文件

WSL 有两类配置文件：

- `/etc/wsl.conf`：发行版内部配置，只影响当前 Ubuntu。
- `%UserProfile%\.wslconfig`：Windows 用户级全局配置，影响 WSL 2 虚拟机。

修改后通常需要执行：

```powershell
wsl --shutdown
```

再重新进入 Ubuntu。

### `/etc/wsl.conf`

在 Ubuntu 内编辑：

```bash
sudo nano /etc/wsl.conf
```

常用示例：

```ini
[boot]
systemd=true

[user]
default=你的用户名

[interop]
enabled=true
appendWindowsPath=false

[automount]
enabled=true
root=/mnt/
options=metadata,umask=22,fmask=11
mountFsTab=false

[network]
generateHosts=true
generateResolvConf=true
```

说明：

- `systemd=true`：启用 systemd，很多现代服务和工具依赖它。
- `default`：导入发行版后常用，用来恢复默认登录用户。
- `appendWindowsPath=false`：不自动把 Windows PATH 拼到 WSL PATH，减少 `node.exe`、`python.exe` 等 Windows 程序误入 Linux 环境。
- `metadata`：让 `/mnt/c` 等 Windows 挂载目录支持更多 Linux 权限元数据。
- `generateResolvConf=true`：让 WSL 自动生成 DNS 配置。公司网络或代理环境下 DNS 异常时再考虑手动管理。

如果关闭 `appendWindowsPath` 后还需要调用 Windows 程序，可以写完整路径，或只把必要目录加回 PATH。

### `%UserProfile%\.wslconfig`

在 PowerShell 中编辑：

```powershell
notepad $env:UserProfile\.wslconfig
```

基础示例：

```ini
[wsl2]
memory=8GB
processors=4
swap=4GB
localhostForwarding=true
```

说明：

- `memory`：限制 WSL 2 虚拟机最多使用多少内存。
- `processors`：限制 CPU 核心数。
- `swap`：限制 swap 大小。
- `localhostForwarding=true`：NAT 模式下让 Windows 和 WSL 的 localhost 转发更符合直觉，mirrored 模式下该项会被忽略。

Windows 11 22H2 及以上版本，可以按需增加这些网络相关配置：

```ini
[wsl2]
networkingMode=mirrored
dnsTunneling=true
firewall=true
autoProxy=true
```

说明：

- `networkingMode=mirrored`：镜像网络模式，Windows 11 新版 WSL 可用，通常能改善 VPN、IPv6、局域网访问和 Windows/WSL 互访。
- `dnsTunneling=true`：让 DNS 请求通过 Windows 转发，通常比手动维护 `/etc/resolv.conf` 稳定。
- `autoProxy=true`：让 WSL 尝试使用 Windows 代理设置。
- `firewall=true`：让 Windows 防火墙规则应用到 WSL 网络。

如果某个配置项不被当前 WSL 支持，先执行：

```powershell
wsl --update
wsl --version
```

## 网络

### localhost 访问

先区分方向：

- WSL 内启动服务，Windows 浏览器访问：通常使用 `http://localhost:端口`。
- Windows 内启动服务，WSL 访问：NAT 模式下通常使用 Windows 宿主机 IP；mirrored 模式下可以尝试 `http://127.0.0.1:端口`。
- 局域网其他设备访问 WSL 服务：优先使用 mirrored 模式；NAT 模式下通常需要 Windows 端口转发。

如果服务只监听 `127.0.0.1`，可能只能在同一侧访问。开发服务器需要被 Windows 或局域网访问时，通常应监听 `0.0.0.0`，例如：

```bash
npm run dev -- --host 0.0.0.0
python -m http.server 8000 --bind 0.0.0.0
```

### 查看 WSL IP 和 Windows 宿主机 IP

WSL IP：

```bash
hostname -I
ip addr show eth0
```

WSL 里查看 Windows 宿主机地址：

```bash
ip route | awk '/default/ {print $3}'
cat /etc/resolv.conf
```

Windows 侧查看网络：

```powershell
ipconfig
```

### NAT 模式和 mirrored 模式

默认 NAT 模式下：

- WSL 有独立虚拟网卡 IP。
- WSL 访问外网通常没问题。
- Windows 访问 WSL 服务通常可通过 `localhost` 转发。
- WSL 访问 Windows 服务通常用宿主机 IP，不要默认假设 `localhost` 可用。
- 局域网其他机器访问 WSL 服务时，通常需要 Windows 端口转发和防火墙规则。

mirrored 模式下：

- WSL 更像共享 Windows 的网络环境。
- WSL 访问 Windows 服务可以使用 `127.0.0.1`，但 IPv6 localhost `::1` 不适用。
- VPN、局域网、IPv6、多播场景通常更顺。
- 需要 Windows 11 22H2 及以上版本，并建议先把 WSL 更新到较新版本。

启用 mirrored 后：

```ini
[wsl2]
networkingMode=mirrored
localhostForwarding=true
dnsTunneling=true
firewall=true
```

然后：

```powershell
wsl --shutdown
```

### 局域网访问 WSL 服务

如果局域网其他设备需要访问 WSL 内服务，优先尝试 mirrored 网络模式。

如果只能使用 NAT 模式，可以在 Windows 侧做端口转发。示例：把 Windows 的 `0.0.0.0:3000` 转发到 WSL 的 `172.x.x.x:3000`：

```powershell
$wslIp = (wsl hostname -I).Trim().Split()[0]
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=3000 connectaddress=$wslIp connectport=3000
New-NetFirewallRule -DisplayName "WSL 3000" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 3000
```

查看和删除转发：

```powershell
netsh interface portproxy show all
netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=3000
```

WSL IP 会变化，NAT 转发可能需要重建。

### DNS 和代理

DNS 异常时先看：

```bash
cat /etc/resolv.conf
nslookup github.com
```

如果公司网络要求代理，可以临时设置：

```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export all_proxy=socks5://127.0.0.1:7890
```

需要长期使用时放进 `~/.bashrc`，或使用 Windows 侧 `.wslconfig` 的 `autoProxy=true`。

如果代理运行在 Windows 上，而 WSL 内 `127.0.0.1` 访问不到，可以改用宿主机 IP：

```bash
export WIN_HOST=$(ip route | awk '/default/ {print $3}')
export http_proxy=http://$WIN_HOST:7890
export https_proxy=http://$WIN_HOST:7890
```

## 环境变量

### PATH 管理

检查 PATH：

```bash
echo "$PATH" | tr ':' '\n'
```

如果看到大量 Windows 路径，例如 `/mnt/c/Windows/...`、`/mnt/c/Program Files/...`，可能会导致 WSL 调到 Windows 侧的 `node.exe`、`python.exe`、`git.exe`。

推荐在 `/etc/wsl.conf` 中关闭自动追加 Windows PATH：

```ini
[interop]
enabled=true
appendWindowsPath=false
```

然后按需在 `~/.bashrc` 中只添加必要工具：

```bash
export PATH="$HOME/.local/bin:$HOME/bin:$PATH"
```

### WSLENV

`WSLENV` 用来控制 Windows 和 WSL 之间哪些环境变量可以互相传递，以及路径是否自动转换。

PowerShell 示例：

```powershell
$env:WSLENV="PROJECT_ROOT/p:HTTP_PROXY:HTTPS_PROXY"
$env:PROJECT_ROOT="C:\Users\你的用户名\project"
wsl -- bash -lc 'echo "$PROJECT_ROOT"'
```

常用标记：

- `/p`：路径在 Windows 和 WSL 之间转换。
- `/l`：变量是路径列表。
- `/u`：只从 Windows 传到 WSL。
- `/w`：只从 WSL 传到 Windows。

不确定时不要把敏感变量放入 `WSLENV`，例如 token、私钥路径、生产数据库连接串。

### Windows 和 WSL 命令互调

WSL 调 Windows 程序：

```bash
/mnt/c/Windows/System32/notepad.exe README.md
/mnt/c/Windows/explorer.exe .
/mnt/c/Windows/System32/cmd.exe /c ver
/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -NoProfile -Command '$PSVersionTable.PSVersion'
```

如果保留了 Windows PATH 自动追加，也可以直接使用 `notepad.exe`、`explorer.exe`、`cmd.exe`。

Windows 调 WSL 命令：

```powershell
wsl -d Ubuntu -- pwd
wsl -d Ubuntu -- bash -lc "cd ~/code && ls"
```

路径转换：

```bash
wslpath -w ~/code
wslpath -u 'C:\Users\你的用户名'
```

## 备份、恢复和克隆环境

### tar 备份

```powershell
wsl --shutdown
New-Item -ItemType Directory -Force D:\WSL\backup
$backup = "D:\WSL\backup\ubuntu-$(Get-Date -Format yyyyMMdd).tar"
wsl --export Ubuntu $backup
Get-Item $backup
```

从 tar 恢复为新发行版：

```powershell
New-Item -ItemType Directory -Force D:\WSL\Ubuntu-Restore
wsl --import Ubuntu-Restore D:\WSL\Ubuntu-Restore D:\WSL\backup\ubuntu-YYYYMMDD.tar --version 2
wsl -d Ubuntu-Restore
```

这里的 `ubuntu-YYYYMMDD.tar` 需要替换成实际导出的备份文件名。

### VHDX 备份

WSL 2 也可以导出为 VHDX。大环境迁移时 VHDX 更接近原始虚拟磁盘形态：

```powershell
wsl --shutdown
wsl --export Ubuntu D:\WSL\backup\ubuntu.vhdx --vhd
wsl --import Ubuntu-Restore D:\WSL\Ubuntu-Restore D:\WSL\backup\ubuntu.vhdx --vhd --version 2
```

如果手上已经有一个 ext4 格式的 `.vhdx`，也可以直接登记：

```powershell
wsl --import-in-place Ubuntu-Restore D:\WSL\Ubuntu-Restore\ext4.vhdx
```

### 克隆临时实验环境

```powershell
wsl --shutdown
wsl --export Ubuntu D:\WSL\backup\ubuntu-base.tar
New-Item -ItemType Directory -Force D:\WSL\Ubuntu-Test
wsl --import Ubuntu-Test D:\WSL\Ubuntu-Test D:\WSL\backup\ubuntu-base.tar --version 2
```

删除实验副本前，确认要删的是临时环境：

```powershell
wsl -l -v
wsl --unregister Ubuntu-Test
```

`wsl --unregister` 会永久删除该发行版的数据、设置和已安装软件。只在确认备份可用或发行版确实不要时执行。

## 常见问题

### 忘记 Ubuntu 密码

PowerShell 中以 root 进入：

```powershell
wsl -d Ubuntu -u root
```

重置用户密码：

```bash
passwd 你的用户名
```

### `code .` 打不开 VS Code

先确认 Windows 已安装 VS Code 和 WSL 扩展。然后：

```powershell
wsl --shutdown
```

重新进入 Ubuntu 后再执行：

```bash
code .
```

### Git 提交出现大量换行变化

检查：

```bash
git config --global core.autocrlf
git config --global core.eol
git diff --check
```

建议：

```bash
git config --global core.autocrlf input
git config --global core.eol lf
```

项目内再用 `.gitattributes` 固定换行。

### WSL 里误用了 Windows 的 node 或 python

检查：

```bash
which node
which python
which git
```

如果输出在 `/mnt/c/...` 下，优先关闭 `appendWindowsPath`，再安装 WSL 内对应工具。

### apt 或 git 网络很慢

排查顺序：

```bash
ping -c 4 github.com
nslookup github.com
curl -I https://github.com
env | grep -i proxy
```

可能原因：

- DNS 配置异常。
- Windows 代理没有被 WSL 使用。
- VPN 不支持默认 NAT 模式，尝试 mirrored 网络。
- 公司网络需要证书或代理。

### 释放 WSL 占用资源

```powershell
wsl --shutdown
```

如需长期限制资源，在 `%UserProfile%\.wslconfig` 中设置 `memory`、`processors`、`swap`。

## 参考资料

- [Microsoft Learn: Install WSL](https://learn.microsoft.com/windows/wsl/install)
- [Microsoft Learn: Basic commands for WSL](https://learn.microsoft.com/windows/wsl/basic-commands)
- [Microsoft Learn: Advanced settings configuration in WSL](https://learn.microsoft.com/windows/wsl/wsl-config)
- [Microsoft Learn: Accessing network applications with WSL](https://learn.microsoft.com/windows/wsl/networking)
- [Microsoft Learn: Set up a WSL development environment](https://learn.microsoft.com/windows/wsl/setup/environment)
- [Microsoft Learn: Get started using Git on WSL](https://learn.microsoft.com/windows/wsl/tutorials/wsl-git)
- [Git Credential Manager: WSL support](https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/wsl.md)
