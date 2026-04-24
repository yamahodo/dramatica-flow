# 文档目录 / Onboarding Index

**受众**：新加入的 AI 模型 / 协作者 / 自己几周后再来看这个项目的你
**目的**：让你在 5 分钟内知道"读什么、为什么读、按什么顺序读"

---

## 项目一句话

我们在设计一个 **AI 长篇小说创作系统**，核心思想是把"**剧情演绎层**"（多 agent 世界模拟）和"**撰写成文层**"（把演绎整理成文字）彻底分离。这是对 `dramatica-flow` 原项目的架构级重构构想。

---

## 所有文档一览

| 文件 | 定位 | 状态 | 篇幅 |
|---|---|---|---|
| `2026-04-24-simulation-first-writing-system-design.md` | **v2 架构设计文档**（骨架） | 🟡 **部分过时** | 长 |
| `2026-04-25-storybox-study-notes.md` | StoryBox 论文深度研读笔记 | 🟢 active | 中 |
| `2026-04-25-research-landscape.md` | 6 领域相关工作调研 | 🟢 active | 长 |
| `2026-04-25-component-borrowing-decisions.md` | **最新组件借鉴决策** | 🟢 **ground truth** | 中 |

### 🔥 冲突时以哪个为准

**按优先级**（上层覆盖下层）：

1. `component-borrowing-decisions.md`（2026-04-25）—— **最新决策，优先于 v2**
2. `storybox-study-notes.md` + `research-landscape.md`（2026-04-25 调研事实）
3. `simulation-first-writing-system-design.md` v2（2026-04-24 架构骨架，部分已被修订）

**v2 里已过时的主张**（被 borrowing-decisions 修正）：
- ❌ "Replay 可作为独立畅销成品" —— 改为"中间表示 IR + 作者审计工具链"
- ❌ "Dramatica 和 Save the Cat 并列用" —— 改为"只用 Save the Cat"
- ❌ "Stanford Generative Agents 解决了角色内核" —— 降级为"短程社交场景验证，长程是开放风险"
- ❌ "系统永不 override 角色决策" 哲学化表述 —— 改为"不直接重写台词/内心"的可操作约束
- ❌ 时间步长范式（隐含） —— 明确为场景驱动 + 跨幕时间跳跃

---

## 按场景选读（最短路径）

### 🆕 你是新加入的模型，完全不了解项目
**读这两份就够**（30 分钟）：
1. `component-borrowing-decisions.md` 的 "TL;DR 总清单" + "自研清单" 两节
2. v2 设计文档的 §0 背景 + §2 整体架构图 + §3-8 快览章节名

其他文档**按需查**。

### 🛠 你要基于现有讨论写 v3 设计文档
**按顺序读**：
1. v2 设计文档（作为骨架起点）
2. `component-borrowing-decisions.md`（**用它覆盖 v2 中过时部分**）
3. `storybox-study-notes.md`（补具体参数、schema、工程细节）
4. `research-landscape.md` §综合判断（确认差异化仍站得住）

### 🧪 你要设计 MVP 实验
**按顺序读**：
1. `component-borrowing-decisions.md` 附录 A（可抄的工程栈）+ 组件 10（评测 stack）
2. `storybox-study-notes.md` §6 工程细节速查表 + §11.3 验证路径
3. 然后去读 StoryBox 论文原文跑 Frozen City parity baseline

### 📚 你要深入某个学术领域
**路径**：
1. `research-landscape.md` 找到对应领域（6 个章节分类）
2. 对应论文 arXiv 链接直接跳转

---

## 文档详述

### 1. `2026-04-24-simulation-first-writing-system-design.md`（v2 设计文档）

**是什么**：经过 brainstorming（7 段对话式澄清）+ 一轮独立 red-team review 产出的架构骨架文档。包含 14 章，从背景动机到理论启发到尚待决定的问题。

**读它是为了**：
- 理解系统的整体架构图（§2）
- 理解每个核心组件的**设计意图**（§3-8）
- 看到完整的理论启发链（§12 理论来源索引）

**注意**：
- 哲学化章节（§9 作者心理契约、§11 IP 归属）是**产品战略而非工程 spec**
- v2 的部分主张已被 2026-04-25 的 borrowing-decisions **修订**——读 v2 时心里要挂一份"已过时主张"清单（见上节）

**关键章节**：
- §0.5 参考同路人对照表
- §3.4 两条可操作铁律（不 override、信息边界）
- §5.4 摘取（cherry-pick）的边界
- §8.2 骨架-涌现冲突解决协议

---

### 2. `2026-04-25-storybox-study-notes.md`（StoryBox 研读笔记）

**是什么**：对 StoryBox (AAAI 2026, arXiv:2510.11618) 的 38 页论文深读笔记。StoryBox 是目前**公开发表的最接近我们架构**的学术工作。

**读它是为了**：
- 理解为什么"时间步长范式"不适合长篇（§10）
- 拿到可直接抄的**工程参数表**（§6 工程细节速查表）
- 了解 StoryBox 的**真实定位**（他们是生活模拟器，不是叙事机器——§10）
- 看到他们**砍掉了 Stanford Agents 的记忆流**这个意外事实（§2.2）

**最硬的数据点**：StoryBox Fig 5 显示 14/30 天模拟**反而掉分**。这是我们 10 万字目标（他们 8 倍字数）的最硬风险警示。

**关键章节**：
- §6 工程细节速查表（embedding、存储、参数）
- §10 StoryBox 是生活模拟器不是叙事机器（核心洞察）
- §11 对我们设计的 6 条调整建议
- §12 关键风险数据点

---

### 3. `2026-04-25-research-landscape.md`（6 领域调研）

**是什么**：对 **2024-2026 年** AI 故事生成及相关领域的广度扫描。分 6 个领域：Multi-Agent Story Generation / Narrative Planning / Drama Management / Computational Narratology / Interactive Storytelling / Scene-Based Generation。

**读它是为了**：
- 避免只盯 StoryBox 做井底优化
- 找到**每个子问题是否已有成熟方案**（见"已有成熟方法可直接抄"表）
- 找到**开放问题**（我们的机会）
- 确认**差异化是否仍站得住**

**重大发现**：
- ⭐ **Open-Theatre** (EMNLP Demos 2025, GitHub 开源) —— 原以为可以 fork，但架构假设错配（玩家即时互动 vs 作者离线创作），不 fork 只读
- ⭐ **Drama Llama** (Mateas 组 2025-01) —— Façade 的直接学术继任，trigger schema 可抄
- ⭐ **RELATE-Sim** (2025-10) —— 已做场景驱动，相似度 60%
- ⭐ **Triangle Framework** (AIIDE 2025) —— 唯一对我们架构做清晰学术定位的工作

**关键章节**：
- §综合判断（开放问题 + 必读优先级）
- 6 个领域的 "可借鉴点 + 局限" 表格
- 必读优先级 Top 5

---

### 4. `2026-04-25-component-borrowing-decisions.md`（组件借鉴决策）🟢 **ground truth**

**是什么**：对蓝图 10 个核心组件逐一判定"可借鉴 / 部分改造 / 必须自研"的**设计决策文档**。基于 StoryBox 研读 + landscape 调研综合。

**读它是为了**：
- **立即知道每个组件应该怎么实现**
- 看到"**可直接抄**"和"**必须自研**"两个清单
- 拿到附录 A 的 **YAML 格式工程栈清单**（可直接当 copy-paste 基材）
- 理解与 v2 的关系（覆盖哪些、保留哪些）

**核心判断**：
- 可抄占工程量 ~60%
- 必须自研 6 项（按优先级）：GM agent / cherry-pick 心理兼容 / Replay 工具链 / 中文适配 / 场景 schema / 10 万字摘要层级
- 最强未占领差异化：GM agent 的"中立裁判"定位

**如果你只读一份文档，读这个。**

---

## 文档关系图

```
┌─────────────────────────────────┐
│  v2 设计文档 (2026-04-24)       │
│  架构骨架、哲学、14 章           │ ← 起点
│  部分主张已过时                  │
└─────────────────────────────────┘
         ↓ 触发研究
┌──────────────────────┐  ┌─────────────────────┐
│ StoryBox 研读笔记    │  │ Landscape 调研      │
│ (2026-04-25)         │  │ (2026-04-25)        │
│ 最接近工作的深读     │  │ 6 领域广度扫盘      │
└──────────────────────┘  └─────────────────────┘
         ↓                         ↓
         └────────────┬────────────┘
                      ↓
┌─────────────────────────────────────┐
│  Component Borrowing Decisions      │
│  (2026-04-25) 🟢 GROUND TRUTH       │
│  10 个组件的设计决策                 │
│  覆盖 v2 过时主张                    │
└─────────────────────────────────────┘
                      ↓ 未来
              [v3 设计文档]
              （尚未写，基于 borrowing 决策 + v2 骨架）
```

---

## 关键术语词典

| 术语 | 含义 | 首次定义 |
|---|---|---|
| **模拟层** | 多 agent 世界模拟，产生事件流 | v2 §2 |
| **成文层** | 把事件流翻译成小说文字 | v2 §7 |
| **Replay 层** | 模拟和成文之间的结构化中间表示 | v2 §6 |
| **GM agent** | 世界规则裁定 + NPC 调度 + 环境播报（不推动情节） | borrowing §2 |
| **Drama Manager (DM)** | 导演接口层，通过命运变量 + 时钟 + Front 推动情节 | v2 §4 |
| **命运变量** | DM 注入到世界的外部事件（不 override 角色决策） | v2 §5 |
| **压力时钟 / Countdown Clock** | 倒计时情节推进器 | v2 §4，Apocalypse World |
| **Front** | 威胁前线集群 | Apocalypse World |
| **场景驱动** | 以场景（scene）为基本单位，对立于连续时间步长 | storybox §10 |
| **世界树** | 多世界线的分叉/cherry-pick 数据结构 | v2 §5 |
| **Parity 实验** | 拿 StoryBox Frozen City premise 做对照基线 | borrowing §10 |

---

## 重要警示

1. **不要 fork Open-Theatre**——架构假设错配，读代码可以，fork 会被绑架
2. **不要重建 evaluation**——Autodesk + StoryBox + ISS + SCORE + Dramaturge 叠起来就够强
3. **中文适配 landscape 零覆盖**——要么最大护城河要么最大陷阱，不要假设西方理论栈能自动迁移
4. **StoryBox 的 Stanford Agents 记忆流其实被砍了**——我们保留是差异化，但是开放风险
5. **10 万字级别无人验证**——所有现存工作都在 1 万字量级及以下

---

## 下一步待决

v3 设计文档尚未写。主要选项：
- **立即写 v3**（基于 borrowing-decisions 覆盖 v2）
- **先做 MVP 实验**（用 StoryBox Frozen City 作 parity），拿数据后再写 v3

两条路都有支持。
