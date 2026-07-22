---
name: codebase-design
description: 设计深度模块的共享词汇。当用户想要设计或改进模块的接口、找深化机会、决定 seam 放哪、让代码更可测试或 AI 可导航，或另一个 skill 需要深度模块词汇时使用。
---

# 代码库设计

设计**深度模块**：大量行为藏在一个小接口后面，位于干净的 seam 上，通过该接口可测试。无论代码正在被设计还是重构，都使用这些语言和原则。目标是对调用者的 leverage、对维护者的 locality、对每个人的可测试性。

## 术语表

准确使用这些术语——不要用 "component"、"service"、"API" 或 "boundary" 替代。一致的词汇就是全部意义。

**Module**——任何有接口和实现的东西。故意规模不可知：函数、类、包、或跨层切片。*避免*：unit、component、service。

**Interface**——调用者必须知道才能正确使用模块的一切：类型签名，还有不变量、排序约束、错误模式、必需配置和性能特征。*避免*：API、signature（太窄——它们只指类型级表面）。

**Implementation**——模块内部的东西，它的代码体。与 **Adapter** 不同：一个东西可以是一个实现大的小 adapter（Postgres repo）或一个实现小的大 adapter（内存 fake）。当 seam 是主题时用 "adapter"；否则用 "implementation"。

**Depth**——接口的 leverage：（调用者或测试）每学一个接口单位能触发多少行为。模块是**深的**当大量行为藏在小接口后面，是**浅的**当接口几乎和实现一样复杂。

**Seam**（Michael Feathers）——一个你可以在不编辑该地方的情况下改变行为的地方；模块接口所在的*位置*。把 seam 放哪是它自己的设计决策，与 seam 后面是什么不同。*避免*：boundary（被 DDD 的 bounded context 重载）。

**Adapter**——在 seam 上满足接口的具体东西。描述*角色*（它填充什么槽），不是*实质*（里面是什么）。

**Leverage**——调用者从深度获得的东西：每学一个接口单位获得更多能力。一个实现回报 N 个调用站点和 M 个测试。

**Locality**——维护者从深度获得的东西：变更、bug、知识和验证集中在一个地方而不是散布在调用者中。修一次，到处修好。

## 深度 vs 浅层

**深度模块** = 小接口 + 大量实现：

```
┌─────────────────────┐
│   Small Interface   │  ← 少方法，简单参数
├─────────────────────┤
│                     │
│  Deep Implementation│  ← 复杂逻辑隐藏
│                     │
└─────────────────────┘
```

**浅模块** = 大接口 + 少量实现（避免）：

```
┌─────────────────────────────────┐
│       Large Interface           │  ← 多方法，复杂参数
├─────────────────────────────────┤
│  Thin Implementation            │  ← 只是透传
└─────────────────────────────────┘
```

设计接口时，问：

- 能减少方法数量吗？
- 能简化参数吗？
- 能把更多复杂性藏在里面吗？

## 原则

- **深度是接口的属性，不是实现的。** 深度模块内部可以由小的、可 mock 的、可交换的部分组成——它们只是不是接口的一部分。模块可以有**内部 seam**（私有的，给自己的测试用）以及接口处的**外部 seam**。
- **删除测试。** 想象删除模块。如果复杂性消失了，它是 pass-through。如果复杂性重新出现在 N 个调用者中，它在赚它的存在。
- **接口是测试表面。** 调用者和测试跨同一个 seam。如果你想*越过*接口测试模块，模块可能是错误的形状。
- **一个 adapter 意味着假设 seam。两个 adapter 意味着真实的。** 不要引入 seam 除非有东西实际上在它上面变化。

## 为可测试性设计

好的接口让测试自然：

1. **接受依赖，不要创建它们。**

   ```typescript
   // 可测试
   function processOrder(order, paymentGateway) {}

   // 难测试
   function processOrder(order) {
     const gateway = new StripeGateway();
   }
   ```

2. **返回结果，不要产生副作用。**

   ```typescript
   // 可测试
   function calculateDiscount(cart): Discount {}

   // 难测试
   function applyDiscount(cart): void {
     cart.total -= discount;
   }
   ```

3. **小表面面积。** 更少的方法 = 需要的测试更少。更少的参数 = 更简单的测试 setup。

## 关系

- 一个 **Module** 恰好有一个 **Interface**（它呈现给调用者和测试的表面）。
- **Depth** 是 **Module** 的属性，相对于其 **Interface** 衡量。
- **Seam** 是 **Module** 的 **Interface** 所在的地方。
- **Adapter** 坐在 **Seam** 上并满足 **Interface**。
- **Depth** 为调用者产生 **Leverage**，为维护者产生 **Locality**。

## 被拒绝的框架

- **深度作为实现行数与接口行数的比率**（Ousterhout）：奖励填充实现。我们改用深度即 leverage。
- **"Interface" 作为 TypeScript `interface` 关键字或类的公共方法**：太窄——这里的接口包括调用者必须知道的每个事实。
- **"Boundary"**：被 DDD 的 bounded context 重载。说 **seam** 或 **interface**。

## 深入

- **给定依赖深化一个集群**——见 [DEEPENING.md](DEEPENING.md)：依赖类别、seam 规范、和 replace-don't-layer 测试。
- **探索替代接口**——见 [DESIGN-IT-TWICE.md](DESIGN-IT-TWICE.md)：启动并行子 agent 以几种完全不同的方式设计接口，然后在深度、locality 和 seam 放置上比较。
