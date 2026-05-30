# ESM 运行时对象 Schema 草案

## 1. 文档定位

- 状态：草案
- 主题：`ESM` 六个核心运行时对象的最小语义 schema
- 适用范围：`Phase 1` 单房间、单实例、单权威总线优先
- 作用：为 `ESM`、事件总线、角色智能体、司命和工作台之间的协议层对齐提供最小字段约束

本稿只冻结语义层 schema，不冻结：

- 最终编码格式
- JSON / Protobuf / FlatBuffers 选型
- 数据库存储结构
- 语言级类定义

## 2. 设计目标

当前要把以下 6 个对象稳定成可执行 schema 草案：

1. `ActionRequest`
2. `ActionResolutionResult`
3. `ConstraintStateResult`
4. `ObjectStateResult`
5. `EnvironmentStateResult`
6. `BodyStateResult`

目标不是做完整世界协议大全，而是先稳定回答三件事：

- 世界收到了什么请求
- 世界是否允许它成立
- 成立后到底改变了哪些物理事实

## 3. 共同前提

### 3.1 继承公共信封

除 `ActionRequest` 可作为内部请求对象存在外，其余所有结果对象都必须能落入事件总线公共信封。

默认公共字段采用 canonical 口径：

- `event_id`
- `event_type`
- `room_id`
- `scene_id`
- `zone_id`
- `source`
- `routing`
- `priority`
- `ttl`
- `durability`
- `producer_ts`
- `causation_id`
- `correlation_id`

### 3.2 边界约束

- `ESM` 只结算物理事实，不解释证据意义
- 司命只能发高层 `environment_request`
- 角色不能绕过 `ESM` 直接宣告世界结果成立
- 角色对 `ESM` 结果的承接分为“自身身体直通路径”和“外部世界完整感知链”

### 3.3 对象族关系

最小闭环应固定为：

```text
ActionRequest
-> ActionResolutionResult / ConstraintStateResult
-> ObjectStateResult / EnvironmentStateResult / BodyStateResult
```

## 4. ActionRequest

### 4.1 用途

回答：

- 当前世界收到了什么动作 / 环境请求
- 请求来自谁
- 指向哪些实体

### 4.2 字段

- `request_id`
- `request_type`
- `room_id`
- `scene_id`
- `zone_id`
- `source`
- `target_entity_refs`
- `action_profile`
- `intent_strength`
- `constraints_hint`
- `producer_ts`
- `causation_id`
- `correlation_id`

### 4.3 字段说明

#### `request_id`

本次请求唯一 ID。

#### `request_type`

建议枚举：

- `interact`
- `move`
- `use_object`
- `apply_force`
- `toggle_state`
- `environment_request`

#### `source`

建议结构：

- `source.layer`
- `source.system`
- `source.actor_id`
- `source.object_id` 可选

#### `target_entity_refs`

数组。允许：

- 物体
- 区域
- 身体代理

#### `action_profile`

该请求希望触发的动作轮廓，例如：

- `open`
- `push`
- `ignite`
- `conceal`
- `shift_light`

#### `intent_strength`

建议枚举：

- `hint`
- `low`
- `medium`
- `high`

#### `constraints_hint`

调用方已知的先验约束提示，例如：

- 预期有锁
- 预期有遮挡

### 4.4 语义约束

- `ActionRequest` 是“请求”，不是已成立事实
- 它不能直接被角色或司命当成物理真值

## 5. ActionResolutionResult

### 5.1 用途

回答：

- 本次请求在 `ESM` 看来是否成立
- 若成立，结算到了什么程度

### 5.2 字段

- `result_id`
- `request_ref`
- `resolution_status`
- `resolved_entities`
- `applied_state_changes`
- `stable_state_summary`
- `producer_ts`
- `causation_id`
- `correlation_id`

### 5.3 字段说明

#### `resolution_status`

建议枚举：

- `accepted`
- `partially_accepted`
- `rejected`

#### `resolved_entities`

本次结算实际触及的实体列表。

#### `applied_state_changes`

本次结算实际应用的状态变化摘要。

#### `stable_state_summary`

结算后世界进入的稳定态摘要。

### 5.4 语义约束

- `ActionResolutionResult` 回答的是“请求是否成立”
- 它不替代具体结果对象
- 即使 `accepted`，仍可继续生成更具体的：
  - `ObjectStateResult`
  - `EnvironmentStateResult`
  - `BodyStateResult`

## 6. ConstraintStateResult

### 6.1 用途

回答：

- 本次请求为什么没成立
- 被什么约束挡住

### 6.2 字段

- `constraint_result_id`
- `request_ref`
- `constraint_type`
- `constraint_code`
- `constraint_summary`
- `blocking_entity_refs`
- `producer_ts`
- `causation_id`
- `correlation_id`

### 6.3 字段说明

#### `constraint_type`

建议枚举：

- `distance_constraint`
- `occlusion_constraint`
- `collision_constraint`
- `lock_state_constraint`
- `material_constraint`
- `state_transition_constraint`
- `cooldown_constraint`
- `resource_constraint`

#### `constraint_code`

机器可读的细粒度失败码，例如：

- `door_locked`
- `line_of_sight_blocked`
- `out_of_range`

#### `constraint_summary`

给工作台 / 日志的人类可读摘要。

#### `blocking_entity_refs`

哪些实体构成了当前阻塞。

### 6.4 语义约束

- 失败时优先生成 `ConstraintStateResult`
- 不得伪造成功结果代替失败解释

## 7. ObjectStateResult

### 7.1 用途

回答：

- 某个物体的物理状态发生了什么变化

### 7.2 字段

- `object_result_id`
- `request_ref` 可选
- `entity_id`
- `previous_state`
- `current_state`
- `change_summary`
- `visibility_state`
- `producer_ts`
- `causation_id`
- `correlation_id`

### 7.3 字段说明

#### `entity_id`

发生变化的物体实体。

#### `previous_state`

变化前状态，例如：

- `closed`
- `intact`
- `visible`

#### `current_state`

变化后状态，例如：

- `ajar`
- `broken`
- `hidden`

#### `visibility_state`

可见性摘要，可选建议枚举：

- `hidden`
- `partially_visible`
- `visible`

### 7.4 语义约束

- `ObjectStateResult` 只描述物体物理状态
- 不描述证据意义

## 8. EnvironmentStateResult

### 8.1 用途

回答：

- 某个区域环境场或环境结构发生了什么变化

### 8.2 字段

- `environment_result_id`
- `request_ref` 可选
- `zone_id`
- `field_delta_summary`
- `structural_change_summary`
- `affected_entity_refs`
- `producer_ts`
- `causation_id`
- `correlation_id`

### 8.3 字段说明

#### `field_delta_summary`

建议是环境场变化摘要，至少允许：

- `temperature_delta`
- `humidity_delta`
- `smoke_density_delta`
- `noise_level_delta`
- `light_level_delta`
- `visibility_level_delta`

#### `structural_change_summary`

环境结构变化摘要，例如：

- `door_opened_partial`
- `light_dimmed`
- `smoke_zone_created`

#### `affected_entity_refs`

受本次环境变化影响的实体集合。

### 8.4 语义约束

- `EnvironmentStateResult` 描述区域或结构环境变化
- 不描述角色主观感受

## 9. BodyStateResult

### 9.1 用途

回答：

- 某角色身体事实是否进入某状态

### 9.2 字段

- `body_result_id`
- `request_ref` 可选
- `actor_id`
- `body_state_type`
- `previous_body_state`
- `current_body_state`
- `severity`
- `producer_ts`
- `causation_id`
- `correlation_id`

### 9.3 字段说明

#### `body_state_type`

建议枚举：

- `pain`
- `injury`
- `balance`
- `oxygen`
- `fatigue`
- `mobility_constraint`

#### `previous_body_state`

例如：

- `stable`
- `normal`

#### `current_body_state`

例如：

- `impaired`
- `critical`
- `restricted`

#### `severity`

建议枚举：

- `low`
- `medium`
- `high`
- `critical`

### 9.4 语义约束

- `BodyStateResult` 是身体物理事实
- 角色情绪解释不在这里
- 若进入角色自身承接路径，可进一步映射成 `SelfBodyPerceivedEvent`

## 10. 六对象关系

### 10.1 分工

- `ActionRequest`
  回答：世界收到了什么请求

- `ActionResolutionResult`
  回答：请求是否被接受

- `ConstraintStateResult`
  回答：请求为什么没成立

- `ObjectStateResult`
  回答：物体状态如何变化

- `EnvironmentStateResult`
  回答：环境场或环境结构如何变化

- `BodyStateResult`
  回答：角色身体事实如何变化

### 10.2 依赖关系

- `ActionResolutionResult` 与 `ConstraintStateResult` 依赖 `ActionRequest`
- `ObjectStateResult / EnvironmentStateResult / BodyStateResult` 通常依赖 `ActionResolutionResult`
- `ConstraintStateResult` 成立时，不应同时伪造成功结果对象

## 11. 最小语义示例

### 11.1 ActionRequest

```json
{
  "request_id": "req_001",
  "request_type": "environment_request",
  "room_id": "room_01",
  "scene_id": "scene_study",
  "zone_id": "zone_door",
  "source": {
    "layer": "L2",
    "system": "siming",
    "actor_id": "siming_orchestrator"
  },
  "target_entity_refs": ["door_01"],
  "action_profile": "open",
  "intent_strength": "low",
  "constraints_hint": ["possible_lock"],
  "producer_ts": "2026-05-31T11:32:00Z",
  "causation_id": "cand_001",
  "correlation_id": "corr_001"
}
```

### 11.2 ConstraintStateResult

```json
{
  "constraint_result_id": "con_001",
  "request_ref": "req_001",
  "constraint_type": "lock_state_constraint",
  "constraint_code": "door_locked",
  "constraint_summary": "Door is still locked",
  "blocking_entity_refs": ["door_01", "lock_01"],
  "producer_ts": "2026-05-31T11:32:00Z",
  "causation_id": "req_001",
  "correlation_id": "corr_001"
}
```

## 12. 当前冻结结论

当前先冻结 `ESM` 六个核心运行时对象的最小语义 schema：

- `ActionRequest`
- `ActionResolutionResult`
- `ConstraintStateResult`
- `ObjectStateResult`
- `EnvironmentStateResult`
- `BodyStateResult`

后续若进入具体协议实现，必须继续保持以下边界：

- 不把请求伪装成既成结果
- 不把失败伪装成成功
- 不把环境 / 物体 / 身体结果混成一个单对象
- 不让 `ESM` 结果承担证据解释或角色心理解释
