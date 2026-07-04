# RAG 典型数据集与任务类型总结

基于本仓库收录的 20+ 篇文献，整理 RAG 领域的评估数据集、查询/任务类型分类，以及各方法在不同任务上的适用性。

---

## 一、任务/查询类型分类

### 1.1 单跳事实查询（Single-Hop Factoid）

| 维度 | 说明 |
|------|------|
| 定义 | 直接事实查找，仅需 1 个文档/段落即可回答 |
| 典型问题 | "X公司的CEO是谁？""产品A的发布日期？" |
| 相关论文 | RAG原始论文, LightRAG (Low-level), UnWeaver, Keyword Search |
| 最佳方案 | 向量 RAG 或关键词搜索即可；图谱方法收益极小 |
| 代表数据集 | Natural Questions, WebQuestions, TriviaQA |

### 1.2 多跳推理（Multi-Hop Reasoning, 2-4 文档链式）

| 维度 | 说明 |
|------|------|
| 定义 | 需要跨 2-4 个相关文档/段落的链式推理 |
| 典型问题 | "A的导师和B的雇主之间是什么关系？""X的父亲的兄弟在哪家公司工作？" |
| 相关论文 | FlowRAG, LinearRAG, HippoRAG, PathRAG, HyperGraphRAG, MeMo, UnWeaver, OMD-GraphRAG |
| 最佳方案 | 图谱方法带来 2-3x 准确率提升；HippoRAG 性价比最高 |
| 代表数据集 | HotpotQA, 2WikiMultiHopQA, MuSiQue, MultiHop-RAG |

### 1.3 全局聚合（Global Aggregation, 20+ 文档并行）

| 维度 | 说明 |
|------|------|
| 定义 | 需要遍历大量文档进行统计计算的查询 |
| 典型问题 | "有多少人在X公司工作过？""谁的工作年限最长？""列出薪资最高的5位员工" |
| 相关论文 | GlobalRAG, Microsoft GraphRAG |
| 最佳方案 | 文档级检索 + 符号计算工具（纯 LLM 失败）；移除聚合工具后 F1 下降 79.2% |
| 代表数据集 | GlobalQA (13,000+ QA, 2,000+ 简历, 23 领域) |

**GlobalQA 四种子任务：**

| 子任务 | 占比 | 示例 |
|--------|------|------|
| 计数 (Counting) | 16.7% | "有多少人具有X属性？" |
| 极值 (Max-Min) | 33.9% | "谁的X最多/最少？" |
| 排序 (Sorting) | 16.3% | "按Y属性排列这些候选人" |
| Top-k 提取 | 33.9% | "列出Z最高的5个" |

### 1.4 Schema 约束查询（Ontology-Grounded）

| 维度 | 说明 |
|------|------|
| 定义 | 需要遵循领域特定结构约束的查询 |
| 典型问题 | "符合指南X的治疗方案有哪些？""满足合规要求Y的操作流程？" |
| 相关论文 | OG-RAG, OMD-GraphRAG, KnowLP |
| 最佳方案 | 本体约束检索（+40% 正确性，+55% 事实召回）|
| 代表数据集 | Agriculture (大豆/小麦种植), Medical (高血压指南) |

### 1.5 N元关系查询（N-ary Relations）

| 维度 | 说明 |
|------|------|
| 定义 | 涉及 3+ 实体同时关联的事实查询 |
| 典型问题 | "哪位医生在什么时间以什么剂量给患者Y开了药物X治疗疾病Z？" |
| 相关论文 | HyperGraphRAG, OG-RAG |
| 最佳方案 | 超图表示；信息论证明二元图对此必然有损 |
| 代表数据集 | HyperGraphRAG benchmark (5领域 × 512题, 256 binary + 256 n-ary) |

### 1.6 路径/因果推理（Path/Causal Reasoning）

| 维度 | 说明 |
|------|------|
| 定义 | 追踪实体间的显式关系链路 |
| 典型问题 | "从A到B的因果链是什么？""这两个事件之间的关联路径？" |
| 相关论文 | PathRAG, FlowRAG, OMD-GraphRAG |
| 最佳方案 | 基于流/路径的剪枝（PathRAG 60% 胜率 vs GraphRAG, 44% token 缩减）|
| 代表数据集 | HotpotQA, 2WikiMultiHopQA |

### 1.7 长文档深度理解（Long-Document Understanding）

| 维度 | 说明 |
|------|------|
| 定义 | 针对单个长文档的复杂查询（书籍、手册、法规）|
| 典型问题 | "第3.2节关于规则X的例外是什么？考虑第1章的定义" |
| 相关论文 | BookRAG, PageIndex, Doc-to-LoRA |
| 最佳方案 | 层次结构索引或推理式导航；Doc-to-LoRA 可突破窗口 4x+ |
| 代表数据集 | NarrativeQA, NIAH (Needle-in-a-Haystack) |

### 1.8 时间推理（Temporal Reasoning）

| 维度 | 说明 |
|------|------|
| 定义 | 需要时间序列推断的查询 |
| 典型问题 | "X和Y日期之间发生了什么事件？""按时间顺序排列..." |
| 相关论文 | OMD-GraphRAG, Is Grep All You Need? |
| 最佳方案 | 目前所有方法的最弱环节；OMD-GraphRAG 仅 65.13% F1（vs 推理型 90.53%）|
| 代表数据集 | MultiHop-RAG (temporal subset), LongMemEval |

### 1.9 精确回忆（Exact Recall / Literal Span）

| 维度 | 说明 |
|------|------|
| 定义 | 需要逐字回忆特定信息的查询 |
| 典型问题 | "三次对话前我说了什么？""合同第X条的原文？" |
| 相关论文 | Keyword Search (Amazon), Is Grep All You Need? (PwC), Sirchmunk |
| 最佳方案 | 词法搜索（Grep）一致优于向量检索；在 inline 交付模式下差距最大 |
| 代表数据集 | LongMemEval (116 questions), FinanceBench |

### 1.10 对比查询（Comparison）

| 维度 | 说明 |
|------|------|
| 定义 | 比较多个实体的属性差异 |
| 典型问题 | "方法A、B、C在方法上有什么不同？" |
| 相关论文 | OMD-GraphRAG |
| 最佳方案 | 双通道检索（精确+语义融合）；中等难度 |
| 代表数据集 | MultiHop-RAG (comparison subset) |

---

## 二、主要评估数据集汇总

### 2.1 多跳推理基准

| 数据集 | 规模 | 使用论文 | 特点 |
|--------|------|----------|------|
| HotpotQA | 标准多跳 QA | FlowRAG, LinearRAG | 开放域，2跳为主 |
| 2WikiMultiHopQA | 标准多跳 QA | FlowRAG, LinearRAG | 基于 Wikipedia，2-4跳 |
| MuSiQue | 2,417 题 (UnWeaver) | FlowRAG, LinearRAG, MeMo, UnWeaver | 组合式 3-4 跳，难度最高 |
| MultiHop-RAG | 149 篇长篇新闻 | OMD-GraphRAG, OG-RAG | 推理/对比/时间三类查询 |

### 2.2 领域特定数据集

| 数据集 | 规模 | 使用论文 | 领域 |
|--------|------|----------|------|
| STaRK-Prime | 129K 实体, 8.1M 关系 | Is GraphRAG Needed? | 精准医疗 |
| Agriculture | 600K tokens / 85 docs | LightRAG, OG-RAG | 农业（大豆/小麦）|
| CS | ~1M tokens | LightRAG, HyperGraphRAG | 计算机科学 |
| Legal | ~2M tokens | LightRAG, HyperGraphRAG | 法律 |
| Medical | 不同规模 | FlowRAG, LinearRAG, HyperGraphRAG | 医疗（高血压等）|
| COVID-QA | 1,234 题 | UnWeaver | COVID 医学文献 |
| eManual | 522 题 | UnWeaver | 技术手册 |
| Tech-QA | 596 题 | UnWeaver | 技术支持 |
| FinanceBench | 金融 QA | Keyword Search (Amazon) | 金融报告 |

### 2.3 论文新提出的基准

| 数据集 | 规模 | 提出论文 | 独特价值 |
|--------|------|----------|----------|
| **GlobalQA** | 13,000+ QA, 2,000+ 简历, 23 领域 | GlobalRAG | 首个全局聚合评估；42.6% 查询需 20+ 文档 |
| **HyperGraphRAG benchmark** | 5域 × 512题 (256 binary + 256 n-ary) | HyperGraphRAG | 区分二元/N元关系来源的查询 |
| **LongMemEval** (子集) | 116 题, 39-66 sessions | Is Grep All You Need? | 长记忆对话，6 类查询 |

### 2.4 经典 NLP 基准（RAG 原始论文使用）

| 数据集 | 使用论文 | 任务类型 |
|--------|----------|----------|
| Natural Questions | RAG 原始 | 开放域事实 QA |
| WebQuestions | RAG 原始 | Web 事实 QA |
| TriviaQA | RAG 原始 | 知识问答 |
| MS-MARCO | RAG 原始 | 抽象式 QA |
| FEVER | RAG 原始 | 事实验证（3-way/2-way）|
| NarrativeQA | MeMo | 长叙事理解 |
| BrowseComp-Plus | MeMo | 复杂浏览理解 |

### 2.5 教育/学习类数据集

| 数据集 | 规模 | 使用论文 | 特点 |
|--------|------|----------|------|
| Junyi | 835 知识点, 525,061 学习者 | KnowLP | K12 数学 |
| MOOCCubeX | 443 知识点, 629 学习者 | KnowLP | MOOC 课程 |
| ASSISTments2009 | 167 知识点, 4,217 学习者 | KnowLP | 数学辅导 |

---

## 三、任务类型 × 最佳方法矩阵

| 任务类型 | 最佳方法 | 性能参考 | 不适合的方法 |
|----------|----------|----------|-------------|
| 单跳事实 | Vector RAG / 关键词搜索 | 图谱收益极小 | GraphRAG（过度设计）|
| 多跳推理 | LightRAG / FlowRAG / HyperGraphRAG | 2-3x 准确率提升 | 纯向量（无法跨文档关联）|
| 全局聚合 | GlobalRAG (Doc-level + 符号工具) | 现有方法 <2 F1→6.63 F1 | 所有 chunk-level 方法（-94.5%）|
| Schema 约束 | OG-RAG / OMD-GraphRAG | +40% 正确性 | 无本体方法 |
| N元关系 | HyperGraphRAG | +7.45 F1 | 二元图（信息论证明有损）|
| 路径推理 | PathRAG / FlowRAG | 60% 胜率 vs GraphRAG | 社区摘要方法 |
| 长文档理解 | BookRAG / PageIndex / Doc-to-LoRA | 突破窗口 4x+ | Chunk-level RAG |
| 精确回忆 | Grep / 关键词搜索 | 90%+ RAG 性能 | 向量检索（语义噪声）|
| 时间推理 | 所有方法均弱 | 最好仅 65.13% F1 | — |

---

## 四、关键发现

### 4.1 检索-生成鸿沟
检索覆盖率 83.5%，但 LLM 仅利用 47.9%。前 10% 位置的信息被利用率是后部的 3.5 倍。**更多检索 ≠ 更好生成。**

### 4.2 图谱的真正价值区间
- **图谱明确有效:** 多跳推理（2-3x 提升）、N 元关系（+7.45 F1）
- **图谱可能无效:** 单跳事实（无收益）、已有结构化 KG 时（简单 1 跳就够）
- **图谱适得其反:** 关系抽取噪声可能让性能低于 vanilla RAG（36-54% vs 62.87% 上下文相关性）

### 4.3 成本-任务匹配原则
- 简单任务用简单方法（关键词搜索 → 零成本，90%+ 效果）
- 复杂任务才值得复杂方案（GraphRAG $33K/rebuild 仅在全局综合时值得）
- 从简单方案开始，用数据验证是否需要升级
