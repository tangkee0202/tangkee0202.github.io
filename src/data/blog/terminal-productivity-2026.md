---
title: "Terminal 效率：改变我工作流的现代 CLI 工具"
description: "认识一批更快、更智能、开发体验更好的现代 CLI 工具，它们可以替代经典 Unix 工具。"
pubDatetime: 2026-01-18T10:00:00Z
tags:
  - terminal
  - productivity
  - linux
  - cli
  - tools
draft: false
---

CLI 生态经历了一场安静的革命。由 Rust 和 Go 编写的新工具正在替代已有几十年历史的 Unix 程序，在几乎不牺牲速度的同时，加入颜色、Syntax Highlighting、Fuzzy Search 和 Git 感知能力。下面是我日常使用的一些工具。

## Table of contents

## Shell：Zsh + Starship

[Starship](https://starship.rs) 是一种投入很少、却能明显改善体验的命令提示符。它支持多种 Shell，运行速度非常快，并能显示 Git 分支、Node/Python/Rust 版本和上一条命令状态等相关上下文。

```toml file=~/.config/starship.toml
# 极简但信息充足的样式
format = """
$directory\
$git_branch\
$git_status\
$nodejs\
$rust\
$python\
$cmd_duration\
$line_break\
$character"""

[git_branch]
symbol = " "
style = "bold purple"

[git_status]
conflicted = "⚔️ "
ahead = "⇡${count}"
behind = "⇣${count}"
modified = "✎${count}"
untracked = "?${count}"

[cmd_duration]
min_time = 2_000
format = "耗时 [$duration](bold yellow)"
```

## 经典工具的现代替代品

### `ls` → `eza`（原名 `exa`）

```bash
eza --tree --level=2 --icons --git    # 显示图标和 Git 状态的目录树
eza -la --sort=modified               # 详细列表，按修改时间排序
```

### `find` → `fd`

```bash
# find：命令较长，可读性一般
find . -name "*.ts" -not -path "*/node_modules/*"    # [!code --]

# fd：更直观，并且默认遵守 .gitignore
fd -e ts                    # 查找项目中的所有 .ts 文件   # [!code ++]
fd -e ts --exec bat {}      # 使用 bat 打开每个结果       # [!code ++]
```

### `grep` → `ripgrep`（`rg`）

```bash
# 经典 grep
grep -r "useEffect" src/ --include="*.tsx"      # [!code --]

# rg：速度更快，并且遵守 .gitignore
rg "useEffect" --type ts                         # [!code ++]
rg "TODO|FIXME|HACK" --type ts --stats           # [!code ++]
rg "deprecated" -l                               # 只显示文件名 # [!code ++]
```

### `cat` → `bat`

`bat` 在 `cat` 的基础上增加了语法高亮、行号、分页和内置 Git 差异显示：

```bash
bat src/components/Header.astro     # 带颜色和行号
bat --diff file.ts                  # 显示 Git 行内修改
```

### `cd` → `zoxide`

它会学习你经常访问的目录，之后只需输入几个字母就能跳转：

```bash
z astro      # 跳到最常访问的 ~/projects/my-astro-blog
z blog src   # 使用多个关键词匹配
zi           # 结合 fzf 的交互模式
```

## 终端复用器：现代配置的 `tmux`

```bash file=~/.tmux.conf
# 使用更顺手的前缀键
set -g prefix C-a
unbind C-b

# 使用直观按键拆分窗格
bind | split-window -h -c "#{pane_current_path}"  # [!code highlight]
bind - split-window -v -c "#{pane_current_path}"  # [!code highlight]

# Alt + 方向键切换窗格，无需前缀
bind -n M-Left  select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up    select-pane -U
bind -n M-Down  select-pane -D

# 启用鼠标
set -g mouse on

# 256 色
set -g default-terminal "tmux-256color"
```

## 模糊查找器：`fzf`，让其他工具能力倍增

`fzf` 可以把任何列表变成交互式查找器，只需把命令结果通过 `| fzf` 传给它。

```bash
# 在命令历史中搜索
使用集成 fzf 的 CTRL+R

# 预览并切换 Git 分支
git branch | fzf --preview 'git log --oneline {}' | xargs git checkout

# 选择并终止进程
ps aux | fzf --multi | awk '{print $2}' | xargs kill

# 查找并打开文件
fd -e ts | fzf --preview 'bat --color=always {}' | xargs nvim
```

## 现代 Git 界面：`lazygit`

这是一个 Git 终端界面，可以直观显示仓库中正在发生什么：

```bash
lazygit   # 打开界面
```

它的主要功能包括：

- 按文件或按行查看差异
- 选择性暂存，甚至只暂存单独几行
- 可视化解决冲突
- 通过交互方式执行 rebase

## 我的基础 `.zshrc` 配置

```bash file=~/.zshrc
# 使用延迟加载保持启动速度
export PATH="$HOME/.cargo/bin:$HOME/.local/bin:$PATH"

# 现代命令别名
alias ls='eza --icons'
alias ll='eza -la --icons --git'
alias tree='eza --tree --icons'
alias cat='bat'
alias find='fd'
alias grep='rg'
alias lg='lazygit'

# fzf 集成
source <(fzf --zsh)

# zoxide
eval "$(zoxide init zsh)"

# starship
eval "$(starship init zsh)"
```

> 提高终端效率最值得的投入，并不是不断学习新工具，而是熟练掌握已经在用的工具。但当现代工具能以更好的体验和数倍速度完成同一件事时，切换成本通常一周内就能收回。
