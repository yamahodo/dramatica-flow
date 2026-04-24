# 文档目录 / Onboarding Index

**受众**：新加入的 AI 模型 / 协作者 / 自己几周后再来看这个项目的你
**目的**：让你在 5 分钟内知道"读什么、为什么读、按什么顺序读"
**最新更新**：2026-04-25（v3 设计文档完成）

---

## 项目一句话

我们在设计一个 **AI 长篇小说创作系统**，核心思想是把"**剧情演绎层**"（多 agent 世界模拟）和"**撰写成文层**"（把演绎整理成文字）彻底分离。这是对 `dramatica-flow` 原项目的架构级重构构想。

---

## 所有文档一览

| 文件 | 定位 | 状态 | 篇幅 |
|---|---|---|---|
| `2026-04-24-simulation-first-writing-system-design.md` | **v3 架构设计文档**（主 spec） | 🟢 **ground truth** | 长 |
| `2026-04-25-component-borrowing-decisions.md` | 组件借鉴决策（v3 的决策过程记录） | 🟡 历史归档 | 中 |
| `2026-04-25-storybox-study-notes.md` | StoryBox 论文深度研读笔记（支撑材料） | 🟢 active | 中 |
| `2026-04-25-research-landscape.md` | 6 领域相关工作调研（支撑材料） | 🟢 active | 长 |

### 🔥 冲突时以哪个为准

**按优先级**（上层覆盖下层）：

1. **v3 设计文档** (`2026-04-24-simulation-first-writing-system-design.md`) —— **主 spec，最新 ground truth**
2. `component-borrowing-decisions.md` —— v3 的决策依据，**已被 v3 完整吸收**
3. `storybox-study-notes.md` + `research-landscape.md` —— 支撑材料

> **注**：文件名带 `2026-04-24` 但内容已是 **v3**（2026-04-25 修订）。文件名保持初版日期作为项目识别锚点，内容版本见文档内 frontmatter 和 §18 修订日志。

---

## 按场景选读（最短路径）

### 🆕 你是新加入的模型，完全不了解项目
**读一份就够**：
- **v3 设计文档**（`2026-04-24-simulation-first-writing-system-design.md`）

关键章节快览：
- §0 背景与动机 + §0.5 参考同路人 + §1 System Invariants → 了解定位
- §2 整体架构 + §2.5 时空建模范式 → 理解系统运行方式
- §3-8 各组件 → 按需查阅
- §13 评测 stack + §14 参考实现清单 → 知道哪些不用自研
- §17 下一步建议 → 知道当前 focus

### 🛠 你要继续修订设计（v3 → v4）
**按顺序读**：
1. v3 设计文档（当前 spec）
2. §18 修订日志（看 v1→v2→v3 怎么演化的）
3. 如果需要理解某个决策的**形成过程**：
   - `component-borrowing-decisions.md`（决策依据归档）
   - `storybox-study-notes.md` / `research-landscape.md`（调研原始材料）

### 🧪 你要设计或跑 MVP 实验
**按顺序读**：
1. v3 §13（评测 stack）+ §14（可直接抄的工程栈）+ §17（5 条可证伪假设）
2. `storybox-study-notes.md` §6 工程细节速查表 + §11.3 验证路径
3. StoryBox 论文原文（跑 Frozen City parity baseline）

### 📚 你要深入某个学术领域
**路径**：
1. `research-landscape.md` 找到对应领域（6 个章节分类）
2. 对应论文 arXiv 链接直接跳转

### 🏗 你要实现某个具体组件
**路径**：
1. v3 对应章节（§3 角色、§4 DM、§5 多世界线、§6 Replay、§7 成文、§8 长篇节奏）
2. v3 §14 参考实现清单 YAML
3. 若需更深工程细节：`component-borrowing-decisions.md` 对应组件 + `storybox-study-notes.md`

---

## 文档详述

### 1. `2026-04-24-simulation-first-writing-system-design.md`（v3 主 spec）🟢

**是什么**：整合 brainstorming（v1）+ red-team review（v2）+ StoryBox 研读 + landscape 调研 + 组件借鉴决策后的**当前主架构 spec**。18 章。

**读它是为了**：
- 理解系统整体架构和每个组件的设计决策
- 拿到可直接抄的工程栈（§14）
- 知道哪些是未验证断言（标注 ⚠）、哪些有验证动作+死线
- 知道下一步要做什么（§17）

**v3 相对 v1/v2 的关键演进**：
- ✅ 新增 §1 System Invariants（取代哲学化"核心哲学"，改为可验证约束）
- ✅ 新增 §2.5 时空建模范式（场景驱动 vs 时间步长）
- ✅ 新增 §13 评测 stack + §14 参考实现清单 + §15 Open-Theatre 可读参考点
- ✅ 升级 §3 角色 agent（StoryBox schema）/ §4 DM（Drama Llama trigger）/ §5 多世界线（WHAT-IF plot-tree）/ §6 Replay IR（StoryBox 存储栈）/ §7 成文层（HoLLMwood + Dramaturge）
- ✅ §0.5 同路人表更新（+ RELATE-Sim / Drama Llama / Open-Theatre / Triangle Framework 等）
- ✅ §9/§10/§11 保留并加**验证动作+死线**
- ❌ **彻底放弃 "Replay 可独立销售" 主张**
- ❌ Dramatica 与 Save the Cat 并列已合并为只用 Save the Cat

**关键章节**：
- §0.5 参考同路人（避免闭门造车）
- §1 System Invariants（6 条可验证约束）
- §2.5 时空建模范式（最深的差异化之一）
- §8.2 骨架-涌现冲突解决协议
- §14 参考实现清单 YAML
- §17 下一步 + MVP 5 假设

---

### 2. `2026-04-25-component-borrowing-decisions.md`（决策过程归档）🟡

**是什么**：v3 写作前的**决策依据文档**，对 10 个核心组件逐一判定"可抄 / 部分改造 / 必须自研"。

**状态**：**内容已被 v3 完整吸收**。保留作为**决策过程记录**——想知道"为什么 v3 决定 X"时查这里能看到对比表和理由。

**读它是为了**：
- 理解某个 v3 决策的**来龙去脉**
- 看到每个组件的"可借鉴选项对比表"
- 附录 B 列出了明确不用的产品及理由（Open-Theatre 错配等）

---

### 3. `2026-04-25-storybox-study-notes.md`（StoryBox 研读）🟢

**是什么**：对 StoryBox (AAAI 2026, arXiv:2510.11618) 38 页论文的深读笔记。StoryBox 是目前公开发表的**最接近我们架构**的学术工作。

**读它是为了**：
- 理解为什么"时间步长范式"不适合长篇（§10）—— 这是 v3 §2.5 的理论基础
- 拿到可直接抄的**工程参数表**（§6 工程细节速查表）
- 了解 StoryBox 的**真实定位**（生活模拟器，不是叙事机器）
- 看到他们**砍掉了 Stanford Agents 记忆流**这个意外事实（§2.2）

**最硬的数据点**：StoryBox Fig 5 显示 14/30 天模拟**反而掉分**。这是我们 10 万字目标（他们 8 倍字数）的最硬风险警示。

---

### 4. `2026-04-25-research-landscape.md`（6 领域调研）🟢

**是什么**：对 2024-2026 年 AI 故事生成及相关领域的广度扫描。6 个领域：Multi-Agent Story Generation / Narrative Planning / Drama Management / Computational Narratology / Interactive Storytelling / Scene-Based Generation。

**读它是为了**：
- 避免只盯 StoryBox 做井底优化
- 找到**每个子问题是否已有成熟方案**
- 找到**开放问题**（我们的机会）
- 深入某个具体论文（有 arXiv 链接索引）

**重大发现（已全部纳入 v3）**：
- Open-Theatre（架构错配，只读代码不 fork）
- Drama Llama（Façade 继任，trigger schema 可抄）
- RELATE-Sim（场景驱动部分已做）
- Triangle Framework（学术定位）

---

## 文档关系图

```
┌─────────────────────────────────────────┐
│  v3 设计文档 (current spec) 🟢           │  ← 主 spec
│  整合所有调研和决策                       │
│  📄 2026-04-24-simulation-first-...md    │
└─────────────────────────────────────────┘
        ↑ 决策过程归档
┌─────────────────────────────────┐
│ Component Borrowing Decisions 🟡 │  ← 已被 v3 吸收
└─────────────────────────────────┘
        ↑ 支撑材料
┌──────────────────────┐  ┌─────────────────────┐
│ StoryBox 研读笔记 🟢 │  │ Landscape 调研 🟢   │
└──────────────────────┘  └─────────────────────┘
        ↑ 调研触发
┌─────────────────────────────────┐
│  v2/v1 (已被 v3 覆盖)           │  ← 历史版本见 §18 修订日志
└─────────────────────────────────┘
```

---

## 关键术语词典

| 术语 | 含义 | v3 定义位置 |
|---|---|---|
| **模拟层** / **Sandbox** | 多 agent 世界模拟，产生事件流 | §2 |
| **成文层** / **Novel 层** | 把事件流翻译成小说文字 | §7 |
| **Replay 层** / **IR** | 模拟和成文之间的结构化中间表示 | §6 |
| **GM agent** | 世界规则裁定 + NPC 非戏剧调度 + 环境播报（**不推动情节**） | §1 #2 |
| **Drama Manager (DM)** | 导演接口层，通过命运变量 + 压力时钟 + Front 推动情节 | §4 |
| **命运变量** / **Fate Variable** | DM 注入的外部事件（不 override 角色） | §4.1 |
| **压力时钟** / **Countdown Clock** | 倒计时情节推进器 | §4.1 |
| **Front** | 威胁前线集群 | §4.1 |
| **Trigger** | 自然语言 IF-THEN 规则（抄 Drama Llama） | §4.2 |
| **场景驱动** / **Scene-Driven** | 以场景（scene）为基本单位 | §2.5 |
| **场景四元组** | (time_coord, space_coord, participants, trigger_hook) | §2.5.2 |
| **世界树** / **Plot-Tree** | 多世界线分叉数据结构 | §5.2 |
| **Cherry-pick 心理兼容检查** | 跨分支摘取时的角色状态兼容判定 | §5.4 |
| **System Invariants** | 架构级可验证约束（取代"核心哲学"） | §1 |
| **Parity 实验** | 用 StoryBox Frozen City premise 做对照基线 | §17 |

---

## 重要警示

1. **不要 fork Open-Theatre**——架构假设错配，读代码可以，fork 会被绑架（见 v3 §15）
2. **不要重建 evaluation**——Autodesk + StoryBox + ISS + SCORE + Dramaturge 叠起来就够强（见 v3 §13）
3. **中文适配 landscape 零覆盖**——要么最大护城河要么最大陷阱，不要假设西方理论栈自动迁移（见 v3 §10）
4. **StoryBox 的 Stanford Agents 记忆流其实被砍了**——我们保留是差异化，但是开放风险（见 v3 §3.5）
5. **10 万字级别无人验证**——所有现存工作都在 1 万字量级及以下（见 v3 §8.5）

---

## 当前焦点（2026-04-25）

**v3 完成，进入 MVP 实验准备阶段。**

下一步是 **MVP 实验（2 周）**——用 Frozen City parity 拿数据验证 v3 的 5 条可证伪假设。实验细节将产出 `2026-04-26-mvp-experiment-spec.md`（待写）。

实验完成后：
- 若 parity 达标 → 进入 10 万字 + 中文 阶段（v3.1 规划）
- 若 parity 失败 → post-mortem，v3 需要哪里重写由数据决定
