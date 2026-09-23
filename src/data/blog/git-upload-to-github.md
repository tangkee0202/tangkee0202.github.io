---
title: "从创建本地仓库到上传 GitHub：Git 完整入门"
description: "面向初学者，详细记录 Git 身份配置、本地仓库初始化、连接 GitHub、首次推送和日常更新的完整流程。"
pubDatetime: 2026-09-23T21:30:00+08:00
featured: true
tags:
  - git
  - github
  - 工具学习
draft: false
---

Git 是一个版本管理工具，可以记录项目每一次修改；GitHub 则可以在线保存 Git 仓库，方便备份、分享和协作。本文从一个普通的本地文件夹开始，完整演示如何创建 Git 仓库并上传到 GitHub。

## Table of contents

## 一、检查 Git 是否安装成功

在 VS Code 中通过“终端 → 新建终端”打开 PowerShell，然后运行：

```powershell
git --version
```

如果能够看到类似 `git version 2.x.x` 的内容，说明 Git 已经可以使用。如果提示找不到 `git` 命令，需要先安装 Git，并在安装完成后重新启动 VS Code。

## 二、配置 Git 用户信息

第一次使用 Git，需要设置提交记录中显示的名字和邮箱：

```powershell
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

例如：

```powershell
git config --global user.name "Tangkee"
git config --global user.email "tangkee0202@users.noreply.github.com"
```

这里可以使用 GitHub 提供的隐私邮箱，避免公开真实邮箱。配置完成后检查结果：

```powershell
git config --global user.name
git config --global user.email
```

`--global` 表示该配置适用于这台电脑上的所有 Git 项目。如果某个项目需要使用不同身份，可以进入项目目录后去掉 `--global` 再进行设置。

## 三、准备本地项目文件夹

可以使用已有项目，也可以新建一个文件夹。例如在 PowerShell 中创建项目：

```powershell
New-Item -ItemType Directory my-project
Set-Location my-project
```

如果项目已经存在，需要先进入对应目录：

```powershell
Set-Location "C:\Users\你的用户名\Documents\my-project"
```

在 VS Code 中，也可以通过“文件 → 打开文件夹”打开项目，然后新建终端。运行下面的命令确认当前位置：

```powershell
Get-Location
```

之后的 Git 命令都应该在项目根目录中执行。

## 四、创建本地 Git 仓库

在项目根目录中运行：

```powershell
git init
```

Git 会创建一个隐藏的 `.git` 目录，用来保存提交历史和仓库配置。不要手动修改或删除这个目录。

将默认分支名称统一改成 `main`：

```powershell
git branch -M main
```

检查仓库状态：

```powershell
git status
```

如果能够看到 `On branch main`，说明本地仓库已经创建成功。

## 五、创建 `.gitignore`

并非所有文件都应该上传到 GitHub。例如依赖包、构建结果和环境变量通常不需要提交。

在项目根目录中新建 `.gitignore` 文件。对于 Node.js 或 Astro 项目，可以加入：

```gitignore
node_modules/
dist/
.env
.env.*
!.env.example
```

其中 `.env` 可能包含邮箱、密钥或其他个人配置，通常不应公开。提交前一定要运行 `git status`，检查是否有隐私文件被意外加入。

## 六、创建第一次本地提交

先查看当前文件：

```powershell
git status
```

将需要保存的文件加入暂存区：

```powershell
git add .
```

这里的 `.` 表示当前项目中的所有变更。如果只想添加指定文件，也可以使用：

```powershell
git add README.md
```

再次检查暂存内容：

```powershell
git status
```

确认无误后创建第一次提交：

```powershell
git commit -m "Initial commit"
```

提交可以理解为项目在某个时间点的快照。引号中的文字是提交说明，应当简短描述这次修改。

## 七、在 GitHub 创建远程仓库

登录 GitHub 后，点击右上角的 `+`，选择 **New repository**：

1. 在 **Repository name** 中填写仓库名称。
2. 根据需要选择 **Public** 或 **Private**。
3. 如果本地项目已经存在，不要勾选添加 README、`.gitignore` 或 License，保持远程仓库为空可以减少首次推送冲突。
4. 点击 **Create repository**。

创建完成后，复制仓库的 HTTPS 地址，例如：

```text
https://github.com/username/my-project.git
```

`username` 和 `my-project` 需要替换为自己的 GitHub 用户名和仓库名。

## 八、连接本地仓库与 GitHub

回到 VS Code 终端，添加名为 `origin` 的远程仓库：

```powershell
git remote add origin https://github.com/username/my-project.git
```

检查远程地址：

```powershell
git remote -v
```

如果 fetch 和 push 后面显示的地址正确，就表示连接成功。

## 九、第一次推送到 GitHub

运行：

```powershell
git push -u origin main
```

参数含义如下：

- `origin`：远程仓库的默认名称。
- `main`：要推送的分支。
- `-u`：让本地 `main` 记住对应的远程分支，以后可以直接运行 `git push`。

Windows 第一次推送时，Git 可能会询问凭据管理方式。可以选择 `manager` 并允许浏览器打开 GitHub 登录页面，然后按照提示登录和授权。不要把密码或访问令牌写入项目文件。

推送成功后，终端通常会显示：

```text
main -> main
```

刷新 GitHub 仓库页面，就能看到本地文件和第一次提交。

## 十、另一种方式：克隆已有的 GitHub 仓库

如果 GitHub 仓库中已经存在模板、README 或其他文件，不要在另一个文件夹中重复运行 `git init`。更简单的方式是直接克隆：

```powershell
git clone https://github.com/username/my-project.git
Set-Location my-project
```

克隆会自动下载文件和历史记录、创建本地仓库、添加 `origin`，并建立本地分支与远程分支的关联。

之后只需要在克隆得到的文件夹中修改项目并正常提交即可。这也是使用 GitHub 模板创建项目后最推荐的操作方式。

## 十一、日常更新和上传流程

完成第一次推送后，每次修改项目都可以按照以下顺序操作。

### 1. 查看改动

```powershell
git status
```

常见状态包括：

- `modified`：已经修改的文件。
- `untracked`：尚未被 Git 管理的新文件。
- `nothing to commit`：当前没有可以提交的修改。

### 2. 暂存改动

添加全部改动：

```powershell
git add .
```

或者只添加指定文件：

```powershell
git add src/data/blog/my-post.md
```

### 3. 创建提交

```powershell
git commit -m "Add a new blog post"
```

### 4. 推送到 GitHub

首次使用过 `-u` 后，可以直接运行：

```powershell
git push
```

也可以完整写成：

```powershell
git push origin main
```

## 十二、上传之前同步远程修改

如果同一个仓库可能在另一台电脑上修改，或者有其他协作者，开始工作前可以先运行：

```powershell
git pull origin main
```

它会将 GitHub 上的最新提交同步到本地。同步之后再编辑、提交并推送，可以降低冲突概率。

## 十三、常见问题

### `fatal: not a git repository`

当前终端不在 Git 仓库中。先使用 `Set-Location` 进入正确的项目目录；如果这是一个全新项目，则运行 `git init`。

### `remote origin already exists`

项目已经设置过 `origin`。先运行 `git remote -v` 检查。如果地址错误，可以更新它：

```powershell
git remote set-url origin https://github.com/username/my-project.git
```

### 推送被拒绝：`non-fast-forward`

这通常表示 GitHub 上存在本地没有的提交。先同步远程内容：

```powershell
git pull --rebase origin main
git push origin main
```

如果出现冲突，不要强制推送。应先检查并解决冲突，再继续提交。

### 不小心暂存了 `.env`

在提交之前，将它移出暂存区：

```powershell
git restore --staged .env
```

然后确认 `.gitignore` 中包含 `.env`。如果敏感信息已经推送到 GitHub，仅仅删除文件并不代表信息已经安全，应立即更换相关密码或密钥。

### `nothing to commit, working tree clean`

这不是错误，它表示当前文件与最近一次提交完全一致，没有新的修改需要提交。

## 十四、完整命令速查

全新本地项目第一次上传：

```powershell
git init
git branch -M main
git status
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/username/my-project.git
git push -u origin main
```

以后日常更新：

```powershell
git status
git add .
git commit -m "Describe what changed"
git push
```

Git 最重要的习惯不是记住所有命令，而是每次提交前都认真查看 `git status`，确保只上传真正需要的文件，并为每次提交写清晰的说明。
