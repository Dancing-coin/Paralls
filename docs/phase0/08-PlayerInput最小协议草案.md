# Player Input 最小协议草案

## 状态

- 状态：第一轮草案
- 作用：冻结 `Phase 0` demo 中从 Godot 客户端送往后端的最小玩家输入协议
- 上游约束：
  - [01-Phase0启动方案.md](/d:/Projects/Paralls/docs/phase0/01-Phase0启动方案.md)
  - [03-Demo联调清单.md](/d:/Projects/Paralls/docs/phase0/03-Demo联调清单.md)

## 1. 统一定义

`Player Input` 在当前 demo 中不指键鼠原始输入，而指：

> Godot 客户端在本地收束后的结构化玩家意图事件。

因此：

- 键盘 / 鼠标 / 视角变化留在 Godot 本地
- 送到后端的是“玩家想做什么”，不是“玩家按下了什么”

## 2. 当前最小输入集

`Phase 0` 当前建议只冻结 4 类 `Player Input`：

1. `move_intent`
2. `dialogue_submit`
3. `interact_intent`
4. `focus_target_change`

## 3. 统一外壳

所有 `Player Input` 当前至少包含：

- `player_id`
- `room_id`
- `actor_id`
- `intent_type`
- `producer_ts`

其中：

- `player_id`：现实操作者或旅人侧标识
- `actor_id`：当前单局内被驾驶的剧中角色实例
- `room_id`：当前单局会话容器
- `producer_ts`：客户端产生该意图的时间

## 4. 各类意图

### 4.1 `move_intent`

表示玩家想移动到哪里，或开始 / 停止移动。

建议最小字段：

- `intent_type: move`
- `move_mode`：`start / stop / update`
- `direction` 或 `target_point`

说明：

- `Phase 0` 可先只用 `target_point`
- 不需要把每帧键位变化上送后端

### 4.2 `dialogue_submit`

表示玩家向某个角色发出一句话。

建议最小字段：

- `intent_type: dialogue`
- `target_actor_id`
- `content`
- `context_hint` 可选

说明：

- 当前只要求支撑最小对话闭环
- 不要求完整多轮上下文控制协议

### 4.3 `interact_intent`

表示玩家想和某个物体交互。

建议最小字段：

- `intent_type: interact`
- `target_object_id`
- `interaction_type`

说明：

- `Phase 0` 至少只要支撑 1 个关键物体
- `interaction_type` 可先很粗，如 `inspect / use / trigger`

### 4.4 `focus_target_change`

表示玩家把注意力切到谁或什么上。

建议最小字段：

- `intent_type: focus`
- `target_actor_id` 或 `target_object_id`

说明：

- 这类输入在 `Phase 0` 可选，但建议保留
- 它可用于角色回应、最小凝视、对话对象确定和司命催化目标选择

## 5. 不属于当前 Player Input 的内容

以下内容当前不应直接上送后端业务层：

- `W/A/S/D pressed`
- `mouse delta`
- `camera yaw / pitch`
- 原始点击 down / up 流
- 每帧本地移动原始输入噪音

它们属于：

- Godot 本地输入层
- Godot 本地表现总线

不属于：

- 后端最小玩家意图协议

## 6. 当前通过标准

若 `Phase 0` demo 中以下四类意图都能被后端正确承接并产生可观察结果，则 `Player Input` 最小协议通过：

- 移动意图
- 对话提交
- 物体交互
- 目标聚焦变化（若本轮启用）

## 7. 一句话收束

当前 `Phase 0` 的 `Player Input` 不是设备输入流，而是：

> 用于驱动角色、物体、环境与司命最小闭环的结构化玩家意图事件。
