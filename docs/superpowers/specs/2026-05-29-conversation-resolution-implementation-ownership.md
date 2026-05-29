# 会话确认链实现分工矩阵

## 1. 文档定位

- 状态：实现前分工附录
- 主题：会话机制相关事实、确认事件、本地状态与记忆沉淀的实现归属
- 适用范围：`Phase 1` 单房间单实例优先
- 作用：把已冻结的语义边界进一步压成“谁实现什么、谁不该实现什么”的分工矩阵

本附录不替代架构主文档，也不冻结具体编码格式、类名或库表结构。

## 2. 总体分工

### 2.1 `L1 / ESM`

负责：

- 生产原始空间事实
- 生产原始声学事实
- 生产遮挡、门窗、噪音、站位、靠近、退开等边界事实
- 回写环境状态变化

不负责：

- 成员资格判断
- 高阶知晓关系判断
- 知识状态判断

### 2.2 事件总线

负责：

- 承载公共信封
- 进行 `room / scene / zone / dialog_group / direct` 路由
- 进行优先级、TTL、可靠性治理
- 传输原始事实与确认事件

不负责：

- 推断谁被纳入会话
- 推断谁知道谁在场
- 推断谁知道了什么

### 2.3 司命高阶知识图谱

负责：

- 基于事实计算接入关系
- 基于接入关系计算知晓关系
- 基于知晓和排斥计算成员资格
- 基于接入与传播路径计算知识状态
- 生成六件套确认事件

不负责：

- 虚构底层空间事实
- 直接改写角色本地主观状态
- 直接输出角色内心独白

### 2.4 角色智能体

负责：

- 消费自身可知版本的确认事件
- 更新本地会话参与状态
- 更新五类记忆
- 通过 `L2/L3/L4` 解释、规划并执行行为

不负责：

- 持有全局图谱真值
- 越权读取其他角色完整会话状态
- 用本地状态反向覆盖司命确认结果

## 3. 事实生产归属

### 3.1 `L1` 必产事实

至少包括：

- `position_changed`
- `distance_band_changed`
- `scene_entered / scene_left`
- `zone_entered / zone_left`
- `voice_started / voice_ended`
- `voice_mode_changed`
- `door_opened / door_closed`
- `window_opened / window_closed`
- `occlusion_changed`
- `noise_source_changed`
- `approach_behavior_detected`
- `retreat_behavior_detected`
- `group_tightened`
- `position_shifted`

### 3.2 `ESM` 必产事实

至少包括：

- 会影响传播边界的环境状态变化
- 会影响角色感知能力的身体状态变化
- 会影响遮挡、开放度或暴露风险的交互结果

### 3.3 事实生产硬约束

- 事实事件必须是可观察、可追溯、可复盘的
- 不允许在 `L1/ESM` 层直接产出“成员资格成立”“知道了什么”这类高阶结论

## 4. 六件套确认事件归属

### 4.1 `AccessResolution`

生产者：

- 司命高阶知识图谱

主要输入：

- `L1/ESM` 原始事实
- 可感知信息编译层候选上下文

主要消费者：

- 角色会话参与状态子模块
- 角色记忆系统
- 角色 `L2`
- 司命知识传播逻辑

### 4.2 `AwarenessResolution`

生产者：

- 司命高阶知识图谱

主要输入：

- `AccessResolution`
- 会话开放度与空间边界事实

主要消费者：

- 司命成员资格判定逻辑
- 角色 `L2`
- 角色会话参与状态子模块

### 4.3 `ConversationBoundaryChange`

生产者：

- 司命高阶知识图谱

主要输入：

- 音量变化
- 站位收紧
- 门窗开合
- 遮挡变化
- 新监听者进入

主要消费者：

- 可感知信息编译层
- 角色 `L3/L4`
- 司命后续图谱判断

### 4.4 `MembershipResolution`

生产者：

- 司命高阶知识图谱

主要输入：

- `AccessResolution`
- `AwarenessResolution`
- 排斥信号
- 会话开放度

主要消费者：

- 角色会话参与状态子模块
- 角色 `L2/L3/L4`
- 司命天平系统
- 司命冲突与案件生成器

### 4.5 `PrivacyRiskResolution`

生产者：

- 司命高阶知识图谱

主要输入：

- `ConversationBoundaryChange`
- `AccessResolution`
- 当前会话开放度
- 新进入者、边界变化、音量变化等事实

主要消费者：

- 角色 `L3/L4`
- 司命天平系统
- 司命冲突与案件生成器

### 4.6 `KnowledgeResolution`

生产者：

- 司命高阶知识图谱

主要输入：

- `AccessResolution`
- `MembershipResolution`
- 会话共享路径
- 复述 / 显式分享 / 推断链

主要消费者：

- 角色 `L2`
- 角色记忆系统
- 司命事实核心
- 信息博弈逻辑

## 5. 角色侧承接矩阵

### 5.1 会话参与状态子模块

直接消费：

- `MembershipResolution`
- `AccessResolution`
- 必要时的 `AwarenessResolution`

直接写入：

- `primary_conversation_id`
- `conversation_role_state`
- `heard_from_ts`
- `member_from_ts`
- `knowledge_confirm_level`
- `privacy_pressure`
- `exposure_risk`
- `exclusion_signal_state`
- `join_path`
- `speech_entitlement`
- `attention_focus_in_conversation`
- `active_conversation_refs`

### 5.2 记忆系统

直接消费：

- `AccessResolution`
- `KnowledgeResolution`
- 部分 `MembershipResolution`

沉淀到：

- 事件记忆
- 观察记忆
- 知识记忆
- 社交记忆
- 高阶记忆

### 5.3 `L2`

直接消费：

- 本地会话参与状态
- `KnowledgeResolution`
- 必要时的 `AwarenessResolution`

主要产出：

- 主观解释
- 风险理解
- 是否被接纳 / 排斥的主观判断
- 内心独白与知识确认倾向

### 5.4 `L3`

直接消费：

- 本地会话参与状态
- `PrivacyRiskResolution`
- `ConversationBoundaryChange`

主要产出：

- 插话 / 沉默 / 退开 / 降声 / 转移 / 接纳 / 排斥 等候选行为

### 5.5 `L4`

直接消费：

- `L3` 候选行为
- 本地会话参与状态
- 风险与边界变化结果

主要产出：

- 语音参数
- 站位变化
- 朝向变化
- 社交距离变化
- 微表情与小动作

## 6. Phase 1 实现顺序

### 6.1 第一批

- `L1/ESM` 原始事实产出
- 事件总线公共信封与路由
- `AccessResolution`

### 6.2 第二批

- `AwarenessResolution`
- `MembershipResolution`
- 角色会话参与状态子模块最小落地

### 6.3 第三批

- `KnowledgeResolution`
- 角色记忆系统联动
- `L2/L3/L4` 行为链路联动

### 6.4 第四批

- `ConversationBoundaryChange`
- `PrivacyRiskResolution`
- 更细的隐私、偷听、会话收紧行为调优

## 7. 不得跨层偷做的事

### 7.1 `L1/ESM` 不得偷做

- 不得输出成员资格结论
- 不得输出知晓关系真值
- 不得输出知识状态真值

### 7.2 事件总线不得偷做

- 不得自己推断 `passive_member / active_member`
- 不得自己生成 `KnowledgeResolution`
- 不得把角色本地主观状态塞回公共信封

### 7.3 角色侧不得偷做

- 不得把本地 `knowledge_confirm_level` 当作全局真值
- 不得把 `privacy_pressure` 回写成图谱侧风险事实
- 不得倒推其他角色的完整图谱状态并当作已确认事实

## 8. 联调检查表

### 8.1 事实链

- `L1/ESM` 是否产出了足够的原始事实
- 原始事实是否都带有足够的 `room / scene / zone` 上下文

### 8.2 确认事件链

- 是否先有 `AccessResolution`
- 是否在接入后才出现 `AwarenessResolution`
- 是否在知晓关系成立后才出现 `MembershipResolution`
- 是否在接入成立后才出现 `KnowledgeResolution`

### 8.3 角色承接链

- 角色本地会话运行态是否只存本地态
- 记忆系统是否按五类拆写
- `L2/L3/L4` 是否真的消费这些确认事件，而不是绕过它们直接硬编码逻辑

## 9. 当前冻结结论

当前这条实现分工链已经稳定为：

- `L1/ESM` 生产原始事实
- 事件总线承载公共信封
- 司命高阶知识图谱产出六件套确认事件
- 角色本地承接并沉淀为状态与记忆

后续若进入实现分工排期、联调任务拆分或服务边界划分，应以本附录作为责任归属的第一参考，而不是重新从主文档里抽象解释。 
