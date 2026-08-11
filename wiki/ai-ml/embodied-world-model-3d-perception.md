---
title: 具身智能 3D 理解与世界模型闭环仿真（李镇团队）
description: 香港中文大学（深圳）李镇（Zhen Li）团队在 2026 Agent 大会的演讲——L3/L4 自动驾驶与具身 AI 的鲁棒泛化 3D 理解：PiSA-Bench 原生 3D 数据基准、VisionPAD/SQS 视觉中心预训练、DLWM 双潜在世界模型、InfiniVerse 世界模型仿真、DriveGEN OOD 鲁棒性、View2Cap 3D 情境感知、CLEA 闭环具身智能体
aliases: [Zhen Li, 李镇, PiSA-Bench, VisionPAD, DLWM, SQS, InfiniVerse, DriveGEN, View2Cap, CLEA, 世界模型, 具身智能, 多模态数据合成, World Model, Xiaoyu Ye, 叶晓宇]
tags: [ai-ml, concept, paper]
sources: ["2026/08/11/前沿探索与超级Agent论坛/03-李镇-跨越真实物理鸿沟：具身智能中的多模态数据合成与世界模型闭环仿.pdf"]
created: 2026-08-11
updated: 2026-08-11
---

# 具身智能 3D 理解与世界模型闭环仿真（李镇团队）

> 2026 Agent 大会「前沿探索与超级Agent论坛」演讲《跨越真实物理鸿沟：具身智能中的多模态数据合成与世界模型闭环仿真》（Bridging the Real Physical Gap — Multimodal Data Synthesis and Closed-Loop Simulation of World Models in Embodied Intelligence）。
>
> Talk by：**Zhen Li（李镇）**，Associate Professor (Tenured)；Presenter：**Dr. Xiaoyu Ye（叶晓宇）**；FNII-Shenzhen, 香港中文大学（深圳）（CUHK-Shenzhen），2026-07-25 深圳。

主题：**Robust and Generalized 3D Understanding for L3/L4 AD and Embodied AI**（L3/L4 级自动驾驶与具身 AI 的鲁棒泛化 3D 理解）。三大块内容：

1. **Multimodal 3D Data**——配对 3D-Text 原生数据与基准（PiSA-Bench）
2. **Pre-training**——自动驾驶视觉预训练（VisionPAD / SQS / DLWM）
3. **Robust and Generalized 3D Perception**——OOD 场景鲁棒感知（DriveGEN / View2Cap / CLEA）

背景：L2-L4 级自动驾驶（Tesla AutoPilot、RoboTaxi）；自动驾驶世界模型（Wayve GAIA-1、GAIA-2——2.6B 美元融资）。

## 一、多模态 3D 数据：PiSA-Bench

### 现状挑战

- 当前 3D MLLM 的错误类型：Wrong Comprehension、Wrong Attribute、Missing Object（示例：问"这是什么"，模型答"sushi"并编造食材——"Missing Noodles!"）
- 分析结论：**训练数据 captioning 是错的、充满幻觉 → 模型没有 3D 知识**（Unqualified / Lack 3D instruction data）
- 现有 3D benchmark 无法专门评估 3D 任务：**评估的属性不够**

### PiSA-Bench 方法

- **PiSA-Bench**：涵盖**七个方面属性**的综合性 3D 评估基准——评估的属性包括描述（Description）、颜色（Color）、形状（Shape）、用途（Usage）、类别（Class）等维度（幻灯片示例逐项给出各属性的 caption 对比，现有 Cap3D 等基准的 caption 错误如把 storage box 描述成 truck）
- 方法：用 **native 3D instruction data**（原生 3D 指令数据）训练——一个生成富含 3D 空间语义的指令点语言（instruction point-language）数据集的框架，三阶段：Stage 1 3D-space Data Annotation（3D 空间数据标注：深度/空间/几何信息）、Stage 2 2D-space Data Refinement（2D 空间数据精化）、Stage 3 Iterative 3D Data Bootstrap（迭代式 3D 数据自举）
- 3D captioning 基准量化结果：含 Human evaluation、GPT-4o evaluation、Traditional metrics（GPT-4o、Sentence-BERT、SimCSE、Corr. 等指标）三类评估

### 3D generative classification 实验结果（Objaverse / PiSA-Bench）

- 两种提示：Instruction-typed (I) 提示 "What is this?"；Completion-typed (C) 提示 "This is an object of..."
- `*` 表示用 **182K 数据**训练的模型。示例：PointLLM-7B 在 PiSA-Bench (I) 45.00/(C) 46.50/平均 45.75，而 PointLLM-PiSA3-7B 60.00/67.50/63.75；PointLLM-PiSA-13B* 67.50/65.00/66.25（+12.50）；PointLLM-PiSA2-7B 62.50/62.50/62.50（+15.00）
- 结论：用 3D 原生指令数据训练后（部分配合 182K 数据），生成式分类准确率显著提升

## 二、预训练：VisionPAD 与 SQS

### VisionPAD：视觉中心预训练范式（Vison-Centric Pre-training for Autonomous Driving）

作者 Haiming Zhang、Wending Zhou 等（FNII/SSE CUHK-Shenzhen、HKUST、Huawei Noah's Ark Lab）。

- **Vision-centric 3D 感知任务**：输入多视图相机图像，输出 3D bounding boxes（3D object detection）、3D semantic occupancy、map segmentation；优势：成本效益、通用对象表示、适合统一模型
- **规模化挑战**：缺乏大规模 3D 标注；大参数模型训练耗时。解决思路：预训练
- **现有预训练方法的不足**：Contrastive——性能差且不适合多视图 AD 数据；MAE——耗时；Rendering-based——严重依赖 LiDAR 显式深度监督
- **VisionPAD 动机（只依赖视觉输入的高效自监督预训练）**：
  - 首次在视觉中心感知模型中引入更高效的 **anchor-based 3DGS（3D Gaussian Splatting）表示**
  - **photometric consistency 模块**：不利用 LiDAR，把几何信息注入 volume feature
  - **自监督 volume velocity estimation 模块**：增强运动线索
- 框架：任意视觉中心感知模型构建 volumetric features → 基于 volume feature 用浅 MLP 构建 anchor-based 3DGS → photometric consistency loss + 自监督速度估计保证预训练性能
- 实验：nuScenes（3D object detection、map segmentation）、nuScenes-Occ3D（3D semantic occupancy）；指标 NDS / mAP / mIoU / IoU；fine-tune 严格遵循官方配置无修改
- 数据效率（有限数据下）：NDS 与 mAP 上相对 Baseline 与 UniPAD 均有提升（如 NDS 图中最大提升 +7.99、+5.63、+4.45 等）

### SQS：Sparse Query-based Splatting（NeurIPS 2025 Spotlight）

作者 Haiming Zhang、Yiyao Zhu 等（CUHK-SZ、HKUST、Huawei Noah's Ark Lab）。

- 现有范式：Dense BEV/Volume-centric（BEVFormer、BEVDet、OccFormer 等）；Sparse query-centric（Sparse4D、SparseBEV、SparseOcc、OPUS 等）
- **SQS**：可集成进**任意稀疏 query 感知模型**的自监督预训练范式——接受 Gaussian queries 进行预训练并用于预测；以 RGB 图像 + depth 为监督
- 预训练阶段不同 SPM 的 query 角色不同难以共享，因此提出基于 **Query Interaction** 的即插即用框架，微调时充分利用预训练 query 中封装的知识
- 实验：nuScenes 3D object detection；SurroundOcc 稠密占用标注做 3D semantic occupancy；结果提升数据效率

## 三、DLWM：双潜在世界模型（Dual Latent World Models）

作者 Yiyao Zhu（HKUST）、Haiming Zhang 等（CUHK-SZ、USTC、Huawei Foundation Model Department）。

### 场景表示谱系与 Gaussian-centric 前沿

| 表示 | 优点 | 缺点 |
|------|------|------|
| Voxel-based | 详细 3D 几何与精确空间占用 | 极高的内存与计算开销 |
| BEV-based | 计算高效的 2D 表示 | 牺牲垂直细节与稠密 3D 几何信息 |
| Sparse-query | transformer 架构推理速度极快 | 只提供粗粒度 object-centric 场景知识 |
| **Gaussian-centric（前沿）** | 3D semantic Gaussians 集合，细节与效率的最优平衡 | |

### 三个挑战

1. **Scalability**：Gaussian-centric 方法依赖大量手动标注，昂贵且限制规模化
2. **Temporal Coherence**：现有预训练主要关注静态 3D 几何，未能显式学习与建模时间动态
3. **Latent Mismatch**：置换等价性（permutation equivalence）导致连续帧的 Gaussian queries 之间没有一对一对应

### 方法：两阶段自监督预训练

**Stage 1 学习静态表示**：从多视角视频用**自监督重建**（在 depth 和 semantic maps 上）学习 3D Gaussian 场景表示——感知模块预测一组 3D Gaussians（shape、position、attributes），渲染重建输入 depth 与 semantic maps。关键优势：**无需人工标注**。

**Stage 2 双潜在世界模型**（两个并行的潜在世界模型，服务不同下游任务）：

- **Model A：Gaussian-Flow-Guided Latent World Model**（学习场景物理动态，服务 3D Occupancy Perception 与 4D Occupancy Forecasting）：1) Predict Flow（为每个 3D Gaussian 生成位移向量）→ 2) Propagate（用 flow + ego-motion 对齐把 Gaussians 投影到 t+1）→ 3) Rasterize（渲染预测的未来 BEV latent B̂ₜ₊₁）→ 4) Supervise（与 GT BEV latent Bₜ₊₁ 对比）。下游：3D Occupancy Perception 用 Gaussian-to-Occupancy Splatting；4D Occupancy Forecasting 按 ego trajectory 对齐当前 Gaussians 到下一帧、新观察区域用随机 Gaussians 补全、3D sparse convolution + refinement 建模
- **Model B：Ego-Planning-Guided Latent World Model**（为 Motion Planning 优化，保证未来场景预测可行动）：1) 用当前 Gaussian 场景特征预测 ego trajectory（T̂）→ 2) 以该轨迹为条件预测未来 BEV latent（B̂ₜ₊₁）→ 3) 与 GT BEV（Bₜ₊₁）对比监督。关键洞察：创建运动感知上下文，让规划目标引导时间场景学习

### 实验设置与结果

- 数据集：nuScenes（1000 条手工标注序列）；SurroundOcc（扩展 nuScenes，18 类的稠密 3D 语义占用标注）
- 实现：ResNet101 图像编码器 + FPN；**25,600 个 Gaussians**；AdamW + learning rate warming up
- 指标：mIoU / IoU；Planning 用 L2 Error 与 Collision Rate

**主结果**（SurroundOcc-nuScenes val）：

| 任务 | Baseline（无预训练） | DLWM（Ours） | 提升 |
|------|---------------------|--------------|------|
| 3D Occupancy Perception | 20.83 mIoU | 21.85 mIoU | +1.02 mIoU，超所有先前方法（含 Gaussian 方法） |
| 4D Occupancy Forecasting（预测未来 3 秒） | 15.09 Avg. mIoU | 17.77 Avg. mIoU | +2.68 mIoU，**新 SOTA** |
| Motion Planning（nuScenes 3 秒时域） | 0.55 m（平均 L2 Error） | **0.46 m** | ↓16% |

结论：DLWM 通过双潜在世界模型统一空间理解（Stage 1）与运动推理（Stage 2），把静态场景与动态行为建模桥接起来；规划引导世界模型预训练对多辅助任务方法具有竞争力。

## 四、InfiniVerse：占用引导的无界场景生成

（Xiaoyu Ye、Leheng Li 等，FNII/SSE CUHK-Shenzhen、HKUST、Huawei Noah's Ark Lab）

- **挑战**：现有方法缺乏长时程一致性、几何稳定性与可控性——要么无显式 3D 而累积误差，要么依赖 BEV/HD maps 而难以支持长视频
- **方法**：从**单个多视图帧**重建 3D occupancy，并沿任意轨迹通过 sketch-and-refine 扩展：粗 occupancy 引导视频，视频再精化 3D——闭合 2D-3D 循环
- **效果**：Waymo / nuScenes 上 SOTA（**FID 6.4、FVD 67.97**）；首个单帧长视频、2D-3D 对齐、任意轨迹、文本驱动的天气/风格控制
- 应用：AD Simulator、低成本数据增强

## 五、DriveGEN：可控 T2I 扩散生成提升 OOD 鲁棒性

（Hongbin Lin、Zilu Guo 等：FNII-Shenzhen、SSE CUHK-Shenzhen、NUS、NTU、宝鸡文理学院、中山大学）

- **挑战**：高数据收集成本与多样真实场景限制训练数据多样性（尤其尾类 case，如各种天气）；一旦分布偏移，训练好的模型性能骤降；OOD 测试数据上的性能下降可能引发交通事故与严重安全风险
- **方法**：**training-free 可控 Text-to-Image (T2I) 扩散生成**做训练数据增强，提升 vision-centric 3D 检测器的鲁棒性（单目与多视角均验证）
- 另有 **test-time 方案**（第 62 页仅标题，细节未展开）

## 六、View2Cap：赋予 LLM 3D 情境感知

（Zhihao Yuan 等：FNII/SSE CUHK-Shenzhen、IHPC A*STAR、香港大学）

- 3D 场景理解任务：RGB-D 扫描 + 语言指令 → 3D visual grounding、captions、answers——帮助 Agent 理解 3D 世界
- 关键差异：3D 中**第一人称（egocentric）观察者的情境会变化**，导致不同描述（如 "left"/"right"）；现有 LLM 方法忽视 egocentric 视角，使用全局视角数据集
- **View2Cap 数据集**：自动生成（无需 3D 标签或大量手动标注）——利用数据采集时的扫描轨迹作为第一人称导航 + VLM 生成高质量 captions 与 QA 对；用 GPT-4 做验证与精化
- **Situation Grounding (SG) 模块**：显式预测观察者视点位置与朝向——把每个物体当作锚点，只预测锚点到情境位置的偏移与锚点旋转到情境旋转的角度差；将 pose 估计转化为分类问题，训练更容易、空间推理更强
- 训练两阶段：region-text alignment（VLM + RGB-D 视频生成配对点云与 caption，微调 LLM 对齐点云编码器与区域 caption 特征）+ situation-aware instruction tuning（多视图图像 + 对应动作生成 QA 数据）
- 结果：对比 LEO，Scan2Cap **+2.8 CIDEr**、SQA3D **+4% EM@1**

## 七、CLEA：闭环具身智能体

（Mingcong Lei、Ge Wang 等，arXiv:2503.00729, 2025）

- **动机**：真实环境部分可观测、动态——对 MLLM 不是简单的 "predict the next token" 任务（示例：Task: Pick up pills → Action: Open the upper drawer）
- **贡献**：1) 提出适合真实世界规划的**闭环规划（closed-loop planning）算法**；2) 在真实环境中为动态任务（需要 find 对象）部署开源 MLLM-Agent
- **方法**：Closed Loop Planner + Simple Sequential Memory
- **实验**：真实厨房环境、两台机器人完成长时程任务；设计覆盖多数厨房任务的 skill pool（SLAM 实现导航技能、Inverse kinematics 实现操作技能）

## Take aways（结论）

1. 预训练对特征表示仍然有用；但**该设计哪种特征表示（3D/4D）仍是开放问题**
2. 感知与推理/规划之间应有 trade-off，尤其开放世界驾驶
3. 泛化与鲁棒不仅针对 OOD，还包括**意外 corner cases**
4. **世界模型不仅用于仿真**，还承载世界先验——环境几何、交通先验、物理交互

## 相关页面

- [[next-gen-agent-form]] — 下一代 Agent 形态探索（同为 2026 Agent 大会论坛演讲，「从数字走向物理」与具身智能同题）
- [[enterprise-agi-framework]] — 企业级 AGI 框架（SoI 智能系统：世界模型作为数字孪生/仿真的学术对照）
- [[agent-multi-agent-collaboration]] — 多 Agent 协作（CLEA 双机器人任务分配）
- [[glm5-model]] — 多模态模型（3D MLLM 能力基础）
