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

## System Tools

### cargo
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
