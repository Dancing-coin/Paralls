# 运行时四核边界一致性审计：司命 / 事件总线 / 角色智能体 / ESM

## 状态

- 状态：第一轮审计稿
- 审计范围：
  - [司命设计文档.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/司命设计文档.md)
  - [事件总线与感知链路设计.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/事件总线与感知链路设计.md)
  - [角色智能体设计文档.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/角色智能体设计文档.md)
  - [ESM设计文档.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/ESM设计文档.md)
  - 以及各自关键协作文档
- 作用：识别当前四个运行时核心在职责边界、消息分层、物理真值、角色承接和高层催化上的一致性状态

## 1. 审计结论摘要

总体判断：

- 主边界已经基本成型
- “谁生产事实、谁解释事实、谁催化叙事、谁承接主观意义”这四条主链已大体对齐
- 仍存在若干“命名不统一、对象族未完全统一、字段口径有残留旧版本、阶段归属未完全收紧”的问题

当前建议结论：

- 可以进入下一阶段的 contract / schema / task list 设计
- 但在正式开始实现前，必须先补掉本审计中的 `P0 / P1` 缺口

## 2. 已对齐的核心边界

### 2.1 司命不是低层控角器

当前四套文档已基本一致：

- 司命只能回写高层消息
- 角色必须经过 `L2 / L3 / 三重过滤器`
- 不能直接写角色最终动作、最终信念或低层姿态

结论：已对齐

### 2.2 `ESM` 不是司命下属

当前四套文档已基本一致：

- `ESM` 是 `L1` 独立环境状态管理器
- 司命只发高层环境请求
- 物理结果以 `ESM` 结算为准

结论：已对齐

### 2.3 角色不直接消费全局原始事实流

当前四套文档已基本一致：

- 司命接收更广的原始/结构化事件
- 角色接收的是 `Per-Character` 感知事件
- `ESM` 结果进入总线后仍需经过感知链

结论：已对齐

### 2.4 事件总线不负责业务解释

当前四套文档已基本一致：

- 总线负责承载与分发
- 不负责证据解释
- 不负责角色感知解释
- 不负责环境物理结算

结论：已对齐

## 3. 部分对齐但仍有残留差异的边界点

### 3.1 时间字段命名不完全统一

现状：

- 角色智能体与事件总线契约已偏向 `producer_ts`
- 司命与事件总线契约仍大量使用 `world_ts`
- 司命内部状态对象又保留 `world_ts`

判断：

- 这不一定是错误，但当前缺少“什么时候用 `producer_ts`，什么时候用 `world_ts`”的统一说明

风险：

- 后续 schema 设计会出现时间字段重复、语义冲突或乱用

建议：

- 定一条全局规则：
  - 事件信封统一 `producer_ts`
  - 房间 / 模型快照 / 读模型可保留 `world_ts`

优先级：`P0`

### 3.2 `ESM` 输出对象命名不完全统一

现状：

- [ESM设计文档.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/ESM设计文档.md) 推荐：
  - `ActionResolutionResult`
  - `ObjectStateResult`
  - `EnvironmentStateResult`
  - `BodyStateResult`
  - `ConstraintStateResult`
- [角色智能体/18-角色智能体与 ESM 协作协议.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/角色智能体/18-角色智能体与 ESM 协作协议.md) 使用：
  - `EnvironmentImpactResult`

判断：

- 这是命名未完全收口，不是架构冲突

风险：

- 后续接口定义会出现同义对象并存

建议：

- 把 `EnvironmentImpactResult` 改收敛到 `EnvironmentStateResult`

优先级：`P0`

### 3.3 司命与事件总线契约中的信封字段仍带旧口径

现状：

- [司命/10-司命与事件总线契约.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/司命/10-司命与事件总线契约.md) 仍建议：
  - `producer`
  - `source_actor_id`
  - `target_actor_ids`
  - `world_ts`
- [角色智能体/19-角色智能体与事件总线契约.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/角色智能体/19-角色智能体与事件总线契约.md) 已切换到：
  - `source.layer`
  - `source.system`
  - `source.actor_id`
  - `routing.*`
  - `producer_ts`

判断：

- 这是当前最明确的结构性未对齐点

风险：

- 一旦开始做总线 schema，会直接出现两套 canonical

建议：

- 以 [角色智能体/19-角色智能体与事件总线契约.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/角色智能体/19-角色智能体与事件总线契约.md) 的 canonical 口径为主
- 回改 [司命/10-司命与事件总线契约.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/司命/10-司命与事件总线契约.md)

优先级：`P0`

### 3.4 `ESM` 阶段归属仍有历史残留

现状：

- [ESM设计文档.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/ESM设计文档.md) 已按 Phase 1 基础交互冻结
- 但 [04-Phase映射表.md](/d:/Projects/Paralls/docs/consolidation/04-Phase映射表.md) 仍写：
  - `ESM 基础交互` 不在 Phase 1，而在 Phase 2

判断：

- 这与当前你刚补进 `Phase 0` 的最小 `ESM` 设计已经不一致

风险：

- 后续排期、待办、实现前计划会误判 `ESM` 的当前必要性

建议：

- 把 `ESM 基础交互` 调整为：
  - `Phase 0`：最小验证
  - `Phase 1`：基础交互正式纳入

优先级：`P0`

## 4. 缺口：当前没完全写清的地方

### 4.1 `L1`、事件总线、`ESM` 三者的动作请求矩阵还没冻结

缺什么：

- 谁可以发 `ActionRequest`
- 谁可以发 `environment_request`
- 哪些请求先走客户端前检
- 哪些请求必须走 `ESM` 权威结算

影响：

- 正式 schema 难写

建议新增：

- `ESM/02-动作结算与约束接口`

优先级：`P1`

### 4.2 `SelfBodyPerceivedEvent` 与 `BodyStateResult` 的映射链还没正式冻结

缺什么：

- 什么时候 `BodyStateResult` 可直接变成 `SelfBodyPerceivedEvent`
- 什么时候仍要走完整感知链

影响：

- 角色身体状态与环境状态的承接实现会不一致

建议新增：

- 角色与 `ESM` 协作协议补一张映射表

优先级：`P1`

### 4.3 司命 `environment_request` 与 `ESM ActionResolutionResult` 的失败码/约束码未统一

缺什么：

- 失败码族
- 约束标签族

影响：

- 工作台和 replay 解释会断裂

建议新增：

- `ESM` 专题文档中增加约束码族最小表

优先级：`P1`

## 5. 冲突等级评估

### `P0` 必须先修

1. 事件总线 canonical 信封字段两套口径并存
2. `ESM` 输出对象命名未完全统一
3. `ESM` 在 Phase 映射中的阶段归属与现状不一致

### `P1` 应尽快补

1. 动作请求矩阵未冻结
2. `BodyStateResult -> SelfBodyPerceivedEvent` 映射未冻结
3. `environment_request` / `ConstraintStateResult` 约束码未冻结

### `P2` 后续优化

1. 工作台联跳对象统一主键策略
2. 各系统 replay 说明图统一格式

## 6. 建议修改清单

### 立即修改

1. [司命/10-司命与事件总线契约.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/司命/10-司命与事件总线契约.md)
- 对齐到 `source.* / routing.* / producer_ts`

2. [角色智能体/18-角色智能体与 ESM 协作协议.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/角色智能体/18-角色智能体与 ESM 协作协议.md)
- 将 `EnvironmentImpactResult` 收敛到 `EnvironmentStateResult`

3. [04-Phase映射表.md](/d:/Projects/Paralls/docs/consolidation/04-Phase映射表.md)
- 修正 `ESM 基础交互` 的阶段口径

### 紧接着补

1. `ESM/02-动作结算与约束接口`
2. `ESM/05-ESM与事件总线契约`
3. `ESM/06-ESM与角色智能体协作协议`
4. `ESM/07-ESM与司命协作协议`

## 7. 一句话收束

当前四核主边界已经大体成立，但还没到可以“无痛开始实现”的程度。真正的阻塞点不是抽象架构，而是三件很具体的事：

- 事件信封 canonical 口径未全量统一
- `ESM` 输出对象命名未全量统一
- `ESM` 的阶段归属与动作请求矩阵仍未完全冻结
