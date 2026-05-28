# 司命高阶知识图谱 v0.1 设计稿

## 1. 文档定位

- 状态：设计稿
- 主题：司命高阶知识图谱在会话成员资格、隐私与知识可见性场景中的最小可执行模型
- 适用范围：`Phase 1` 单房间单实例优先
- 作用：为司命、事件总线、`L1` 事实上抛器和角色智能体之间提供最小高阶知识判定契约

本稿不尝试冻结司命完整高阶知识图谱总设计，只收敛“会话与知识可见性”这一个最小闭环。

## 2. 设计目标

当前需要让司命高阶知识图谱至少能稳定回答以下问题：

- 谁进入了谁的可感知范围
- 谁知道谁在场
- 谁知道别人知道自己在场
- 谁只是偷听者，谁已成为沉默成员或主动成员
- 某角色从什么时间点开始真正接入一场会话
- 某角色对某条信息当前是未知、怀疑、确认还是公开得知

本稿的目标不是表示“整个世界里谁知道什么”，而是先稳定表示“单场会话里谁在场、谁被纳入、谁听到了什么、这些内容从何时起对谁成立”。

## 3. v0.1 范围

### 3.1 v0.1 要覆盖的内容

- 会话中的在场可知
- 会话中的成员资格
- 会话中的知识获取来源
- 新加入者的上下文时间线边界
- 私密会话、局部会话、公开会话的最小开放度模型

### 3.2 v0.1 不覆盖的内容

- 完整证据图谱
- 全量谎言传播模型
- 跨局知识继承
- 持续世界长期社交图
- 多房间并发的完整分布式知识同步
- 图数据库选型与最终存储结构

## 4. 关键边界

### 4.1 与 L1 的边界

所有空间接入相关底层事实，必须来自 `L1` 执行侧事实上抛器或 `ESM` 状态回写，例如：

- 位置变化
- 距离变化
- 进入 / 离开 `scene`
- 进入 / 离开 `zone`
- 发声开始 / 结束
- 音量模式
- 门窗开合
- 遮挡变化
- 站位变化
- 社交距离变化

司命高阶知识图谱不允许反向虚构这些底层事实。

### 4.2 与事件总线的边界

事件总线负责传输：

- 原始空间事实
- 原始声学事实
- 候选感知上下文
- 由司命确认后的会话关系结果

事件总线不负责高阶知识推理。

### 4.3 与角色智能体的边界

角色智能体只接收自身可知版本的结果，不读取司命持有的全局知识图谱真值。

角色本地只维护：

- 当前会话参与状态
- 自己的知识状态
- 记忆沉淀结果

不维护全局图谱本体。

## 5. 最小职责范围

### 5.1 在场可知

图谱至少要回答：

- 谁进入了谁的可感知范围
- 谁知道谁在场
- 谁知道别人知道自己在场

### 5.2 会话成员资格

图谱至少要回答：

- 某角色当前是 `overhearer / candidate_member / passive_member / active_member / excluded / rejoin` 中哪一种
- 这个判断从何时开始成立

### 5.3 知识获取来源

图谱至少要回答：

- 某信息是通过公开对话获得
- 通过局部对话获得
- 通过私密对话获得
- 通过偷听获得
- 通过显式分享获得
- 通过推断获得

### 5.4 上下文时间线边界

图谱至少要回答：

- 某角色从何时开始真正接入这场会话
- 某角色是否有资格知道更早内容

默认规则是：

- 没有接入，就没有知识
- 后加入者不自动获得更早历史
- 更早内容只能通过复述、再次共享或额外推断进入其知识状态

## 6. 最小对象集

### 6.1 Actor

表示会产生感知、知识和会话行为的主体。

当前至少包括：

- AI 角色
- 玩家投影角色
- 必要时的关键 NPC

### 6.2 Conversation

表示一段运行时会话实例，而不是一段纯文本。

它至少承载：

- 所属 `room`
- 所属 `scene`
- 所属 `zone`
- 当前开放度
- 成员态变化
- 时间线边界

### 6.3 InformationItem

表示可被会话传播、获取、共享、偷听或误解的信息单元。

它可以是：

- 一句明确发言
- 一个关键词
- 一段局部事实提示
- 一个“有人说过某事”的片段

### 6.4 PerceptionAccess

表示某个角色对某段会话或某条信息的接入记录。

至少表达：

- 谁接入了
- 接入了哪场会话或哪条信息
- 从何时开始接入
- 接入质量如何

### 6.5 MembershipState

表示角色与会话之间当前的成员关系判定结果。

### 6.6 KnowledgeClaim

表示某角色当前对某条信息持有什么知识状态。

### 6.7 AwarenessRelation

表示：

- 谁知道谁在场
- 谁知道别人知道自己在场

v0.1 只要求支持会话机制所需的有限高阶关系，不扩成无限递归模型。

## 7. 最小关系集

### 7.1 `participates_in`

`Actor -> Conversation`

表示角色当前与这场会话存在参与关系。

### 7.2 `has_membership_state`

`Actor + Conversation -> MembershipState`

表示角色在这场会话中的当前成员态。

### 7.3 `has_access_to`

`Actor -> PerceptionAccess -> Conversation / InformationItem`

表示角色对会话或其中信息的接入情况。

### 7.4 `contains_information`

`Conversation -> InformationItem`

表示一场会话包含哪些信息单元。

### 7.5 `knows`

`Actor -> KnowledgeClaim -> InformationItem`

表示角色对某条信息当前持有何种知识状态。

### 7.6 `aware_of_presence`

`Actor -> Actor`，带 `Conversation` 上下文。

表示某角色知道另一角色在当前会话面内。

### 7.7 `aware_of_awareness`

`Actor -> Actor -> Actor`，带 `Conversation` 上下文。

表示某角色知道另一角色知道第三者在场。

在会话机制中，最关键的特例是：

- `A` 知道 `B` 知道 `A` 在场

### 7.8 `excludes`

`Actor / Conversation -> Actor`

表示某角色或某场会话对另一角色发出了排斥信号。

### 7.9 `shares_with`

`Actor / Conversation -> Actor / group -> InformationItem`

表示某条信息通过哪种共享行为被正式传播。

## 8. 最小状态枚举与时间字段

### 8.1 状态枚举

#### `ConversationOpenLevel`

- `open`
- `local`
- `private`

#### `MembershipRoleState`

- `overhearer`
- `candidate_member`
- `passive_member`
- `active_member`
- `excluded`
- `rejoin`

#### `KnowledgeState`

- `unknown`
- `suspected`
- `confirmed`
- `public`

#### `AccessQuality`

- `none`
- `presence_only`
- `fragment`
- `gist`
- `full`

#### `AwarenessLevel`

- `none`
- `presence_known`
- `mutual_presence_known`

#### `ExclusionSignalLevel`

- `none`
- `implicit_weak`
- `implicit_strong`
- `explicit`

#### `JoinPath`

- `self_approach`
- `explicit_invite`
- `group_expansion`
- `speak_and_join`
- `rejoin_after_break`

### 8.2 时间字段

- `conversation_started_at`
- `access_started_at`
- `membership_started_at`
- `last_state_changed_at`
- `information_emitted_at`
- `knowledge_started_at`
- `exclusion_started_at`

关键约束：

- `access_started_at`、`membership_started_at` 与 `information_emitted_at` 必须可比较
- 成员资格时间与知识时间必须分离
- 排斥时间必须可回溯

## 9. 最小输出集

### 9.1 MembershipResolution

至少输出：

- `conversation_id`
- `actor_id`
- `membership_role_state`
- `membership_started_at`
- `last_state_changed_at`
- `join_path`
- `exclusion_signal_level`

### 9.2 AccessResolution

至少输出：

- `actor_id`
- `conversation_id`
- `information_item_id`（如需）
- `access_quality`
- `access_started_at`

### 9.3 KnowledgeResolution

至少输出：

- `actor_id`
- `information_item_id`
- `knowledge_state`
- `knowledge_started_at`
- `knowledge_source`

`knowledge_source` 当前至少支持：

- `public_dialogue`
- `local_dialogue`
- `private_dialogue`
- `eavesdrop`
- `explicit_share`
- `inference`

### 9.4 AwarenessResolution

至少输出：

- `conversation_id`
- `subject_actor_id`
- `object_actor_id`
- `awareness_level`
- `awareness_started_at`

### 9.5 PrivacyRiskResolution

至少输出：

- `conversation_id`
- `actor_id`
- `privacy_pressure`
- `exposure_risk`
- `risk_source`

### 9.6 ConversationBoundaryChange

至少输出：

- `conversation_id`
- `conversation_open_level`
- `changed_at`
- `change_cause`

## 10. 最小判断流程

### 10.1 原始事实输入

`L1/ESM` 上抛：

- 位置、距离、进入离开 `scene/zone`
- 发声起止、音量模式、声源位置
- 门窗开合、遮挡变化、噪音变化
- 站位、靠近、停留、退开、围拢、回避

### 10.2 事件总线分发

事件总线负责：

- 封装公共信封
- 按 `room / scene / zone / dialog_group / direct` 路由
- 做优先级、TTL 和可靠性治理

### 10.3 候选上下文生成

可感知信息编译层生成：

- 候选听觉事件
- 候选视觉在场事件
- 候选接近 / 回避事件
- 候选隐私边界变化事件

### 10.4 图谱判断顺序

图谱最小判断顺序固定为：

1. 接入判定
2. 互相知晓判定
3. 成员资格判定
4. 知识状态判定

不能颠倒为：

- 先判成员资格再判接入
- 先给知识状态再判听到没听到

### 10.5 结果下发

图谱把以下结果下发给角色侧和司命其他子系统：

- 成员资格结果
- 接入结果
- 知识状态结果
- 必要的互相知晓结果
- 隐私风险结果
- 会话边界变化结果

### 10.6 角色本地更新

角色智能体用这些结果更新：

- 会话参与状态子模块
- 五类记忆
- `L2` 理解层
- `L3` 规划层
- `L4` 执行层

### 10.7 新事实回流

角色一旦：

- 插话
- 靠近
- 退开
- 压低音量
- 关门
- 转移站位
- 回避

就会再次形成 `L1` 可观察事实，并重新进入下一轮判断。

## 11. 当前冻结结论

司命高阶知识图谱 v0.1 的目标不是表示“整个世界里谁知道什么”，而是先稳定表示：

- 谁在当前会话面内
- 谁知道谁在场
- 谁是偷听者、候选成员、沉默成员、主动成员或被排斥者
- 某角色从什么时间点开始真正接入这场会话
- 某条信息从什么时间点起对某角色成立为怀疑、确认或公开知识

它必须保持以下边界：

- 原始空间与声学事实来自 `L1/ESM`
- 事件总线只传输事实与确认结果
- 司命负责高阶知识推理
- 角色侧只接收自身可知版本并维护本地运行态与记忆沉淀
