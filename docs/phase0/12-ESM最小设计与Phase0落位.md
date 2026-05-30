# ESM 最小设计与 Phase 0 落位

## 状态

- 状态：第一轮草案
- 作用：冻结 `Phase 0` demo 中 `ESM` 的最小设计、最小落位和最小联调范围
- 上游约束：
  - [ESM设计文档.md](/d:/Projects/Paralls/docs/phase1/core/01-运行时核心/ESM设计文档.md)
  - [Godot源码底层基础设施与运行时约束.md](/d:/Projects/Paralls/docs/phase1/core/00-总纲/Godot源码底层基础设施与运行时约束.md)
  - [01-Phase0启动方案.md](/d:/Projects/Paralls/docs/phase0/01-Phase0启动方案.md)
  - [03-Demo联调清单.md](/d:/Projects/Paralls/docs/phase0/03-Demo联调清单.md)
  - [10-ObjectEnvironmentResult最小协议草案.md](/d:/Projects/Paralls/docs/phase0/10-ObjectEnvironmentResult最小协议草案.md)

## 1. 文档目标

本文件只回答：

1. `Phase 0` demo 里的 `ESM` 到底做什么
2. 它最小要依赖 Godot 的哪些底层组件
3. 它最小需要打通哪些输入输出

它不回答：

- 完整 `ESM` 物理系统
- 完整天气 / 生长 / 长时间环境传播
- 完整证据链闭环

## 2. `Phase 0` 中的 `ESM` 定位

`Phase 0` 中的 `ESM` 不是完整环境模拟器，而是：

> 支撑“单个物体交互 + 单段环境或物体状态变化 + 可回写结构化结果”的最小环境状态执行内核。

它至少要证明：

- 世界状态不是假 UI
- 交互后真的会有状态变化
- 状态变化可以被角色和司命承接

## 3. `Phase 0` 最小职责

当前只冻结 4 项最小职责：

1. 关键物体状态变化
2. 最小环境状态变化
3. 动作是否成立的最小结算
4. 结构化结果回写

### 3.1 关键物体状态变化

至少支持 1 个关键物体：

- 可查看
- 可触发
- 可进入新状态

例如：

- `idle -> opened`
- `visible -> hidden`
- `dry -> wet`

### 3.2 最小环境状态变化

至少支持 1 段最小环境变化：

- 门半开
- 灯变暗
- 区域出现烟雾 / 噪音 / 遮挡

### 3.3 动作是否成立的最小结算

`Phase 0` 必须能回答：

- 玩家或角色这次交互是否真的成立
- 如果不成立，是因为什么约束失败

### 3.4 结构化结果回写

`Phase 0` 必须把结果回写为结构化对象，而不是只改客户端表现。

## 4. 与 Godot 底层组件的最小对齐

基于项目内 `/.tmp/godot/` 源码，`Phase 0` 最小 `ESM` 建议优先依赖：

### 4.1 `World3D` + `PhysicsDirectSpaceState3D`

用于：

- 空间查询
- 接触点判断
- 可行性预检

### 4.2 `Area3D`

用于：

- 最小区域触发
- 玩家 / 角色进入关键区域
- 环境变化触发条件

### 4.3 `RayCast3D` / `ShapeCast3D`

用于：

- 交互目标确认
- 近距离碰撞候选收束

### 4.4 `PhysicsBody3D`

用于：

- 推门 / 受阻 / 简单位移约束

### 4.5 `SceneTree` / `MessageQueue`

用于：

- 主线程安全地把状态变化应用到场景表现

## 5. `Phase 0` 最小输入

`ESM` 当前至少接收：

1. `interact_intent`
2. 最小 `environment_request` 或 `situation_shift`
3. 系统时钟 / tick

### 5.1 来自玩家

- `target_object_id`
- `interaction_type`

### 5.2 来自司命

- `target_environment_id` 或 `target_object_id`
- 最小环境变化意图

## 6. `Phase 0` 最小输出

建议正式固定 4 类：

1. `object_interaction_result`
2. `environment_state_result`
3. `visible_feedback_result`
4. `constraint_state_result`

### 6.1 `constraint_state_result`

这是 `Phase 0` 当前应该补上的最小对象，用来回答：

- 为什么这次交互没成立
- 是距离不够、遮挡、锁定、碰撞还是状态不允许

## 7. `Phase 0` 最小状态机建议

当前只建议冻结最小状态机，不做扩展：

### 7.1 物体状态机

- `idle -> interacted`
- `closed -> opened`
- `visible -> hidden`
- `intact -> disturbed`

### 7.2 环境状态机

- `stable -> shifted`
- `lit -> dimmed`
- `clear -> obscured`

## 8. 联调观察点

联调 `ESM` 时至少看 3 件事：

1. 场景里真的变化了什么
2. 结构化结果回写了什么
3. 角色或司命有没有承接这次变化

## 9. `Phase 0` 成功标准

若以下 4 件事同时成立，则 `Phase 0` 的 `ESM` 最小设计通过：

1. 至少 1 次物体交互成功结算
2. 至少 1 次环境或物体状态变化成功回写
3. 至少 1 次失败交互能给出结构化约束原因
4. 至少 1 次状态变化被角色或司命承接

## 10. 一句话收束

`Phase 0` 里的 `ESM` 不是完整世界模拟器，而是：

> 建立在 Godot 现成空间查询、区域触发和物理约束入口之上，用来证明“交互真的改变世界状态，而且这个改变能被回写、被承接”的最小环境执行内核。
