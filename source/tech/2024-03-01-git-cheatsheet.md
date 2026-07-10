---
title: Git 常用命令速查
date: 2024-03-01 09:00:00
updated: 2024-03-15 16:45:00
notebook: tech-notes
menu_id: tech
tags: [Git, 工具]
excerpt: 日常开发中最常用的 Git 命令与常见场景速查
---

## 基础操作

```bash
# 初始化仓库
git init

# 克隆远程仓库
git clone <repo-url>

# 查看状态
git status

# 查看提交历史
git log --oneline --graph
```

## 分支管理

```bash
# 列出所有分支
git branch -a

# 创建并切换分支
git checkout -b feature/x

# 切换分支
git switch main

# 合并分支
git merge feature/x

# 删除已合并的本地分支
git branch -d feature/x
```

## 远程同步

```bash
# 拉取并 rebase（推荐，避免合并节点）
git pull --rebase origin main

# 推送当前分支并建立追踪
git push -u origin feature/x
```

## 常见场景

### 场景 1：撤销最近一次未 push 的提交

```bash
git reset --soft HEAD~1   # 保留改动在暂存区
git reset --mixed HEAD~1  # 保留改动在工作区（默认）
```

### 场景 2：临时切换分支

```bash
git stash             # 暂存当前改动
git switch other      # 切去做别的事
git switch back
git stash pop         # 恢复改动
```

### 场景 3：把某次提交搬到当前分支

```bash
git cherry-pick <commit-hash>
```
