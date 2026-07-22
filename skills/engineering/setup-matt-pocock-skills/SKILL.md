---
name: setup-matt-pocock-skills
description: 为这个 repo 配置工程 skills——设置它的 issue tracker、triage 标签词汇和领域文档布局。在其他工程 skill 首次使用前运行一次。
disable-model-invocation: true
---

# Setup Matt Pocock's Skills

搭建工程 skill 假设的每个 repo 配置：

- **Issue tracker** — issue 住在哪（默认 GitHub；本地 markdown 也开箱即用）
- **Triage 标签** — 五个规范 triage 角色使用的字符串
- **领域文档** — `CONTEXT.md` 和 ADR 住在哪，以及读取它们的消费者规则

这是一个 prompt 驱动的 skill，不是确定性脚本。探索，展示你发现的，与用户确认，然后写入。

## 流程

### 1. 探索

查看当前 repo 理解它的初始状态。读取所有存在的东西；不要假设：

- `git remote -v` 和 `.git/config` — 这是一个 GitHub repo 吗？是哪个？
- repo 根目录的 `AGENTS.md` 和 `CLAUDE.md` — 哪一个存在？里面已有 `## Agent skills` 段落吗？
- repo 根目录的 `CONTEXT.md` 和 `CONTEXT-MAP.md`
- `docs/adr/` 和任何 `src/*/docs/adr/` 目录
- `docs/agents/` — 这个 skill 之前输出是否已存在？
- `.scratch/` — 表明已在使用本地 markdown issue tracker 约定
- `triage` skill 是否已安装？（这个 skill 旁边有一个 `triage` skill 文件夹，或 `triage` 在你的可用 skills 中。）这决定 Section B 是否运行。
- Monorepo 信号 — `pnpm-workspace.yaml`、`package.json` 中的 `workspaces` 字段，或有自己 `src/` 的已填充的 `packages/*`。只在真正的大型多包 repo 中呈现；它们的缺席意味着单上下文，这几乎是每个 repo。

### 2. 展示发现并提问

总结什么存在、什么缺失。然后按顺序处理每个 section——一次一个 section，一个回答，然后下一个。

每个 section 以推荐答案开头，让用户可以用一个词接受。只在选择真正分叉时给出一行解释；当探索已经解决了 section 时跳过整个 section（Section B 当 `triage` 没安装时，Section C 当没有 monorepo 时）。

**Section A — Issue tracker。**

> 解释：\"Issue tracker\" 是这个 repo 的 issue 住的地方。像 `to-tickets`、`triage`、`to-spec` 和 `qa` 这样的 skill 从它读取并向它写入——它们需要知道是调用 `gh issue create`、在 `.scratch/` 下写 markdown 文件，还是遵循你描述的其他工作流。选你实际为这个 repo 追踪工作的地方。

默认立场：这些 skill 是为 GitHub 设计的。如果 `git remote` 指向 GitHub，提议它。如果指向 GitLab（`gitlab.com` 或自托管主机），提议 GitLab。否则（或用户偏好），提供：

- **GitHub** — issues 住在 repo 的 GitHub Issues（使用 `gh` CLI）
- **GitLab** — issues 住在 repo 的 GitLab Issues（使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI）
- **本地 markdown** — issues 作为文件住在 repo 下的 `.scratch/<feature>/` 中（适合个人项目或无远程的 repo）
- **其他**（Jira、Linear 等）— 让用户用一段话描述工作流；skill 会把它记录为自由格式散文

选择记录在 `docs/agents/issue-tracker.md` 中。GitHub 和 GitLab 模板带有一个 \"PRs as a request surface\" 标志，默认**关闭**——保持关闭不要打开它；想要外部 PR 在 triage 队列中的用户可以在文件中稍后翻转标志。

**Section B — Triage 标签词汇。** 如果 `triage` skill 没安装就完全跳过这个 section（探索告诉你的）——未安装的 skill 不需要标签。

如果已安装，问恰好一个问题：

> 你想保留默认 triage 标签吗？（推荐：**是**）

默认值是五个规范角色，每个标签字符串等于其名称：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`。在**是**上，原样写入它们。只有在用户说不的时候——通常因为他们的 tracker 已经用了其他名字（如 `bug:triage` 替代 `needs-triage`）——收集覆盖，使 `triage` 应用现有标签而不是创建重复。

**Section C — 领域文档。** 默认 **single-context** — 在 repo 根目录一个 `CONTEXT.md` + `docs/adr/`。这适合几乎所有 repo；不问直接写入。

只当探索发现 monorepo 信号时才提供 **multi-context** — 一个根 `CONTEXT-MAP.md` 指向每个上下文的 `CONTEXT.md` 文件。然后确认他们想要哪个布局。

### 3. 确认并编辑

给用户展示草稿：

- 要添加的 `## Agent skills` 块到正在编辑的 `CLAUDE.md` / `AGENTS.md` 中（见步骤 4 的选择规则）
- `docs/agents/issue-tracker.md`、`docs/agents/domain.md` 和 `docs/agents/triage-labels.md` 的内容（最后仅在 `triage` 已安装时）

让他们在写入前编辑。

### 4. 写入

**选择要编辑的文件：**

- 如果 `CLAUDE.md` 存在，编辑它。
- 否则如果 `AGENTS.md` 存在，编辑它。
- 如果两者都不存在，问用户要创建哪个——不要替他们选。

当 `CLAUDE.md` 已存在时绝不创建 `AGENTS.md`（反之亦然）——总是编辑已经存在的那一个。

如果选择的文件中已存在 `## Agent skills` 块，原地更新其内容而不是追加重复。不要覆盖用户对周围段的编辑。

这个块：

```markdown
## Agent skills

### Issue tracker

[issue 在哪被追踪的一行总结]。见 `docs/agents/issue-tracker.md`。

### Triage labels

[标签词汇的一行总结]。见 `docs/agents/triage-labels.md`。

### Domain docs

[布局的一行总结 — \"single-context\" 或 \"multi-context\"]。见 `docs/agents/domain.md`。
```

只在 `triage` 已安装且 Section B 运行时包含 `### Triage labels` 子块，并写入 `docs/agents/triage-labels.md`。当没安装时，两者都省略。

然后使用这个 skill 文件夹中的种子模板写入 docs 文件：

- [issue-tracker-github.md](./issue-tracker-github.md) — GitHub issue tracker
- [issue-tracker-gitlab.md](./issue-tracker-gitlab.md) — GitLab issue tracker
- [issue-tracker-local.md](./issue-tracker-local.md) — 本地 markdown issue tracker
- [triage-labels.md](./triage-labels.md) — 标签映射（仅在 `triage` 已安装时）
- [domain.md](./domain.md) — 领域文档消费者规则 + 布局

对于 \"other\" issue tracker，从头用用户的描述写 `docs/agents/issue-tracker.md`。

### 5. 完成

告诉用户 setup 完成了，哪些工程 skill 现在会从这些文件读取。提到他们之后可以直接编辑 `docs/agents/*.md`——只有在想要切换 issue tracker 或从头开始时才需要重新运行这个 skill。
