# 组件借鉴决策表

**日期**：2026-04-25
**基于**：StoryBox 研读笔记 + 6 领域 Landscape 调研
**目的**：对蓝图的 10 个核心组件，逐一判定"可借鉴 / 部分改造 / 必须自研"
**定位**：v2 → v3 之间的设计决策基材，非完整设计文档

---

## 决策原则

1. **没有合适的就不借鉴**——宁可自研也不强塞
2. **借鉴判断只看"架构假设是否匹配"**——不看论文新旧、不看是否开源
3. **差异化 = 自研项**——不等同于"我们造轮子"，等同于"landscape 没人做好"
4. **评测绝不自研**——现成 benchmark 叠起来就够强

---

## TL;DR：总决策清单

### ⭐ 可以直接抄（占工程量约 60%）

- StoryBox persona schema（Innate/Learned/Currently/Lifestyle + Daily Plan Requirement）
- StoryBox event schema + 存储栈（sqlite3 + FAISS + jina-embeddings-v3 dim 512）
- Drama Llama natural-language trigger schema
- HoLLMwood Writer + Editor 双 agent 分工
- Plug-and-Play Dramaturge 三阶段成文打磨（Global / Scene / Coordinated Revision）
- 整个评测 stack（Autodesk ASP benchmark + StoryBox LLM-eval + ISS + SCORE 三维）

### ⚠️ 部分借鉴 / 改造

- Park 2023 memory/reflection 三件套（要补长程对策）
- WHAT-IF plot-tree 数据结构（要扩展回收 + cherry-pick）
- STORY2GAME precondition/effect（作为 GM 裁定基础）
- RELATE-Sim 场景驱动范式（借思路不抄 schema）
- Triangle Framework "landmark-guided hybrid" 话术

### 🏗️ 必须自研（landscape 零参考）

1. **GM agent 的"中立裁判"定位** — 所有现有 director 都 override 角色
2. **多世界线的 cherry-pick 心理兼容检查** — 无人做过
3. **Replay 工具链**（可视化、diff、回放、审计面板）— 没人暴露给作者
4. **中文网文适配** — landscape 零覆盖
5. **场景驱动的通用 scene schema** — 只有单一场景版本
6. **10 万字级别的 hierarchical summarization** — StoryBox 30 天就掉分

---

## 组件详述

### 1. 角色 agent 内核

**要解决的**：让角色能自主决策、有记忆、保持人设

| 来源 | 方案 | 决策 | 理由 |
|---|---|---|---|
| StoryBox persona schema | Innate / Learned / Currently / Lifestyle / Daily Plan Requirement + 5 层 Living Area colon-path | ✅ **直接抄字段** | 经过 78 人评估 + ISS 验证；命名稳定 |
| Park 2023 memory/reflection/retrieval | 事件流 + 定期反思 + 向量检索 | ✅ **用，但承认长程风险** | StoryBox 自己砍掉了，说明短程社交场景外泛化不稳；我们保留是差异化但要补长程对策 |
| SCORE Dynamic State Tracking | 符号化追踪 character consistency / emotional / plot | ⚠️ **不放角色内部，放 Replay 元信息层** | 角色不该有全局 state 视角（违反信息边界铁律） |
| RELATE-Sim Scene Master state | 场景级 state 追踪 | ❌ **不抄** | 他们专做情侣关系，schema 不通用 |
| Stanislavski super-objective / given circumstances | 心理学内核 | ✅ v2 已有，保留 | |

**最终组合**：StoryBox 字段 + Park 2023 记忆机制 + Stanislavski 心理内核。

---

### 2. GM agent（世界权威、NPC 调度、环境裁定）

**要解决的**：谁维护世界一致性、谁演 NPC、谁判定"角色能不能做这件事"

| 来源 | 方案 | 决策 | 理由 |
|---|---|---|---|
| StoryBox | 没有 GM，环境是被动 YAML 树 | ❌ 他们没做 | — |
| IBSEN Director | director 会主动 reschedule plot | ❌ **反面案例** | 违反我们"不 override 角色"铁律 |
| CoDi director | director 可引入新事件、选 NPC | ❌ 同 IBSEN 问题 | 他们的 director = IBSEN 变体 |
| STORY2GAME (Riedl) | LLM 生成 action 的 **preconditions/effects** | ✅ **部分借鉴** | 把"角色能不能做 X"形式化为 precondition 检查，作为世界裁定基础 |

**结论**：**landscape 没有合适的 GM agent 参考**。所有现有"director"都会 override 角色决策。**GM agent 的"中立裁判"定位是我们必须自研的核心差异化**。

**自研要点**：
- GM **不推动情节**（那是 Drama Manager 的事）
- GM **只做三件事**：世界规则裁定 / NPC 非戏剧性调度 / 环境事件播报
- 用 STORY2GAME 的 precondition/effect 作为裁定形式化基础

---

### 3. Drama Manager（命运变量 + 时钟 + Front）

**要解决的**：作者怎么不 override 角色还能推动情节

| 来源 | 方案 | 决策 | 理由 |
|---|---|---|---|
| **Drama Llama** (Mateas 组 2025-01) | 作者用**自然语言写 trigger**（条件+效果），LLM 判触发 | ⭐⭐⭐ **直接抄 trigger schema** | 命运变量的最佳接口形态；Mateas 组 = Façade 直接继任 |
| Apocalypse World Fronts + Clocks | TRPG 威胁前线 + 倒计时 | ✅ v2 已有，保留 | — |
| Triangle Framework (AIIDE 2025) | "landmark-guided hybrid" 定位 | ✅ **抄学术话术** | 给 DM 一个清晰学术锚点 |
| Façade Drama Manager (2005) | 高度预写 beat 池 + Aristotelian arc | ⚠️ 太古董 | Drama Llama 是现代化继任 |

**最终形态**：作者写一句自然语言 trigger（"if 林尘拒绝丹药 then 师父出现"），DM 判断触发时机并把事件注入 sandbox。命运变量 + 时钟 + Front 三位一体配置是我们的。

---

### 4. 多世界线 / 世界树

**要解决的**：分叉、cherry-pick、心理兼容检查

| 来源 | 方案 | 决策 | 理由 |
|---|---|---|---|
| WHAT-IF (Penn 2024) | plot-tree 数据结构 + meta-prompt 生成替代分支 | ✅ **抄 plot-tree 数据结构** | 树状存储现成 |
| Git 分支模型 | branch / cherry-pick / rebase | ✅ 概念借鉴 | UI/UX 参考 |

**结论**：**landscape 里几乎全要自研**。WHAT-IF 只做"分叉"，没做"回收"和"cherry-pick 心理兼容检查"。

**自研要点**：
- 分支回收机制（设为正史、存档、废弃）
- **cherry-pick 心理兼容性检查**——把分支 A 的对白摘到正史 B 时，检查正史 B 里的角色心理状态能否支撑该对白
- 摘取三档：直接摘 / 改写摘 / 禁止摘

这是我们**最硬的差异化**，也意味着没 schema 可抄。

---

### 5. Replay 层（结构化中间表示）

**要解决的**：结构化事件存储 + 可审计 + 作者审稿面板

| 来源 | 方案 | 决策 | 理由 |
|---|---|---|---|
| **StoryBox event schema** (Fig 2) | Event ID / Start-End / Duration / Participants / Location / Description / Detail | ⭐⭐⭐ **直接抄** | 结构清晰、已验证 |
| **StoryBox 存储栈** | sqlite3 + FAISS + jina-embeddings-v3 (dim 512) | ✅ **直接抄整个栈** | 工程组合经过验证 |
| SCORE 三维 state | character / emotional / plot 三维 | ✅ **作为 Replay 元信息字段** | 补齐状态追踪维度 |

**结论**：**数据模型 + 存储实现基本全抄**。我们自己要做的是**工具链**：可视化面板、diff、回放、审计 UI——这是真正的差异化。

**自研要点**：
- 事件可视化面板
- 跨世界线 diff 视图
- 场景回放器
- 作者审计入口（"为什么林尘这里这么说？" → 跳转到相关记忆/情境）

---

### 6. 成文层

**要解决的**：Replay → 小说散文的翻译

| 来源 | 方案 | 决策 | 理由 |
|---|---|---|---|
| **HoLLMwood** (EMNLP 2024) | Writer + Editor 双 agent 反馈循环 | ✅ **抄双 agent 分工** | 胜率 76-84%，成熟 |
| **Plug-and-Play Dramaturge** (2025-10) | Global Review → Scene Review → Hierarchical Revision 三阶段 | ✅ **抄三阶段打磨** | script-level +53.2%, scene-level +65.1% |
| R² (2025-03) | causal plot graph + sliding window 跨章 | ⚠️ **不用** | 输入是成品小说反推 graph；我们 Replay 已结构化不需要 |
| LongWriter-Zero 32B | RL 训练的超长生成 | ⚠️ **候选备用** | MVP 用 API 即可；生产阶段再考虑自训 |
| Genette 叙事话语五维 | 时序/时长/频率/聚焦/声音 | ✅ v2 已有，保留 | |

**最终组合**：Writer agent（HoLLMwood 风格）+ Editor agent（Dramaturge 三阶段）+ Genette 五维控制台。这个栈**基本不用自研**。

---

### 7. 场景驱动调度

**要解决的**：场景切换而非时间步长；支持跨幕任意时间跳跃

| 来源 | 方案 | 决策 | 理由 |
|---|---|---|---|
| **RELATE-Sim Scene Master** | 基于 Turning Point Theory 逐场景推进 | ⭐ **借鉴范式，不抄 schema** | 他们只做情侣关系；范式思路可借 |
| R² scene-by-scene | 基于因果图逐场景生成 | ❌ **不用** | 输入是成品小说 |
| Beyond Direct Generation | logline → outline → scene 层级分解 | ⚠️ **部分借鉴层级思路** | 但他们完全 top-down 无模拟 |

**结论**：**场景驱动核心要自研**。RELATE-Sim 给了范式灵感（戏剧理论指导 scene 位置），但具体 schema 要自己设计。

**自研要点**：
- 场景四元组：`(time_coord, space_coord, participants, trigger_hook)`
- 相对时间戳（"上次见他三年前"而非绝对模拟时间）
- 场景间任意时间跳跃支持
- 场景 entry/exit 的 agent 状态切换协议

---

### 8. 长篇节奏管理

**要解决的**：10 万字不散架

| 来源 | 方案 | 决策 | 理由 |
|---|---|---|---|
| Save the Cat 15 节拍 | 卷级骨架 | ✅ v2 已有，保留 | |
| Apocalypse World Fronts + Clocks | 时钟层 | ✅ v2 已有，保留 | |

**结论**：**landscape 未提供新点**，v2 已完整。

**核心风险**：StoryBox Fig 5 显示 14/30 天模拟反而掉分。我们瞄准 10 万字 ≈ 他们 8 倍字数。这是**还没人解决的开放问题**——也意味着 hierarchical summarization 要做到他们做不到的层数，**这是自研硬骨头**。

---

### 9. 中文网文适配

**landscape 状态**：**零覆盖**。StoryBox 20 个 story 全英文；所有 Drama Management 工作全英文；所有 evaluation prompt 全英文。

**结论**：**完全自研，无可借鉴**。

**自研路径候选**（尚未决定）：
- 网文头部作品做 few-shot 风格样本
- LongWriter-Zero 32B + 中文网文 LoRA 微调
- 人机协作校稿回注风格
- 专用 evaluation prompt（覆盖爽感、打脸、章末钩子等网文特有维度）

**这是我们潜在的最大护城河**——也可能是最大陷阱。

---

### 10. 评测 stack

**要解决的**：如何客观评估故事质量

| 来源 | 方案 | 决策 | 来源地址 |
|---|---|---|---|
| **Autodesk ASP benchmark** (2025-06) | causal soundness / character intentionality / dramatic conflict | ⭐⭐⭐ **直接用** | HuggingFace: ADSKAILab/LLM-narrative-planning-taskset |
| **StoryBox LLM-eval prompt** | 6 维 pairwise（Plot / Creativity / Character / Language / Conflict / Overall） | ⭐⭐⭐ **直接用** | arXiv 2510.11618 Table 2/3 |
| **ISS** (Identity Stable Set) | 角色行为一致性 0-10 分 | ⭐⭐⭐ **直接用** | StoryBox Table 3 prompt |
| SCORE 三维评测 | character / emotional / plot | ✅ **可叠加** | arXiv 2503.23512 |
| Dramaturge 双层胜率 | script-level / scene-level | ✅ **补充** | arXiv 2510.05188 |

**结论**：**评测完全不用自研**。现有工作叠起来就是一个极强的 benchmark stack。

---

## 自研清单（landscape 零参考的硬骨头）

按优先级排序：

### 1. GM agent 的"中立裁判"定位 ⭐⭐⭐
- 所有现有 director 都违反铁律
- 未被占领的强差异化点
- 风险低（不需要新算法，只需要新定位 + STORY2GAME precondition/effect 形式化）

### 2. 多世界线的 cherry-pick 心理兼容检查 ⭐⭐⭐
- WHAT-IF 只分不归
- 核心原创算法
- 风险中（心理状态建模 + 兼容性判定都要设计）

### 3. Replay 工具链（审计面板、diff、回放）⭐⭐
- StoryBox 把 Replay 藏在 pipeline 内部
- 工程量中
- 护城河强度：产品级差异化

### 4. 中文网文适配 ⭐⭐
- landscape 零覆盖
- 工程量大（语料 + 微调 + evaluation）
- 是最大护城河 **或** 最大陷阱

### 5. 场景驱动通用 scene schema ⭐
- RELATE-Sim 只做单一场景
- 理论清晰，工程中等
- 跨幕时间跳跃的记忆时间戳改造是关键

### 6. 10 万字级 hierarchical summarization ⭐
- StoryBox 30 天就撑不住
- 开放研究问题
- 可能要做 3-4 层摘要（StoryBox 只做 2 层）

---

## 与 v2 / v3 的关系

本文档**不是 v3**。它是 v3 的**决策基材**——把"借鉴什么"这件事一次性说清楚，避免 v3 写作时反复纠结。

v3 写作时：
- **§3 角色 agent** → 按组件 1 的最终组合写
- **§4 Drama Manager** → 按组件 3 的 trigger schema 写
- **§6 Replay 层** → 按组件 5 的抄 + 自研工具链写
- **§7 成文层** → 按组件 6 的栈写
- **§8 长篇节奏** → 按组件 8 的保留 + 自研摘要层级写
- **§新增 §2.5 时空建模范式** → 按组件 7 的场景四元组写
- **§新增 §附录 评测 stack** → 按组件 10 直接抄
- **§9 中文适配** → 按组件 9 保留"无可借鉴，自研"判断

---

## 附录 A：可直接抄的工程栈清单

```
persona_schema:
  source: StoryBox Table 4
  fields: [Name, Age, Innate, Learned, Currently, Lifestyle,
           Living_Area (5-level colon path), Daily_Plan_Requirement]

memory_mechanism:
  source: Park et al. 2023 (Generative Agents)
  components: [event_stream, reflection, retrieval]
  caveat: 短程社交场景验证，长程需自补对策

event_storage:
  source: StoryBox
  stack:
    db: sqlite3
    vector: FAISS
    embedding_model: jinaai/jina-embeddings-v3
    embedding_dim: 512
    context_cap: 102400 tokens

event_schema:
  source: StoryBox Fig 2
  fields: [event_id, start_time, end_time, duration,
           participants, location, description, detail]

trigger_schema:
  source: Drama Llama (arXiv:2501.09099)
  format: natural-language condition + effect
  types: [basic, ending]

writer_editor_loop:
  source: HoLLMwood (EMNLP 2024)
  agents: [writer, editor]
  feedback_prompt: paper §3.3

revision_pipeline:
  source: Plug-and-Play Dramaturge (arXiv:2510.05188)
  stages: [global_review, scene_review, hierarchical_coordinated_revision]

world_tree_structure:
  source: WHAT-IF (arXiv:2412.10582)
  structure: plot-tree
  extension_needed: [回收操作, cherry-pick 心理兼容检查]

state_tracking:
  source: SCORE (arXiv:2503.23512)
  dimensions: [character_consistency, emotional_coherence, plot_element]
  retrieval: TF-IDF + embedding 双通道

precondition_formalization:
  source: STORY2GAME (Riedl, arXiv:2505.03547)
  use: GM agent 裁定 action 合法性的基础

evaluation_stack:
  - source: Autodesk ASP benchmark (HF: ADSKAILab/LLM-narrative-planning-taskset)
    metrics: [causal_soundness, character_intentionality, dramatic_conflict]
  - source: StoryBox LLM-eval (arXiv 2510.11618 Table 2/3)
    metrics: [plot, creativity, character_development, language_use, conflict_quality, overall]
  - source: StoryBox ISS prompt (Table 3)
    metric: character_behavior_consistency (0-10)
  - source: SCORE
    metrics: [character, emotional, plot_element] 三维
  - source: Dramaturge
    metrics: [script_level, scene_level] 双层胜率
```

---

## 附录 B：架构假设错配的产品（明确不用）

- **Open-Theatre**：面向"玩家即时互动"（同步短窗高交互），我们要做"作者离线创作"（异步长窗低交互高可审计）。架构假设根本不同，fork 等于"拿 Unity 做 PDF 排版"。可以**读代码作参考**，但不作为基础
- **IBSEN / CoDi**：director 会 override 角色，违反我们铁律。反面借鉴
- **AI Dungeon / Character.AI**：单 prompt 架构，已被证明不可扩展
- **R²**：输入是成品小说反推 plot graph，不做模拟

---

## 核心判断

**可抄的占工程量约 60%——如果全抄，至少省 60% 工作。**

**真正自研的部分恰好就是我们的差异化——这不是巧合，是因为 landscape 没覆盖的地方本身就是机会。**

**最强未占领的差异化点**：GM agent 的"中立裁判"定位。所有现有系统的 director 都违反这个定位。

**最大护城河 or 陷阱**：中文网文适配。landscape 零覆盖。
