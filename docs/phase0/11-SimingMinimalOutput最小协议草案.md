# Siming Minimal Output 最小协议草案

## 状态

- 状态：第一轮草案
- 作用：冻结 `Phase 0` demo 中司命作为最小叙事引擎时的输出协议
- 上游约束：
  - [01-Phase0启动方案.md](/d:/Projects/Paralls/docs/phase0/01-Phase0启动方案.md)
  - [03-Demo联调清单.md](/d:/Projects/Paralls/docs/phase0/03-Demo联调清单.md)
  - [09-AIOutput最小协议草案.md](/d:/Projects/Paralls/docs/phase0/09-AIOutput最小协议草案.md)
  - [10-ObjectEnvironmentResult最小协议草案.md](/d:/Projects/Paralls/docs/phase0/10-ObjectEnvironmentResult最小协议草案.md)

## 1. 统一定义

`Siming Minimal Output` 指：

> 司命在 `Phase 0` 中作为最小叙事引擎，对角色、环境或场景节奏施加的一次最小高层催化输出。

它不等于：

- 完整单局导演输出
- 完整高阶知识图谱确认事件
- 完整冲突生成器结果

## 2. 当前最小输出集

`Phase 0` 当前建议只冻结 3 类最小输出：

1. `narrative_nudge`
2. `attention_prompt`
3. `situation_shift`

## 3. 统一外壳

所有司命最小输出当前至少包含：

- `room_id`
- `output_type`
- `target_actor_id` 或 `target_environment_id`
- `causation_id`
- `producer_ts`

## 4. 各类输出

### 4.1 `narrative_nudge`

表示司命对某个角色施加一次最小叙事催化。

建议最小字段：

- `output_type: narrative_nudge`
- `target_actor_id`
- `nudge_summary`
- `nudge_intensity`

说明：

- 当前只要求让角色产生可观察的最小承接
- 不要求完整冲动系统

### 4.2 `attention_prompt`

表示司命让某个角色或玩家侧更容易注意到某个对象、事件或变化。

建议最小字段：

- `output_type: attention_prompt`
- `target_actor_id`
- `target_object_id` 或 `target_environment_id`
- `prompt_summary`

说明：

- 例如“让角色更关注门的变化”
- 当前只要求证明最小叙事引擎能把局部焦点推起来

### 4.3 `situation_shift`

表示司命触发一次局部情境变化，使场景从“静态状态”进入“有戏状态”。

建议最小字段：

- `output_type: situation_shift`
- `target_environment_id` 或 `target_actor_id`
- `shift_summary`
- `expected_effect`

说明：

- 这条最像“让戏发生起来”的最小版本
- 当前只要求它能被角色与环境承接

## 5. 不属于当前司命输出的内容

以下内容当前不应进入 `Phase 0` 的司命最小输出协议：

- 完整事实锁定链控制
- 完整多角色节奏编排
- 完整高阶知识确认家族
- 完整案件生成与冲突生成链

## 6. 当前通过标准

若 `Phase 0` demo 中至少出现 1 次司命输出，并且它能引发：

- 角色响应
或
- 环境 / 物体变化
或
- 玩家可观察到的局部剧情变化

则 `Siming Minimal Output` 最小协议通过。

## 7. 一句话收束

当前 `Phase 0` 的司命输出不是完整导演协议，而是：

> 用来证明司命能在最小场景里作为叙事引擎介入，并让角色、物体、环境共同形成一段短剧情变化的最小高层催化输出。
