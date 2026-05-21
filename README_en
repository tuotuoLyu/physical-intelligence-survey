# From π0 to π0.7: A Survey of Physical Intelligence's Roadmap Toward General-Purpose Robot Foundation Models

**Scope of this report.** This survey systematically reviews 14 publicly released works from Physical Intelligence (PI) between October 2024 and April 2026, covering the company's first generalist policy π0, the action tokenizer FAST, the open-world generalization model π0.5, the real-time control system RTC, the Knowledge Insulating (KI) training paradigm, the human-in-the-loop reasoning system Hi Robot, the experience-learning model π\*0.6, human-to-robot transfer, the Moravec challenge, the Physical Intelligence Layer, long- and short-term memory (MEM), online RL Token, and the most recent steerable model π0.7.

**Purpose of this writing.** Along five threads — single-model architecture → training-data paradigm → inference systems → learning signals → deployment ecosystem — we distill the internal logic, technical evolution, and open problems of this research program.

---

## 1. Background and Research Threads

Robotics has long been haunted by Moravec's paradox: tasks "hard" for humans — chess, theorem proving — yield easily to symbolic systems, while tasks "easy" even for toddlers — grasping, folding, cleaning — remain extraordinarily difficult for machines. Since 2024, Physical Intelligence has pushed a unified roadmap targeting "general-purpose physical intelligence models," based on a central hypothesis: as with the GPT trajectory in NLP, only when model scale, cross-embodiment data scale, and training methodology are jointly mature can robotics break out of the bottleneck of isolated skill learning and exhibit genuine open-world generalization.

Around this hypothesis, PI's work unfolds along five interlocking threads: (1) model architecture and training objectives (the Vision-Language-Action (VLA) backbone shared by π0, π0.5, π0.7); (2) action representation (the FAST tokenizer, which makes autoregressive VLAs viable); (3) inference and deployment (Real-Time Chunking, Physical Intelligence Layer); (4) learning signals (Hi Robot's human-in-the-loop signal, the online reinforcement learning of π\*0.6 and RL Token, human-video transfer); (5) system-level capability extensions (the MEM long- and short-term memory, the Moravec Robot Olympics, the steerability of π0.7). Below we unpack each work along the chronological axis, and synthesize the overall picture in Section 16.

---

## 2. π0: The First Generalist Robot Policy (October 2024)

π0 is PI's first publicly released generalist policy, positioned as a "prototype foundation model." Its core architecture attaches an **action expert** to a pretrained Vision-Language Model (VLM) backbone, and adopts **flow matching** as the training objective for the continuous action distribution. Given multi-view images and a natural-language instruction, the model produces an action chunk of length H in one shot, avoiding the cumulative error and latency of step-by-step autoregressive decoding.

The training data consists of multi-robot, multi-task teleoperation demonstrations, the scale of which crosses the conventional "single-embodiment, single-skill" paradigm for the first time. This architecture and training recipe have been repeatedly inherited and refined in subsequent work. π0 also validates a key judgment: through imitation learning alone, given sufficiently diverse data, a single-weight model can already exhibit substantial zero-shot performance on real-world tasks such as folding laundry and clearing tableware [3].

It should be noted that π0 did not emerge in isolation. The Open X-Embodiment project [2] simultaneously catalyzed the open ecosystem for cross-embodiment data, providing both an important counterpart and a data foundation for generalist VLAs; π0's contribution lies in walking through the full combination — "VLM backbone + action expert + flow matching + cross-embodiment demonstrations" — in one pass, and validating it directly on production-grade hardware.

---

## 3. FAST: A 5× Speedup for Autoregressive VLAs via Action Tokenization (January 2025)

Flow matching is powerful, but incompatible with the autoregressive (AR) decoding stack of existing VLMs, making it hard to reuse all the tooling brought by large-scale text pretraining. FAST [1] tackles this problem head-on. The core observation is that robot action sequences are highly redundant in the time domain and low-bandwidth; naïvely quantizing each per-dimensional floating-point action into a discrete token yields very long, highly predictable sequences, leaving the Transformer to fit trivial temporal correlations rather than task semantics.

FAST's solution is **frequency-domain tokenization**: each action chunk is first mapped to the spectral domain via a discrete cosine transform (DCT), and the coefficients are then quantized and compressed with byte-pair encoding (BPE). Because action energy is concentrated at low frequencies, the compressed token sequence is dramatically shortened, and AR training efficiency rises accordingly. The paper reports that, under equal data, training a generalist policy with FAST is roughly 5× faster than prior best discretization schemes, while maintaining or improving task success rate [1].

The significance of FAST is not merely "speed" — it enables PI to maintain two technical stacks in parallel: the **π0 line** (flow-matching action expert, continuous output, low latency) and the **π0-FAST line** (autoregressive discrete actions, benefiting from VLM tooling and interpretable intermediate representations). This "dual-stack" strategy provided the flexibility for subsequent open-source releases and system-level deployment.

---

## 4. Open-Sourcing π0 (February 2025)

In February 2025, PI released the weights and code for both π0 and π0-FAST. These were among the first industry-grade generalist VLAs to enter the open-source ecosystem, and their impact rapidly compounded over the following year: a large body of subsequent work — DiG-Flow [10], Shallow-π knowledge distillation [11], AT-VLA tactile injection [12], among others — directly used π0 or π0-FAST as a baseline for refinement, distillation, extension, or robustness analysis.

Methodologically, the open-source release was not only a product decision but a scientific one: it forced PI to publicly disclose key details of the training recipe and inference stack, allowing external researchers to verify the reproducibility of the "VLM backbone + cross-embodiment data + flow matching/FAST" route under controlled comparisons, and quickly exposing its failure modes (e.g., out-of-distribution brittleness, language-perturbation fragility). Much subsequent work has been organized around the problems thus exposed.

---

## 5. Hi Robot: Human-in-the-Loop Hierarchical Reasoning (February 2025)

Hi Robot takes "understanding complex instructions and executing them step by step" as its objective, and proposes a **hierarchical reasoning** structure: a high-level VLM decomposes abstract instructions (e.g., "Clean this table, but leave the red cup") into an executable sequence of sub-instructions, while a low-level policy (i.e., π0 or π0-FAST) executes them in turn. The key design is support for **human-in-the-loop feedback**: the user can interject corrective language during execution ("No, don't throw that cup away"), and the high-level module rewrites the remaining sub-instructions on the fly.

This work is the first within PI's program to introduce a "System 2"-style slow-thinking capability and to open up the "task planning — action execution" loop to human input. It directly foreshadows the design motivations for the subsequent MEM long- and short-term memory and the steerability of π0.7: to deploy a generalist policy in long-horizon, interactive, correctable real-world scenarios, action generation alone is not enough; the planning layer and the human-machine protocol must be explicitly modeled.

---

## 6. π0.5: Open-World Generalization (April 2025)

π0.5 is the direct successor to π0, and its central claim is "executing long-horizon tasks in homes the robot has never seen before." The report shows a mobile manipulator completing cleaning and tidying tasks in **entirely unfamiliar** kitchens and bedrooms, and argues that this generalization arises from **heterogeneous co-training**: cross-embodiment teleoperation data, cross-task web-multimodal data, cross-scene video, and high-level semantic annotations are all jointly incorporated into training.

π0.5 argues that open-world generalization is not simply a "more robot data" problem; rather, the model must be exposed during training to supervision signals that are diverse in source but closer to the deployment distribution. This idea is further formalized as a "cross-embodiment data organization" problem by independent follow-up work such as Data Analogies [4], and continues to shape the training recipe of π0.6/0.7.

---

## 7. KI: Train Fast, Run Fast, Generalize Better (May 2025)

Knowledge Insulating (KI) [9] resolves a seemingly contradictory engineering paradox: when a VLA is fine-tuned on large amounts of robot-action data, it tends to **destroy the valuable semantic priors in the VLM backbone**, leading to degradation of instruction-following ability and weaker generalization. KI proposes to "insulate" the VLM representation during training, so that gradients from the action head do not contaminate the semantic backbone, while a carefully designed interface still allows the action head to read information from the semantic features efficiently.

The empirical result is a triple gain: training throughput increases substantially, inference latency drops, and language-conditioned generalization outperforms the directly end-to-end fine-tuned baseline [9]. This paper also provides crucial empirical evidence for the subsequent debate of "when should a VLA be end-to-end and when should it be modular," and has been reused by open-source VLAs such as NORA [13] and MetaVLA.

---

## 8. RTC: Real-Time Execution of Action-Chunking Flow Policies (June 2025)

The larger the VLA, the slower the inference — but the physical world will not wait for the model, and the tension between slow thinking and real-time control becomes increasingly sharp in the era of large VLAs. Real-Time Action Chunking (RTC) [14] addresses this problem head-on. Its core idea is to decouple "chunk-level generation" from "step-level execution": the model asynchronously generates the next action chunk, and the controller triggers the next generation request before the current chunk is consumed, stitching the chunks together in an asymptotically consistent way.

RTC preserves the high-precision output of flow-matching / diffusion policies, while at the same time maintaining smooth, real-time control frequencies under inference latencies on the order of 300 ms, enabling large VLAs to drive contact-rich, tightly paced tasks (e.g., bimanual coordination, dynamic contact) [14]. This system-level contribution was subsequently extended by several asynchronous-execution works [5] and acceleration-patch works [6], and became one of the core components of the actual deployment stack underlying the PI Layer (see Section 10).

---

## 9. Moravec's Paradox and the "Robot Olympics" (December 2025)

In December 2025, PI released a particularly provocative collection of demonstrations: using the latest π-series models with task-specific fine-tuning to take on a series of operational tasks long regarded by researchers as "high difficulty" — fine assembly, dynamic balancing, multi-step assembly, and more — together constituting a "robot olympics."

The scientific value of this work lies in **reverse-validating the viability of the foundation-model paradigm**: each of these challenge tasks would have required months of dedicated engineering under a traditional task-specific learning framework, whereas starting from a single generalist π model, only a small amount of targeted fine-tuning suffices to solve them. This parallels exactly the NLP story of GPT — "general base + a small amount of fine-tuning → cross-task capability" — and provides strong empirical evidence that Moravec's paradox is being meaningfully shaken at the level of robotics.

---

## 10. The Physical Intelligence Layer: A Deployment Ecosystem for General-Purpose Physical Intelligence (February 2026)

By early 2026, π-series models had been deployed to real production settings by several partners. In *The Physical Intelligence Layer*, PI proposes the concept of a "physical intelligence layer": viewing general-purpose physical intelligence models as a **callable infrastructure layer** across hardware and industries, analogous to the operating-system layer of the cloud era. The argument is that once the underlying model has sufficient generality, robotics applications will see a "Cambrian explosion" — every application that previously required a bespoke perception-planning-control stack can now be incubated atop the same π model at the pace of weeks or even days.

The article is itself a product/strategy manifesto, with relatively light technical content, but it delineates the product boundary for subsequent works like π0.7: the model must be explicitly **steerable** by partners, it must support long-horizon memory, and it must be able to continue improving from deployment experience. These three requirements correspond directly to the research themes of Sections 11–14.

---

## 11. Human-Video-to-Robot Transfer Emerges in VLAs (December 2025)

This work investigates a long-standing open question: human manipulation videos differ greatly from robotic embodiments (no joint correspondence, different viewpoints, mismatched kinematics) — can VLAs benefit from them without explicitly modeling these differences? The answer is yes: when the model is large enough, **transfer from human videos to robotic tasks emerges spontaneously**.

The report shows that as the scale of the VLA backbone and cross-modal data grow together, the model gradually learns from "watching a human do X" the high-level intent and temporal structure needed for "the robot to do X" — without any additional human-robot explicit alignment loss. This finding is consistent with survey works such as Learning by Watching [15], and provides direct empirical justification for extending the robot training set with video data.

---

## 12. π\*0.6: A VLA That Learns from Experience (November 2025)

π\*0.6 takes "the ability to keep learning after training" as a core feature. Building on π0/π0.5, it introduces a **real-world reinforcement learning (real-world RL)** training pipeline: execution trajectories and sparse success/failure signals during deployment are fed back into the training loop, and used to **simultaneously improve task success rate and execution throughput**.

The key contribution of this work is embedding the RL signal in a generalist policy rather than in task-specific policies, avoiding the trap of "engineering a separate reward for every task." The results directly preview the direction of Section 14, where RL Token pushes this idea further into finer-grained actions, and underpin the "continuous improvement after deployment" promise of the PI Layer.

---

## 13. MEM: A VLA with Both Long-Term and Short-Term Memory (March 2026)

Multi-Scale Embodied Memory (MEM) introduces **multi-scale memory** to a VLA: short-term memory serves step-level action coherence, while long-term memory maintains environment and intent state across hours and across tasks. The report shows a π+MEM system completing complex household tasks lasting more than ten minutes and composed of dozens of sub-steps (e.g., "prepare a three-course breakfast"), with humans allowed to interrupt and add constraints mid-execution.

The technical significance of MEM is that it resolves the system bottleneck of "action-chunk-based policies lacking cross-chunk state." Simply lengthening the context window cannot resolve this, because the token cost of visual input is prohibitive; MEM takes a hierarchical abstraction approach — the lower layer retains recent visual/action frames, while the upper layer summarizes environment state into a semantic structure reusable by the planner. This idea forms a structural loop with Hi Robot's hierarchical reasoning, giving PI's system, for the first time, the operational capability for cross-hour consistency on long-horizon tasks.

---

## 14. RL Token: Efficient Online RL for Precision Manipulation (March 2026)

This work is directly aimed at high-precision, time-sensitive tasks (insertion, alignment, fine assembly): a dedicated **RL Token** is "carved out" of the VLA, so that online RL only needs to update parameters associated with that token, rather than the entire VLA backbone. The result is that a few hours of real-world data suffice to substantially raise both throughput and success rate on precision tasks that had previously been hard to crack.

Methodologically, RL Token is a refinement of the π\*0.6 idea: the coarse formulation of "whole-model online learning" is sharpened to "preserve general-purpose capability, while applying RL signals in a targeted way at critical decision points." Conceptually, this "modular online learning" is highly complementary to KI's "knowledge insulation": the latter protects the general-purpose semantic prior from being broken, while the former permits fine-grained local optimization outside that protected scope.

---

## 15. π0.7: The Latest Model with Steerability and Emergent Capabilities (April 2026)

π0.7 is the most recent generation, positioned as **steerable** and exhibiting **qualitatively step-change generalization**. Steerability concretely means that partners can change the model's behavior explicitly through structured language, examples, or small amounts of tuning — without retraining the entire model.

The report highlights three categories of emergent capability: (1) zero-shot cross-scene transfer no longer degrades linearly across a broader distribution; (2) multimodal long-instruction following improves substantially; (3) without dedicated fine-tuning, the model spontaneously produces reasonable grasping and use strategies for novel tools and novel objects. This release rhetorically moves "general-purpose physical intelligence" from an engineering goal to a foundation-model layer that can be productized and customized on demand by external teams — and constitutes the present-day node of this roadmap.

---

## 16. Synthesis: The Internal Logic of a Unified Research Program

Taken together, these 14 works show that PI's research is not a series of isolated product iterations, but unfolds around a clearly articulated thesis: **a general-purpose physical intelligence model = a strong semantic-prior backbone + cross-embodiment heterogeneous data + an efficient action representation + a real-time inference system + a continuous learning signal + a steerable interface.** Each paper maps precisely to one or more of these six components.

The backbone and training objective are carried forward by π0 [3], π0.5, KI [9], and π0.7 in a relay; the action representation is provided by FAST [1] and the open-source π0-FAST; real-time inference is addressed by RTC [14]; continuous learning is jointly supported by π\*0.6, RL Token, and human-video transfer; long-horizon and interactive capability is closed-looped by Hi Robot and MEM; the steerable interface and the surrounding ecosystem are crystallized in the PI Layer and π0.7. The Moravec Olympics provides end-to-end capability validation on the most challenging tasks.

From the angle of how the field has evolved, this roadmap has imposed three structural effects on the broader VLA research community. First, it has **established the de facto standard of VLM backbone + action expert** — almost all subsequent VLAs (NORA [13], SpatialVLA [16], JARVIS-VLA [17], and so on) develop within this architecture. Second, the **open-sourcing of FAST and π0** has moved the community from "each lab closed-source" to "controlled comparisons on a common baseline," which is what makes systematic robustness studies such as DiG-Flow [10] and LIBERO-Plus [18] possible. Third, **RTC and the PI Layer** were the first to raise "how does a VLA actually run in real production environments?" as a first-class question, catalyzing system-level follow-up work such as Kairos [19] and Speedup Patch [6].

**Open problems** remain significant. (1) Robustness: independent evaluations such as LIBERO-Plus [18] show that current VLAs (including the π0 family) remain brittle to language perturbations and sensor noise. (2) Verifiability: steerability means partners can inject constraints, but there is at present no formal method for verifying that the model actually respects those constraints. (3) Data transparency: the concrete recipes and licensing of cross-embodiment heterogeneous data are not publicly disclosed, making it difficult for external researchers to rigorously reproduce π0.5/0.7. (4) Real cost: the true expense of online RL and long-horizon deployment — in hardware, energy, and human supervision — has scarcely been quantified in public materials. Together, these four points define a clear window of opportunity for the next phase of research.

---

## References

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

**Note.** Of the 14 PI works covered above, π0 [3], π0.5, Hi Robot, π\*0.6, human-to-robot transfer, the Moravec Olympics, MEM, RL Token, π0.7, and the Physical Intelligence Layer are published as PI official blog posts; some of them do not yet have a fully corresponding paper on arXiv or in a peer-reviewed venue, and this report summarizes their methods and contributions based on the blog descriptions. FAST [1], KI [9], and RTC [14] already have formal papers and are cited with their arXiv identifiers. All external comparative and extension works ([2, 4–8, 10–13, 15–19]) have been verified via paper_search, and their references correspond to real, retrievable paper records.
