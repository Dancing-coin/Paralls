# 第 1 组

- 编号: 1
- 区间: 001-050
- 用户节点 ID: 1
- 助手节点 ID: 2
- 用户时间: 05/18/2026 10:59:35
- 助手时间: 05/18/2026 10:59:35
- 返回区间索引: [001-050.md](../../../indexes/ranges/001-050.md)
- 返回总索引: [all-turns.md](../../../indexes/all-turns.md)

## 用户

怎么使用kimodo动作模型来控制3d角色？

## DeepSeek

`Kimodo` 是 NVIDIA 推出的一种运动学动作扩散模型（Kinematic Motion Diffusion Model）[reference:0][reference:1]，能让你通过简单的指令来生成和控制 3D 角色的动作[reference:2]。它的核心是利用 AI，根据你的输入自动生成逼真的动画，简化了传统繁琐的制作流程。

### ⚙️ Kimodo 的核心控制方式

Kimodo 提供了**命令行工具 (CLI)** 和**交互式网页演示 (Interactive Demo)** 两种使用方式[reference:3]，主要有以下几种控制方法：

*   **📝 文字描述 (Text Prompts)**：用自然语言直接告诉模型你想要什么动作，例如 "一个人向前走，然后蹲下捡东西" ("A person walks forward, then crouches to pick something up")[reference:4][reference:5]。这是让 AI 理解你意图最直接的方式。

*   **🗝️ 关键帧约束 (Keyframe Constraints)**：在动作序列的特定时间点，锁定角色的姿势，AI 会负责补全关键帧之间的过渡动画[reference:6]。这很适合用来精准纠正动作的特定瞬间，可分为：
    *   **全身约束 (Full-Body Constraints)**：设定特定帧中角色全身所有关节的精确位置[reference:7]。
    *   **末端执行器约束 (End-Effector Constraints)**：只控制手或脚的位置与旋转。例如，你可以让角色的手精确地触碰到桌上的一个点[reference:8]。

*   **🗺️ 路径与航点控制 (Path & Waypoint Control)**：在 2D 平面上规划角色的整体移动路径，模型会确保角色严格按照你设定的路线行走[reference:9]。这非常适合控制角色的走位。

### 🛠️ 两种核心工作流程

你可以根据自己的习惯和需求，选择下面两种工作流程：

1.  **基于 UI 的演示模式 (新手友好)**：这是上手最快的方式。
    1.  **启动**：通过 `python -m kimodo.demo` 启动服务，然后在浏览器打开 `http://127.0.0.1:7860`[reference:10]。
    2.  **编辑**：在一个**时间轴 (Timeline Editor)** 上，直观地添加**文本描述**或拖拽式的**关键帧约束**[reference:11][reference:12]。
    3.  **预览与导出**：可以**实时查看**生成效果，调整**回放速度**，甚至可以一次生成**多个样本**进行比较，最后将满意的动作和约束条件**导出**[reference:13]。

2.  **使用命令行接口 (CLI) 进行批处理**：适合批量、参数化地生成动作。
    1.  **基础用法**：使用一行命令即可快速生成动作。
        ```bash
        # 示例：用 Kimodo-SOMA-RP-v1 模型生成一段行走动画
        kimodo_gen --prompt "一个人向前走" --model kimodo-soma-rp-v1 --duration 5.0 --num_samples 3 --output ./output/walk
        ```
        主要参数说明：
        *   `--prompt`: 描述动作的文本[reference:14]。
        *   `--model`: 选择模型，如 `kimodo-soma-rp-v1`[reference:15]。
        *   `--duration`: 动作时长（秒）[reference:16]。
        *   `--num_samples`: 生成多少个变体供挑选[reference:17]。
        *   `--output`: 结果保存目录[reference:18]。

    2.  **高级控制**：在 Python API 中使用 `constraint_lst` 参数，可以传入复杂的约束，如关键帧姿势、末端执行器目标等，实现更精细的控制[reference:19]。

### 💡 上手前的准备与建议
*   **硬件要求**：本地生成大约需要 **17GB 的显存 (VRAM)**，推荐 RTX 3090 或以上级别显卡。如果显存不足，可以通过环境变量将文本编码器移至 CPU 来降低显存占用[reference:20][reference:21]。
*   **模型选择**：推荐选择基于完整 700 小时 **Bones Rigplay 数据集**训练的模型，如 `Kimodo-SOMA-RP` 系列，它们拥有更全面的动作表现力[reference:22][reference:23]。
*   **格式兼容**：生成的动画可导出为 `.npz` 或 `.bvh` 格式，能轻松导入 Unreal Engine、Unity、Maya 等主流创作工具[reference:24]。

如果想了解某个具体操作或控制方式的更多细节，可以随时再问我～
