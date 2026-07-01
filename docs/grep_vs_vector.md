# Is Grep All You Need? Agent Harness 如何重塑智能体搜索

> 论文: [Is Grep All You Need? How Agent Harnesses Reshape Agentic Search](https://arxiv.org/html/2605.15184v1/)
> 作者: Sahil Sen, Akhil Kasturi, Elias Lumer, Anmol Gulati, Vamse Kumar Subbiah
> 机构: PricewaterhouseCoopers, U.S.
> 日期: 2026-05-14

## 1. 研究动机

在 LLM 智能体系统中，RAG 被广泛采用，但**检索策略与智能体架构/工具调用设计之间的交互作用**研究不足。关键实践维度——工具输出如何呈现、对噪声语料的鲁棒性——缺乏系统性研究。

核心问题：**在智能体框架中，词法搜索（grep）是否足够好？语义搜索的优势是否被高估了？**

## 2. 研究设计

### 2.1 因子实验设计：三维度交叉

| 维度 | 选项 |
|------|------|
| 检索模式 | grep-only vs vector-only |
| Harness 类型 | Chronos (自定义) vs 三种 CLI (Claude Code, Codex, Gemini CLI) |
| 交付方式 | Inline（内联到对话）vs File-based（写入磁盘） |

### 2.2 数据集

- **LongMemEval 基准**的 116 题子集
- 六个类别：知识更新、多会话、单会话变体、时间推理

### 2.3 测试模型

- Claude Opus 4.6
- Claude Haiku 4.5
- GPT-5.4
- Gemini 3.1 Pro
- Gemini 3.1 Flash-Lite

### 2.4 评估方式

- 二元准确率评估
- GPT-4o 作为评判模型

## 3. 核心实验结果

### 3.1 实验一：完整 Haystack 比较

**关键发现——Inline 交付模式:**

> "词法搜索在所有 harness-model 配对中一致强于密集检索：inline grep 在每个配对中都超越 inline vector。"

| Harness + Model | Grep (Inline) | Vector (Inline) | 差距 |
|----------------|---------------|-----------------|------|
| Chronos + Claude Opus 4.6 | **93.1%** | 83.6% | +9.5pp |
| Claude Code + Claude Opus 4.6 | **76.7%** | 75.0% | +1.7pp |
| Gemini CLI + Gemini 3.1 Flash-Lite | **87.1%** | 67.2% | +19.9pp |

**Programmatic（文件）交付模式:**
- 10个 harness-model 配对中，5个 vector 反超 grep
- 文件交付引入的组合式工具使用挑战可以**反转检索比较结果**

### 3.2 实验二：噪声扩展

测试会话限制：s5, s10, s20, s30, full (39-66 sessions)

**关键观察:**
- 检索器排序取决于 harness——不存在普遍的单调退化模式
- Grep 优势在不同配置间变化
- 在较低会话数下 vector 有时表现更好

## 4. 核心方法

### 4.1 Chronos 自定义 Harness

- 动态 prompting：系统指令随问题类别变化
- 类别条件化的引导
- 受控的工具可用性

### 4.2 检索实现

| 检索类型 | 实现 |
|----------|------|
| 词法（Grep） | 正则匹配 |
| 语义（Vector） | 近似最近邻 + 重排序 |

## 5. 三大核心贡献

### 5.1 检索-Harness 交互

检索效果在架构不同的 harness 之间**不稳定**，尽管使用完全相同的语料。

### 5.2 噪声鲁棒性

性能随无关内容增长的演变是复杂的交互模式，而非简单的退化曲线。

### 5.3 跨栈异质性

> "在 Chronos 和 provider CLI 之间移动同一个backbone，准确率变化幅度相当于更换检索器。"

## 6. 关键发现总结

| 发现 | 解释 |
|------|------|
| Grep 通常优于 Vector | 答案常依赖于字面跨度（literal spans） |
| Harness 比检索器更重要 | 更换 harness 的影响 ≈ 更换检索器 |
| 文件交付可反转结果 | 引入组合式工具使用的额外挑战 |
| 弱模型更受益于 Grep | 词法检索的精度偏差对弱模型帮助更大 |
| 不存在通用最优 | 需要针对具体 harness+模型+任务调优 |

## 7. 适用范围与局限

**适用范围:**
- 长记忆对话 QA
- 答案依赖于逐字跨度的任务
- 结构化或半结构化数据

**局限（作者声明）:**
> "在需要语义综合的领域，密集检索和混合路由可能表现不同。"

## 8. 核心洞察

这篇论文提出了一个**反直觉但重要的观点**：

1. **检索策略不是孤立的选择** ——同一个检索策略在不同 harness 中表现截然不同。这意味着架构设计（如何呈现结果、如何引导工具使用）比检索算法本身可能更关键。

2. **Grep 的意外优势** ——在精确回忆任务中，简单的关键词匹配比语义搜索更可靠。这挑战了"语义搜索总是更好"的常见假设。

3. **交付方式是隐藏变量** ——内联 vs 文件的结果呈现方式可以完全反转检索策略的优劣。这意味着 RAG 系统的"最后一公里"设计至关重要。

**实践启示:**
- 在选择检索策略前，先确定智能体架构和工具交互设计
- 对于精确信息检索任务，不要忽视简单的词法搜索
- 系统性评估应覆盖检索×harness×交付方式的完整组合空间
- 不同模型能力（强/弱）可能需要不同的检索策略配对
