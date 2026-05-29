# Specs Index

本目录用于存放在主文档回填前的独立设计稿与协议草案。

当前与“运行时会话机制”直接相关的 spec 建议按以下顺序阅读：

1. [2026-05-28-conversation-membership-and-privacy-design.md](/d:/Projects/Paralls/docs/superpowers/specs/2026-05-28-conversation-membership-and-privacy-design.md)
   - 主题：会话成员资格、隐私、偷听、加入、沉默成员、时间线裁切

2. [2026-05-28-siming-high-order-knowledge-v0.1-design.md](/d:/Projects/Paralls/docs/superpowers/specs/2026-05-28-siming-high-order-knowledge-v0.1-design.md)
   - 主题：司命高阶知识图谱 `v0.1` 的最小对象、关系、状态、输出与判断流程

3. [2026-05-29-conversation-resolution-events-schema-draft.md](/d:/Projects/Paralls/docs/superpowers/specs/2026-05-29-conversation-resolution-events-schema-draft.md)
   - 主题：会话确认事件 schema 草案
   - 当前状态：完整六件套语义已稳定；其中六件套已全部进入 core 的协议边界描述

阅读原则：

- 先看机制，再看图谱，再看协议
- 先看语义边界，再看字段与 schema
- 若主文档与 spec 草案短期内出现细节差异，以已回填到 `docs/consolidation/` 和 `docs/phase1/core/` 的冻结口径为准

若当前已经进入实现前接口与联调准备阶段，可继续阅读：

4. [2026-05-29-conversation-resolution-events-api-appendix.md](/d:/Projects/Paralls/docs/superpowers/specs/2026-05-29-conversation-resolution-events-api-appendix.md)
   - 主题：六件套确认事件的实现前接口附录

5. [2026-05-29-conversation-resolution-implementation-ownership.md](/d:/Projects/Paralls/docs/superpowers/specs/2026-05-29-conversation-resolution-implementation-ownership.md)
   - 主题：会话确认链的事实生产、确认事件产出、角色本地承接与联调分工矩阵
