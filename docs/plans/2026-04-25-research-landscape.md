# AI 小说创作 Research Landscape 调研（2024-2026）

**调研日期**：2026-04-25
**范围**：6 个领域
**目的**：避免只盯 StoryBox 做井底优化；找出更广的同路人/可借鉴工作
**已排除的熟知工作**：StoryBox / StoryVerse / IBSEN / Agents' Room / StoryWriter（正文细节见各自笔记）

---

## 1. Multi-Agent Story Generation（查漏补缺）

### 1.1 HoLLMwood — 角色 agent + Editor 反馈循环
- **出处**：Chen, Zhu et al., EMNLP Findings 2024, arXiv:2406.11683
- **核心**：Writer + Editor + Actors 三类；Editor 给 Writer 写修订建议；Actors 互动丰富角色
- **相关性**：强。Editor agent 对应我们 Novel 层之后的"校稿 agent"
- **可借鉴**：Editor→Writer feedback prompt 模板（§3.3）；Plan-then-Write 胜率 76-84%
- **局限**：~2k-3k 词短篇；Actors 陪跑式，非真正独立 agent

### 1.2 Plug-and-Play Dramaturge — 剧本迭代修订
- **出处**：arXiv:2510.05188（2025-10）
- **核心**：Global Review → Scene-level Review → Hierarchical Coordinated Revision 三阶段分治
- **相关性**：强——我们 Novel 层之后"稿件打磨"阶段的现成蓝图
- **可借鉴**：Global/Scene 双层审稿；script-level +53.2%，scene-level +65.1%
- **局限**：输入已是剧本，不覆盖"从模拟到剧本"

### 1.3 LongWriter / LongWriter-Zero — RL 训超长生成
- **出处**：原版 arXiv:2408.07055（ICLR 2025，清华 THUDM）；Zero 版 arXiv:2506.18841（ICLR 2026，Qwen2.5-32B + RL）
- **核心**：AgentWrite 拆子任务；Zero 版用纯 RL 做长度+质量 reward
- **相关性**：中——场景驱动架构天然回避单次超长生成，但 Novel 层每幕扩写可用
- **可借鉴**：LongWriter-Zero 在 WritingBench 8.69 超过 GPT-4o/DeepSeek-R1；**32B 开源权重可直接微调**
- **局限**：只解决"单轮长输出"，不解决跨幕一致性/伏笔/弧光

### 1.4 SCORE — 符号+RAG 的长故事一致性框架
- **出处**：arXiv:2503.23512（2025-03）
- **核心**：Dynamic State Tracking（符号）+ Context-Aware Summarization + Hybrid Retrieval
- **相关性**：强。他们的 State Tracking 就是我们 Replay 层元信息的实例化
- **可借鉴**：character consistency / emotional coherence / plot element tracking 三维评测；TF-IDF+embedding 双通道检索
- **局限**：未覆盖多 agent 模拟；State Tracking schema 未公开

### 1.5 CollabStory — 多 agent 接力写作
- **出处**：arXiv:2406.12665（2024-05）
- **核心**：最多 5 agent 顺序接力写段落
- **相关性**：弱——接力式不是"同时模拟"架构
- **可借鉴**：evaluation metrics (LLM-as-judge 维度)
- **局限**：本质 chained writer

---

## 2. Narrative Planning（Mark Riedl 学派）

### 2.1 STORY2GAME — Riedl 组
- **出处**：Zhou, Basavatia, Siam, Chen, Riedl. arXiv:2505.03547（2025-05）
- **核心**：LLM 生成 story → populate world → 生成 action 的 preconditions/effects → game engine 执行
- **相关性**：强——precondition/effect 是 GM agent 做世界裁定的形式化基础
- **可借鉴**：用 LLM 生成动作的前置条件与效果作为世界一致性锚
- **局限**：IF 游戏范式（小空间、少角色），长篇未验证

### 2.2 Can LLMs Generate Good Stories? — Narrative Planning benchmark
- **出处**：Farrell, Ware et al. arXiv:2506.10161（2025-06，Autodesk + UKY）
- **核心**：基于经典叙事规划文献构建 ASP 可验证 benchmark（causal soundness / character intentionality / dramatic conflict）
- **相关性**：强——**直接可用 benchmark**。taskset 公开：HuggingFace `ADSKAILab/LLM-narrative-planning-taskset`
- **可借鉴**：ASP 形式化三核心属性；GPT-4 tier 在 causal 过关但 intentionality/conflict 弱
- **局限**：符号 + 小规模评测，不做生成

### 2.3 LLMs as Narrative Planning Heuristics/Search Guides
- **出处**：Farrell, Ware et al. FDG 2025, DOI: 10.1145/3723498.3723815
- **核心**：LLM 当 heuristic 引导经典 planner 搜索（非生成文本）
- **相关性**：中——混合符号+LLM，和我们"LLM 模拟 + DM 裁判"同构
- **可借鉴**：LLM 建议 promising branches + 符号 planner 保 validity 的分工
- **局限**：传统 narrative planner state space 在长篇不可行

### 2.4 Pareto-Based Narrative Planning
- **出处**：Siler, Fisher, Ware. AIIDE 2025, DOI: 10.1609/aiide.v21i1.36837
- **核心**：多目标 Pareto 前沿选 NPC 动作（"最好"与"最安全"权衡）
- **相关性**：中——GM 控制 NPC 行为有启发（不只追目标，还要"不蠢"）
- **可借鉴**：多目标决策替代单目标
- **局限**：符号 planner 基础，LLM 迁移未做

### 2.5 From Emergence to Planning — Triangle Framework ⭐
- **出处**：Senanayake. AIIDE 2025, DOI: 10.1609/aiide.v21i1.36858
- **核心**：叙事系统定位在三角区域：emergent MAS / reactive rules / centralized planner，中间是"landmark-guided hybrid"
- **相关性**：**非常强——目前最清晰对我们系统做学术定位的工作**
- **可借鉴**：landmark = 命运变量/Countdown Clock 的学术框架化；我们架构正好在三角中间
- **局限**：position paper，无具体实现

---

## 3. Drama Management（LLM 时代的 Facade 继任）

### 3.1 Drama Llama — storylets + LLM 自然语言 trigger ⭐
- **出处**：Kreminski et al. arXiv:2501.09099（2025-01，Santa Cruz + Mateas 组）
- **核心**：作者用自然语言写 triggers（条件+效果），LLM 判 trigger 激活 + 生成 stage direction
- **相关性**：**非常强——Facade 的直接学术继任，Mateas 组出品**
- **可借鉴**：natural-language triggers 作为作者接口比命运变量更具体；Basic/Ending trigger 二分
- **局限**：6 作者、每本 3-4 triggers 短剧；未证长篇；无模拟层、无 Replay IR

### 3.2 Open-Theatre — 开源互动戏剧 toolkit ⭐⭐⭐
- **出处**：arXiv:2509.16713（EMNLP Demos 2025）；**GitHub: johnnie193/Open-Theatre**
- **核心**：集成 one-for-all / director-actor / hybrid 三种架构，统一 hierarchical 记忆系统
- **相关性**：**最强——开源、可做 parity baseline**
- **可借鉴**：Director–Global Actor 架构；developer console / player console / monitor 三件套 UI；hierarchical memory
- **局限**：面向"玩家即时互动"而非"作者离线创作"

### 3.3 Towards Enhanced Immersion and Agency
- **出处**：arXiv:2502.17878（ACL 2025, Tsinghua/Peking）
- **核心**：Playwriting-guided Generation + Plot-based Reflection；悬念胜率 10%→32%，情感张力 10%→48%
- **相关性**：强——首次把戏剧理论显式嵌入 LLM prompt
- **可借鉴**：playwriting prompt schema（亚里士多德六要素）；Plot-based Reflection 作为角色 agent 自修正
- **局限**：玩家驱动互动剧；不做长篇

### 3.4 CoDi — Director-Actor Planner-Editor 四角色
- **出处**：Kim, Yoo, Cheong. AIIDE 2025, DOI: 10.1609/aiide.v21i1.36811；GitHub: Speeditidious/CoDi
- **核心**：planner + character + director + editor 四 agent；director 可引入新事件、选 NPC
- **相关性**：强——四 agent 分工对应 GM/DM/角色/成文
- **可借鉴**：director 作为"叙事目标驱动"而非"情节直接指挥"
- **局限**：短剧；director 可直接插事件（IBSEN 问题变体，要避开）

### 3.5 DramaLLM — Drama-Interaction 六要素
- **出处**：aclanthology.org/2024.findings-acl.196；GitHub: vickywu1022/DramaLLM
- **核心**：Narrative Chain + Auto-Drama + Sparse Instruction Tuning
- **相关性**：中——六要素（plot/character/thought/diction/spectacle/interaction）可作评测
- **局限**：短剧脚本、评测驱动

---

## 4. Computational Narratology

**诚实判断：该领域近 2 年无 breakthrough 论文对接 LLM 时代。**

### 4.1 Narrative Theory-Driven LLM Methods — Survey
- **出处**：arXiv:2602.15851
- **核心**：把叙事理论（形式主义/结构主义/认知叙事学）和 LLM 方法 mapping
- **相关性**：中——参考索引
- **局限**：survey，无原创

### 4.2 CMN Workshop 第 8 届（ACL 2025）
- **窗口**：活跃但 landmark 缺席，多为小规模 case study

### 4.3 A Survey on LLMs for Story Generation
- **出处**：aclanthology.org/2025.findings-emnlp.750（EMNLP Findings 2025）
- **核心**：近 3 年 LLM 故事生成 taxonomy（agent-based / planning-based / multi-stage）
- **相关性**：强——landscape 起点最权威

### 4.4 Marie-Laure Ryan 学派 — 近 2 年无新作对接 LLM
Ryan 1991 年 Possible Worlds 专著仍是主引用。

---

## 5. Interactive Storytelling（Storytron 后）

### 5.1 WHAT-IF — 分支叙事 meta-prompting ⭐
- **出处**：Huang, Martin, Callison-Burch. arXiv:2412.10582（Wordplay @ EMNLP 2025, Penn）
- **核心**：给定线性故事，每个关键决策点 meta-prompt 生成替代分支，树状结构维护
- **相关性**：**强——我们"世界树"的直接学术对照**
- **可借鉴**：plot-tree 数据结构；major plot points 作为 meta-prompt 锚
- **局限**：从已完成故事做分支（非从模拟中自然分叉）；短篇

### 5.2 Fable Showrunner — 商业 demo
- **出处**：2023 SHOW-1，2025 Amazon Alexa Fund 投资 SHOW-2
- **技术细节全黑盒**
- **相关性**：中——产品层验证商业可行性
- **局限**：无公开论文

### 5.3 AI Dungeon / Inworld / Friends-and-Fables
- **技术细节基本闭源**
- 反面案例（单 prompt 混层失败）仍适用
- 无新技术可学

### 5.4 Emily Short / Nick Montfort — 2024-2025 无学术产出
- Emily Short 2024-01 离 Failbetter 进入 Wizards of the Coast；blog 活跃无论文
- Montfort 无对 LLM 叙事的专门学术论文

### 5.5 RELATE-Sim — Scene Master 建模长期关系 ⭐⭐⭐
- **出处**：arXiv:2510.00414（2025-10）
- **核心**：基于 Turning Point Theory，Scene Master 逐 turning point 推进
- **相关性**：**极强——最接近我们"场景驱动"的已发表工作**
- **可借鉴**：Turning Point Theory 挑 scene 位置；Scene Master state tracking schema
- **局限**：只模拟情侣关系垂直场景，长篇未验证

---

## 6. 场景驱动 / Scene-Based Generation（我们的核心差异化）

**重要发现：有人做了，但都比我们浅。组合差异化仍成立。**

### 6.1 RELATE-Sim（见 §5.5）— 最直接对标
- **与我们相似度 ~60%**：Scene Master 框定 scenario → 给有限选项 → 推进叙事 → 推断 state 变化
- **我们的差异**：(a) 他们单线，我们多世界线；(b) 他们短对话，我们 10 万字；(c) 他们只关系科学，我们通用创作

### 6.2 R² — Novel-to-Screenplay with Causal Plot Graphs
- **出处**：arXiv:2503.15655（2025-03）
- **核心**：Reader 建因果 plot graph（贪心去环）+ Rewriter 按 scene 生成
- **相关性**：强——"scene by scene generation"明确；胜率 +51.3%
- **可借鉴**：causal plot graph 作为场景间因果连接的形式化；sliding window 跨章读取
- **局限**：输入是已有小说（适配而非生成），不做模拟

### 6.3 Narrative-to-Scene Generation for 2D Games
- **出处**：arXiv:2509.04481（2025-09）
- **相关性**：弱——2D 游戏布景，不是叙事结构
- **可借鉴**：三个 time frame 的时间结构化 prompting

### 6.4 LLM-Based Authoring of Agent-Based Narratives through Scene Descriptions
- **出处**：arXiv:2512.20550（Purdue, 2025-12）
- **核心**：拖拽 agent+object 到 scene，自动生成 SceneDirector 动作序列
- **相关性**：中——"场景作为作者工作单元"的 UI 实例
- **局限**：短 scenario；动作执行只驱动 Unity 动画

### 6.5 Beyond Direct Generation — Decomposed Screenwriting
- **出处**：arXiv:2510.23163（2025-10）
- **核心**：logline → outline → scene 三层分解
- **相关性**：中——层级分解接近我们，但完全 top-down，无模拟层
- **可借鉴**：logline 作为最顶层不变量

### 6.6 Ink / Yarn Spinner 学术化 — 近期停滞
分支叙事脚本语言流行，学术化 2024-2025 冷清。2025 有一本业界书《Narrative Systems Design for Games》做 survey，非学术。

---

## 综合判断

### 已有成熟方法可直接抄

| 子问题 | 现成方法 | 出处 |
|---|---|---|
| 单次长输出 | LongWriter-Zero 32B 开源权重微调 | arXiv:2506.18841 |
| 稿件打磨 | Dramaturge 三阶段 review | arXiv:2510.05188 |
| 长故事一致性 | SCORE Dynamic State Tracking + 双通道检索 | arXiv:2503.23512 |
| 自然语言 trigger | Drama Llama trigger schema | arXiv:2501.09099 |
| 四角色分工 | CoDi planner/character/director/editor | AIIDE 2025 |
| 场景级 Scene Master | RELATE-Sim state schema | arXiv:2510.00414 |
| 因果图 | R² causal plot graph | arXiv:2503.15655 |
| benchmark | Autodesk/UKY ASP taskset (HF 公开) | arXiv:2506.10161 |

### 开放问题 = 机会

1. ⭐ **10 万字级场景驱动** — 所有现有工作都在短篇/垂直场景
2. ⭐ **中文网文特化** — landscape **零覆盖**
3. **多世界线回归 + cherry-pick 心理兼容** — WHAT-IF 只分不归
4. **作者心理契约** — 学术不关心、商业闭源
5. **Replay 作为可审计产物的作者面板** — 没人暴露给作者看
6. **长程伏笔跨幕追踪** — 所有系统只做到章内

### 差异化在更广 landscape 下的判断

| 原差异化 | 最终判断 |
|---|---|
| Drama Manager | ✅ 站得住。独特的不是"加 DM"而是 **命运变量 + 时钟 + Front 三位一体** 配置 |
| 多世界线 | ⚠️ WHAT-IF 已做树结构但只分不归；我们的"回收 + cherry-pick 兼容检查"仍独特。定位应明确为"**素材期工具**" |
| Replay 作为 IR | ⚠️ **"可独立销售"主张应彻底放弃**；保留"IR + 可审计面板"工程价值 |
| GM agent | ✅ 仍独特。现有系统要么角色独立（StoryBox），要么 director 直接插剧情（IBSEN/CoDi），没人把 GM 定位为"中立裁判" |
| 场景驱动 | ⚠️ RELATE-Sim/R² 部分已做。独特性退守到"**场景驱动 + 跨幕时间跳跃 + 多世界线 + 作者节奏控制**"四件套组合 + 10 万字 + 中文 |

### 必读优先级（5 篇）

1. ⭐⭐⭐ **RELATE-Sim** (arXiv:2510.00414) — 最接近场景驱动
2. ⭐⭐⭐ **Drama Llama** (arXiv:2501.09099) — Mateas 组；trigger schema
3. ⭐⭐⭐ **Open-Theatre** (arXiv:2509.16713 + GitHub) — **必跑 demo** 做 parity baseline
4. ⭐⭐ **Can LLMs Generate Good Stories?** (arXiv:2506.10161) — 现成 benchmark
5. ⭐⭐ **Triangle Framework** (AIIDE 2025) — 唯一清晰学术定位

### 可选深读

- R² (arXiv:2503.15655) causal plot graph
- Enhanced Immersion and Agency (arXiv:2502.17878) Playwriting prompt schema
- LongWriter-Zero (arXiv:2506.18841) 成文层模型训练

---

## 诚实声明

- Marie-Laure Ryan、Chris Crawford 继承者、Emily Short 学术产出三处**近期冷清**
- Inworld AI / AI Dungeon 技术细节**闭源**，无法评估
- Computational Narratology 在 LLM 时代**缺乏奠基性新工作**
- arXiv 2512.20550、2510.05188 等 2025 末期论文尚未进入 peer review 验证周期
