# Dotfile Repo

## Desc
This repo traces dotfiles of Linux `$home`, to async between different devices.

> Repo dont includes any **sensitive information** (like `sk-xxx`, `ssh`, `AK` etc.). All sensitive informations are stored in `*.local`, which needed async by hand.

## Init
```bash
# 1. 创建 bare repository
git init --bare $HOME/.dotfiles

# 2. 创建别名方便操作
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'

# 3. 配置不显示未跟踪文件（避免显示整个主目录的文件）
dotfiles config --local status.showUntrackedFiles no

# 4. 将别名写入 .bashrc（永久生效）
echo "alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'" >> $HOME/.bashrc

# 5. 添加并提交你的 dotfiles
dotfiles add .bashrc .config/nvim .config/i3 .gitconfig  # 按需添加
dotfiles commit -m "Initial dotfiles commit"

# 6. 关联远程仓库（先到 GitHub 创建空仓库）
dotfiles remote add origin git@github.com:你的用户名/dotfiles.git
dotfiles push -u origin main
```

## Usage
```bash
# 查看状态
dotfiles status

# 添加新文件
dotfiles add .config/新配置

# 提交更改
dotfiles commit -m "更新某配置"

# 推送到远程
dotfiles push

# 拉取远程更改
dotfiles pull
```

## Restore In New Device
```bash
# 1. 克隆 bare repository
git clone --bare git@github.com:你的用户名/dotfiles.git $HOME/.dotfiles

# 2. 定义别名
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'

# 3. 检出文件（注意：会覆盖现有文件！）
dotfiles checkout

# 4. 如果第3步有冲突，备份冲突文件后重试
mkdir -p $HOME/.config-backup
dotfiles checkout 2>&1 | grep -E "\s+\." | awk '{print $1}' | xargs -I{} mv {} $HOME/.config-backup/{}
dotfiles checkout

# 5. 设置不显示未跟踪文件
dotfiles config --local status.showUntrackedFiles no
```

## Package Sync (`pkg-manifest`)

`~/.local/bin/pkg-manifest` 把各包管理器安装的软件导出成清单，清单随本仓库同步，
换机后一条命令还原工具链。支持 **apt / cargo / npm / flatpak**。

```bash
pkg-manifest export           # 导出本机清单 -> ~/.config/pkg/manifest/*.txt
pkg-manifest list             # 查看本机已安装（可用 `list cargo` 只看一类）
pkg-manifest diff             # 对比清单与本机：缺什么、多什么（有缺失项时返回码 1）
pkg-manifest apply            # 按清单安装缺失的包（apt 会先 apt-get update）
pkg-manifest apply --dry-run  # 只打印将执行的命令，不实际安装
```

别名：`save`=export、`install`=apply、`status`=diff；不指定管理器则处理全部。

### 清单文件

| 文件 | 内容 |
| --- | --- |
| `~/.config/pkg/manifest/apt.txt` | `apt-mark showmanual`，已剔除发行版自带的 required/important 基础包（`--apt-all` 可保留） |
| `~/.config/pkg/manifest/cargo.txt` | `cargo install --list` 的顶层 crate（不含依赖） |
| `~/.config/pkg/manifest/npm.txt` | `npm -g` 全局包（排除 node 自带的 npm/corepack） |
| `~/.config/pkg/manifest/flatpak.txt` | `应用ID<TAB>来源仓库`，还原时按记录的 remote 安装 |

### 只在本机安装的包

在同目录建 `<mgr>.local`（已在 `.gitignore` 中，永远不会提交），每行一个包名，支持 `#` 注释：

```bash
# ~/.config/pkg/manifest/apt.local
amd64-microcode        # 硬件相关，别的机器不需要
firmware-nvidia-graphics
```

导出与比较时都会跳过这些包，避免把显卡固件、微码之类装到不匹配的机器上。

### 换新机流程

```bash
dotfiles checkout          # 取回 dotfiles（含清单与脚本）
pkg-manifest diff          # 看缺哪些
pkg-manifest apply         # 装回来
```

注意事项：

- `rustup`、`node`(fnm)、`flatpak` 本体等属于基础工具，需先自行安装，脚本才能接管各自的包。
- `cargo install` 是本地编译，慢且依赖系统库（如 `libssl-dev`、`pkg-config`），apt 部分建议先跑。
- npm 全局包装在 node 版本目录下，换机后 node 大版本变了要重新 `pkg-manifest apply npm`。
- 装完新软件后：`pkg-manifest export && dotfiles add .config/pkg .local/bin && dotfiles commit -m "packages"`。

## 追踪范围

- **追踪**：shell / 终端 / 编辑器 / CLI 工具配置、KDE 应用偏好与主题、技能目录，以及上面的软件清单。
- **不追踪**（见 `$HOME/.gitignore`）：密钥凭据（`.ssh/`、`.bashrc.local`、`.config/opencode/`、`.local/share/keyrings/` …）
  与仅本机状态（`kwinrc` 屏幕布局、`kxkbrc` 键盘布局、`ktimezonedrc`、历史记录与缓存 …）。
- `~/.local/bin` 只跟踪 `pkg-manifest`，其余（uv/pip 生成的软链等）忽略；新增要同步的脚本需在
  `.gitignore` 中加一行 `!.local/bin/<脚本名>`。

## System Tools

### cargo

一般情况下直接 `pkg-manifest apply cargo` 即可；全新环境可先手动装一批常用工具：

```bash
cargo install bat btm duf eza fd-find mwget ripgrep uv zoxide
```

### tmux
Install `tmux plugin manager` to manage plugins.
```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

[tmux prefix] + :
source-file ~/.tmux.conf

[tmux prefix] + I
```
