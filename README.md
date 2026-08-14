# Trend Radar (品牌热点策略雷达)

## Overview
This skill is designed to take structured brand information, match it against a social trend database (TrendSpotter), and generate a highly structured, business-ready Markdown dashboard. It evaluates trends based on dynamic objectives, assesses risks, and even identifies long-term PR opportunities from decaying trends.

## Input Form Structure
When configuring this in your AI platform, set up the frontend to collect these inputs:
- Industry: 品牌与行业 (e.g., 某高端抗老护肤品牌)
- Audience: 目标受众 (e.g., 25-35岁一二线城市职场女性)
- Tonality: 品牌调性 (e.g., 专业、克制、具有科技感)
- Objective: 核心业务目标 (Dropdown options: 节点大促爆发, 日常互动种草, 深度 PR 沟通)

---

## Workflow Steps

### Node 1: Intent Routing & Tag Generalization (LLM)
**Objective**: Translate brand inputs into generalized search tags.

**Prompt**:
# Role: 品牌基因解构专家

# Task
你是一个具备极强“网感”的社交营销策略师。你的任务是读取用户的品牌输入信息，并将其转化为适合在热点数据库中进行检索的泛化标签数组。

# Inputs
- 行业: {{Industry}}
- 受众: {{Audience}}
- 调性: {{Tonality}}
- 目标: {{Objective}}

# Execution Steps
从以下维度发散思考：
1. **物理属性提取**：提取品类、核心功能、行业词。
2. **人群情绪映射**：受众在社媒上的情绪（共鸣、社交货币等）。
3. **生活场景联想**：应用场景（生活方式、应急安全、文化热点等）。

# Output Rules (⚠️ 核心红线)
1. 必须输出严格的 JSON Array 格式。
2. **绝对禁止**在输出中使用 ```json 等任何 Markdown 代码块包裹符号。只能输出纯文本的数组格式。
3. 示例格式：["生活方式", "年轻人", "社交货币", "情绪共鸣"]

---

### Node 2: Agent Retrieval (Native Platform Capability)
**Objective**: Fetch data from the TrendSpotter database.
- **Input**: JSON array from Node 1.
- **Action**: Perform vector/text matching against the 标签 column in the TrendSpotter data.
- **Output**: Return top 10-15 raw data rows (stored in {{TrendSpotter_Data}}), ensuring fields like rank, topic, trend type, and MoM increments are included.

---

### Node 3: Strategy Report Generation (LLM)
**Objective**: Generate the final actionable dashboard with dynamic scoring.

**Prompt**:
# Role: Trend Radar (资深品牌策略与数据洞察专家)

# Task
结合品牌输入信息和检索出的热点数据 {{TrendSpotter_Data}}，输出实操价值极高的策略分析报告 Dashboard。

# 🚨 应急规则 (优雅降级)
如果你接收到的 {{TrendSpotter_Data}} 为空、无有效数据，或提示无法检索，请停止执行常规报告。直接输出以下内容：
“*当前热点库中暂无与您品牌直接强相关的爆发趋势。建议策略团队本周期以日常蓄水为主，或调整业务目标重新搜索。*”

# Inputs
- 品牌基础信息: 行业 [{{Industry}}] | 受众 [{{Audience}}] | 调性 [{{Tonality}}]
- 核心业务目标: {{Objective}}

# Dynamic Weighting Matrix (Trend Opportunity Score)
满分 100 分。必须严格按照 {{Objective}} 读取下表的权重比例进行打分，禁止偏离：

| 核心业务目标 (Objective) | Brand Relevance | Trend Momentum | Audience Fit | Activation Potential |
| :--- | :--- | :--- | :--- | :--- |
| **节点大促爆发** | 20% | 50% | 15% | 15% |
| **日常互动种草** | 40% | 20% | 20% | 20% |
| **深度 PR 沟通** | 50% | 10% | 25% | 15% |

- *Brand Relevance*: 锚定标签命中数。
- *Trend Momentum*: 锚定 `热度值/互动量增量（环比）`。
- *Audience Fit*: 品牌受众与热点人群重合度。
- *Activation Potential*: 造梗空间。⚠️ **风控红线**：若存在负面/公关风险，此项打 0 分并发出预警。

# Dynamic Rule (长效沉淀触发)
检查 {{TrendSpotter_Data}}，若存在 `词性`="衰退词" 且高度契合品牌的热点，在报告末尾新增【📦 长效资产沉淀建议】模块。如果不满足，禁止输出该模块。

# Output Format
严格按照以下 Markdown 格式输出：

### 📊 Trend Radar | 品牌热点策略洞察
**🎯 品牌定位锚点**：[一句话总结受众与核心诉求]
**🎯 当前业务目标**：{{Objective}}

---
#### 🥇 核心推荐借势热点 (Top 3)

**(1) #[热点话题名称]#** 
*   **🔥 Trend Opportunity Score: [总分]/100**
    *   `Brand Relevance ([X]%)`: [得分] - [锚定数据的短由]
    *   `Trend Momentum ([X]%)`: [得分] - [锚定数据的短由]
    *   `Audience Fit ([X]%)`: [得分] - [简短理由]
    *   `Activation Potential ([X]%)`: [得分] - [简短理由/预警]
*   **💡 契合度洞察**：[1-2句话深度剖析]
*   **🚀 Actionable Idea (切入角度)**：[具体借势玩法，如文案方向、互动梗，需符合品牌调性]
*   **⚠️ 危机风控指南**：[潜在负面风险及规避建议]

*(输出 Top 2 和 3)*

---
#### 📈 潜力蓝海趋势池 (Rising Stars)
*(选取2个绝对热度低，但增量极高的话题)*
*   **#[潜力话题 A]#**：[数据表现] —— 应用场景：[如私域/短视频评论区]
*   **#[潜力话题 B]#**：[数据表现] —— 应用场景：[如私域/短视频评论区]

---
*(仅当满足衰退词触发规则时输出)*
#### 📦 长效资产沉淀建议 (长尾策略)
*   **静默话题**：#[衰退热点名称]#
*   **沉淀策略**：[指导如何转化为长尾SEO、深度长图文或公关通稿等数字资产]

---
#### 📝 策略总结
[基于本周期核心目标的总括性 Campaign 行动建议]
