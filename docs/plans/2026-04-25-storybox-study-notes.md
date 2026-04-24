# StoryBox 深度研读笔记

**研读日期**：2026-04-25
**目的**：对照我们《模拟优先 AI 小说创作系统》设计，提取可抄细节与差异化锚点
**相关文件**：`2026-04-24-simulation-first-writing-system-design.md`（我们的设计 v2）

---

## 0. 论文元信息

- **标题**：*StoryBox: Collaborative Multi-Agent Simulation for Hybrid Bottom-Up Long-Form Story Generation Using Large Language Models*
- **作者**：Chen, Pan, Li
- **发表**：**AAAI 2026**（arXiv:2510.11618，v3 于 2026-03-19）
- **项目页**：https://storyboxproject.github.io（占位页，未发 demo/代码）
- **规模**：正文 8 页 + 附录 30 页（prompt 模板、评估、案例全文等）

---

## 1. 整体架构

**两阶段、两类 agent**（§Methodology, p3）：

```
┌── Stage 1: Sandbox Simulation (bottom-up) ─────────────┐
│  N 个 Character agents (只有这一种 agent)               │
│     ↓ 每 1 小时一个模拟步                                │
│  Events DB (sqlite3, 带 description + detail)           │
└────────────────────────────────────────────────────────┘
                          ↓
┌── Stage 2: Storyteller Agent (top-down 组织) ──────────┐
│  1. Event Summarization (daily by character → windowed) │
│  2. Story Information Generation (Title → metadata)     │
│  3. Iterative chapter-by-chapter writing                │
│     ├─ Retrieval: keyword + dense (FAISS, dim=512)      │
│     └─ Prev chapter summary appended to next            │
└────────────────────────────────────────────────────────┘
```

**关键**：**只有 character agent + 1 个 Storyteller**。**没有 GM、没有 Director、没有世界权威**。论文 §Related Work 明确与 IBSEN 的 Director-Actor 对比，理由是"top-down 会压制自然发展"。

---

## 2. 角色 agent 实现

### 2.1 Persona 结构（Table 4 原文）

```
Name, Age
Innate (3 形容词)
Learned (多句背景)
Currently (当前处境+动机)
Lifestyle (日常节奏)
Living Area (colon-separated 5 层路径)
Daily Plan Requirement (list of 4 bullets)
```

### 2.2 关键意外：**没用 Stanford Agents 的记忆流**

- 论文声称 "Inspired by Generative Agents... we adopt and modify it"
- 但**只借用了 Persona 结构**，**完全砍掉**了 memory stream / reflection / retrieval
- 角色"记忆"= **每天重生成 daily plan**（不累积、不反思）
- 没有 theory of mind、没有对其他角色意图的建模

### 2.3 他们怎么绕过长程一致性问题？

**答案不是"解决了"，而是"绕过了"**：
- Fig 5 Simulation Duration Study：**7 天是最佳值**
- **14 天和 30 天反而掉分**
- 原文（p7）：*"Excessive events in longer simulations may even burden the model's ability to synthesize information effectively"*

---

## 3. Storyteller Agent（成文层）

### 3.1 Event Summarization 两层结构

- **Layer 1**：按角色按日摘要（chronological）
- **Layer 2**：Dynamic window 将日摘要分组再聚合
- "**The size of the dynamic window is automatically determined by the LLM**"——**论文未披露具体参数**，对复现不利

### 3.2 Story Information Generation

自顶向下生成：`Title → Type → Background → Themes → Chapter Titles → Conflicts per Chapter → Plot Points per Chapter`

**重要**：所有这些都是**事后 top-down 塞进去的**，不是从模拟中涌现的。Plot Points 与 sandbox 模拟**无本质关联**。

### 3.3 迭代式写章

- 逐章增量写，不是一次性全文
- 每章"可能需要多次迭代"——**具体次数未披露**
- 每章写完生成摘要，附加到 history 给下章

### 3.4 哪些事件值得写？

原文（§Story Generation, p4）：
> *"the information retrieval process also acts as a dynamic filtering mechanism"*

**没有显式 McKee 式价值转换判断**，全靠 embedding 相似度 + chapter 主题/冲突/plot point 的引导信号**隐式筛选**。

---

## 4. Drama Manager 对等物（极简版）

**他们基本没有 Drama Manager**。

### 4.1 用户介入接口

- 仅限**初始化阶段**：可手动指定 premise、setting、characters、chapter 数、conflicts per chapter、plot points per chapter
- **模拟一旦开始，用户无法介入**
- 4 小时墙钟跑完就出来了

### 4.2 "戏剧张力注入"的唯一机制：abnormal_factor = 0.3

- 每个规划步 **30% 概率**让角色脱离日程
- Ablation（Fig 6）显示：**去掉这个随机扰动，Creativity / Character Development / Conflict Quality 三项降幅最大**
- 即"戏剧张力"本质上靠一个随机性钮在撑

### 4.3 对比我们的命运变量

- StoryBox 的 abnormal_factor = **无语义全局随机数**
- 我们的命运变量 = **有语义定向扰动**
- 消融实验反向证明"有目的扰动"就是正确方向——他们做了最简化版本，我们要做定向版本

---

## 5. 评估方法

### 5.1 数据集

- 20 个自建 story settings（Table 1）
- **全英文**，偏科幻/奇幻/悬疑
- **无中文、无修仙/网文题材**

### 5.2 人工评估

- 78 位评审（literature / CS / 其他）
- 6 维 Pairwise 比较：Plot / Creativity / Character Development / Language Use / Conflict Quality / Overall
- 自研 PickMan 中英双语平台
- Latin Square 设计

### 5.3 自动评估

- LLM 打分（prompt 见 Table 2, 3）
- 特殊指标：**Character Behavior Consistency** 基于 ISS (Identity Stable Set)，0-10 分
- Table 3 原 ISS prompt 可直接复用为我们的 benchmark

### 5.4 结果

- **StoryBox 6 项全部第一**（Fig 4, p5）
- 平均字数 **~12,000 词**
- IBSEN ~10,000 词；vanilla LLMs ~1,000 词

### 5.5 未被论文承认的局限

- Fig 4 Creativity 其实只微弱领先
- Table 7 Chapter 5 "Thawing Resilience" **结尾过于光明**，主题漂移明显，**论文未讨论**
- Fig 5 14/30 天掉分的解释软弱

---

## 6. 工程细节速查表

| 项 | 值 | 备注 |
|---|---|---|
| Sandbox LLM | `gpt-4o-mini` | temperature 0.8，60s 超时，最多 5 次重解析 |
| ISS 评估 LLM | `llama3.1:8b-instruct-fp16` | |
| 上下文 | **102,400 tokens** 硬帽 | 约 128k 的 80% |
| Embedding | `jinaai/jina-embeddings-v3` | **dim 512** |
| 向量索引 | FAISS | |
| 存储 | sqlite3 | 时间戳起点 `2024-09-01 12:00` |
| 模拟步长 | **1 小时/步** | |
| 默认模拟时长 | **7 天** | 14/30 天反而掉分 |
| Abnormal Factor | **0.3** | |
| 对话硬约束 | 每次 chat = **4 轮**（2 exchanges per agent） | 防 LLM 水话 |
| 环境模型 | 5 层 YAML 树：World/Region/Zone/Area/Object | |
| 角色数 | Story 1 有 6 人 | |
| 墙钟参考 | 6 人 × 7 天 sim + 成文 ≈ 4 小时 | 14 天约 7 小时 |

### 6.1 Event 记录结构（Fig 2 原文）

```
[Event ID] 5
[Start Time] 2024-09-01 9:00
[End Time]   2024-09-01 10:30
[Duration]   01:30:00
[Participants] John Smith
[Location]   Nexus Valley:Codespire Tower:Innovation Hub:Room 42
[Description] Starting coding on NLP models
[Detail]     <扩展：环境、时段、状态、情绪、动机>
```

### 6.2 他们没说的工程细节（复现时要自决）

- Retrieval 的 top-k
- Keyword + dense 两路融合策略
- Dynamic window 的窗口大小与层级数
- 每章迭代次数

---

## 7. 与我们设计的关键对比

| 维度 | StoryBox | 我们 | 依据 |
|---|---|---|---|
| 多 agent 模拟 | ✓ | ✓ | §MAS |
| GM / 世界权威 agent | ✗ | ✓ | Fig 2 |
| 模拟层/成文层分离 | ✓ | ✓ | §Methodology |
| Drama Manager | **极弱**（仅初始化+30% 随机） | ✓ | §Story Info Gen |
| 角色记忆流 | **✗**（每日重生成） | ✓ | 全篇无 memory/reflection |
| 角色 reflection | ✗ | ✓ | 同上 |
| 分层摘要 | ✓（2 层） | ✓ | §Event Summarization |
| 事件向量检索 | ✓（jina-v3, dim 512） | ✓ | More Impl p15 |
| 多世界线分叉 | ✗ | ✓ | 全篇无 |
| Replay 作为 IR | **部分**（sqlite3 events 已是 IR，但未显式宣称） | ✓（显式） | Impl p15 |
| 压力时钟 / 定向事件注入 | ✗ | ✓ | 仅全局 abnormal_factor |
| 骨架/节拍层 | ✗ | ✓（Save the Cat） | 只有 chapter-level conflicts |
| 长篇验证字数 | **~12k 词** | 目标 10 万+ | Fig 4 |
| 中文场景 | ✗ | ✓ | Table 1 |
| 伏笔追踪 | ✗ | 设计中 | 全篇无 |
| **时间建模** | **连续时间步长**（1 小时/步） | **场景驱动 + 时间跳跃** | 见 §10 |

---

## 8. 他们未解决、我们需要面对的问题

1. **多角色情绪/决策互相矛盾**：StoryBox 角色完全独立规划，**无 theory of mind**
2. **长期伏笔追踪**：论文通篇无伏笔概念；Plot Points 是 top-down 塞入，非涌现
3. **中文**：数据集 0 个中文；embedding 虽多语种，但 pipeline prompt 全英
4. **网文节奏/爽感**：他们追求"literary novel"式连贯性，不是每 3000 字爽点
5. **作者心理契约**：StoryBox 完全不让用户实时介入（放弃了我们作为工具的价值主张）
6. **时间建模**：见下文 §10——这是最根本的错配

---

## 9. 三个差异化点的重新评估

| 差异化点 | 状态 | 理由 |
|---|---|---|
| **Drama Manager** | ✅ **站得住且关键** | StoryBox 消融实验反向证明"有目的扰动"正确，他们只是做了最简化。我们定向版本明确更强 |
| **多世界线分叉** | ⚠️ **站得住，但要降级** | 1 条线 4 小时墙钟 → 10 条 40 小时。**定位为素材期工具，不是主流程** |
| **Replay 作为 IR** | ⚠️ **部分被占先** | StoryBox sqlite3 events 已是结构化 IR。我们要强化的是**工具链**（可视化/回放/diff/审计），不是 IR 本身 |

### 9.1 新发现的隐含差异

- **GM agent**：我们原构想的"GM agent 作为世界权威 / NPC / 环境裁定"是真正的新东西（StoryBox 环境是被动 YAML 树）
- **角色记忆流**：StoryBox 砍掉了 Park 2023 的 memory/reflection，我们保留。**如果我们能在 10w 字级别验证记忆流有效，是强差异化**——但这也是最大工程风险（§3.5）
- **时间建模**：见下节——这是最深的差异

---

## 10. 关键洞察：StoryBox 是生活模拟器，不是叙事机器

### 10.1 时间步长范式的真实起源

StoryBox 的"1 小时/步 × 7 天"范式直接继承自 **Stanford Generative Agents 的 Smallville**——25 个 agent 在小镇过日常生活的实验。那个实验的原初目标是**生活模拟**，故事只是副产品。

StoryBox 的真实定位是：
> **"让 agent 自由生活 7 天，事后整理出 1.2 万字"**

不是：
> **"用 agent 写一部长篇小说"**

### 10.2 为什么时间步长不适合长篇小说

小说的时间本质是**跳跃**的：

```
关键戏：一分钟对话写 2000 字
     ↓（跳 3 天）
日常戏：一笔带过 "三日后..."
     ↓（跳 3 年）
闪回戏：回忆十年前那个冬夜
     ↓（跳回当下）
高潮戏：十分钟决战写一整章
```

**连续时间步长无法建模这种节奏**。这解释了为什么：
- 14/30 天模拟反而掉分（生活模拟超过一定时长稀释戏剧密度）
- Plot Points 只能事后 top-down 塞（模拟中没有"情节"概念）
- 伏笔追踪零实现（生活模拟不需要）

### 10.3 我们的正确范式：场景驱动（scene-driven）

**❌ StoryBox 范式（时间驱动）**：
```
for hour in 1..168:
    for agent in agents:
        agent.act(hour)
```

**✅ 我们的范式（场景驱动）**：
```
while 卷未完结:
    scene = 下一幕规划  # 用户/Drama Manager 决定
        # "悬崖对峙" / "三年后重逢" / "童年闪回"
    for agent in scene.participants:
        agent.进入场景(scene.context)
    演绎(scene)          # 几分钟或几小时
    记录 Replay(scene)
    # 场景结束，时间可以跳任意长度
```

### 10.4 对设计的具体影响

1. **§3 模拟层**：基本单位是 **"幕（scene）"**，不是"小时步长"
2. **§8 长篇节奏**新增：**时间跳跃机制**（一幕到下一幕可跳任意长度）
3. **§4 Drama Manager** 职责扩充：**调度下一幕的时空坐标**（地点 + 时间 + 在场角色）是场景驱动的核心控制点
4. **角色记忆**时间戳改为**相对时间**（"上次见他是三年前在断崖"），不是"t=168:00"

### 10.5 这是比原以为更强的差异化

不只是"我们加了 Drama Manager"，而是**整个时间-空间建模范式根本不同**：

- **StoryBox = 生活模拟器**（连续时间 / agent 自主度过日常 / 事后整理）
- **我们 = 叙事机器**（离散场景 / 场景内涌现 / 作者节奏控制）

这两个范式**不可互相转换**。StoryBox 无法只改参数就变成长篇小说架构。

---

## 11. 对我们设计的 6 条具体调整建议

### 11.1 立即可做（不用写代码）

1. **角色卡 schema 对齐 StoryBox 字段**（Innate / Learned / Currently / Lifestyle / Daily Plan Requirement + 5 层 colon-path Living Area）作为基础字段。我们额外加的（关系网 -100~+100、super-objective、MBTI）作为**补充字段**而非替换，方便 diff baseline
2. **世界用 5 层树 YAML**（World/Region/Zone/Area/Object）。直接采用 Fig 7 + Table 5 schema
3. **事件存 sqlite3 + 向量化双索引**：dim 512 FAISS + keyword 两路。我们必须补齐 StoryBox 遮盖的 top-k 与融合策略——建议起手 top-k=20，rerank MMR 去冗余

### 11.2 架构决策

4. **Drama Manager 必须做成运行时可介入接口**，不只是初始化参数。Fronts/Clocks 应以 event 形式注入 sandbox（让角色 agent 感知"地震发生了"/"倒计时推进了"），这是甩开 StoryBox 的关键产品差异
5. **采用场景驱动（scene-driven）架构**，拒绝时间步长范式。这是比 Drama Manager 更深的差异化点——**整个时间-空间建模不同**

### 11.3 验证路径

6. **10w 字验证之前先做 1.2w 字的 StoryBox parity 实验**：拿他们 Story 1 "Frozen City" premise，用我们的架构跑一遍，用他们 Table 2/3 的 LLM-eval prompt 打分对比。parity 拿不到，不要急着上 10w 字

---

## 12. 关键风险数据点（必入风险登记册）

- **Fig 5 Simulation Duration Study 曲线**：StoryBox 在 **14 天/30 天性能下降**
- 原文归因：*"Excessive events in longer simulations may even burden the model's ability to synthesize information effectively"*
- **对我们的含义**：我们瞄准 10w 字 ≈ 他们 8 倍字数
  - 要么沙盒"时长"压得更短（靠戏剧密度而非时长）→ 这已经由我们的场景驱动范式天然解决
  - 要么 summarization 要做到他们做不到的层数
- **这是我们 §3.5 长程记忆风险登记的最硬数据点**

---

## 13. 小结

**StoryBox 对我们的价值**：
- ✅ **工程 schema 可大量直接抄**（角色卡、环境 YAML、事件结构、embedding 选型、评估 prompt）
- ✅ **benchmark 提供了**（Frozen City premise + Table 2/3 LLM-eval = 现成 parity baseline）
- ✅ **反向验证我们的差异化**（他们的 Drama Manager 极弱 = 我们方向对；他们时间步长掉分 = 我们场景驱动对）
- ⚠️ **警告了长程记忆风险**（30 天就撑不住 → 我们 10w 字是硬挑战）

**StoryBox 不能给我们的**：
- 长篇节奏（他们根本没尝试）
- 多世界线（完全没做）
- 中文 / 网文适配（零覆盖）
- 运行时用户介入（架构不支持）
- 伏笔追踪（架构不支持）
- 场景驱动时间建模（范式错配）

**一句话总结**：StoryBox 是目前最接近的公开工作，工程细节可大量借鉴，但它在架构深度上**不是我们的竞品，是我们的起点**。
