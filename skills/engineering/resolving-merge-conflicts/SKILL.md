---
name: resolving-merge-conflicts
description: "当你需要解决进行中的 git merge/rebase 冲突时使用。"
---

1. **看当前状态** merge/rebase。检查 git 历史，和冲突文件。

2. **找每个冲突的主要来源。** 深入理解每个变更为什么被做，原始意图是什么。读 commit 消息，检查 PR，检查原始 issues/tickets。

3. **解决每个 hunk。** 尽可能保留两个意图。不兼容时，选匹配 merge 声明的目标的那个并注明权衡。**不要**发明新行为。总是解决；永远不要 `--abort`。

4. 发现项目的**自动检查**并运行它们——通常先 typecheck，然后 tests，然后 format。修复 merge 打破的任何东西。

5. **完成 merge/rebase。** 暂存所有东西并 commit。如果 rebasing，继续 rebase 过程直到所有 commits 都被 rebase。
