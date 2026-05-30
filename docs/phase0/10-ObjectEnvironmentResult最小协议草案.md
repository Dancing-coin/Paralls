# Object / Environment Result 最小协议草案

## 状态

- 状态：第一轮草案
- 作用：冻结 `Phase 0` demo 中物体 / 环境侧回给场景、角色和司命的最小结果协议
- 上游约束：
  - [01-Phase0启动方案.md](/d:/Projects/Paralls/docs/phase0/01-Phase0启动方案.md)
  - [03-Demo联调清单.md](/d:/Projects/Paralls/docs/phase0/03-Demo联调清单.md)
  - [08-PlayerInput最小协议草案.md](/d:/Projects/Paralls/docs/phase0/08-PlayerInput最小协议草案.md)

## 1. 统一定义

`Object / Environment Result` 指：

> 玩家、角色或司命对物体、环境进行交互后，由最小 `ESM` 回写出来的最小结构化结果。

它不等于：

- 完整 `ESM` 物理状态机
- 完整证据链
- 完整视觉事实系统

## 2. 当前最小结果集

`Phase 0` 当前建议只冻结 4 类结果：

1. `object_interaction_result`
2. `environment_state_result`
3. `visible_feedback_result`
4. `constraint_state_result`

## 3. 统一外壳

所有结果当前至少包含：

- `room_id`
- `source_type`
- `target_object_id` 或 `target_environment_id`
- `result_type`
- `causation_id`
- `producer_ts`

## 4. 各类结果

### 4.1 `object_interaction_result`

表示一次物体交互后的最小结构化结果。

建议最小字段：

- `result_type: object_interaction_result`
- `target_object_id`
- `interaction_type`
- `result_summary`
- `state_changed`

说明：

- 当前至少支撑“玩家点了物体 -> 世界给出结构化反馈”
- `state_changed` 用来区分只是查看，还是世界内状态发生了变化

### 4.2 `environment_state_result`

表示环境或物体状态变化对外可见的最小结果。

建议最小字段：

- `result_type: environment_state_result`
- `target_environment_id` 或 `target_object_id`
- `previous_state`
- `current_state`
- `change_summary`

说明：

- 当前只要求 1 段最小环境 / 物体状态变化
- 不要求完整连续物理仿真链

### 4.3 `visible_feedback_result`

表示玩家或角色可直接观察到的结果反馈。

建议最小字段：

- `result_type: visible_feedback_result`
- `target_object_id` 或 `target_environment_id`
- `feedback_mode`
- `feedback_payload`

说明：

- `feedback_mode` 可先很粗，如 `text / static_image / state_hint`
- 这条主要服务 `Phase 0` 演示，不是长期正式协议

### 4.4 `constraint_state_result`

表示一次交互、状态变化或环境请求没有成立时，`ESM` 返回的最小约束结果。

建议最小字段：

- `result_type: constraint_state_result`
- `target_object_id` 或 `target_environment_id`
- `constraint_type`
- `constraint_summary`

说明：

- 这条结果对 `Phase 0` 很重要，因为它证明世界不是“点一下就永远成功”
- 它也是后续正式 `ConstraintStateResult` 的前身

## 5. 不属于当前 Result 的内容

以下内容当前不应进入 `Phase 0` 的结果协议：

- 完整 `ESM` 数值状态
- 完整证据链事件树
- 完整视觉事实输出全集
- 多轮因果传播历史

## 6. 当前通过标准

若 `Phase 0` demo 中以下两件事都成立，则 `Object / Environment Result` 最小协议通过：

- 玩家点击物体后得到稳定结构化反馈
- 至少 1 段环境 / 物体状态变化可被回写并被角色或司命承接
- 至少 1 次失败交互能返回结构化约束结果

## 7. 一句话收束

当前 `Phase 0` 的 `Object / Environment Result` 不是完整世界状态协议，而是：

> 用来证明最小 `ESM` 会把玩家、角色或司命的交互结算成一个可观察、可承接、可继续被角色和司命利用的最小结果对象。
