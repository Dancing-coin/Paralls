# 会话确认事件 Schema 草案

## 1. 文档定位

- 状态：草案
- 主题：`MembershipResolution`、`AccessResolution`、`KnowledgeResolution` 三类会话确认事件的最小语义 schema
- 适用范围：`Phase 1` 单房间单实例优先
- 作用：为事件总线、司命高阶知识图谱和角色智能体之间的协议层对齐提供最小字段约束

本稿只冻结语义层 schema，不冻结：

- 最终编码格式
- 传输协议
- JSON / Protobuf / FlatBuffers 选型
- 数据库存储结构

## 2. 设计目标

当前需要把三类会话确认事件的语义边界收紧为可执行草案：

- `MembershipResolution`
- `AccessResolution`
- `KnowledgeResolution`

目标不是做完整协议大全，而是先稳定回答三件事：

- 我在这场会话里的社会身份是什么
- 我从什么时候开始接入、接入了多少
- 我现在到底知道了什么

## 3. 共同前提

### 3.1 继承公共信封

三类确认事件都继承事件总线公共信封，不另造外壳。

默认已存在的公共字段包括：

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

### 3.2 判断顺序

高阶知识图谱内部判断顺序必须保持：

1. 接入判定
2. 互相知晓判定
3. 成员资格判定
4. 知识状态判定

因此，对外事件的语义依赖也必须保持：

- `KnowledgeResolution` 不得跳过 `AccessResolution`
- `MembershipResolution` 不得伪装成原始空间事实

### 3.3 边界约束

- 原始空间与声学事实来自 `L1/ESM`
- 事件总线只传输公共信封与确认结果
- 司命高阶知识图谱负责产生确认结果
- 角色侧只持有自身可知版本

## 4. MembershipResolution

### 4.1 用途

回答：

- 某角色当前在某场会话中的成员身份是什么
- 该身份从何时开始成立
- 是否刚发生状态切换

### 4.2 Payload 字段

- `conversation_id`
- `actor_id`
- `membership_role_state`
- `previous_membership_role_state` 可选
- `membership_started_at`
- `last_state_changed_at`
- `join_path`
- `mutual_awareness_confirmed`
- `exclusion_signal_level`
- `conversation_open_level_snapshot` 可选

### 4.3 字段说明

#### `conversation_id`

当前会话实例 ID。

#### `actor_id`

当前被判定成员态的角色。

#### `membership_role_state`

当前成员态，枚举：

- `overhearer`
- `candidate_member`
- `passive_member`
- `active_member`
- `excluded`
- `rejoin`

#### `previous_membership_role_state`

上一状态，便于角色侧做状态迁移解释。

#### `membership_started_at`

当前成员态开始成立的时间。

#### `last_state_changed_at`

最近一次成员态切换时间。

#### `join_path`

进入路径，枚举：

- `self_approach`
- `explicit_invite`
- `group_expansion`
- `speak_and_join`
- `rejoin_after_break`

#### `mutual_awareness_confirmed`

布尔值，表示构成成员资格所需的互相知晓是否成立。

#### `exclusion_signal_level`

排斥信号强度，枚举：

- `none`
- `implicit_weak`
- `implicit_strong`
- `explicit`

#### `conversation_open_level_snapshot`

当前会话开放度快照，可选，枚举：

- `open`
- `local`
- `private`

### 4.4 语义约束

- 它是司命高阶知识图谱的判断结果，不是 `L1` 原始事实
- 它不描述角色具体知道了哪些内容
- 它不能替代 `KnowledgeResolution`

## 5. AccessResolution

### 5.1 用途

回答：

- 某角色是否真正接入了这场会话或其中某条信息
- 从何时开始接入
- 接入质量如何

### 5.2 Payload 字段

- `conversation_id`
- `actor_id`
- `information_item_id` 可选
- `access_quality`
- `access_started_at`
- `access_modalities`
- `access_path`
- `bounded_by_private_context`

### 5.3 字段说明

#### `conversation_id`

当前会话实例 ID。

#### `actor_id`

当前接入者。

#### `information_item_id`

若当前分辨的是“对某条具体信息的接入”，则填写；若先回答“是否接入整场会话”，则可为空。

#### `access_quality`

接入质量，枚举：

- `none`
- `presence_only`
- `fragment`
- `gist`
- `full`

#### `access_started_at`

该角色开始实际接入的时间点，是 `heard_from_ts` 的外部来源。

#### `access_modalities`

建议为数组，最小支持：

- `audio`
- `visual`
- `mixed`

#### `access_path`

接入路径，枚举：

- `direct_presence`
- `public_broadcast`
- `local_proximity`
- `eavesdrop`
- `explicit_share`

#### `bounded_by_private_context`

布尔值，表示本次接入是否受到私密会话边界限制。

### 5.4 语义约束

- `AccessResolution` 只回答“接入到了多少”，不回答“是否被接纳”
- 它不能直接推出成员资格
- 它不能直接推出知识状态

## 6. KnowledgeResolution

### 6.1 用途

回答：

- 某角色当前对某条信息处于什么知识状态
- 这种状态是通过什么来源形成的
- 从何时开始成立

### 6.2 Payload 字段

- `information_item_id`
- `actor_id`
- `knowledge_state`
- `knowledge_started_at`
- `knowledge_source`
- `derived_from_access_quality`
- `supersedes_previous_state` 可选

### 6.3 字段说明

#### `information_item_id`

该知识状态针对哪条信息单元。

#### `actor_id`

当前持有该知识状态的角色。

#### `knowledge_state`

知识状态，枚举：

- `unknown`
- `suspected`
- `confirmed`
- `public`

#### `knowledge_started_at`

当前知识状态从何时起成立。

#### `knowledge_source`

知识来源，枚举：

- `public_dialogue`
- `local_dialogue`
- `private_dialogue`
- `eavesdrop`
- `explicit_share`
- `inference`

#### `derived_from_access_quality`

表明这条知识主要建立在哪种接入质量之上，枚举：

- `none`
- `presence_only`
- `fragment`
- `gist`
- `full`

#### `supersedes_previous_state`

可选，表示这次是否覆盖掉更早的知识状态。

### 6.4 语义约束

- `KnowledgeResolution` 不能跳过 `AccessResolution` 的时间边界
- 后加入者不能在 `access_started_at` 之前直接获得 `confirmed/public`，除非后续发生显式分享或公开披露
- `knowledge_source=eavesdrop` 不应默认生成 `confirmed`

## 7. 三者关系

### 7.1 角色分工

- `MembershipResolution`
  回答：我在会话里的社会身份是什么

- `AccessResolution`
  回答：我从什么时候开始接入、接入了多少

- `KnowledgeResolution`
  回答：我现在到底知道了什么

### 7.2 禁止混写

以下推断都不成立：

- 既然是成员，所以必然知道全部
- 既然听到了片段，所以必然被接纳
- 既然知道了什么，所以一定从头都在场

## 8. AwarenessResolution

### 8.1 用途

回答：

- 谁知道谁在场
- 谁知道别人知道自己在场
- 哪些知晓关系足以支撑成员资格判断

### 8.2 Payload 字段

- `conversation_id`
- `subject_actor_id`
- `object_actor_id`
- `awareness_level`
- `awareness_started_at`
- `derived_from_access_quality`
- `supports_membership_resolution`

### 8.3 字段说明

#### `subject_actor_id`

谁在“知道”。

#### `object_actor_id`

被知道在场的是谁。

#### `awareness_level`

知晓等级，枚举：

- `none`
- `presence_known`
- `mutual_presence_known`

#### `awareness_started_at`

这条知晓关系从何时起成立。

#### `derived_from_access_quality`

这条知晓关系主要建立在哪种接入质量之上。

#### `supports_membership_resolution`

布尔值，表示该知晓关系是否足以支撑成员资格判断。

### 8.4 语义约束

- 它回答的是知晓关系，不回答知识内容
- 它不能替代 `MembershipResolution`
- 它不能脱离 `conversation_id` 单独当长期社交关系使用

## 9. PrivacyRiskResolution

### 9.1 用途

回答：

- 这场会话当前的隐私是否正在被破坏
- 哪个角色的暴露风险正在上升
- 是否需要驱动降声、换位、停话或转移

### 9.2 Payload 字段

- `conversation_id`
- `actor_id`
- `privacy_pressure`
- `exposure_risk`
- `risk_source`
- `risk_started_at`
- `bounded_by_private_context`

### 9.3 字段说明

#### `actor_id`

这条隐私 / 暴露风险主要作用到谁。

#### `privacy_pressure`

当前隐私紧张度，当前只要求可分级表达，不冻结具体数值制式。

#### `exposure_risk`

当前暴露风险，当前同样只要求可分级表达。

#### `risk_source`

风险来源，枚举建议至少包括：

- `new_listener_approach`
- `door_or_window_opened`
- `noise_drop`
- `line_of_sight_exposed`
- `speaker_volume_increase`
- `group_expansion`

#### `risk_started_at`

该风险从何时开始成立。

#### `bounded_by_private_context`

表示该风险是否发生在 `private` 边界内。

### 9.4 语义约束

- 它不直接判定是否已泄密
- 它只输出风险和压力，不输出最终知识扩散结果
- 角色侧可以基于它调整行为，但不能把它当成既成知识事实

## 10. ConversationBoundaryChange

### 10.1 用途

回答：

- 会话边界是否变化了
- 会话是否从 `open` 收紧为 `local/private`
- 传播裁剪是否应重新计算

### 10.2 Payload 字段

- `conversation_id`
- `previous_conversation_open_level`
- `conversation_open_level`
- `changed_at`
- `change_cause`
- `boundary_scope_snapshot`

### 10.3 字段说明

#### `previous_conversation_open_level`

变化前的开放度。

#### `conversation_open_level`

变化后的开放度，枚举：

- `open`
- `local`
- `private`

#### `changed_at`

变化发生时间。

#### `change_cause`

变化原因，枚举建议至少包括：

- `volume_lowered`
- `group_tightened`
- `door_closed`
- `position_shifted`
- `new_listener_entered`
- `explicit_private_signal`

#### `boundary_scope_snapshot`

可选快照，用于记录变化时的 `scene / zone / dialog_group` 范围。

### 10.4 语义约束

- 它回答的是传播边界变化，不是成员资格变化
- 它不能单独推出谁已被排斥或谁已接入
- 它主要服务感知链重新裁剪、偷听概率重算和角色行为调整

## 11. 六件套关系

### 11.1 角色分工

- `AccessResolution`
  回答：我从什么时候开始接入、接入了多少

- `AwarenessResolution`
  回答：我和别人是否互相知道在场

- `ConversationBoundaryChange`
  回答：这场会话的开放边界是否变化了

- `MembershipResolution`
  回答：我在会话里的社会身份是什么

- `PrivacyRiskResolution`
  回答：这场会话现在危险不危险

- `KnowledgeResolution`
  回答：我现在到底知道了什么

### 11.2 推荐顺序

若把六类都排入最小顺序，当前建议：

1. `AccessResolution`
2. `AwarenessResolution`
3. `ConversationBoundaryChange`
4. `MembershipResolution`
5. `PrivacyRiskResolution`
6. `KnowledgeResolution`

语义边界上，第 3 和第 4 在某些实现里可局部迭代，但都不能早于接入与知晓。

## 12. 语义示例

### 12.1 MembershipResolution 示例

场景：`C` 走近 `A/B` 的局部对话，双方已互相知晓，`C` 尚未开口，因此被判定为 `passive_member`。

```json
{
  "conversation_id": "conv_study_001",
  "actor_id": "char_c",
  "membership_role_state": "passive_member",
  "previous_membership_role_state": "candidate_member",
  "membership_started_at": "2026-05-29T20:14:08Z",
  "last_state_changed_at": "2026-05-29T20:14:08Z",
  "join_path": "group_expansion",
  "mutual_awareness_confirmed": true,
  "exclusion_signal_level": "none",
  "conversation_open_level_snapshot": "local"
}
```

### 12.2 AccessResolution 示例

场景：`D` 站在书房门外，听到一点耳语，只捕捉到片段。

```json
{
  "conversation_id": "conv_study_001",
  "actor_id": "char_d",
  "information_item_id": "info_utterance_014",
  "access_quality": "fragment",
  "access_started_at": "2026-05-29T20:14:11Z",
  "access_modalities": ["audio"],
  "access_path": "eavesdrop",
  "bounded_by_private_context": true
}
```

### 12.3 KnowledgeResolution 示例

场景：`D` 虽然只偷听到片段，但已经形成“怀疑 A 和 B 在讨论某个秘密”的知识状态。

```json
{
  "information_item_id": "info_secret_topic_003",
  "actor_id": "char_d",
  "knowledge_state": "suspected",
  "knowledge_started_at": "2026-05-29T20:14:13Z",
  "knowledge_source": "eavesdrop",
  "derived_from_access_quality": "fragment",
  "supersedes_previous_state": false
}
```

## 13. 当前冻结结论

当前先冻结六类会话确认事件的最小语义 schema：

- `MembershipResolution`
- `AccessResolution`
- `KnowledgeResolution`
- `AwarenessResolution`
- `PrivacyRiskResolution`
- `ConversationBoundaryChange`

其中前三类构成最小闭环，后三类构成支撑与修正上下文：

- `MembershipResolution`：角色在会话中的社会身份
- `AccessResolution`：角色从何时开始真正接入、接入了多少
- `KnowledgeResolution`：角色对具体信息当前到底知道了什么
- `AwarenessResolution`：会话中的互相知晓关系
- `PrivacyRiskResolution`：当前隐私与暴露风险
- `ConversationBoundaryChange`：会话开放边界的变化

后续若进入具体协议实现，必须继续保持以下边界：

- 不把六类事件混成一类
- 不把它们回写伪装成 `L1` 原始事实
- 不让 `KnowledgeResolution` 越过 `AccessResolution` 的时间边界
- 不让成员资格替代知识状态，不让接入质量替代成员资格
- 不让边界变化或风险评估伪装成知识确认结果
