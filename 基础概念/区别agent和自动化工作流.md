# 区分 Agent 和自动化工作流

> 用来梳理「Agent」与「传统自动化工作流」之间的概念差异和适用场景。

## 核心概念对比

- 什么是 Agent
- 什么是自动化工作流（如流水线、定时任务）
- 二者的共同点与差异

## 典型使用场景

- 什么时候更适合用 Agent
- 什么时候传统工作流更简单稳定

## 设计思路对比

- 状态与记忆的处理方式
- 决策逻辑：规则 vs 智能
- 可观测性与可控性

## 实践示例

- 选一个具体问题，分别用「Agent」和「自动化工作流」的思路来设计


### 工作流

标准工作流

![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1771896701948-0b232b5e-b2f1-45f6-ae51-aeb4fbd59504.png)

下面的接入了四个不同的新闻来源，提取新闻热点，并且接入了ollama，ollama运行了一个本地模型，做成一个干净的格式，再写到飞书。这也是工作流，而不是agent

![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1771897029120-fba52aea-b200-47be-9505-6b13c01ec388.png)

![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1771897310918-84549c4b-41f0-4ff6-bc04-12444c995092.png)

### Agent

三个关键组件：大脑 记忆 工具

![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1771897710513-b5beec7e-4495-4713-9f9b-f5d38b216591.png)

![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1771897727948-ed18f4ed-a27a-4db4-8d6b-7dd1848f6cf6.png)

可能是之前的对话，也可能是文档、知识库

![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1771897895109-22af900a-6ee8-442b-af60-6dae20445453.png)

工具是agent与外界互动的主要方式，可以有API

![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1771897980695-dcec4175-393b-407e-88fc-24659f89c3fe.png)

原则

![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1771915545604-fd9847a5-b183-4b73-9e12-b57fa6e4af64.png)

![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1771915822302-c5cfb4da-7c85-4e66-9fa8-ac4a329391d2.png)