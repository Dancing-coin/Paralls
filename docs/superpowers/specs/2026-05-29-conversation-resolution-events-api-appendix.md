# 会话确认事件 API 附录（v0.1）

## 1. 文档定位

- 状态：附录草案
- 主题：`MembershipResolution`、`AccessResolution`、`KnowledgeResolution`、`AwarenessResolution`、`PrivacyRiskResolution`、`ConversationBoundaryChange` 六类事件的实现前接口附录
- 适用范围：`Phase 1` 单房间单实例优先
- 作用：把已经冻结的语义 schema 转成更接近工程实现的接口页，供服务端、运行时和角色侧联调

本附录不冻结：

- 最终编码格式
- 具体传输协议
- SDK 形态
- 持久化库表结构

## 2. 共用接口约束

### 2.1 事件外壳

六类确认事件都继承统一公共信封：

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
- `payload`

### 2.2 角色分工

- `L1/ESM`：生产原始空间、声学、遮挡和交互事实
- 事件总线：承载公共信封与分发
- 司命高阶知识图谱：输出确认事件
- 角色智能体：消费自身可知版本并更新本地状态与记忆

### 2.3 核心约束

- 这六类事件都不是 `L1` 原始事实
- 它们不得回写伪装成原始空间事件
- `KnowledgeResolution` 不得越过 `AccessResolution` 的时间边界
- `PrivacyRiskResolution` 不等于角色本地 `privacy_pressure`
- `AwarenessResolution` 必须能表达会话所需的二阶知晓目标

## 3. 事件目录

### 3.1 最小闭环事件

- `MembershipResolution`
- `AccessResolution`
- `KnowledgeResolution`

### 3.2 支撑与修正事件

- `AwarenessResolution`
- `PrivacyRiskResolution`
- `ConversationBoundaryChange`

## 4. MembershipResolution

### 4.1 用途

回答：

- 某角色当前在某场会话中的成员身份是什么
- 该身份从何时开始成立
- 是否刚发生状态切换

### 4.2 生产者

- 司命高阶知识图谱

### 4.3 主要消费者

- 角色会话参与状态子模块
- 角色 `L2/L3/L4`
- 司命冲突与案件生成器
- 司命天平系统

### 4.4 Payload 字段表

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `conversation_id` | 是 | 会话实例 ID |
| `actor_id` | 是 | 被判定成员态的角色 |
| `membership_role_state` | 是 | 当前成员态 |
| `previous_membership_role_state` | 否 | 上一成员态 |
| `membership_started_at` | 是 | 当前成员态开始成立时间 |
| `last_state_changed_at` | 是 | 最近一次成员态切换时间 |
| `join_path` | 是 | 进入路径 |
| `mutual_awareness_confirmed` | 是 | 构成成员资格所需的互相知晓是否成立 |
| `exclusion_signal_level` | 是 | 当前排斥信号强度 |
| `conversation_open_level_snapshot` | 否 | 会话开放度快照 |

### 4.5 枚举

`membership_role_state`

- `overhearer`
- `candidate_member`
- `passive_member`
- `active_member`
- `excluded`
- `rejoin`

`join_path`

- `self_approach`
- `explicit_invite`
- `group_expansion`
- `speak_and_join`
- `rejoin_after_break`

`exclusion_signal_level`

- `none`
- `implicit_weak`
- `implicit_strong`
- `explicit`

### 4.6 校验规则

- `membership_started_at <= last_state_changed_at`
- 若 `membership_role_state=passive_member|active_member|rejoin`，则 `mutual_awareness_confirmed=true`
- 若 `membership_role_state=excluded`，则 `exclusion_signal_level != none`

## 5. AccessResolution

### 5.1 用途

回答：

- 某角色是否真正接入了这场会话或其中某条信息
- 从何时开始接入
- 接入质量如何

### 5.2 生产者

- 司命高阶知识图谱

### 5.3 主要消费者

- 角色会话参与状态子模块
- 角色记忆系统
- 角色 `L2`
- 司命知识传播判断

### 5.4 Payload 字段表

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `conversation_id` | 是 | 会话实例 ID |
| `actor_id` | 是 | 当前接入者 |
| `information_item_id` | 否 | 如在信息粒度上判定接入，则填写 |
| `access_quality` | 是 | 接入质量 |
| `access_started_at` | 是 | 实际接入开始时间 |
| `access_modalities` | 是 | 接入通道数组 |
| `access_path` | 是 | 接入路径 |
| `bounded_by_private_context` | 是 | 是否受私密会话边界限制 |

### 5.5 枚举

`access_quality`

- `none`
- `presence_only`
- `fragment`
- `gist`
- `full`

`access_modalities`

- `audio`
- `visual`
- `mixed`

`access_path`

- `direct_presence`
- `public_broadcast`
- `local_proximity`
- `eavesdrop`
- `explicit_share`

### 5.6 校验规则

- 若 `access_quality=none`，则 `information_item_id` 可为空
- 若 `access_path=eavesdrop`，则 `bounded_by_private_context` 通常为 `true`
- `access_started_at` 不得晚于依赖它的 `knowledge_started_at`

## 6. KnowledgeResolution

### 6.1 用途

回答：

- 某角色当前对某条信息处于什么知识状态
- 这种状态是通过什么来源形成的
- 从何时开始成立

### 6.2 生产者

- 司命高阶知识图谱

### 6.3 主要消费者

- 角色 `L2`
- 角色记忆系统
- 司命事实核心
- 信息博弈逻辑

### 6.4 Payload 字段表

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `information_item_id` | 是 | 知识状态针对的信息单元 |
| `actor_id` | 是 | 当前持有该知识状态的角色 |
| `knowledge_state` | 是 | 当前知识状态 |
| `knowledge_started_at` | 是 | 知识状态成立时间 |
| `knowledge_source` | 是 | 知识来源 |
| `derived_from_access_quality` | 是 | 建立该知识时依赖的接入质量 |
| `supersedes_previous_state` | 否 | 是否覆盖更早状态 |

### 6.5 枚举

`knowledge_state`

- `unknown`
- `suspected`
- `confirmed`
- `public`

`knowledge_source`

- `public_dialogue`
- `local_dialogue`
- `private_dialogue`
- `eavesdrop`
- `explicit_share`
- `inference`

`derived_from_access_quality`

- `none`
- `presence_only`
- `fragment`
- `gist`
- `full`

### 6.6 校验规则

- 若 `knowledge_source=eavesdrop`，默认不应直接生成 `confirmed`，除非 `derived_from_access_quality=full`
- `knowledge_started_at` 不得早于对应 `access_started_at`，除非 `knowledge_source=explicit_share|inference`

## 7. AwarenessResolution

### 7.1 用途

回答：

- 谁知道谁在场
- 谁知道别人知道自己在场
- 哪些知晓关系足以支撑成员资格判断

### 7.2 生产者

- 司命高阶知识图谱

### 7.3 主要消费者

- 司命成员资格判定逻辑
- 角色 `L2`
- 角色会话参与状态子模块

### 7.4 Payload 字段表

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `conversation_id` | 是 | 会话实例 ID |
| `subject_actor_id` | 是 | 谁在“知道” |
| `object_actor_id` | 是 | 被知道在场的是谁 |
| `target_actor_id` | 否 | 二阶知晓时的目标主体 |
| `awareness_level` | 是 | 知晓等级 |
| `awareness_started_at` | 是 | 知晓成立时间 |
| `derived_from_access_quality` | 是 | 基于何种接入质量形成 |
| `supports_membership_resolution` | 是 | 是否足以支撑成员资格判断 |

### 7.5 枚举

`awareness_level`

- `none`
- `presence_known`
- `mutual_presence_known`

### 7.6 校验规则

- 若表达二阶知晓，则应填写 `target_actor_id`
- 若 `supports_membership_resolution=true`，则 `awareness_level` 不得为 `none`

## 8. PrivacyRiskResolution

### 8.1 用途

回答：

- 这场会话当前的隐私是否正在被破坏
- 哪个角色的暴露风险正在上升
- 是否需要驱动降声、换位、停话或转移

### 8.2 生产者

- 司命高阶知识图谱

### 8.3 主要消费者

- 角色 `L3`
- 角色 `L4`
- 司命天平系统
- 司命冲突与案件生成器

### 8.4 Payload 字段表

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `conversation_id` | 是 | 会话实例 ID |
| `actor_id` | 是 | 风险主要作用到谁 |
| `exposure_risk` | 是 | 图谱侧暴露风险判断 |
| `risk_source` | 是 | 风险来源 |
| `risk_started_at` | 是 | 风险成立时间 |
| `bounded_by_private_context` | 是 | 是否发生在私密会话上下文 |

### 8.5 枚举

`risk_source`

- `new_listener_approach`
- `door_or_window_opened`
- `noise_drop`
- `line_of_sight_exposed`
- `speaker_volume_increase`
- `group_expansion`

### 8.6 校验规则

- 它不输出角色主观 `privacy_pressure`
- 它不输出最终知识扩散结果
- 它只代表图谱侧的风险判断

## 9. ConversationBoundaryChange

### 9.1 用途

回答：

- 会话边界是否变化了
- 会话是否从 `open` 收紧为 `local/private`
- 感知裁剪与传播范围是否应重新计算

### 9.2 生产者

- 司命高阶知识图谱

### 9.3 主要消费者

- 可感知信息编译层
- 角色 `L3`
- 角色 `L4`
- 司命图谱后续判断

### 9.4 Payload 字段表

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `conversation_id` | 是 | 会话实例 ID |
| `previous_conversation_open_level` | 是 | 变化前开放度 |
| `conversation_open_level` | 是 | 变化后开放度 |
| `changed_at` | 是 | 变化发生时间 |
| `change_cause` | 是 | 变化原因 |
| `boundary_scope_snapshot` | 否 | 变化时的 `scene / zone / dialog_group` 快照 |

### 9.5 枚举

`conversation_open_level`

- `open`
- `local`
- `private`

`change_cause`

- `volume_lowered`
- `group_tightened`
- `door_closed`
- `position_shifted`
- `new_listener_entered`
- `explicit_private_signal`

### 9.6 校验规则

- 它回答的是传播边界变化，不是成员资格变化
- 它不能单独推出谁已被排斥或谁已接入

## 10. 依赖关系

- `ConversationBoundaryChange` 依赖原始空间 / 声学 / 边界事实
- `AccessResolution` 依赖原始空间 / 声学 / 感知上下文
- `AwarenessResolution` 依赖 `AccessResolution`
- `MembershipResolution` 依赖 `AccessResolution` 与 `AwarenessResolution`
- `PrivacyRiskResolution` 依赖 `ConversationBoundaryChange`、`AccessResolution` 与当前会话开放度
- `KnowledgeResolution` 依赖 `AccessResolution`，并通常受成员态、共享路径与会话边界影响

## 11. 最低硬约束

- `AwarenessResolution` 不能早于 `AccessResolution`
- `MembershipResolution` 不能早于 `AccessResolution` 与 `AwarenessResolution`
- `KnowledgeResolution` 不能跳过 `AccessResolution`
- `ConversationBoundaryChange` 与 `AccessResolution` 可伴随事实变化反复重算，但都不得伪装成原始事实

## 12. 当前冻结结论

当前六类会话确认事件均已具备最小实现前接口语义：

- `MembershipResolution`
- `AccessResolution`
- `KnowledgeResolution`
- `AwarenessResolution`
- `PrivacyRiskResolution`
- `ConversationBoundaryChange`

后续若进入具体协议实现，必须保持：

- 公共信封与 `payload` 语义分层
- 图谱侧风险判断与角色本地主观压力分层
- 二阶知晓表达能力
- “接入 -> 知晓 -> 成员资格 -> 知识状态”的依赖边界
