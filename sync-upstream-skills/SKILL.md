---
name: sync-upstream-skills
description: Use when syncing this skills repository with upstream open-source repos (superpowers, obsidian-skills, Humanizer-zh) to pull the latest community updates via git subrepo
---

# Sync Upstream Skills

## Overview

此仓库（`my-skills`）通过 `git subrepo` 跟踪三个上游开源 skills 仓库。本 skill 用于将上游最新变更 pull 到本地各子目录，并处理与本地私有修改的合并。

## 前提：确认 git-subrepo 已安装

```bash
which git-subrepo || brew install git-subrepo
```

## 管理的上游仓库

| 目录 | 上游仓库 | skills 位置 |
|------|----------|-------------|
| `superpowers` | `git@github.com:obra/superpowers.git` | `superpowers/skills/` |
| `obsidian-skills` | `git@github.com:kepano/obsidian-skills.git` | `obsidian-skills/skills/` |
| `Humanizer-zh` | `git@github.com:op7418/Humanizer-zh.git` | 根目录 `SKILL.md` |

## 同步流程

### 同步所有上游

```bash
cd ~/workspace/dotfiles/skills
git subrepo pull superpowers
git subrepo pull obsidian-skills
git subrepo pull Humanizer-zh
```

### 只同步某一个

```bash
git subrepo pull superpowers
```

## 冲突处理

`git subrepo pull` 内部执行 merge。若出现冲突：

1. 查看冲突文件：`git diff --name-only --diff-filter=U`
2. 逐个读取冲突文件，理解两侧改动意图：
   - 上游改动（社区更新）
   - 本地改动（私有定制）
3. 语义合并：保留本地私有定制，同时纳入上游改进
4. 标记解决：`git add <file>`
5. 完成合并：`git commit`

## 同步后操作

只推送到私有远端，**不要** push 回上游：

```bash
git push origin main
```

如需将本地改进贡献回上游，请整理为干净的 PR 提交，不要直接 `git subrepo push`。

## 常见问题

| 问题 | 解决方案 |
|------|----------|
| `git: 'subrepo' is not a git command` | `brew install git-subrepo` |
| pull 后出现 merge conflict | 按上方冲突处理流程操作 |
| 某个 subrepo 上游分支变了 | 编辑 `.gitrepo` 中的 `branch` 字段后重新 pull |
