# 模拟优先的 AI 小说创作系统：架构设计

**初版**：2026-04-24（v1，brainstorming 产出）
**修订**：2026-04-25（v2，含一轮独立 red-team review 修订）
**最新**：2026-04-25（**v3**，整合 StoryBox 研读 + 6 领域 landscape 调研 + 组件借鉴决策）
**状态**：可执行架构 spec（含未验证断言，标注验证动作+死线）
**定位**：修正主流的前沿探索

---

## 0. 背景与动机

Dramatica-Flow 用 Dramatica 叙事理论驱动 AI 长篇写作。本构想更激进：**把"剧情演绎"与"撰写成文"彻底分层**——先让多 agent 模拟出故事，再按需提炼成文字。

**反面教训**：Storytron、AI Dungeon 因"模拟与讲述混在一层"失败；Narratology vs Ludology 学术共识是两种逻辑不可互相归约，需要分层。

**正面参照**：日本 Group SNE"跑团→Replay→Novel"产业模式 38 年验证分层可行；StoryBox (AAAI 2026) 学术级验证 12k 字可跑通。

---

## 0.5 参考同路人（必读）

**本架构不是凭空发明。下表是差异化锚点与借鉴点，不是竞品列表。**

| 项目 | 年份 | 相关性 | 与本架构的关系 |
|---|---|---|---|
| **StoryBox** (AAAI 2026, arXiv:2510.11618) | 2025-10 | ⭐⭐⭐ | 最接近。砍掉了 Stanford Agents 记忆流，**抄工程 schema + 存储栈** |
| **Drama Llama** (Mateas 组, arXiv:2501.09099) | 2025-01 | ⭐⭐⭐ | Façade 直接继任，**抄 natural-language trigger schema** |
| **RELATE-Sim** (arXiv:2510.00414) | 2025-10 | ⭐⭐⭐ | 场景驱动部分已做（单一关系场景），借思路 |
| **Open-Theatre** (EMNLP Demos 2025) | 2025-09 | ⭐⭐ | **架构假设错配**（玩家即时互动 vs 作者离线创作），不 fork 只读参考 |
| **Triangle Framework** (AIIDE 2025) | 2025 | ⭐⭐ | 把我们定位在 "landmark-guided hybrid" 三角中点的学术锚点 |
| **HoLLMwood** (EMNLP 2024) | 2024-06 | ⭐⭐ | **抄 Writer+Editor 双 agent 分工** |
| **Plug-and-Play Dramaturge** (arXiv:2510.05188) | 2025-10 | ⭐⭐ | **抄三阶段稿件打磨** |
| **STORY2GAME** (Riedl, arXiv:2505.03547) | 2025-05 | ⭐ | GM 的 precondition/effect 裁定形式化 |
| **WHAT-IF** (arXiv:2412.10582) | 2024-12 | ⭐ | 世界树 plot-tree 数据结构 |
| **SCORE** (arXiv:2503.23512) | 2025-03 | ⭐ | State Tracking 三维，Replay 元信息层 |
| **IBSEN** (ACL 2024) | 2024 | ❌ | **反面案例**：Director 会 override 角色，违反铁律 |
| **AI Dungeon / Character.AI** | — | ❌ | 反面案例，单 prompt 混层失败 |

**本架构相对同路人的差异化（精确表述，不再泛泛）**：

1. **GM agent 作为中立裁判**——所有现有系统的 director 都 override 角色（IBSEN/CoDi），或没有 GM（StoryBox 环境是被动 YAML）。**这是最强未占领的点**
2. **多世界线回收 + cherry-pick 心理兼容检查**——WHAT-IF 只分不归，无人做兼容检查
3. **"命运变量 + 压力时钟 + Front" 三位一体 Drama Manager 配置**——单项都有人做，组合独特
4. **场景驱动 + 跨幕任意时间跳跃 + 10 万字**——RELATE-Sim 是单一关系场景，未做长篇
5. **中文网文适配**——landscape **零覆盖**
6. **Replay 作为可审计工具链**（不是独立成品）——StoryBox 事件表是隐式 IR，我们暴露给作者

**成熟度**：
- 1 万字连贯长文 2025-10 由 StoryBox 首次证明
- **10 万字级别尚无任何公开系统证明可行**
- 商业侧**零款**工具采用"模拟+成文分层"架构

---

## 1. System Invariants（架构不变量）

> **v3 修订**：v2 的"核心哲学"表述被替换。Invariant 是可验证约束，不是哲学宣言。

| # | Invariant | 可验证形式 |
|---|---|---|
| 1 | **分层不混** | 一个 agent 不同时承担"演"和"写"两职。违反 = 架构 bug |
| 2 | **GM 中立** | GM agent 不推动情节——只做世界规则裁定 / NPC 非戏剧调度 / 环境事件播报。违反 = 变成 IBSEN |
| 3 | **不直接重写** | 作者/DM/系统不直接修改角色 agent 的台词或内心输出。所有干预走外部情境层 |
| 4 | **信息边界** | 角色 agent 只能访问亲历/听说/推理得出的信息。全知污染 = bug |
| 5 | **涌现优先** | 涌现出的"超预设走向"默认保留，除非违反世界规则或 System Invariants |
| 6 | **间接操纵是诚实的** | 改情境 = 改 prompt = 间接影响角色决策。这是**有意为之**的干预通道，不是"完全自由涌现"，也不是"涌现圣洁" |

> **启发来源**：fabula/sjužet（Genette）；Vincent Baker Apocalypse World 的 MC 哲学（"be a fan of the characters"）。水野良 2024 年 CBR 采访"预设世界观硬塞会压垮即兴"。

---

## 2. 整体架构

```
┌─ 静态层：世界规则 / 角色卡 / 初始设定（人工维护）
│
├─ 模拟层（Sandbox）
│   ├─ 1 个 GM agent（中立裁判）
│   ├─ N 个角色 agent（独立、自主、有记忆）
│   ├─ 环境 YAML 树（5 层 World/Region/Zone/Area/Object）
│   ↑ 导演（用户）通过 Drama Manager 介入
│
├─ Replay 层（中间表示 IR）
│   ├─ Event sqlite3 + FAISS 向量索引
│   ├─ 对话 + 旁白 + 元信息
│   └─ 作者审计面板（可视化 / diff / 回放）
│
└─ Novel 层（文学成品）
    ├─ Writer agent（Replay → 散文）
    └─ Editor agent（三阶段打磨）
```

**三层产物都是 first-class**：Replay 不是废料，是可审计的作者工具。但 **Replay 作为独立销售品的主张已放弃**（见 §6）。

---

## 2.5 时空建模范式（v3 新增章节）

### 2.5.1 核心判断

**StoryBox 的时间步长范式（1 小时/步 × 7 天）不适合长篇小说。**

StoryBox Fig 5 显示 14/30 天模拟反而掉分——他们的范式在生活模拟层面有效，但小说的时间本质是**跳跃**的：

```
关键戏：1 分钟对话写 2000 字
    ↓（跳 3 天）
日常戏：一笔带过 "三日后..."
    ↓（跳 3 年）
闪回戏：回忆十年前那个冬夜
```

### 2.5.2 我们的范式：场景驱动 (Scene-Driven)

**基本单位是"幕（scene）"，不是"小时步长"**。

**场景四元组**：
```
Scene = (
  time_coord:    # 绝对时间或相对时间戳
  space_coord:   # 5 层 colon-path 地点
  participants:  # 在场角色列表
  trigger_hook:  # 触发该幕的命运变量 或 "自然衔接"
)
```

**场景循环伪代码**：
```python
while 卷未完结:
    scene = next_scene(...)  # 用户/DM 决定下一幕时空
    for agent in scene.participants:
        agent.enter(scene.context)
    play(scene)              # 持续几分钟 或 几小时
    record_replay(scene)
    # 场景结束，时间可跳任意长度
```

### 2.5.3 对其他组件的影响

- 角色记忆时间戳从**绝对时间**（t=168:00）改为**相对时间**（"上次见他三年前"）
- DM 的核心操作界面变为 **"调度下一幕的时空坐标"**
- GM agent 负责"跨时间跳跃后的世界状态一致性"

### 2.5.4 未验证断言 ⚠

场景驱动 > 时间步长——本断言**尚无我方实测数据**。**验证动作**：MVP H2 实验对照（见 §17）。

> **启发来源**：RELATE-Sim Scene Master（基于 Turning Point Theory），但 RELATE-Sim 仅做情侣关系场景，我们扩展到通用叙事长篇。

---

## 3. 模拟层：角色 agent 内核

### 3.1 角色卡（Persona Schema）

**基础字段**（抄 StoryBox Table 4 schema，经过 78 人评估 + ISS 验证）：
- `Name`
- `Age`
- `Innate`（3 形容词）
- `Learned`（多句背景）
- `Currently`（当前处境 + 动机）
- `Lifestyle`（日常节奏）
- `Living_Area`（5 层 colon-separated 路径）
- `Daily_Plan_Requirement`（4 条 bullet）

**本架构补充字段**（我们的差异化）：
- `super_objective`：人生渴望（Stanislavski）
- `relationship_network`：对每个主要角色的关系强度 -100~+100
- `known_info`：信息边界记录（他知道什么、在何时何地以何方式知道的）

### 3.2 记忆流（Memory Mechanism）

**借鉴 Park et al. 2023（Generative Agents）的三件套**：
- **事件流** (Event Stream)：按时间存储亲历 / 听说 / 推理事件
- **反思** (Reflection)：每 N 幕从散乱事件提炼"领悟"
- **检索** (Retrieval)：决策时按情境召回相关记忆

⚠ **效力边界**：Park 2023 仅在 **25 agent × 2 天 × 社交场景**验证。长程（10 万字级）是**开放风险**。

### 3.3 决策循环

```
感知场景 → 召回相关记忆 → 结合性格/目标/关系推理 → 输出
                                                    ├─ 对话
                                                    ├─ 动作
                                                    └─ 内心活动
```

### 3.4 两条可操作铁律

1. **不直接重写角色台词或内心** ——所有干预走情境层
2. **信息边界严格**——角色只知亲历/听说/推理之事

> **v3 修订**：v2 的"铁律 3 允许犯错"被删除——LLM 实际难以稳定"犯错"而不极端化。改为 3.5 的工程关注点。

### 3.5 已知工程风险

| 风险 | 描述 | 对策 |
|---|---|---|
| **长程记忆退化** | 百万字级向量检索精度塌陷、reflection 漂移、人设偏移 | 分层摘要（StoryBox Layer 1+2）+ 主动老化 + 章外人设重锚 |
| **人设不稳定** | 长序列中角色说话趋向均值 | DPO 训练（参考 Wayfarer-Large）+ 每幕前重注入性格摘要 |
| **"犯错"失败** | LLM 被要求犯错时要么回归最优解要么极端化 | MBTI/九型等性格向量 + 情绪状态机约束，而非 prompt 里说"你有缺点" |
| **StoryBox Fig 5 警示** | StoryBox 14/30 天模拟反而掉分 | 我们场景驱动范式天然规避（不追求连续时长），但 10 万字 summarization 层级要做到 3-4 层（StoryBox 只做 2 层） |

> **启发来源**：3.1 → StoryBox Table 4 schema；3.2 → Park et al. 2023 memory/reflection/retrieval；3.5 长程对策 → StoryBox hierarchical summarization + Wayfarer DPO；超值导向 → Stanislavski

---

## 4. 导演层：Drama Manager

### 4.1 三大操作

| 操作 | 作用 | 来源 |
|---|---|---|
| **压力时钟** (Countdown Clock) | 挂倒计时（"魔族入侵 0/12"），到点触发世界事件 | Apocalypse World |
| **命运变量注入** (Fate Variable) | 关键时刻往世界扔外部变量（让某人出现、让灾难降临） | 本架构原创 |
| **Front 管理** | 威胁前线集群（NPC + 资源 + 时钟） | Apocalypse World |

### 4.2 Trigger Schema（抄 Drama Llama）

作者用**自然语言**写 trigger：
```
IF <condition>
THEN <effect>
```

示例：
```
IF 林尘拒绝丹药 AND 当前场景在悬崖
THEN 师父出现
```

**两类 trigger**（Drama Llama 二分）：
- **Basic**：推进常规情节
- **Ending**：触发结局相关事件

**判定者是 LLM**——给 DM 的 LLM 看全局 Replay 状态 + trigger 表，由它决定何时激活。

### 4.3 三档切换（用户自选）

| 档 | 谁设 trigger | 谁激活 | 适合 |
|---|---|---|---|
| 手动 | 作者 | 作者手动命令 | 关键章、心中有画面 |
| 半自动 | 作者 | LLM 判断时机 | 日常创作（默认） |
| 全自动 | LLM 建议 | LLM 激活 | 素材期、番外 |

### 4.4 场景时空调度（v3 新增）

DM 的**主要工作台**不是"写下一章大纲"，而是决定下一幕的四元组：
```
next_scene = {
    time_coord:    "三年后" | "当晚" | "绝对 2024-09-04"
    space_coord:   "黑风崖顶" | "林家演武场"
    participants: [林尘, 黎]
    trigger_hook: "[命运变量] 师父闯入" 或 "自然衔接"
}
```

### 4.5 学术定位

本架构定位在 **Triangle Framework** 的 "landmark-guided hybrid"——emergent MAS 与 centralized planner 之间的中点。我们的**命运变量 / 压力时钟 = landmarks**。

> **启发来源**：Drama Llama (Mateas 组)、Apocalypse World (Vincent Baker)、Triangle Framework (AIIDE 2025)。

---

## 5. 多世界线演绎

### 5.1 分叉原则

**命运变量分叉**（改情境），**不是决策分叉**（指定行为）：
- ✅ 分支 A：师父没出现 / B：师父出现 / C：信物泄露
- ❌ 分支 A：林尘接了 / B：林尘拒绝

### 5.2 世界树

**数据结构**：抄 WHAT-IF plot-tree
- 根节点 = 故事开头
- 分支节点 = 命运变量分叉点
- 每条分支独立演绎若干幕

**稀疏分叉**：不是每幕分叉，只在戏剧关键点分叉（十几幕可能 2-3 个）。

### 5.3 操作

| 操作 | 作用 |
|---|---|
| 设为正史 | 某分支成主线，其余自动标为存档 |
| 存档 | 保留不用（番外用） |
| 废弃 | 彻底删除 |
| **摘取** (cherry-pick) | 把分支 A 的对白/场景摘到正史 B |
| 回分叉点 | 历史任意节点重新分叉 |

### 5.4 cherry-pick 心理兼容检查（本架构原创）

跨分支摘取不是无条件：

| 摘取档 | 条件 | 风险 |
|---|---|---|
| **直接摘** | 正史角色心理状态能支撑该对白/动作 | 低 |
| **改写摘** | 需轻微调整措辞使兼容 | 中，**明确承认改意思** |
| **禁止摘** | 正史角色根本不会说出这句话（内心完全不兼容） | 不可用 |

**兼容性判定**：比对两分支在该时点角色的 `super_objective` / `known_info` / 情感状态，若差异超过阈值则禁止直接摘。

### 5.5 根本取舍

纯命运分叉的多样性受限于角色性格空间。性格注定拒绝的角色，无论多少命运变量都不会自然接受——**加码命运 vs 接受"这不是他的故事"**。

### 5.6 定位：素材期工具

⚠ **成本警告**：墙钟成本线性膨胀（StoryBox 6 人 × 7 天 ≈ 4 小时；10 条分支 ≈ 40 小时）。

**产品定位**：多世界线是**素材期工具**，不是主流程。主创作线单主干，关键点偶尔分叉探索。

---

## 6. Replay 层（中间表示 IR）

> **v3 修订**：v2 的"first-class 独立销售品"主张**彻底放弃**。Replay 是结构化中间表示 + 作者审计工具。

### 6.1 Event Schema（抄 StoryBox Fig 2）

```
[Event ID]       5
[Start Time]     2024-09-01 9:00
[End Time]       2024-09-01 10:30
[Duration]       01:30:00
[Participants]   [林尘]
[Location]       青峰山:演武场:南角:木桩
[Description]    林尘独自练剑
[Detail]         <扩展：环境、情绪、动机、状态>
```

### 6.2 存储栈（抄 StoryBox）

| 层 | 技术 |
|---|---|
| 事件数据库 | **sqlite3** |
| 向量索引 | **FAISS** |
| Embedding 模型 | **jinaai/jina-embeddings-v3** |
| Embedding 维度 | **512** |
| 上下文 cap | **102,400 tokens** |

### 6.3 元信息字段（SCORE 三维 + 本架构补充）

```
meta:
  character_consistency:  # 角色状态
  emotional_coherence:    # 情感曲线
  plot_element:           # 伏笔/目标追踪
  causal_link:            # 因果链（本架构新增）
  information_flow:       # 信息传播（本架构新增）
```

### 6.4 作者审计工具链（本架构差异化）

StoryBox 的 sqlite3 events 藏在 pipeline 内部，不暴露给作者。我们要暴露：
- **可视化事件流面板**
- **跨世界线 diff 视图**
- **场景回放器**
- **审计入口**："为什么林尘这里这么说？" → 跳转到相关记忆/情境

**这是 Replay 层的真正差异化，不是数据模型本身。**

### 6.5 Retrieval 策略补齐（StoryBox 未披露）

- **top-k = 20**（初始）
- **keyword + dense 双通道融合**（相加 or MMR rerank）
- **Dynamic window** 层级数 **3-4 层**（StoryBox 只做 2 层，撑到 30 天就崩；我们做 3-4 层应对 10 万字）

> **启发来源**：StoryBox event schema + 存储栈；SCORE Dynamic State Tracking 维度。

---

## 7. 成文层：翻译家不是编剧

### 7.1 双 agent 分工（抄 HoLLMwood）

- **Writer agent**：读 Replay → 产出散文
- **Editor agent**：读散文 → 给修订建议 → Writer 迭代

### 7.2 三阶段打磨（抄 Dramaturge）

| 阶段 | 范围 |
|---|---|
| **Global Review** | 全卷级一致性、弧光、伏笔回收 |
| **Scene Review** | 场景级张力、节奏、对话自然度 |
| **Hierarchical Coordinated Revision** | 跨章交叉修订 |

Dramaturge 论文验证：script-level +53.2%，scene-level +65.1%。

### 7.3 三个控制旋钮（Genette 五维简化）

- **POV**：跟随单人 / 全知 / 多视角轮转
- **详略**（时长）：过渡戏 5:1 压缩，高潮戏 1:5 扩写
- **叙述距离**：紧贴内心 vs 冷眼旁观

### 7.4 "不改意思"的精确定义

> 成文层**不得改变**角色 agent 在该幕的**核心决策和情感坐标**。
> 可以：重排句序、增加感官细节、补写与当幕情感一致的心理描写。
> 不可以：让角色做出 Replay 里不存在的决定、或表达与 Replay 记录的情感相反的心理活动。

### 7.5 必避陷阱（水野良教训）

**不要让成文层"按预设世界观"改写 Replay**。水野良 2024 年 CBR 采访承认早期小说糟，因为"硬塞预设压垮玩家即兴"。

涌现的意外是金子——默认保留。

> **启发来源**：HoLLMwood（Writer+Editor）、Plug-and-Play Dramaturge（三阶段）、Genette 叙事话语五维、水野良。

---

## 8. 长篇节奏管理

### 8.1 四层结构

```
全书层：核心命题 / 主角弧光终点 / 大伏笔
  ↓
卷级骨架：Save the Cat 15 节拍（每 10-15 万字一卷）
  ↓
压力时钟层（DM 主要工作台）：3-5 个 Countdown Clock
  ↓
幕级涌现（角色 agent 自主）
```

> **v3 修订**：v2 的 "Dramatica 与 Save the Cat 并列"已移除（两套理论相互批评）。**选 Save the Cat**：节拍具体可验证、网文实践友好。Dramatica 的双层需求保留为**角色卡字段**。

### 8.2 骨架-涌现冲突解决协议 ⚠

当涌现与卷级节拍冲突时：

| 偏离 | 处理 |
|---|---|
| ≤ 2 幕延迟 | 自动容忍 |
| 3-5 幕延迟 | 预警，作者介入：施加更强命运变量 或 改卷级骨架 |
| > 5 幕或方向反转 | 冻结本幕，作者必须做结构决定 |

⚠ 上述阈值**尚无实测支撑**。**验证动作**：MVP H2/H3 实验期间记录自然偏离分布，据此校准。

### 8.3 三条自动警戒线

- **角色弧光停滞**：主角 10 幕无内在变化
- **伏笔超期**：某伏笔 30 幕未推进（**需作者手动标注伏笔**——系统无法自动识别涌现出的伏笔）
- **时钟卡住**：某时钟 15 幕无推进

系统**只预警，不替作者决定**。

### 8.4 网文特化

成文层硬约束：每章末尾必须留悬念钩子。

### 8.5 10 万字开放风险 ⚠

StoryBox Fig 5 显示 14/30 天掉分。我们瞄准 10 万字 ≈ 8 倍字数。

**对策组合**：
- 场景驱动范式（天然规避连续时长问题）
- Hierarchical summarization 做到 3-4 层（StoryBox 只做 2 层）
- 章外人设重锚
- 但**最终验证只能靠实测**

> **启发来源**：Save the Cat（Blake Snyder）；Fronts + Clocks（Vincent Baker）。

---

## 9. 作者心理契约与读者期待

### 9.1 作者的角色转换代价

本系统要求作者**从"创作者"切换到"导演"**——放弃"我赋予主角意志"的心理回报，接受"我观察角色自主行动"。

**未解决**：谁会长期使用一个把自己从主体降为观察者的工具？

**可能缓解**：
- 提供"**强介入模式**"作为逃生阀（允许作者在关键幕直接改写 Replay，标记为"作者手动幕"，不影响其他涌现幕）
- 重新定位为"高阶创作乐趣"——**需用户群前测验证**

### 9.2 读者口味反方向

网文读者喜欢强目的性、强主题、强作者意志（爽文）。涌现产物天然松散、去中心。

**对策**：成文层"网文化"步骤（压缩日常、强化冲突、每章钩子）。这是**市场适配**，不是文学优化。

### 9.3 验证动作（v3 新增）

| 动作 | 死线 |
|---|---|
| 强介入模式 MVP 实现 | v3.1（MVP 实验后） |
| 20-50 人目标用户前测问卷（作者心理契约接受度） | v3.1 前 |
| 10 本头部网文"爽点密度"分析作为成文层调参基准 | v3.1 前 |

---

## 10. 中文语料与风格适配

### 10.1 问题

landscape **零覆盖**。StoryBox 20 story 全英文；所有 Drama Management 工作全英文；所有 evaluation prompt 全英文。

### 10.2 已知局限

- GPT/Claude 对"渡劫""金丹""道心"训练分布远不如英文奇幻
- 直接生成修仙文易翻译腔
- 中文 Replay 可读性未经验证（v2 宣称"可销售"已撤回）

### 10.3 可能路径

1. **风格参考语料库**：10 本头部网文，成文层每次生成引入 few-shot 风格样本
2. **LoRA/DPO 微调**：在专用网文语料上训练成文层（候选 base：LongWriter-Zero 32B）
3. **人机协作校稿回注风格**
4. **专用评测 prompt**：覆盖爽感、打脸、章末钩子等网文特有维度

### 10.4 验证动作（v3 新增）

| 动作 | 死线 |
|---|---|
| 10 本头部网文语料收集 + few-shot 原型 | Parity 实验前 |
| 中文 Frozen City 翻译版 parity 实验 | MVP 第 2 周 |
| 爽点密度 + 章末钩子 evaluation prompt 初稿 | v3.1 前 |

**定位**：这是**最大护城河 or 最大陷阱**，不能模糊处理。

---

## 11. IP 归属

如果创意大部分来自 agent 涌现，谁是作者？中国著作权法对 AI 参与作品权属**尚无成熟判例**。

**验证动作**：商业化前咨询法律意见（死线：商业化前）。

**立场（暂定）**：作者作为 Drama Manager + Curator 的角色构成独创性贡献。以此为默认商业模型设计边界。

---

## 12. 理论来源索引（含效力范围）

| 模块 | 启发 | 来源 | 效力范围 |
|---|---|---|---|
| 两层分层 | fabula/sjužet | Genette / 俄国形式主义 | 理论基础充分 |
| 反面教训 | 模拟+讲述混层 → 失败 | Storytron、AI Dungeon | 经验证据充分 |
| 同路人 | hybrid bottom-up | **StoryBox 2025** | ✅ 最强参考 |
| 分工范式 | planner + writer + scratchpad | Agents' Room ICLR 2025 | 可直接借鉴 |
| 角色记忆 | memory + reflection + retrieval | Park et al. 2023 | ⚠ 仅 2 天 sandbox 验证 |
| 长程摘要 | hierarchical windowed summarization | StoryBox 2025 | 初步验证，30 天已掉分 |
| 人设训练 | DPO 连贯性 | Wayfarer-Large | 产品侧验证 |
| 角色心智 | super-objective / given circumstances | Stanislavski | 类比使用 |
| 信息边界 | possible worlds | Marie-Laure Ryan | 类比使用 |
| 导演接口 | natural-language trigger schema | **Drama Llama 2025** | ✅ Façade 直接继任 |
| 时钟机制 | Fronts + Countdown Clocks | Apocalypse World | TRPG 经验充分 |
| 学术定位 | landmark-guided hybrid | Triangle Framework 2025 | ✅ 唯一清晰定位工作 |
| GM 裁定 | preconditions/effects | STORY2GAME (Riedl) | 部分借鉴 |
| 成文旋钮 | 叙事话语五维 | Genette | 经典 |
| 双 agent 成文 | Writer + Editor | HoLLMwood 2024 | ACL 2024 验证 |
| 稿件打磨 | Global + Scene + Coordinated | Dramaturge 2025 | 双层胜率验证 |
| 长篇骨架 | 15 节拍 | Save the Cat | 工业标准 |
| 世界树 | plot-tree | WHAT-IF 2024 | 学术验证 |
| 场景驱动 | Scene Master | RELATE-Sim 2025 | 单一关系场景验证 |
| 网文语感 | （空缺） | — | ⚠ **landscape 零覆盖，自研** |
| 产业范式 | Replay/Novel 分层 | Group SNE 产业 | 日本验证，中文未验证 |

---

## 13. 评测 Stack（v3 从附录升正文）

**我们不自研评测**。叠加现有工作即可：

| 评测 | 维度 | 来源 |
|---|---|---|
| **Autodesk ASP benchmark** | causal soundness / character intentionality / dramatic conflict | HuggingFace: ADSKAILab/LLM-narrative-planning-taskset |
| **StoryBox 6 维** | Plot / Creativity / Character / Language / Conflict / Overall | arXiv 2510.11618 Table 2/3 |
| **ISS** (Identity Stable Set) | 角色行为一致性 0-10 | StoryBox Table 3 prompt |
| **SCORE 三维** | character / emotional / plot | arXiv 2503.23512 |
| **Dramaturge 双层** | script-level / scene-level 胜率 | arXiv 2510.05188 |

**Parity baseline**：StoryBox "Frozen City" premise 的 1.2 万字复现。

---

## 14. 参考实现清单（可直接抄）

```yaml
persona_schema:
  source: StoryBox Table 4
  fields: [Name, Age, Innate, Learned, Currently, Lifestyle,
           Living_Area (5-level), Daily_Plan_Requirement]
  extension: [super_objective, relationship_network, known_info]

memory_mechanism:
  source: Park et al. 2023
  components: [event_stream, reflection, retrieval]
  caveat: 长程开放风险（§3.5）

event_storage_stack:
  db: sqlite3
  vector: FAISS
  embedding: jinaai/jina-embeddings-v3 (dim 512)
  context_cap: 102400 tokens

event_schema: StoryBox Fig 2

trigger_schema:
  source: Drama Llama
  format: natural-language IF-THEN
  types: [basic, ending]

writer_editor_loop:
  writer: HoLLMwood style
  editor: Dramaturge 3 stages

world_tree:
  data_structure: WHAT-IF plot-tree
  extension: [回收操作, cherry-pick 心理兼容检查]

evaluation: (见 §13)
```

---

## 15. Open-Theatre 可读参考点（v3 新增）

**为什么不 fork**：Open-Theatre 面向"玩家即时互动"（同步短窗高交互），我们要做"作者离线创作"（异步长窗低交互高可审计）。架构假设根本不同，fork = 拿 Unity 做 PDF 排版。

**但值得读代码的地方**：
- `memory/` 分层记忆实现（hierarchical summarization 工程参考）
- `ui/developer_console/` UI 组件（作者审计面板可复用）
- `agents/actor_agent.py` 的记忆检索逻辑
- `configs/architecture_*.yaml` 配置结构

**阅读时的心态**：**抄片段 not fork 项目**。

---

## 16. 尚待决定的问题

1. **实现路径**：改造现有 Dramatica-Flow 代码库 vs 另起新项目
2. **LLM 选型**：模拟层（稳定人格）vs 成文层（文笔）是否用不同模型
3. **"正史确认"触发时机**：每幕 / 回溯
4. **Drama Manager 自动化程度**：导演离线时是否自主推进
5. **Replay 审计面板的 UI 最小集**
6. **涌现质量评估**：超越 Autodesk benchmark 的人工评估怎么做
7. **中文读者对 Replay 的可读性**（§10 前测未做）
8. **网文风格适配最佳路径**（few-shot vs LoRA vs 校稿回注）
9. **IP 归属立场的法律意见**
10. **多世界线分叉的自动触发阈值**（哪些情节点值得 DM 提示分叉）

---

## 17. 下一步建议（优先级排序）

### 🔴 优先级 1：MVP 实验（2 周）

**目标**：用实测数据回答"架构是否真 work"

**5 条可证伪假设**：
- H1: 我们架构在 Frozen City premise 上 StoryBox 6 维至少 3 维胜出
- H2: 场景驱动 vs 时间步长在 Conflict Quality + Character Development 上显著差异
- H3: 有 DM（trigger + clock）vs abnormal_factor=0.3 消融
- H4: 记忆流（memory+reflection）vs 无记忆 的 ISS 差异
- H5: 中文 Frozen City 翻译版 parity 稳健性

**每条假设预先注册判定线**。实验细节见 `docs/plans/` 待写的 `2026-04-26-mvp-experiment-spec.md`。

### 🟡 优先级 2：中文前测（MVP 第 2 周）

- 10 本头部网文语料收集
- few-shot 风格 prototype
- 中文 Frozen City 翻译 parity

### 🟡 优先级 3：心理契约前测（v3.1 前）

- 20-50 人目标用户问卷（作者从创作者到导演的接受度）

### 🟢 优先级 4：学术基础阅读

按顺序：
1. **StoryBox 论文原文**（对照我们已有 notes 验证理解）
2. **Drama Llama 论文**（trigger schema 细节）
3. **Open-Theatre 代码审计**（不 fork，读关键文件）
4. **RELATE-Sim 论文**（场景驱动细节）
5. **Triangle Framework 论文**（学术话术锚定）

---

## 18. 修订日志

- **v1**（2026-04-24）：brainstorming 产出初版
- **v2**（2026-04-25）：独立 red-team review 修订（降级 Stanford Agents 射程、删除涌现圣洁话术、Replay 降为 IR、Dramatica 与 Save the Cat 二选一、新增冲突协议/心理契约/中文适配/IP）
- **v3**（2026-04-25）：整合 StoryBox 研读 + 6 领域 landscape 调研 + 组件借鉴决策——
  - 新增 §0.5 同路人更新、§1 System Invariants 取代"核心哲学"、§2.5 时空建模范式、§13 评测 stack、§14 参考实现、§15 Open-Theatre 参考点
  - 升级 §3 角色 agent（StoryBox schema）、§4 Drama Manager（Drama Llama trigger）、§5 多世界线（WHAT-IF plot-tree）、§6 Replay IR（StoryBox 存储栈）、§7 成文层（HoLLMwood + Dramaturge）
  - §9/§10/§11 保留并加验证动作+死线
  - §12 理论索引补充效力范围列
  - §17 下一步重排，MVP 实验为第一优先级
  - **彻底放弃 Replay 可独立销售主张**

---

## 附：核心一句话

> 传统 AI 写作是"给 AI 一个提纲让它填空"；本架构是"搭一个有生命的故事世界，让它自己长出故事，然后派文学家去记录"。**涌现-控制的张力、长程记忆退化、中文市场验证、作者心理契约**是四块硬骨头，**已有明确验证路径，不再回避**。
