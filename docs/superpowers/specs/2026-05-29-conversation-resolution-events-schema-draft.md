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

## 8. 语义示例

### 8.1 MembershipResolution 示例

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

### 8.2 AccessResolution 示例

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

### 8.3 KnowledgeResolution 示例

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

## 9. 当前冻结结论

当前先冻结三类会话确认事件的最小语义 schema：

- `MembershipResolution`
- `AccessResolution`
- `KnowledgeResolution`

它们共同构成最小协议闭环：

- 角色在会话中的社会身份
- 角色从何时开始真正接入
- 角色对具体信息当前到底知道了什么

后续若进入具体协议实现，必须继续保持以下边界：

- 不把这三类事件混成一类
- 不把它们回写伪装成 `L1` 原始事实
- 不让 `KnowledgeResolution` 越过 `AccessResolution` 的时间边界
- 不让成员资格替代知识状态，不让接入质量替代成员资格
