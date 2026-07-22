---
name: improve-codebase-architecture
description: 扫描代码库找深化机会，把它们呈现为可视化 HTML 报告，然后追问你选中的任何一个。
disable-model-invocation: true
---

# 改善代码库架构

暴露架构摩擦并提出 **deepening opportunities（深化机会）**——把浅模块变成浅模块的重构。目标是可测试性和 AI 可导航性。

这个命令*参考*项目的领域模型并建立在共享设计词汇上：

- 运行 `/codebase-design` skill 获取架构词汇（**module**、**interface**、**depth**、**seam**、**adapter**、**leverage**、**locality**）及其原则（删除测试、"接口是测试表面"、"一个 adapter = 假设 seam，两个 = 真实"）。在每个建议中准确使用这些术语——不要漂移到 "component"、"service"、"API" 或 "boundary"。
- `CONTEXT.md` 中的领域语言给好的 seam 命名；`docs/adr/` 中的 ADR 记录了这个命令不应重新诉讼的决策。

## 流程

### 1. 探索

**扫描前先定范围——YAGNI。** 深化一个模块的回报是让它未来的变更更容易，所以把额外权重放在代码库最近改变的部分。在看你*往哪看*之前决定：

- 如果用户指定了一个方向——一个模块、子系统、痛点——接受它，跳过下方的推断。
- 否则，往回走一大段 commit 历史（`git log --oneline`）找代码库的热点——不断出现的文件和区域——让那些路径先拉你的注意力。如果变化分散没有明确的热点，扩大网。

先读项目的领域词汇表（`CONTEXT.md`）和你触及区域的任何 ADR。

然后用 Agent 工具 `subagent_type=Explore` 走代码库。不要遵循僵化的启发式——有机地探索并注意你经历摩擦的地方：

- 哪里理解一个概念需要在许多小模块之间跳来跳去？
- 哪里模块是**浅的**——接口几乎和实现一样复杂？
- 哪里纯函数只是为了可测试性被提取出来，但真正的 bug 藏在它们被调用的方式中（没有 **locality**）？
- 哪里紧耦合的模块跨它们的 seam 泄漏？
- 代码库的哪些部分未测试，或通过当前接口难以测试？

对你怀疑是浅的东西应用**删除测试**：想象删除它。如果复杂性消失了，它是 pass-through。如果复杂性重新出现在 N 个调用者中，它在赚它的存在。

### 2. 把候选呈现为 HTML 报告

写一个自包含的 HTML 文件到 OS 临时目录，这样什么都不落在 repo 中。从 `$TMPDIR` 解析临时目录，回退到 `/tmp`（或 Windows 上的 `%TEMP%`），写到 `<tmpdir>/architecture-review-<timestamp>.html` 这样每次运行都得到新文件。为用户打开它——Linux 上 `xdg-open <path>`，macOS 上 `open <path>`，Windows 上 `start <path>`——并告诉他们绝对路径。

报告使用 **Tailwind via CDN** 做布局和样式，**Mermaid via CDN** 做图（当 graph/flow/sequence 能可靠传达结构时）。混合 Mermaid 和手写的 CSS/SVG 可视化——当关系是图形状时（调用图、依赖、序列）用 Mermaid，当你想要更编辑化的东西时（mass diagrams、cross-sections、collapse animations）用手工 div/SVG。每个候选得到一个**前后可视化**。要有视觉感。

对每个候选，渲染一个卡片包含：

- **文件**——涉及哪些文件/模块
- **问题**——当前架构为什么造成摩擦
- **解决方案**——用大白话描述会改变什么
- **收益**——用 locality 和 leverage 的术语解释，以及测试会如何改善
- **前后图**——并排，手绘，说明浅度和深化
- **推荐强度**——`Strong`、`Worth exploring`、`Speculative` 之一，渲染为 badge

报告以**顶部推荐**段落结束：你会先处理哪个候选以及为什么。

**领域用 CONTEXT.md 词汇，架构用 `/codebase-design` 词汇。** 如果 `CONTEXT.md` 定义了 "Order"，谈 "the Order intake module"——不是 "the FooBarHandler"，也不是 "the Order service"。

**ADR 冲突**：如果候选与现有 ADR 矛盾，只在摩擦真实到值得重新审视 ADR 时才暴露它。在卡片中明确标记它（例如警告标注：_"与 ADR-0007 矛盾——但因为…值得重新打开"_）。不要列出 ADR 禁止的每个理论重构。

完整 HTML 脚手架、图表模式和样式指南见 [HTML-REPORT.md](HTML-REPORT.md)。

**不要提议接口。** 文件写完后，问用户："你想探索哪个？"

### 3. 追问循环

一旦用户选了候选，运行 `/grilling` skill 和他们一起走决策树——约束、依赖、深化模块的形状、seam 后面是什么、什么测试存活。

副作用在决策结晶时内联发生——运行 `/domain-modeling` skill 保持领域模型最新：

- **用 `CONTEXT.md` 中不存在的概念命名深化模块？** 把术语加到 `CONTEXT.md`。如果文件不存在惰性创建。
- **在对话中精化模糊术语？** 当场更新 `CONTEXT.md`。
- **用户用 load-bearing 的理由拒绝候选？** 提供 ADR，框架为：_"你想让我把这个记录为 ADR 这样未来的架构审查不会重新建议它吗？"_ 只在理由实际上会被未来探索者需要以避免重新建议相同东西时提供——跳过短暂的理由（"现在不值得"）和自明的理由。
- **想探索深化模块的替代接口？** 运行 `/codebase-design` skill 并用它的 design-it-twice 并行子 agent 模式。
