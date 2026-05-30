# AI Output 最小协议草案

## 状态

- 状态：第一轮草案
- 作用：冻结 `Phase 0` demo 中角色智能体回给场景、玩家和后端协调层的最小输出协议
- 上游约束：
  - [01-Phase0启动方案.md](/d:/Projects/Paralls/docs/phase0/01-Phase0启动方案.md)
  - [03-Demo联调清单.md](/d:/Projects/Paralls/docs/phase0/03-Demo联调清单.md)
  - [08-PlayerInput最小协议草案.md](/d:/Projects/Paralls/docs/phase0/08-PlayerInput最小协议草案.md)

## 1. 统一定义

`AI Output` 在当前 demo 中指：

> 角色智能体经过最小理解与响应后，回给场景、玩家和协调层的结构化输出。

它不等于：

- 角色内部 chain-of-thought
- 完整信念状态快照
- 完整具身表达控制流

## 2. 当前最小输出集

`Phase 0` 当前建议只冻结 4 类 `AI Output`：

1. `dialogue_response`
2. `attention_shift`
3. `interaction_reaction`
4. `siming_hook_signal`

## 3. 统一外壳

所有 `AI Output` 当前至少包含：

- `actor_id`
- `room_id`
- `output_type`
- `causation_id`
- `producer_ts`

其中：

- `actor_id`：当前输出的角色实例
- `room_id`：当前单局会话容器
- `causation_id`：这次输出由哪次输入或哪条事件触发
- `producer_ts`：输出产生时间

## 4. 各类输出

### 4.1 `dialogue_response`

表示角色对玩家或另一角色的最小语言回应。

建议最小字段：

- `output_type: dialogue_response`
- `target_actor_id`
- `content`
- `tone`
- `tts_required`

说明：

- `Phase 0` 中最重要的是让“输入一句话 -> 角色回一句话 -> 语音播出来”闭环成立
- `tts_required` 当前默认建议保留，方便后端或客户端决定是否走 TTS

### 4.2 `attention_shift`

表示角色把注意力转向某个对象、角色或环境变化。

建议最小字段：

- `output_type: attention_shift`
- `target_actor_id` 或 `target_object_id`
- `attention_reason`

说明：

- 这不是完整 gaze 协议
- 只是最小“角色注意到了什么”的回写
- 它可作为后续场景表现或司命最小催化的观察点

### 4.3 `interaction_reaction`

表示角色对物体或环境状态变化的最小承接结果。

建议最小字段：

- `output_type: interaction_reaction`
- `target_object_id` 或 `target_environment_id`
- `reaction_type`
- `reaction_text` 可选

说明：

- 例如“看到门半开后出声提醒”
- 或“注意到物体变化后给出一句评价”
- 当前只要求证明角色不是纯问答壳

### 4.4 `siming_hook_signal`

表示角色输出里那些可被最小司命引擎继续观察和利用的钩子。

建议最小字段：

- `output_type: siming_hook_signal`
- `signal_type`
- `signal_strength`
- `summary`

说明：

- 这不是完整司命协作协议
- 只是给 `Phase 0` 留一个最小观察点
- 例如“角色明显被环境变化吸引”“角色开始怀疑另一角色”“角色对物体变化做出警觉反应”

## 5. 不属于当前 AI Output 的内容

以下内容当前不应直接作为 `Phase 0` 的 `AI Output` 回写：

- 完整人格状态
- 内部怀疑值 / 信任值全量快照
- 完整会话确认事件
- 全量 FACS / 骨骼 / 动作树参数
- 未过滤的内部推理文本

它们属于：

- 角色内部状态
- `Phase 1` 更完整的运行时协议

## 6. 当前通过标准

若 `Phase 0` demo 中以下 4 类输出至少有 3 类能被稳定观察到，则 `AI Output` 最小协议通过：

- 语言回应
- 注意力转移
- 物体 / 环境承接反应
- 司命可观察钩子

其中 `dialogue_response` 必须是必选项。

## 7. 一句话收束

当前 `Phase 0` 的 `AI Output` 不是完整角色协议，而是：

> 用于证明角色能说话、能注意到变化、能对物体和环境做出承接，并能给最小司命引擎留下可利用观察点的最小结构化输出。
