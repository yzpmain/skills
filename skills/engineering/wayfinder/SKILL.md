---
name: wayfinder
description: 规划一个巨大的工作块——超过一个 agent 能容纳的——作为你 issue tracker 上的决策 ticket 共享地图，逐一解决它们直到通往目的地的路清晰。
disable-model-invocation: true
---

一个松散的想法到了——太大一个 agent 装不下，裹在雾中：从这里到**目的地**的路还看不清。Wayfinding 是关于找到那条路，不是冲向目的地。这个 skill 把路绘制成 repo issue tracker 上的一个**共享地图**，然后逐一解决它的**决策 ticket**——问题是其解决是决策的，不是要执行的构建切片——直到路线清晰。

目的地因工作而异，命名它是绘制的第一步——它塑造每个 ticket。它可能是交接和迭代的 spec、规划开始前要锁定的决策、或像数据结构迁移那样的就地变更。地图是领域无关的——工程工作、课程内容，任何适合这个形状的。

## 规划，不执行

Wayfinder 默认是**规划**：每个 ticket 解决一个决策，当地图完成时路就清晰了——在有人去做这件事之前没有东西要决定。直接去做工作的拉力通常是你已到达地图边缘该交接的信号。一个工作可以在它的 **Notes** 中覆盖这个——把执行带进地图本身——但缺省产出决策，不是可交付物。

## 按名称引用

每个地图和 ticket 都是一个 issue，所以它有一个**名称**——它的标题。在人类读的所有东西中——叙述、地图的 Decisions-so-far——按那个名称引用它，永远不要用裸 id、号或 slug。一墙的 `#42, #43, #44` 是不可读的；名称一目了然。id 和 URL 不会消失——名称包裹它的链接——但它们骑在名称*里面*，永远不替代它。

## 地图

地图是这个 repo issue tracker 上的单个 issue，标有 `wayfinder:map`——规范工件。它的 tickets 是地图的子 issues。

地图是一个**索引**，不是存储。它列出已做的决策并指向持有其细节的 tickets；一个决策恰好住在一个地方——它的 ticket——所以地图从不重述它，只 gist 它并链接。

**地图、它的子 tickets、阻塞和前沿查询物理住在哪是 tracker 特定的。** Issue tracker 应该已经提供给你了——如果没有，运行 `/setup-matt-pocock-skills`。查阅 tracker 文档的 "Wayfinding operations" 段落了解*这个* repo 如何表达它们。如果没有提供 tracker，默认用本地 markdown tracker。

### 地图正文

整个地图低分辨率，每会话加载一次。Open tickets **不**列出来——它们是 open 子 issues，通过查询找到。

```markdown
## Destination

<到达这个地图终点看起来像什么——这个工作正在找到它的路的 spec、决策或变更。一到两行；每个会话在选择 ticket 之前定向到它。>

## Notes

<领域；每个会话应该咨询的 skills；这个工作的长期偏好>

## Decisions so far

<!-- 索引——每个关闭的 ticket 一行：足够判断相关性，然后放大链接查看 ticket 持有的细节 -->

- [<closed ticket title>](link) — <答案的一行 gist>

## Not yet specified

<!-- 见 "未知迷雾（fog of war）"：范围内的雾，你还不能 ticket；随着前沿推进毕业 -->

## Out of scope

<!-- 见 "Out of scope"：被裁定在目的地之外的工作；关闭，永远不毕业 -->
```

### Tickets

每个 ticket 是地图的**子 issue**；tracker 的 issue id 是它的身份。它的正文是问题，大小适合一个 100K token agent 会话：

```markdown
## Question

<这个 ticket 解决的决策或调查>
```

每个 ticket 带有一个 `wayfinder:<type>` 标签——`research`、`prototype`、`grilling`、`task` 之一（见 [Ticket Types](#ticket-types)）。

一个会话通过把 ticket 分配给驱动地图的 dev 来**认领**它，**首先**，在任何工作之前，这样并发会话跳过它。那个 assignee *就是* claim：一个 open、未分配的 ticket 是未认领的。

阻塞使用 tracker 的**原生**依赖关系——至关重要因为它在 tracker 自己的 UI 中*可视化*地呈现前沿，这样人类不用打开地图就能看到什么是可取的。只有缺乏原生阻塞的 tracker 才回退到正文约定。一个 ticket 在阻塞它的每个 ticket 都关闭时是**未阻塞的**；**前沿**是 open、未阻塞、未认领的子——已知边缘。

答案不是正文的一部分——它在解决时记录（见 [Work through the map](#work-through-the-map)）。解决 ticket 时创建的资产链接到 issue 不从粘贴。

## Ticket Types

每个 ticket 要么是 **HITL**——human in the loop，与一个为自己说话的人一起工作——要么是 **AFK**，仅由 agent 驱动。HITL ticket 只通过那个实时交换解决；agent 永远不代表人类那边（一个回答自己问题的追问 agent 已经破坏了这个）。

- **Research**（AFK）：阅读文档、第三方 API 或本地资源如知识库，暴露一个决策等待的事实。由 `/research` **子 agent** 解决。当需要当前工作目录之外的知识时使用。
- **Prototype**（HITL）：通过制作一个便宜的、粗糙的、具体的工件来提高讨论保真度——一个大纲、粗略的 take、存根、或通过 /prototype skill 的 UI/logic 代码。把原型链接为资产。当 "它应该长什么样" 或 "它应该怎么表现" 是关键问题时用。
- **Grilling**（HITL）：通过 /grilling 和 /domain-modeling skills 的对话，一次一个问题。默认情况。
- **Task**（HITL 或 AFK）：在*决策*之前必须发生的人工工作——没有东西要决定、原型或研究，但在它完成之前讨论被阻塞。注册一个服务以便其 API 可以被判断、配置访问、移动数据以便其形状可以被看到。这是*做*而不是决定的唯一类型——它通过解锁一个决策而不是交付目的地来赢得它的位置。Agent 在可以的地方单独驱动它（AFK）；否则它交给人类一个精确的清单（HITL）。解决时工作完成；答案记录做了什么和任何结果事实（凭证位置、新 URL、行数）后续 tickets 依赖的。

## 未知迷雾（fog of war）

地图是*故意*不完整的：不要绘制你还看不到的东西。活 tickets 之外是**未知迷雾**——你能判断正在到来但还不能确定的决策和调查的暗淡视图，因为它们挂在仍然开放的问题上。解决一个 ticket 清除它前面的雾，把现在可规格化的任何东西毕业为新 tickets——一次一个，直到通往目的地的路清晰且没有 tickets 剩下。

地图的 **Not yet specified** 段落是那个暗淡视图被写下的地方：怀疑的问题，稍后重新访问的区域。它是朝向目的地的未发现前沿——这里的所有东西都在范围内，只是不够 sharp 到可以 ticket。写多松散或多完整随视图允许；它作为协作者读的努力方向的路标。

**雾还是 ticket？** 测试是现在是否能精确陈述问题——*不是*现在是否能回答它。

- **Ticket 当** 问题已经 sharp——即使它被阻塞你还不能对它行动。
- **Not yet specified 当** 你还不能那么 sharp 地措辞它。不要把雾预切成 ticket 大小的块：它比 ticket 粗糙，一个补丁可能毕业为几个 tickets，或没有，一旦前沿到达它。

**Not yet specified** 排除已经决定的东西（Decisions so far）、已经是活 ticket 的东西、和范围外的东西（下一段）。

## 范围外

雾只朝目的地聚集。目的地固定范围，所以它之外的工作是**范围外的**——它不是雾，不属于 **Not yet specified**。它在地图上得到自己的 **Out of scope** 段落：你有意识地裁定*这个*工作之外的。范围，不是 sharpness，把它放在这里。

范围外的工作永远不毕业——前沿停在目的地——所以它只在目的地被重画时返回，然后作为新努力，不是恢复。

裁定范围外是一个范围行为，不是路线上的步骤。当一个已经存在的 ticket 被发现坐在目的地之外——在绘制时错误地放进去的，或被解决暴露的——**关闭它**（一个关闭的 ticket 明确地离开前沿）并在 **Out of scope** 段落留一行：gist 加为什么它超出范围，链接关闭的 ticket。它留在 **Decisions so far** 之外，它记录实际走的路线——范围边界不是上面的步骤。

## 调用

两种模式。无论哪种，**永远不要每会话解决超过一个 ticket**——研究 ticket 除外。

### 绘制地图

用户用一个松散的想法调用。

1. **命名目的地。** 运行 `/grilling` 和 `/domain-modeling` 会话来确定这个地图正在找到它的路到哪——spec、决策或变更。目的地固定范围，所以它先被确定。
2. **绘制前沿。** 再次追问，这次**breadth-first**：在整个空间扇出而不是在任何一条线上深入，暴露开放的决策和现在可以采取的第一步。**如果这没有暴露雾**——通往目的地的路已经清晰，整个旅程小到适合一个会话——你不需要地图。停下来问用户他们想怎么继续。
3. **创建地图**（标签 `wayfinder:map`）：Destination 和 Notes 已填，Decisions-so-far 为空，雾勾画到 **Not yet specified**。
4. **创建你现在能指定的 tickets** 作为地图的子 issues——然后在**第二轮**中连接阻塞边（issues 需要 id 才能互相引用）。接线把它们排序到前沿和被阻塞的；你不能指定的东西留在雾中——**Not yet specified** 段落。
5. **启动研究子 agents。** 对你刚创建的每个 `research` ticket，启动一个 `/research` 子 agent 并行解决它，把它的发现捕获在一个一次性 `research/<name>` 分支上，带一个来自 ticket 的上下文指针。
6. 停止——绘制是一个会话的工作；它不亲手解决任何东西。

### 走过地图

用户用一个地图调用（URL 或号）。一个 ticket 是**可选的**——没有它，你选下一个决策，不是用户。

1. 加载**地图**——低分辨率视图，不是每个 ticket 正文。
2. 选择 ticket。如果用户指定了一个，用它。否则按顺序取第一个前沿 ticket。**认领它**：在任何工作之前分配给你自己。
3. 解决它——**按需放大**：按需获取任何相关或关闭 ticket 的正文；调用 `## Notes` 段落命名的 skills。如果有疑问，用 `/grilling` 和 `/domain-modeling`。
4. 记录解决：发布答案作为**解决评论**，**关闭** issue，并**追加一个上下文指针**到地图的 Decisions-so-far。
5. 添加新暴露的 tickets（创建然后接线）；毕业答案已使可规格化的任何雾，从 **Not yet specified** 清除每个毕业的补丁这样它只作为它的活 ticket 存在。如果答案揭示一个 ticket——这个或另一个——坐在目的地之外，**裁定它超出范围**而不是在路线上解决它。如果决策使地图的其他部分无效，更新或删除那些 tickets。

用户可以并行运行未阻塞的 tickets，所以预期其他会话会并发编辑 tracker。
