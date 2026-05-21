# 从 π0 到 π0.7：Physical Intelligence 通用机器人基础模型路线综述

**报告范围**：本综述系统梳理 Physical Intelligence (PI) 自 2024 年 10 月至 2026 年 4 月公开的 14 项研究工作，涵盖其首个通用策略 π0、动作分词器 FAST、开放世界泛化模型 π0.5、实时控制系统 RTC、知识隔离训练范式 KI、人在环路推理 Hi Robot、从经验中学习的 π\*0.6、人到机器人迁移、Moravec 挑战、物理智能层、长短期记忆 MEM、在线 RL Token，直至最新的可引导模型 π0.7。

**写作目的**：从单一模型架构 → 训练数据范式 → 推理系统 → 学习信号 → 部署生态五条主线，提炼这一研究计划的内在逻辑、技术演进与开放问题。

---

## 1. 背景与研究主线

机器人学长期受 Moravec 悖论困扰：对人类而言"困难"的下棋、定理证明可被符号系统轻松处理，而抓取、折叠、清洁这类对幼儿都"简单"的物理任务对机器至今仍极其困难。Physical Intelligence 自 2024 年起以"通用物理智能模型"为目标推进一条统一路线，其核心假设是：与 NLP 中 GPT 系列的路径相似，只有当模型规模、跨本体数据规模与训练方法协同到位时，机器人才能跨越孤立技能学习的瓶颈，出现真正的开放世界泛化。

围绕这一假设，PI 的工作呈现五条彼此咬合的主线：(1) 模型架构与训练目标（π0、π0.5、π0.7 一脉相承的 Vision-Language-Action (VLA) 主干）；(2) 动作表征（FAST 分词器，使自回归 VLA 成为可行）；(3) 推理与部署（Real-Time Chunking、Physical Intelligence Layer）；(4) 学习信号（Hi Robot 人在环路、π\*0.6 与 RL Token 的在线强化学习、人到机器人视频迁移）；(5) 系统能力扩展（MEM 长短期记忆、Moravec Robot Olympics、π0.7 的可引导性）。下文按时间脉络逐项展开，最后在第 16 节给出整体综合。

---

## 2. π0：第一代通用机器人策略（2024 年 10 月）

π0 是 PI 公开的首个通用策略，定位为"原型基础模型"。其核心架构在一个预训练 Vision-Language Model (VLM) 主干之上挂接一个**动作专家 (action expert)**，并以**流匹配 (flow matching)** 作为连续动作分布的训练目标。给定多视角图像与自然语言指令，模型一次性生成长度为 H 的动作块 (action chunk)，避免逐步自回归带来的累积误差与延迟。

训练数据由多机器人、多任务遥操作演示构成，规模上首次跨越了"单本体单技能"的传统范式。这一架构与训练配方在后续工作中被反复继承与改进。π0 同时验证了一个关键判断：仅通过模仿学习与足够多样的数据，单一权重模型即可在折叠衣物、整理餐具等真实任务上展现可观的零样本性能 [3]。

值得注意的是，π0 并非孤立诞生。Open X-Embodiment 项目 [2] 同期推动了跨本体数据的开源生态，为通用 VLA 提供了重要的对照与数据基础；π0 的贡献在于把"VLM 主干 + 动作专家 + 流匹配 + 跨本体演示"这套组合一次性走通，并直接在生产级硬件上验证。

---

## 3. FAST：让自回归 VLA 训练快 5 倍的动作分词器（2025 年 1 月）

流匹配虽强，但与现有 VLM 的自回归 (AR) 解码栈不兼容，难以复用大规模文本预训练带来的所有工具链。FAST [1] 正面解决这一问题。其核心观察是：机器人动作序列在时域上高度冗余、低带宽，朴素地把每一维浮点动作量化为离散 token 会产生极长、高度可预测的序列，使 Transformer 主要在拟合琐碎的时序相关性而非任务语义。

FAST 的解法是**频域分词**：先用离散余弦变换 (DCT) 把每段动作块映射到频谱，再对系数做量化与字节对编码 (BPE) 压缩。由于动作能量集中在低频，压缩后的 token 序列大幅缩短，AR 训练效率随之大幅提升。论文报告在等量数据下，基于 FAST 的通用策略训练速度比此前最优的离散化方案快约 5 倍，同时维持甚至改进任务成功率 [1]。

FAST 的意义不仅是"快"——它使 PI 后续可以同时维护两条技术栈：**π0 系**（流匹配动作专家，连续输出，低延迟）与 **π0-FAST 系**（自回归离散动作，受益于 VLM 工具链与可解释中间表征）。这种"双栈"策略为后来的开源发布和系统级部署提供了灵活性。

---

## 4. 开源 π0（2025 年 2 月）

PI 在 2025 年 2 月公开了 π0 与 π0-FAST 的权重与代码。这是首批进入开源生态的工业级通用 VLA，其影响在随后一年内迅速放大：大量后续工作（DiG-Flow [10]、Shallow-π 知识蒸馏 [11]、AT-VLA 触觉注入 [12] 等）直接以 π0 或 π0-FAST 为基线进行改进、蒸馏、扩展或鲁棒性分析。

从研究方法论角度，开源不仅是产品决策也是科学决策：它强制 PI 公开训练配方与推理栈的关键细节，使外部研究者可在可控对照下验证"VLM 主干 + 跨本体数据 + 流匹配/FAST"路线的可复现性，并迅速暴露其失败模式（例如分布外鲁棒性、语言扰动脆弱性等），后续工作正是围绕这些暴露出的问题展开。

---

## 5. Hi Robot：人在环路的分层推理（2025 年 2 月）

Hi Robot 把"听懂复杂指令并分步执行"作为目标，提出**分层推理**结构：高层 VLM 负责把抽象指令（如"清理这张桌子，但把红色杯子留下"）分解为可执行的子指令序列，低层策略（即 π0 或 π0-FAST）逐步执行。关键设计是支持**人在环路反馈**：用户可在执行过程中插入纠正性语言（"不，那个杯子不要扔"），高层模块即时改写后续子指令。

这一工作首次在 PI 体系内引入"系统 2"式的慢思考能力，并把"任务规划—动作执行"的回路打开给人类。它直接预示了后续 MEM 长短期记忆与 π0.7 可引导性的设计动机：要把通用策略部署到长时序、可交互、可修正的真实场景，光有动作生成是不够的，必须把规划与人机协议显式建模。

---

## 6. π0.5：开放世界泛化（2025 年 4 月）

π0.5 是 π0 的直接继承者，其关键命题是"在从未见过的真实家庭中执行长时序任务"。报告展示了一台移动操作机器人 (mobile manipulator) 在**完全陌生**的厨房与卧室中完成清洁、整理任务，并主张这种泛化来自**异质数据协同训练 (heterogeneous co-training)**：把跨本体遥操作数据、跨任务网络多模态数据、跨场景视频与高层语义标注一同纳入训练。

π0.5 提出，开放世界泛化不是简单的"更多机器人数据"问题，而是要让模型在训练时就接触到与部署环境分布更接近、却来源各异的监督信号。这一思想被后续 Data Analogies [4] 等独立工作进一步形式化为"跨本体数据组织"问题，并继续影响 π0.6/0.7 的训练配方。

---

## 7. KI：训练快、推理快、泛化更好的 VLA（2025 年 5 月）

Knowledge Insulating (KI) [9] 解决了一个看似矛盾的工程悖论：当 VLA 在大量机器人动作数据上微调时，往往**破坏 VLM 主干中宝贵的语义先验**，导致语言遵循能力退化，泛化变差。KI 提出在训练阶段对 VLM 表征做"知识隔离"，使动作头的梯度不会污染语义骨干，同时通过精心设计的接口让动作头仍能高效地从语义特征中读取信息。

实测结果是三重收益：训练吞吐显著提升、推理延迟下降、且语言条件下的泛化性能优于直接端到端微调的基线 [9]。这一论文也为后续讨论"VLA 何时应该端到端、何时应该模块化"提供了关键的实证证据，并被 NORA [13]、MetaVLA 等开源 VLA 复用。

---

## 8. RTC：动作分块流策略的实时执行（2025 年 6 月）

VLA 越大、推理越慢，但物理世界不会等模型——慢思考与实时控制的张力在大规模 VLA 时代愈发尖锐。Real-Time Action Chunking (RTC) [14] 正面攻击这一问题。其核心思想是把"块级生成"与"步级执行"解耦：模型异步生成下一个动作块，控制器在当前块尚未消耗完毕时即触发下一块的生成请求，并以渐近一致的方式拼接。

RTC 保留了流匹配/扩散策略的高精度输出，又在 300 ms 量级的推理延迟下维持了平滑、实时的控制频率，使大型 VLA 可以驱动接触丰富、节奏紧凑的任务（如双手协调、动态接触）[14]。这一系统级贡献后来被多篇异步执行工作 [5]、加速 patch 工作 [6] 进一步推广，并成为 PI Layer（见第 10 节）实际部署栈的核心组件之一。

---

## 9. Moravec 悖论与"机器人奥运会"（2025 年 12 月）

PI 在 2025 年 12 月发布了一组特别有挑衅性的演示：用最新的 π 系列模型进行专项微调，挑战一系列被研究者长期视为"高难度"的操作任务——从精细装配、动态平衡到多步装配，构成一场"机器人奥运会"。

这项工作的科学价值在于**反向印证基础模型范式的可行性**：每一项挑战任务在传统专项学习框架下都需要数月专门工程，而以一个通用 π 模型为起点，仅通过少量针对性微调即可解决。这与 NLP 中 GPT 系列"通用基座 → 少量微调即跨任务"的故事完全平行，是 Moravec 悖论在机器人学层面被实证撼动的有力证据。

---

## 10. Physical Intelligence Layer：通用物理智能的部署生态（2026 年 2 月）

到 2026 年初，π 系列模型已被多家合作伙伴部署到真实生产场景。PI 在《The Physical Intelligence Layer》一文中提出"物理智能层"的概念：把通用物理智能模型视为一个跨硬件、跨行业的**可调用基础设施层**，类似于云时代的操作系统层。其论点是，一旦底层模型具备足够通用性，机器人应用将出现一次"寒武纪大爆发"——以前需要单独构建感知-规划-控制栈的每个应用，将能在同一个 π 模型之上以数周乃至数天的速度被孵化。

这一篇文章本身是产品/战略宣言，技术内容相对薄弱，但它界定了后续 π0.7 之类工作的产品边界：模型必须可被合作伙伴**显式引导 (steerable)**，必须支持长时序记忆，必须能持续从部署经验中改进。这三条直接对应了第 11–14 节的研究主题。

---

## 11. 从人类视频到机器人的迁移在 VLA 中涌现（2025 年 12 月）

这项工作研究了一个长期悬而未决的问题：人类操作视频与机器人本体差异极大（无关节对应、视角不同、力学不匹配），能否在不显式建模这些差异的前提下，让 VLA 从中获益？答案是肯定的——当模型规模足够大时，**从人类视频到机器人任务的迁移会自发涌现**。

报告显示，随着 VLA 主干规模与跨模态数据规模同步增长，模型逐渐能从"观察人类执行 X"中学到"机器人执行 X"所需的高层意图与时序结构，而不需要额外的人-机器人显式对齐损失。这一发现与 Learning by Watching [15] 等综述工作相互印证，并为后续以视频数据扩展机器人训练集提供了直接的实证依据。

---

## 12. π\*0.6：从经验中学习的 VLA（2025 年 11 月）

π\*0.6 把"训练后能继续学习"作为核心特征。它在 π0/π0.5 的基础上引入一套**真实世界强化学习 (real-world RL)** 训练流程：模型在部署阶段把执行轨迹与稀疏成功/失败信号回流到训练循环中，用以**同时改进任务成功率与执行吞吐量**。

这一工作的关键贡献在于把 RL 信号嵌入到通用策略而非任务专项策略，避免了"每个任务都要单独建一套奖励工程"的陷阱。其结果直接预告了第 14 节 RL Token 把这一思想推进到更精细动作的方向，并支撑了 PI Layer 中"部署后持续改进"这条产品承诺。

---

## 13. MEM：兼具长短期记忆的 VLA（2026 年 3 月）

Multi-Scale Embodied Memory (MEM) 为 VLA 引入**多尺度记忆**：短期记忆服务于步级动作连贯性，长期记忆维护跨小时、跨任务的环境与意图状态。报告展示一台 π+MEM 系统完成超过十分钟、由数十个子步骤构成的复杂家居任务（如"做一顿三道菜的早餐"），并在中途允许人类打断、追加约束。

MEM 的技术意义在于解决"动作块基础策略缺乏跨块状态"这一系统瓶颈：单纯把上下文做长无法解决，因为视觉输入的 token 成本过高；MEM 采用分层抽象——低层保留近期视觉/动作 frames，高层把环境状态摘要为可被规划器复用的语义结构。这一思想与 Hi Robot 的分层推理在结构上形成闭环，使 PI 的系统首次具备"长时任务跨小时一致性"的实操能力。

---

## 14. RL Token：高效在线 RL 改善精密操作（2026 年 3 月）

这项工作直接面向高精度、对时间敏感的任务（如插拔、对位、精细装配），将一个**专门的 RL Token** 从 VLA 中"抽出"，使在线 RL 只需更新该 token 相关参数而非整个 VLA 主干。结果是只需数小时真实数据就能显著提升此前难以攻克的精密任务的吞吐与成功率。

从方法论看，RL Token 是 π\*0.6 思想的精细化：把"模型整体在线学习"细化为"在保留通用能力的同时，对关键决策点定向施加 RL 信号"。这种"模块化在线学习"在概念上与 KI 的"知识隔离"高度互补——前者保护通用语义先验不被破坏，后者则允许在保护范围之外做精细的局部优化。

---

## 15. π0.7：具备可引导性与涌现能力的最新模型（2026 年 4 月）

π0.7 是当前最新一代模型，定位为"**可引导 (steerable)**"且展现**质的跃迁式泛化**。可引导性的具体含义是：合作伙伴可以通过结构化语言、示例或少量调参显式地改变模型行为，而无需重训整个模型。

报告强调三类涌现能力：(1) 跨场景零样本迁移在更广分布上不再线性退化；(2) 多模态长指令遵循显著改善；(3) 在没有专门微调的前提下，自发产生了对新工具、新物体的合理抓取与使用策略。这一发布在叙事上将"通用物理智能"从工程目标推进为一个可被产品化、可被外部团队按需定制的基础模型层，构成本路线的当前节点。

---

## 16. 综合：一条统一研究计划的内部逻辑

把上述 14 项工作放在一起，可以看到 PI 的研究并非孤立产品迭代，而是围绕一个明确的论题展开：**通用物理智能模型 = 强语义先验主干 + 跨本体异质数据 + 高效动作表征 + 实时推理系统 + 持续学习信号 + 可引导接口**。每一篇工作都精准对应到这六个组成中的一项或多项：

主干与训练目标由 π0 [3]、π0.5、KI [9]、π0.7 一脉接力；动作表征由 FAST [1] 与开源 π0-FAST 提供；实时推理由 RTC [14] 解决；持续学习由 π\*0.6、RL Token 与人类视频迁移共同支撑；长时序与可交互能力由 Hi Robot 与 MEM 闭环；可引导接口与生态由 PI Layer 与 π0.7 定型。Moravec 奥运会则在最具挑战的任务上提供端到端的能力检验。

从领域演进角度，这条路线给整个 VLA 研究社区带来了三个结构性影响：第一，**确立了 VLM 主干 + 动作专家的事实标准**，几乎所有后续 VLA（NORA [13]、SpatialVLA [16]、JARVIS-VLA [17] 等）都在这一架构下展开；第二，**FAST 与 π0 的开源**把社区从"各自闭源"推进到"在统一基线上做对照实验"，DiG-Flow [10]、LIBERO-Plus [18] 等系统性鲁棒性研究因此成为可能；第三，**RTC 与 PI Layer** 首次把"VLA 在真实生产环境中怎么跑得动"作为一等公民问题提出，催生了 Kairos [19]、Speedup Patch [6] 等系统级后续工作。

**开放问题**仍然显著。(1) 鲁棒性：LIBERO-Plus [18] 等独立评测显示当前 VLA（含 π0 系）对语言扰动、传感器噪声仍脆弱；(2) 可验证性：可引导意味着合作伙伴可以注入约束，但目前并无形式化方式验证模型确实遵守这些约束；(3) 数据透明度：跨本体异质数据的具体配方与许可问题尚不公开，使外部研究者难以严格复现 π0.5/0.7；(4) 真实成本：在线 RL 与长时序部署对硬件、能源、人工监督的真实开销在公开材料中几乎未被量化。这四点构成下一阶段研究的明确机会窗口。

---

## 参考文献

[1] K. Pertsch, K. Stachowicz, B. Ichter, et al. *FAST: Efficient Action Tokenization for Vision-Language-Action Models*. arXiv:2501.09747, 2025. https://arxiv.org/abs/2501.09747

[2] Embodiment Collaboration. *Open X-Embodiment: Robotic Learning Datasets and RT-X Models*. arXiv:2310.08864, 2023. https://arxiv.org/abs/2310.08864

[3] Physical Intelligence. *π0: Our First Generalist Policy*. PI Blog, 2024-10-31. (Companion technical report; see also references in [1, 9, 14].)

[4] J. Yang, C. Finn, D. Sadigh. *Data Analogies Enable Efficient Cross-Embodiment Transfer*. arXiv:2603.06450, 2026.

[5] P. Wang, K. Hong, C. Peng, et al. *DiscreteRTC: Discrete Diffusion Policies are Natural Asynchronous Executors*. arXiv:2604.25050, 2026.

[6] Z. Wu, J. Ye, Z. Zhang, et al. *Speedup Patch: Learning a Plug-and-Play Policy to Accelerate Embodied Manipulation*. arXiv:2603.20658, 2026.

[7] H. Chen, J. Liu, C. Gu, et al. *Fast-in-Slow: A Dual-System Foundation Model Unifying Fast Manipulation within Slow Reasoning*. arXiv:2506.01953, 2025. DOI: 10.48550/arxiv.2506.01953

[8] K. Kawaharazuka, J. Oh, J. Yamada, et al. *Vision-Language-Action Models for Robotics: A Review Towards Real-World Applications*. IEEE Access, 2025. DOI: 10.1109/access.2025.3609980

[9] D. Driess, J. T. Springenberg, B. Ichter, et al. *Knowledge Insulating Vision-Language-Action Models: Train Fast, Run Fast, Generalize Better*. arXiv:2505.23705, 2025. https://arxiv.org/abs/2505.23705

[10] W. Zhang, Y. Wang, H. Luo, et al. *DiG-Flow: Discrepancy-Guided Flow Matching for Robust VLA Models*. arXiv:2512.01715, 2025.

[11] B. F. Jeon, Y. Choi, T. Kim. *Shallow-π: Knowledge Distillation for Flow-based VLAs*. arXiv:2601.20262, 2026.

[12] X. Li, M. Cai, J. Xu, et al. *AT-VLA: Adaptive Tactile Injection for Enhanced Feedback Reaction in Vision-Language-Action Models*. arXiv:2605.07308, 2026.

[13] C.-Y. Hung, Q. Sun, P. Hong, et al. *NORA: A Small Open-Sourced Generalist Vision Language Action Model for Embodied Tasks*. arXiv:2504.19854, 2025.

[14] K. Black, M. Y. Galliker, S. Levine. *Real-Time Execution of Action Chunking Flow Policies*. arXiv:2506.07339, 2025. https://arxiv.org/abs/2506.07339

[15] C. Eze, C. Crick. *Learning by Watching: A Review of Video-based Learning Approaches for Robot Manipulation*. arXiv:2402.07127, 2024.

[16] D. Qu, H. Song, Q. Chen, et al. *SpatialVLA: Exploring Spatial Representations for Visual-Language-Action Models*. RSS 2025. DOI: 10.15607/rss.2025.xxi.011

[17] M. Li, Z. Wang, K. He, et al. *JARVIS-VLA: Post-Training Large-Scale Vision Language Models to Play Visual Games with Keyboards and Mouse*. ACL Findings, 2025.

[18] S. Fei, S. Wang, J. Shi, et al. *LIBERO-Plus: In-depth Robustness Analysis of Vision-Language-Action Models*. arXiv:2510.13626, 2025.

[19] Y. Dai, G. Ananthanarayanan, L. Cox, et al. *Kairos: A Scalable Serving System for Physical AI*. arXiv:2605.11381, 2026.

---

**说明**：上述 14 项 PI 工作中，π0 [3]、π0.5、Hi Robot、π\*0.6、人到机器人迁移、Moravec 奥运会、MEM、RL Token、π0.7 与 Physical Intelligence Layer 等以 PI 官方博客形式公开，其中部分尚未在 arXiv 或同行评审会议上有完整对应论文，本报告基于博客描述总结其方法与贡献；FAST [1]、KI [9]、RTC [14] 已有正式论文，引用时给出 arXiv 标识。所有外部对照与扩展工作（[2,4–8,10–13,15–19]）均经 paper_search 验证，引用条目对应真实可查的论文记录。
