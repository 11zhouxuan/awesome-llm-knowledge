# Awesome LLM Knowledge

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

系统整理 LLM 结合外部知识的主要使用范式，涵盖 2020-2026 年的代表性研究工作。

## 目录

- [概述](#概述)
- [范式一：LLM 驱动的知识提炼](#范式一llm-驱动的知识提炼llm-powered-indexing)
  - [图谱构建（二元关系）](#图谱构建二元关系)
  - [N元关系与超图](#n元关系与超图)
  - [本体驱动的知识约束](#本体驱动的知识约束)
  - [层次结构索引](#层次结构索引)
  - [Wiki / 持久化知识格式](#wiki--持久化知识格式)
- [范式二：无向量检索 (Vectorless / Embedding-Free)](#范式二无向量检索-vectorless--embedding-free)
  - [推理式检索](#推理式检索)
  - [关键词/Grep式检索](#关键词grep式检索)
  - [实时全文搜索](#实时全文搜索)
- [范式三：智能体驱动的检索与生成](#范式三智能体驱动的检索与生成)
- [范式四：知识即参数 (Knowledge as Parameters)](#范式四知识即参数-knowledge-as-parameters)
- [范式五：视觉检索（Visual RAG）](#范式五视觉检索visual-rag)
- [工业界知识库产品](#工业界知识库产品enterprise-knowledge-products)
- [数据集与任务类型](#数据集与任务类型)
- [综述与基准](#综述与基准)
- [详细解读](#详细解读)

---

## 概述

传统 RAG（Lewis et al., 2020）确立了"参数记忆 + 非参数记忆"的二元架构。6 年来，这一单一范式演化为多种并存的知识使用方式。本仓库按以下五大范式组织文献：

| 范式 | 核心思想 | 代表工作 |
|------|----------|----------|
| LLM-Powered Indexing | LLM 离线提炼知识为结构化表示 | GraphRAG, LightRAG, HyperGraphRAG, LLM Wiki, OKF |
| Vectorless | 用推理/关键词/实时搜索替代向量 | PageIndex, Sirchmunk |
| Agentic | 检索是多轮决策过程 | GlobalRAG, KnowLP |
| Knowledge as Parameters | 知识预压缩为模型参数 | Doc-to-LoRA, MeMo |
| Visual RAG | 视觉表示替代文本解析 | PixelRAG, MAGE-RAG, MM-BizRAG |

---

## 范式一：LLM 驱动的知识提炼（LLM-Powered Indexing）

用 LLM 在离线阶段对文档进行知识提炼，输出为结构化表示（图谱、超图、Wiki 页面等）。核心是"LLM 作为知识构建者"——不同方法的区别在于输出格式和组织方式。

### 图谱构建（二元关系）

- **[GraphRAG](https://arxiv.org/abs/2404.16130)** (Microsoft, 2024) — 使用 Leiden 算法进行层次化社区检测并生成社区摘要，在全局综合任务上表现优异（72-83% 提升），但索引成本极高（~$33K/次重建）。 [[详细解读]](docs/graphrag_vs_hipporag_vs_pathrag_vs_og_rag.md)

- **[LightRAG](https://arxiv.org/abs/2410.05779)** (北邮/港大, 2024) — 双层检索（Low-level 具体查询 + High-level 抽象查询）结合增量更新机制，单次 API 调用 <100 tokens 完成检索，成本降低数千倍。 [[代码]](https://github.com/HKUDS/LightRAG) [[详细解读]](docs/lightrag.md)

- **[UnWeaver](https://arxiv.org/abs/2603.29875)** (Samsung AI Warsaw, 2026) — 仅抽取实体（不构建完整图谱），以 1/17 的成本达到 GraphRAG 级效果。核心论点：VectorRAG 几乎就够了，图谱复杂度被高估。 [[详细解读]](docs/unweaver.md)

- **[LinearRAG](https://arxiv.org/abs/2510.10114)** (香港理工, 2025) — 用 spaCy NER 替代 LLM 关系抽取，构建三层图（实体-句子-段落）+ Personalized PageRank 检索。索引零 token 消耗，速度降低 77%+，在 2Wiki 上绝对提升 +3.80%。量化证明 GraphRAG 的关系抽取反而引入噪声。 [[代码]](https://github.com/DEEP-PolyU/LinearRAG) [[详细解读]](docs/linearrag.md)

- **[FlowRAG](https://arxiv.org/abs/2606.17856)** (华东师范/上海AI实验室, 2026) — 四层异构图（段落-摘要-句子-实体）+ 双粒度实体激活 + 频率感知加权流。索引比 LightRAG 快 14x，token 消耗仅 2%，平均 GPT-Accuracy 58.89%（+1.72% vs LinearRAG）。 [[详细解读]](docs/flowrag.md)

### N元关系与超图

- **[HyperGraphRAG](https://arxiv.org/abs/2503.21322)** (北邮/NTU, 2025) — 用超边表示 n 元关系，突破二元关系限制。信息论证明二元图对 3+ 实体事实必然有损。F1 平均提升 +7.45，构建成本仅 $0.006/1k tokens。 [[详细解读]](docs/hypergraphrag.md)

- **[OG-RAG](https://arxiv.org/abs/2412.15235)** (Microsoft Research, 2024) — 本体约束的超图表示，事实召回提升 55%，回答正确性提升 40%。需要预定义领域本体，适合强监管领域。 [[详细解读]](docs/og_rag.md)

### 本体驱动的知识约束

- **[OMD-GraphRAG](https://arxiv.org/abs/2603.25152)** (中国联通, 2025) — 本体引导抽取 + 多维聚类 + 双通道融合检索。三个模块各贡献约 3% F1 提升，组合达 +9.21%（vs LightRAG）。证明本体在抽取阶段的杠杆效应最高。 [[详细解读]](docs/omd_graphrag.md)

### 层次结构索引

- **[BookRAG](https://arxiv.org/abs/2512.03413)** (2025) — 保留文档原生层次结构（章节/节/小节）的索引方法，利用结构关系改善检索精度。 [[详细解读]](docs/bookrag.md)

- **[PageIndex](https://pageindex.ai/blog/pageindex-intro)** (2025) — LLM 构建 JSON 层次索引（ID/名称/描述/子节点），保留文档原生结构供推理导航。*（也属于范式二：推理式检索）* [[详细解读]](docs/pageindex.md)

- **[GlobalRAG](https://arxiv.org/abs/2510.26205)** (复旦, 2025) — 提出 GlobalQA 基准（13,000+ QA 对），揭示现有方法在全局聚合任务上仅达 1.51 F1。文档级检索 + 智能过滤 + 符号计算工具实现 6.63 F1（+339%）。 [[详细解读]](docs/globalrag.md)

### Wiki / 持久化知识格式

LLM 提炼的知识不一定要存为图谱——输出为人类可读的 Wiki 页面，实现知识复用、可审计和人机协同编辑。

- **[LLM Wiki](https://github.com/nashsu/llm_wiki)** (2025, 13.1k stars) — 桌面应用，将文档自动转化为互联 Wiki 知识库。两步 CoT 摄入、Louvain 社区检测、增量缓存、图可视化。知识可读、可审计、可人工修正。 [[详细解读]](docs/llm_wiki.md)

- **[Google Open Knowledge Format (OKF)](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing)** (Google Cloud, 2026) — 将 LLM-Wiki 模式标准化为供应商中立的开放规范。Markdown + YAML frontmatter + Git，零基础设施投入，从个人工具走向行业标准。 [[详细解读]](docs/google_okf.md)

---

## 范式二：无向量检索 (Vectorless / Embedding-Free)

拒绝"先嵌入-再检索"的预计算范式，用推理、关键词搜索或结构导航替代向量相似度。

### 推理式检索

- **[PageIndex](https://pageindex.ai/blog/pageindex-intro)** (2025) — LLM 通过 JSON 层次索引逐步推理导航，五步迭代检索。无需向量数据库，理解深层含义并跟踪交叉引用。*（也属于范式一：层次结构索引）* [[详细解读]](docs/pageindex.md)

### 关键词/Grep式检索

- **[Keyword Search is All You Need](https://arxiv.org/abs/2602.23368)** (Amazon Science, 2026) — 智能体关键词搜索（rga/pdfgrep）在无向量数据库条件下达到 RAG 90%+ 性能，FinanceBench 上甚至超越传统 RAG。 [[详细解读]](docs/keyword_search_agentic.md)

- **[Is Grep All You Need?](https://arxiv.org/abs/2605.15184)** (PwC, 2026) — Grep 在 inline 交付模式下一致优于 vector 检索。更换 Agent Harness 的影响 ≈ 更换检索器。证明对精确回忆任务，词法匹配 > 语义匹配。 [[详细解读]](docs/grep_vs_vector.md)

### 实时全文搜索

- **[Sirchmunk](https://modelscope.github.io/sirchmunk-web/zh/blog/technical-deep-dive/)** (ModelScope, 2026) — 开源无嵌入搜索引擎，基于 ripgrep-all（100+ 格式）+ 蒙特卡洛证据采样（探索-利用平衡 + 模拟退火收敛）+ ReAct Agent 容错。FAST 模式 2-5 秒。 [[代码]](https://github.com/modelscope/sirchmunk) [[详细解读]](docs/sirchmunk.md)

---

## 范式三：智能体驱动的检索与生成

检索不是单次操作，而是多轮决策过程。LLM Agent 自主决定何时检索、检索什么、何时停止。

- **[GlobalRAG](https://arxiv.org/abs/2510.26205)** (复旦, 2025) — LLM + 符号计算工具（计数/极值/排序/Top-K）。证明纯 LLM 在全局聚合任务上失败，需要程序化工具辅助。移除聚合工具后 F1 下降 79.2%。 [[详细解读]](docs/globalrag.md)

- **[KnowLP](https://arxiv.org/abs/2506.22303)** (暨南大学/Griffith, 2025) — GraphRAG 用于个性化学习路径推荐。多智能体协作：P-Agent（前置路径）+ S-Agent（相似绕道）+ D-Agent（难度匹配）。在 Junyi 数据集上比次优方法提升 46.7%。 [[详细解读]](docs/knowlp.md)

- **[Is Grep All You Need?](https://arxiv.org/abs/2605.15184)** (PwC, 2026) — 揭示 Agent Harness 设计比检索算法更关键。工具输出的呈现方式（inline vs file）可以完全反转检索策略的优劣。 [[详细解读]](docs/grep_vs_vector.md)

---

## 范式四：知识即参数 (Knowledge as Parameters)

不在运行时检索知识，而是将知识预先压缩为模型参数。介于 RAG 和 Fine-tuning 之间的"第三条路"。

- **[Doc-to-LoRA](https://arxiv.org/abs/2602.15902)** (Sakana AI, ICML 2026) — 超网络单次前向传播生成 LoRA 适配器注入主模型，突破原生窗口 4x+ 仍保持接近完美准确率。推理时零额外延迟，但需要访问模型权重。 [[代码]](https://github.com/SakanaAI/doc-to-lora) [[详细解读]](docs/doc_to_lora.md)

- **[MeMo](https://arxiv.org/abs/2605.15156)** (NUS/东京大学/MIT CSAIL, 2025) — 训练独立的 Memory Model 作为知识载体，主 LLM 完全冻结。支持黑盒 API 模型，噪声鲁棒（±1.77%），多语料可合并（K=10 时 5.5× 计算节省）。 [[详细解读]](docs/memo.md)

---

## 范式五：视觉检索（Visual RAG）

用视觉表示替代文本解析作为知识载体——跳过 HTML/PDF 解析，直接在像素空间中完成嵌入、检索和阅读。改变了知识的存在形态（文本→图像）、嵌入方式（文本Embedding→视觉Embedding）和阅读方式（LLM→VLM）。

- **[PixelRAG](https://arxiv.org/abs/2606.28344)** (UC Berkeley, 2026) — 将网页渲染为截图，用视觉嵌入模型（Qwen3-VL-Embedding + LoRA）在 30M Wikipedia 截图上检索，VLM 直接从图像阅读。比文本 RAG 提升高达 18.1%，即使在纯文本任务（NQ、SimpleQA）上也更好。 [[代码]](https://github.com/StarTrail-org/PixelRAG) [[详细解读]](docs/pixelrag.md)

- **[MAGE-RAG](https://arxiv.org/abs/2606.15906)** (2026) — 长文档多模态 QA 的自适应图证据框架。构建动态证据图（页面节点+元素节点，编码包含/阅读顺序/布局邻接/语义关系），Agent 在预算约束下迭代选择-扩展-过滤证据子图。LongDocURL 52.75%，MMLongBench-Doc 53.26%。 [[代码]](https://github.com/laonuo2004/MAGE-RAG)

- **[MM-BizRAG](https://arxiv.org/abs/2606.04231)** (ACL 2026 Industry) — 企业级多模态 RAG。针对复杂企业文档（报告/PPT/表格）的结构感知管道：方向特定的摄入流水线（竖版用布局解析，横版用整体表示）+ LLM 驱动的工件转换。比纯视觉基线提升高达 32 个百分点。

- **[Multimodal Graph RAG](https://arxiv.org/abs/2606.28780)** (2026) — 结合多模态知识图谱与 RAG 实现长程视觉文档理解。提出 DLVQA 基准用于文档级视觉 QA，解决需要全文档理解的跨页面视觉问答。

---

## 工业界知识库产品（Enterprise Knowledge Products）

各大云厂商与商业公司推出的知识库/上下文智能产品。区别于学术论文，这些条目聚焦**可落地的托管服务与商业化产品**——它们如何为企业 AI Agent 提供受治理、可审计、随使用演化的知识基础设施。

- **[AWS Context](https://aws.amazon.com/cn/blogs/machine-learning/context-intelligence-for-your-data-and-ai-agents-at-scale/)** (AWS, 2026-06, Coming Soon) — AWS Summit NYC 发布的托管知识图谱服务，"automatically maps the relationships across your existing data into a knowledge graph and provides agentic search"。图谱元数据以 Apache Iceberg 格式落地 S3，与 Glue Data Catalog / SageMaker Unified Studio / Lake Formation 打通，由 Amazon Quick 的个人知识图谱扩展为组织级图谱。核心机制：**observes which sources produce correct results, which join paths agents rely on, and which curated rules get applied**，按实际使用为数据源排名；每次调用继承调用者的 IAM & Lake Formation 权限（Identity-aware and governed by default）。 [[详细解读]](docs/aws_context.md)

---

## 数据集与任务类型

不同 RAG 方法适用于不同类型的查询任务。详细整理见 [[完整文档]](docs/datasets_and_tasks.md)。

### 任务类型速查

| 任务类型 | 典型问题 | 最佳方法 | 代表数据集 |
|----------|----------|----------|-----------|
| 单跳事实 | "X公司的CEO是谁？" | 向量RAG / 关键词搜索 | Natural Questions, TriviaQA |
| 多跳推理 | "A的导师在哪家公司工作？" | LightRAG / FlowRAG / HyperGraphRAG | HotpotQA, MuSiQue |
| 全局聚合 | "薪资最高的5位员工？" | GlobalRAG (Doc-level + 符号工具) | GlobalQA (13K+ QA) |
| Schema约束 | "符合指南X的治疗方案？" | OG-RAG / OMD-GraphRAG | Agriculture, Medical |
| N元关系 | "医生X用药物Y治疗患者Z？" | HyperGraphRAG | 5域×512题自建基准 |
| 路径推理 | "从A到B的因果链？" | PathRAG / FlowRAG | HotpotQA, 2Wiki |
| 长文档理解 | "第3.2节关于X的例外？" | BookRAG / PageIndex / Doc-to-LoRA | NarrativeQA, NIAH |
| 精确回忆 | "合同第X条原文？" | Grep / 关键词搜索 | LongMemEval, FinanceBench |
| 时间推理 | "X和Y之间发生了什么？" | 所有方法均弱（最好 65% F1） | MultiHop-RAG temporal |

### 关键发现

- **检索-生成鸿沟:** 检索覆盖 83.5%，LLM 仅利用 47.9%——更多检索 ≠ 更好生成 (AWS+Cisco 2026)
- **图谱有效区间:** 多跳推理（2-3x 提升）和 N 元关系（+7.45 F1）明确有效；单跳事实无收益
- **图谱可能有害:** 关系抽取噪声可使性能低于 vanilla RAG（36-54% vs 62.87% 上下文相关性）

---

## 综述与基准

- **[Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401)** (Facebook AI Research, 2020) — RAG 开山之作。确立"参数记忆 + 非参数记忆"二元架构，626M 参数匹配 T5-11B 性能。后续所有 RAG 变体的起点。 [[详细解读]](docs/rag_original.md)

- **[Retrieval-Augmented Generation with Graphs](https://arxiv.org/abs/2501.00309)** (2025, 18位研究者) — GraphRAG 领域综述，提出统一的分析框架（Query Processor → Retriever → Organizer → Generator → Data Source）。 [[详细解读]](docs/graphrag_survey.md)

- **[GraphRAG vs HippoRAG vs PathRAG vs OG-RAG](https://medium.com/graph-praxis/graphrag-vs-hipporag-vs-pathrag-vs-og-rag-choosing-the-right-architecture-for-your-knowledge-graph-a4745e8b125f)** (2026) — 四种图谱 RAG 架构的实践对比与选型指南。 [[详细解读]](docs/graphrag_comparison.md)

- **[Is GraphRAG Needed?](https://arxiv.org/abs/2606.25656)** (AWS + Cisco, 2026) — 9 种 RAG 场景的标准化评估框架。关键发现：基础 RAG + 1跳关系即可匹配 Agentic RAG；检索覆盖 83.5% 但 LLM 仅利用 47.9%（检索-生成鸿沟）；上下文优化可节省 19%-53% token。 [[详细解读]](docs/is_graphrag_needed.md)

- **[Do We Still Need GraphRAG? (RAGSearch)](https://arxiv.org/abs/2604.09666)** (NYU Shanghai, 2025) — 统一基准对比 Dense RAG vs 5种 GraphRAG × 4种 Agent 推理范式。最关键发现：**通用 QA 上 GraphRAG 无优势（差距仅 +0.47），多跳 QA 上 GraphRAG 大幅领先（+27.23）**。Agent 迭代搜索可缩小约 1/3 差距但无法替代图结构。HippoRAG2 在多跳基准上最强且最稳定。 [[详细解读]](docs/ragsearch_benchmark.md)

---

## 详细解读

每篇文献的详细研究报告在 [docs/](docs/) 目录下，包含方法细节、实验数据和核心洞察。

---

## 贡献

欢迎提交 PR 补充相关文献。请遵循现有格式：
- README 中添加一行简介（含论文链接、机构、年份、核心贡献）
- docs/ 中添加详细解读文件

## License

MIT
