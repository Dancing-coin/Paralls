# Phase 0 Demo 节点职责速查表

## 1. 文档目标

本文档为 `Phase 0 Demo 技术构成图` 和 `最小链路图` 提供一张节点职责速查表。

用途：

- 给未来工程师快速判断每个盒子做什么
- 防止把认知层、执行层、视觉事实层和事件总线职责混掉

本文件定位为：

- 职责字典
- 边界速查表

它不负责：

- 解释完整事件链
- 代替技术构成图

## 2. 速查表

| 节点 | 一句话职责 | 不做什么 |
| :--- | :--- | :--- |
| `Player Input` | 收集玩家主动输入，映射为角色级 intent | 不直接改世界结果 |
| `Godot Client` | 本地高频具身执行器 + 玩家输入终端 | 不做角色认知真值判断 |
| `Character Replica A/B` | 客户端角色副本，承载本地表现与采样链 | 不持有后端权威知识状态 |
| `Object` | 被拿取、隐藏、展示、移动的世界内对象 | 不承载角色心理解释 |
| `Environment / ESM` | 物理和环境状态结算 | 不做心理解释，不替代角色理解 |
| `Authority Event Bus` | 世界级事实、角色认知、司命裁决与 replay/audit 总线 | 不收骨骼高频流 |
| `Character Service` | 感知、理解、规划、角色侧主观链 | 不做 Godot 本地骨骼执行 |
| `Siming` | 公平裁判型导演，做失衡判断和最小干预 | 不直接控角，不直接写物理结果 |
| `Visual Fact System` | 把 Godot 本地已成立的可见状态转成结构化视觉事实 | 不做最终心理解释 |
| `Local Presentation Bus` | Godot 内部表现同步、revision 与 debug 副通道 | 不做业务真值传播 |
| `Replay / Audit` | 记录世界链和司命链路，支持回放与工作台解释 | 不做运行时决策 |

## 3. 一句话收束

这张表的意义是：任何人看 `Phase 0` Demo 图时，都能立刻分清“谁负责世界真值、谁负责角色主观链、谁负责本地表现、谁负责桥接”。

## 4. 上位约束

- `Godot Client` 与 `Local Presentation Bus` 的边界解释，统一以上游 [Godot源码底层基础设施与运行时约束.md](/d:/Projects/Paralls/docs/phase1/core/00-总纲/Godot源码底层基础设施与运行时约束.md) 为准。
