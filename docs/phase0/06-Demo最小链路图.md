# Phase 0 Demo 最小链路图

## 1. 文档目标

本文档给出 `Phase 0` 最小可跑闭环的一条示例事件链，帮助未来工程师快速理解系统“怎么跑”，而不是只知道“有哪些盒子”。

本文件定位为：

- 单样板时序链
- 推荐最小跑通路径

它不表示：

- `Phase 0` 中所有链都必须完整实现
- 所有后续阶段系统都必须在本轮 demo 中 fully present

## 2. 推荐样板事件

`Character A` 从桌上拿走物件，`Character B` 持续盯视 `Character A`。

这个样板能同时覆盖：

- 玩家/角色动作
- 物体变化
- gaze
- 视觉事实系统
- 事件总线
- 角色感知
- 司命公平判断

说明：

- 这是推荐样板链，不是唯一剧情
- 它的价值在于用一条链同时压到“角色执行 / 物体变化 / 视觉事实 / 总线 / 司命”这几个关键盒子

## 3. 最小链路图

```text
1. Character A action intent
   -> hide / take / move object

2. Godot / L1 执行动作
   -> AnimationTree / SkeletonModifier3D / object state change

3. Object state changed
   -> object removed from surface

4. Character B gaze sustained
   -> fixed gaze on Character A

5. Visual Fact System 发出：
   - object_removed_from_surface
   - fixed_gaze_on_target

6. Authority Event Bus 分发：
   -> Character Service
   -> Siming
   -> Replay / Audit

7. Character Service
   -> 可感知信息编译
   -> Per-Character 感知过滤
   -> CharacterPerceivedEvent

8. Siming
   -> FairnessStateSnapshot
   -> InterventionCandidate
   -> 如有必要发 opportunity / fact_reveal

9. 回到角色输入链
   -> Character A / B 继续感知、理解、规划、执行
```

## 4. 这条链验证什么

### 4.1 角色执行链

- 角色意图不是直接成功
- 先过 `Godot/L1/ESM`
- Godot 本地执行主轨应按 `AnimationTree + SkeletonModifier3D` 理解，而不是后端逐骨骼高频直控

### 4.2 视觉事实系统

- 本地姿态/物体变化不会直接上传骨骼流
- 先转成结构化 `VisualFactEvent`
- 原始骨骼 / 表情高频流不上业务总线，这条边界以上游 [Godot源码底层基础设施与运行时约束.md](/d:/Projects/Paralls/docs/phase1/core/00-总纲/Godot源码底层基础设施与运行时约束.md) 为准

### 4.3 角色感知链

- 别的角色不是直接收到“物件被拿走了”的全知真值
- 而是收到自己有资格感知到的视觉候选

### 4.4 司命

- 司命用视觉事实做公平判断
- 不直接控角

## 5. 一句话收束

这张图的目的，是让未来工程师在五分钟内理解《开本》的最小闭环到底如何跨越：角色执行、物体变化、视觉事实、权威总线、角色感知和司命判断。
