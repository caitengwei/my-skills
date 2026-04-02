# my-skills

`my-skills` 是一个私有 skills 聚合仓库。

它的职责有三件事：

- 聚合多个开源 skills 仓库
- 承载我的私有修改
- 作为 `dotfiles` 中统一的 skills 来源

当前这个仓库本身不直接开发一套全新的 skills，而是通过 `git subrepo` 跟踪上游仓库，再在需要时叠加本地定制。

## Included Repos

当前包含 3 个上游仓库：

- `Humanizer-zh`
  上游：`git@github.com:op7418/Humanizer-zh.git`
  说明：单 skill 仓库，根目录直接包含 `SKILL.md`

- `obsidian-skills`
  上游：`git@github.com:kepano/obsidian-skills.git`
  说明：多 skill 仓库，实际 skills 位于 `obsidian-skills/skills/`

- `superpowers`
  上游：`git@github.com:obra/superpowers.git`
  说明：多 skill 仓库，实际 skills 位于 `superpowers/skills/`

这些目录都由 `git subrepo` 管理。每个目录下的 `.gitrepo` 文件记录了对应的上游仓库、分支和同步信息。

## Repository Layout

```text
my-skills/
├── Humanizer-zh/
├── obsidian-skills/
└── superpowers/
```

这里保留的是“整个上游仓库”，而不是只抽取内部的 `skills/` 子目录。这样做有几个目的：

- 保留上游 README、脚本、插件元数据和其他辅助文件
- 让 `git subrepo pull` / `git subrepo push` 工作流保持简单
- 在需要排查或继续私有定制时，不会丢掉仓库上下文

## How It Is Used

这个仓库通常不直接手工安装，而是通过 `dotfiles` 统一消费：

- `dotfiles` 把本仓库作为单一 submodule 挂载到 `skills/`
- `dotfiles/setup.sh` 会把这里的顶层仓库目录链接到：
  - `~/.claude/skills/`
  - `~/.codex/skills/`

当前本地环境约定继续使用 `~/.claude` 和 `~/.codex` 作为 skills 安装路径。

## Daily Workflow

### Pull upstream changes

在仓库根目录执行：

```bash
git subrepo pull Humanizer-zh
git subrepo pull obsidian-skills
git subrepo pull superpowers
```

如果只需要同步某一个上游，只执行对应目录即可。

### Make private changes

直接修改对应目录中的文件，例如：

```bash
$EDITOR superpowers/skills/brainstorming/SKILL.md
$EDITOR obsidian-skills/skills/obsidian-cli/SKILL.md
```

修改完成后，像普通仓库一样提交：

```bash
git status
git add .
git commit -m "docs: adjust private skill defaults"
```

### Push this private repo

```bash
git push origin main
```

这一步只会把聚合后的仓库和你的私有修改推送到 `my-skills`，不会影响上游开源仓库。

### Push back to upstream

不要直接 Push 上游，只能以整理好的开源Pull Request形式贡献回上游。

## Notes

- `Humanizer-zh` 是单 skill 仓库，所以仓库根目录就是 skill 本体
- `obsidian-skills` 和 `superpowers` 是多 skill 仓库，skills 在各自的 `skills/` 子目录下
- 如果以后还要引入新的 skills 仓库，优先继续用 `git subrepo`，保持这里的维护模型一致
- 如果只是本地临时实验，不值得长期维护，可以先在这个私有仓库里改，确认稳定后再决定是否保留
