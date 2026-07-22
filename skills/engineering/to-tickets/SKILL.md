---
name: to-tickets
description: 把计划、spec 或当前对话拆成一组 tracer-bullet ticket，每个声明其阻塞边，发布到配置的 tracker——边在本地是每个 ticket 一个文件中的文本，在真实 tracker 上是原生阻塞链接。
disable-model-invocation: true
---

# To Tickets

把计划、spec 或对话拆成一组 **tickets**——tracer-bullet 垂直切片，每个声明**阻塞**它的 tickets。

Issue tracker 和 triage 标签词汇应该已经提供给你了——如果没有，运行 `/setup-matt-pocock-skills`。

## 流程

### 1. 收集上下文

从对话上下文中已有的东西工作。如果用户传了一个引用（spec 路径、issue 号或 URL）作为参数，获取它并读其完整正文和评论。

### 2. 探索代码库（可选）

如果你还没有探索代码库，就这样做以理解代码的当前状态。Ticket 标题和描述应该使用项目的领域词汇表词汇，并尊重你触及区域的 ADR。

寻找机会对代码做预重构让实现更容易。"让改变变容易，然后做容易的改变。"

### 3. 起草垂直切片

把工作拆成 **tracer bullet** ticket。

<vertical-slice-rules>

- 每个切片切过每一层（schema、API、UI、测试）的窄但完整路径——垂直的，不是一层的水平切片
- 一个完成的切片可以独立演示或验证
- 每个切片大小适合一个新鲜上下文窗口
- 任何预重构应该先做

</vertical-slice-rules>

给每个 ticket 它的 **blocking edges**——必须先完成的其他 tickets。没有阻塞者的 ticket 可以立即开始。

**宽重构是垂直切片的例外。** **宽重构**是一个机械变更——重命名一列、重新类型一个共享符号——其**爆炸半径**波及整个代码库，所以一次编辑会打破数千个调用站点，没有垂直切片能落地为绿色。不要把它强行塞进 tracer bullet；把它排序为 **expand–contract**。先 expand：在旧形式旁边添加新形式，这样什么都不破。然后按爆炸半径分批迁移调用站点（每个包、每个目录），每批是自己的 ticket，被 expand 阻塞，因为旧形式仍然存在所以 CI 保持绿色。最后 contract：一旦没有调用者剩下，删除旧形式，在一个被每个迁移批次阻塞的 ticket 中。当甚至批次不能单独保持绿色时，保持序列但让它们共享一个集成分支，所有分支阻塞一个最终的集成和验证 ticket——绿色只在那里承诺。

### 4. 测验用户

把提议的拆分呈现为编号列表。对每个 ticket，显示：

- **标题**：短的描述性名称
- **Blocked by**：哪些其他 tickets（如果有）必须先完成
- **它交付什么**：这个 ticket 让什么端到端行为工作

问用户：

- 粒度感觉对吗？（太粗 / 太细）
- 阻塞边正确吗——每个 ticket 只依赖真正门控它的 tickets 吗？
- 应该合并或进一步拆分任何 ticket 吗？

迭代直到用户批准拆分。

### 5. 发布 tickets 到配置的 tracker

发布批准的 tickets。**如何**取决于 `/setup-matt-pocock-skills` 配置的 tracker——tickets 是一样的，只有阻塞边的形状改变：

- **本地文件** → 在 `.scratch/<feature-slug>/issues/<NN>-<slug>.md` 下每个 ticket 写一个文件，从 `01` 开始按依赖顺序编号（阻塞者先）。每个文件的 "Blocked by" 列出它依赖的编号/标题。使用下方的每个 ticket 文件模板——每个 ticket 一个文件，永远不要合并文件。
- **真实 issue tracker（GitHub、Linear、…）** → 按依赖顺序（阻塞者先）每个 ticket 发布一个 issue，这样每个 ticket 的阻塞边可以引用真实标识符。使用平台的原生阻塞/子 issue 关系（如果有的）；否则把每个 ticket 的 "Blocked by" 设为阻塞 issues。应用 `ready-for-agent` triage 标签（除非另有指示）——tickets 构造上就是 agent 可抓取的。

工作在 **frontier**（前沿）上：任何阻塞者都已完成的 ticket。对纯线性链这意味着从上到下。

不要关闭或修改任何父 issue。

<local-ticket-template>

# <NN> — <Ticket 标题>

**要构建什么：** 这个 ticket 让什么端到端行为工作，从用户视角——不是逐层的实现列表。

**Blocked by：** 门控这个 ticket 的 tickets 的编号/标题，或 "无——可以立即开始"。

**状态：** ready-for-agent

- [ ] 验收标准 1
- [ ] 验收标准 2

</local-ticket-template>

<issue-template>

## 父级

对 tracker 上父 issue 的引用（如果来源是已有 issue，否则省略这个段落）。

## 要构建什么

这个 ticket 让什么端到端行为工作，从用户视角——不是逐层实现。

## 验收标准

- [ ] 标准 1
- [ ] 标准 2

## Blocked by

对每个阻塞 ticket 的引用，或 "无——可以立即开始"。

</issue-template>

两种形式中，避免具体的文件路径或代码片段——它们很快会过时。例外：如果原型产出了一个比散文更精确编码决策的片段（状态机、reducer、schema、类型形状），内联它并简要注明它来自原型。只保留决策丰富的部分——不是可工作的 demo，只是重要的部分。
