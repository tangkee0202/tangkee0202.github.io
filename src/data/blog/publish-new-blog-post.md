---
title: "如何在 Astro 博客中发布一篇新文章"
description: "记录从创建 Markdown 文件、本地预览到使用 GitHub Actions 自动发布博客的完整流程。"
pubDatetime: 2026-09-23T21:40:00+08:00
featured: true
tags:
  - astro
  - 博客
  - github-actions
draft: false
---

这个博客使用 Astro 构建，文章以 Markdown 文件保存在项目中。写完文章并推送到 GitHub 后，GitHub Actions 会自动构建和发布网站。

## Table of contents

## 找到文章目录

博客文章统一保存在：

```text
src/data/blog
```

在 VS Code 左侧依次展开 `src`、`data` 和 `blog`。右键单击 `blog` 文件夹，选择“新建文件”。

文件名建议使用小写英文字母、数字和短横线，例如：

```text
robot-learning-notes-01.md
```

文件名会成为文章网址的一部分，因此不要使用空格，尽量让它简短且容易理解。

## 添加文章信息

每篇文章开头都需要一段 Frontmatter，用来设置标题、描述、发布时间和标签：

```markdown
---
title: "我的第一篇机器人学习记录"
description: "记录今天在机器人学习过程中掌握的知识。"
pubDatetime: 2026-09-23T21:30:00+08:00
tags:
  - 机器人
  - 学习记录
draft: false
---
```

这些字段的作用如下：

- `title`：文章标题。
- `description`：文章摘要，会显示在文章列表和搜索结果中。
- `pubDatetime`：发布时间，`+08:00` 表示中国标准时间。
- `tags`：文章标签，可以填写多个。
- `draft`：设为 `false` 时公开发布；设为 `true` 时作为草稿隐藏。

## 编写正文

Frontmatter 下方就是文章正文，可以使用 Markdown 语法：

```markdown
今天开始记录我的机器人学习过程。

## 今天学到了什么

在这里填写学习内容。

## 遇到的问题

记录问题以及解决方法。

## 总结

写下本次学习的收获和下一步计划。
```

使用 `##` 创建二级标题，使用 `-` 创建列表，使用成对的反引号标记命令或代码。如果文章需要目录，可以在正文前面加入：

```markdown
## Table of contents
```

Astro 会根据文章标题自动生成目录。

## 在本地预览

保存文章后，在项目终端运行：

```powershell
pnpm dev
```

然后打开：

```text
http://localhost:4321
```

进入 Posts 页面检查文章是否显示，并确认标题、段落、代码和标签的排版。预览服务器运行期间，保存文件后页面通常会自动更新。

如果需要停止预览，在终端中按 `Ctrl + C`。

## 提交并发布文章

确认内容没有问题后，将新文章提交到 Git：

```powershell
git status
git add src/data/blog/robot-learning-notes-01.md
git commit -m "Add first robotics learning post"
git push origin main
```

推送完成后，GitHub Actions 会自动运行 `Deploy to GitHub Pages` 工作流。可以在 GitHub 仓库顶部的 Actions 页面查看进度。

当工作流出现绿色对勾后，等待一两分钟并访问：

```text
https://tangkee0202.github.io
```

如果浏览器仍显示旧内容，可以按 `Ctrl + F5` 强制刷新。

## 修改已经发布的文章

直接打开原来的 Markdown 文件进行修改，保存后再次执行：

```powershell
git add src/data/blog/文章文件名.md
git commit -m "Update blog post"
git push origin main
```

每次推送到 `main` 分支后，网站都会自动重新部署，因此不需要在 GitHub Pages 中重复配置。
