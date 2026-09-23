---
title: "如何使用 Git 将本地项目上传到 GitHub"
description: "从检查修改、创建提交到推送远程仓库，记录一套适合初学者的 GitHub 上传流程。"
pubDatetime: 2026-09-23T21:30:00+08:00
tags:
  - git
  - github
  - 工具学习
draft: false
---

Git 是一个版本管理工具，GitHub 则可以用来在线保存 Git 仓库。平时修改完项目后，我们通常需要依次完成三个动作：暂存修改、创建提交、推送到 GitHub。

## Table of contents

## 开始之前

首先确认电脑已经安装 Git：

```powershell
git --version
```

如果能够看到版本号，就说明 Git 已经可以正常使用。然后在 VS Code 中打开项目文件夹，并通过菜单“终端 → 新建终端”打开终端。

## 检查当前修改

运行下面的命令查看哪些文件发生了变化：

```powershell
git status
```

常见状态包括：

- `modified`：文件已经被修改。
- `untracked`：这是一个尚未被 Git 管理的新文件。
- `nothing to commit`：当前没有需要提交的修改。

提交之前最好先检查一次，避免误上传不需要的文件、密码或其他隐私信息。

## 将文件加入暂存区

如果只想添加一个文件，可以指定文件路径：

```powershell
git add src/data/blog/my-post.md
```

如果确认当前目录中的所有修改都需要提交，可以运行：

```powershell
git add .
```

再次运行 `git status`，可以看到即将包含在提交中的文件。

## 创建一次提交

使用下面的命令保存这次修改记录：

```powershell
git commit -m "Add a new blog post"
```

引号中的内容是提交说明，应该简短描述这次做了什么。例如：

```powershell
git commit -m "Update About page"
```

## 推送到 GitHub

当前项目使用 `main` 分支，因此运行：

```powershell
git push origin main
```

其中：

- `origin` 是 GitHub 远程仓库的默认名称。
- `main` 是要推送的分支名称。

第一次推送时，Git 可能会要求登录 GitHub。按照浏览器中的提示完成登录和授权即可，不要将密码或访问令牌写入项目文件。

## 确认是否推送成功

成功后，终端通常会显示类似下面的结果：

```text
main -> main
```

还可以再次运行：

```powershell
git status
```

如果看到分支已经与 `origin/main` 保持一致，说明本地提交已经上传。最后刷新 GitHub 仓库页面，检查最新提交和文件是否出现。

## 常用完整流程

以后每次修改项目，都可以按照下面的顺序操作：

```powershell
git status
git add .
git commit -m "Describe what changed"
git push origin main
```

养成先检查再提交的习惯，可以让项目历史更清晰，也能降低误上传文件的风险。
