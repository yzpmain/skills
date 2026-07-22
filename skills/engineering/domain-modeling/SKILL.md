---
name: domain-modeling
description: 构建和打磨项目的领域模型。当用户想要确定领域术语或通用语言、记录架构决策，或另一个 skill 需要维护领域模型时使用。
---

# 领域建模（Domain Modeling）

在设计时主动构建和打磨项目的领域模型。这是*主动的*实践——挑战术语、发明边缘场景、在术语和决策结晶的瞬间就把它们写下来。（仅仅*读取* `CONTEXT.md` 获取词汇不是这个 skill——那是任何 skill 都能做的一行习惯。这个 skill 是当你*改变*模型时用的，不只是消费它。）

## 文件结构

大多数 repo 有单个上下文：

```
/
├── CONTEXT.md
├── docs/
│   └── adr/
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

如果根目录存在 `CONTEXT-MAP.md`，repo 有多个上下文。map 指向每个上下文住在哪：

```
/
├── CONTEXT-MAP.md
├── docs/
│   └── adr/                          ← 系统级决策
├── src/
│   ├── ordering/
│   │   ├── CONTEXT.md
│   │   └── docs/adr/                 ← 上下文特定决策
│   └── billing/
│       ├── CONTEXT.md
│       └── docs/adr/
```

惰性创建文件——只有当你有东西要写时才创建。如果不存在 `CONTEXT.md`，在第一个术语被解决时创建它。如果不存在 `docs/adr/`，在第一个 ADR 被需要时创建它。

## 会话期间

### 对照术语表挑战

当用户使用的术语与 `CONTEXT.md` 中现有语言冲突时，立即指出。"你的术语表把 'cancellation' 定义为 X，但你似乎意思是 Y——是哪个？"

### 精化模糊语言

当用户用模糊或重载的术语时，提出一个精确的规范术语。"你说的是 'account'——你是指 Customer 还是 User？那是不同的东西。"

### 讨论具体场景

当讨论领域关系时，用具体场景对它们进行压力测试。发明探测边缘情况的场景，强迫用户对概念边界变得精确。

### 与代码交叉引用

当用户陈述某物如何工作时，检查代码是否同意。如果发现矛盾，暴露它："你的代码取消整个 Order，但你刚才说部分取消是可能的——哪个是对的？"

### 内联更新 CONTEXT.md

当一个术语被解决时，当场更新 `CONTEXT.md`。不要批量处理——在它们发生时捕获。格式见 [CONTEXT-FORMAT.md](./CONTEXT-FORMAT.md)。

`CONTEXT.md` 应该完全不含实现细节。不要把 `CONTEXT.md` 当作 spec、scratch pad 或实现决策的存储库。它是术语表，仅此而已。

### 节制地提供 ADR

只有当三个条件都为真时才提供创建 ADR：

1. **难以撤销**——改变主意的成本是有意义的
2. **没有上下文会让人惊讶**——未来的读者会想"他们为什么这样做？"
3. **是真实权衡的结果**——有真正的替代方案，你为特定原因选了一个

如果三个中缺任何一个，跳过 ADR。格式见 [ADR-FORMAT.md](./ADR-FORMAT.md)。
