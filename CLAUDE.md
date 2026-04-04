# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 仓库性质

这是一个私有 skills 聚合仓库（`my-skills`），通过 `git subrepo` 跟踪三个上游开源 skills 仓库，并叠加本地私有定制。它本身不包含业务代码。

## 同步上游

使用内置 skill：直接说"同步上游 skills"，会调用 `.claude/skills/sync-upstream-skills`。

手动执行：

```bash
git subrepo pull superpowers       # 或 Humanizer-zh / obsidian-skills
```

`git-subrepo` 未安装时：`brew install git-subrepo`

## 上游仓库

| 目录 | 上游 | skills 路径 |
|------|------|-------------|
| `superpowers/` | `git@github.com:obra/superpowers.git` | `superpowers/skills/` |
| `obsidian-skills/` | `git@github.com:kepano/obsidian-skills.git` | `obsidian-skills/skills/` |
| `Humanizer-zh/` | `git@github.com:op7418/Humanizer-zh.git` | 根目录 `SKILL.md` |

各目录的 `.gitrepo` 文件记录同步状态（commit、branch、parent），由 `git subrepo` 自动维护，**不要手动编辑**。

## 私有 skills

私有 skills 放在 `.claude/skills/`，不在任何 subrepo 目录内：

```
.claude/skills/
└── sync-upstream-skills/SKILL.md
```

## 贡献回上游

不要 `git subrepo push`。如需贡献，整理为干净的 PR 提交到对应的上游仓库。贡献 `superpowers` 前必须阅读 `superpowers/CLAUDE.md`——该仓库 PR 拒绝率 94%，有严格的提交要求。
