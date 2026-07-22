---
name: writing-great-skills
description: 编写和编辑高质量 skill 的参考——让 skill 可预测的词汇和原则。
disable-model-invocation: true
---

skill 的存在是为了从随机系统中梳理出确定性。**可预测性**——agent 每次运行采取相同的*过程*，而不是产生相同的输出——是根本美德；下方的每个杠杆都服务于它。

**加粗术语**在 [`GLOSSARY.md`](GLOSSARY.md) 中定义；在那里查完整含义。

## 调用（Invocation）

两个选择，交换不同的成本：

- **model-invoked** skill 保留一个 **description**，所以 agent 可以自动触发它，其他 skill 也可以到达它（你仍然可以输入它的名字）。它贡献 **context load**——description 每轮都坐在窗口中。机制：省略 `disable-model-invocation`，写一个面向模型的 description，用丰富的触发短语（"当用户想要…、提到…时使用"）。
- **user-invoked** skill 从 agent 的触及中剥离 description：只有你输入它的名字才能调用它——其他 skill 也不能。零 context load，但它花费 **cognitive load**：*你* 是必须记住它存在的索引。机制：设置 `disable-model-invocation: true`；`description` 变成面向人的——一行总结，去掉触发列表。

只在 agent 必须自己到达 skill 或其他 skill 必须到达它时才选 model-invocation。如果它永远只手动触发，就让它 user-invoked 并不付 context load。

当 user-invoked skill 多过你能记住时，堆积的 cognitive load 通过 **router skill** 治愈：一个 user-invoked skill 命名其他 skill 以及何时到达每个。

## 写 description

model-invoked 的 **description** 做两件事——说明 skill 是什么，以及列出应该触发它的 **branches**。每个词都增加 **context load**，所以 description 比正文更值得狠狠修剪：

- **前置 skill 的 leading word**——description 是它做调用工作的地方。
- **每个 branch 一个触发。** 重命名单个 branch 的同义词是**重复**——"用 TDD 构建功能… 要求测试优先开发" 是一个 branch 写了两次。折叠它们；只保留真正不同的 branch。
- **切掉正文中已有的身份。** 保持 description 为触发，加上任何"当另一个 skill 需要…"的到达子句。

## 信息层级

skill 由两种内容类型构建——**steps** 和 **reference**——自由混合：skill 可以全是 steps、全是 reference、或两者都有。核心决策是用哪个，以及每个坐在**信息层级**的哪个位置——一个按 agent 需要材料的即时程度排名的梯子：

1. **In-skill step**——`SKILL.md` 中的有序动作，主要层级：agent 做什么，按顺序。每个 step 以 **completion criterion** 结束，告诉 agent 工作完成的条件。让它*可检查*（agent 能区分完成和未完成吗？）以及，在重要的地方，*穷尽*（"每个修改的模型都已考虑"，而不是"产出变更列表"）——模糊的标准邀请**提前完成**。
2. **In-skill reference**——`SKILL.md` 中的定义、规则或事实，按需查阅。通常是一个合法的扁平同级集（一个 review 的所有规则在一个横档上）——一个很好的安排，不是异味。*这个 skill 全是 reference。*
3. **External reference**——reference 被推出 `SKILL.md` 到单独文件，由 **context pointer** 到达，只在指针触发时加载。（跨越 *disclosed* 引用——像 `GLOSSARY.md` 这样的兄弟文件，仍然是 skill 的一部分——到完全**外部引用**，它存在于 skill 系统之外，任何 skill 都可以指向它。）

一个要求高的 completion criterion 驱动彻底的 **legwork**——agent 在工作中做的挖掘——无论 skill 有没有 steps，因为"每个规则都应用了"绑定扁平 reference 正如"每个 step 都完成了"绑定一个序列。

推太少下去顶部会膨胀；推太多你会隐藏 agent 真正需要的材料。那个张力就是整个决策。

**渐进式披露**是向下梯子的移动——从 `SKILL.md` 到链接文件——让顶部保持可读。机制：skill 文件夹中一个链接的 `.md` 文件，以它持有的东西命名（这个 skill 把它的完整定义披露给 `GLOSSARY.md`）。一些 skill 以不止一种方式使用，每种不同的方式是一个 **branch**——不同的运行走 skill 的不同路径。分支是最干净的披露测试：内联每个 branch 需要的东西，把只有某些 branch 到达的东西推到指针后面。**context pointer** 的*措辞*，不是它的目标，决定 agent 何时以及多么可靠地到达材料。

梯子决定一片材料*往下坐多远*，**co-location** 决定*一旦到了那里什么坐在它旁边*：把一个概念的定义、规则和注意事项放在一个标题下而不是分散，这样读一部分就带来了它的邻居。

## 何时拆分

**Granularity** 是你把 skill 分得有多细，每个切割花费两种负载之一，所以只在切割值得时才拆分。两个切割：

- **按调用**——当你有一个应该自己触发的不同 **leading word**，或其他 skill 必须到达它时，拆分出一个 **model-invoked** skill。你为新的始终加载的 **description** 付 **context load**，所以那个独立到达必须值得。
- **按序列**——当一个 **steps** 序列中前面的步骤（一个 step 的 **post-completion steps**）诱惑 agent 匆忙完成前面的步骤（**提前完成**）时，拆分它们。把它们保持在视线外鼓励 agent 在当前任务上做更多 **legwork**。

## 修剪

保持每个含义在**单一事实来源**中：一个权威地方，这样改变行为是一次编辑。

检查每一行的**相关性**：它仍然与 skill 做的事情有关吗？

然后逐句（不只是逐行）猎杀 **no-op**：在每个句子上单独运行 no-op 测试，当一个失败时，删除整个句子而不是从中修剪单词。要激进——大多数失败的散文应该走，而不是重写。

## Leading words

**leading word** 是一个已经活在模型预训练中的紧凑概念，agent 在运行 skill 时用它思考（例如 _lesson_、_fog of war_、_tracer bullets_）。在文本中重复（虽然不一定——一个强的 leading word 可能只需要一次），它积累了一个分布式定义，用最少的 token 锚定了一整块行为，通过招募模型已经持有的先验。

它两次服务于可预测性。在正文中它锚定*执行*：agent 每次词出现时到达相同的行为。在 description 中它锚定*调用*：当相同的词住在你的 prompt、文档和代码中时，agent 把那个共享语言链接到 skill 并更可靠地触发它。

寻找机会重构 skill 使用 leading words。在三处拼写的**三元组**（**重复**），一个 description 花一句话暗示一个想法——每个都是乞求**折叠**成单个 token 的段落。例子包括：

- "fast, deterministic, low-overhead" -> _tight_——一个质量在一个阶段重述——变成一个预训练单词（一个 _tight_ 循环）。
- "a loop you believe in" -> _red_——把模糊的门变成二进制可观察状态（循环在 bug 上变 _red_，或者不变）。

你两次赢：更少的 token，*和* 一个更锋利的钩子让 agent 挂它的思考。假设每个 skill 都携带着 leading words 要退休的重述——去找它们。

## 失败模式

用这些诊断用户可能遇到的 skill 问题。

- **提前完成**——在 step 真正完成之前结束，注意力滑向*完成*。防御，按顺序：先 sharpen completion criterion（便宜、本地）；只有当它不可减少地模糊*并且*你观察到匆忙时，通过拆分（序列切割）隐藏 post-completion steps。
- **重复**——相同含义在一个以上的地方。花费维护和 token，并把含义的突出度膨胀到超过它在梯子上的真实排名。
- **沉积物**——因为添加感觉安全而移除感觉风险而沉淀的过时层。任何没有修剪规范的 skill 的默认命运。
- **蔓延**——一个 skill 就是太长，即使每行都是活的且独特的。伤害可读性和可维护性并浪费 token。治疗是梯子：在指针后面披露**reference**，并按**branch**或序列拆分使每条路径只携带它需要的东西。
- **no-op**——模型已经默认遵守的一行，所以你付负载说废话。测试：它对比默认改变行为吗？一个弱的 leading word（当 agent 已经差不多彻底时的 _be thorough_）是一个 no-op；修复是一个更强的词（_relentless_），不是不同的技术。
- **否定**：通过禁止来转向会适得其反：_don't think of an elephant_ 命名了大象并让它更易获得，而不是更少。提示**积极的**——陈述目标行为这样被禁止的那个永远不说；只在作为你无法正面表述的硬护栏时保留一个禁止，然后甚至把它和该做什么配对。
