# Do We Still Need GraphRAG? 智能体搜索系统中 RAG 与 GraphRAG 的基准评估

> 论文: [Do We Still Need GraphRAG? Benchmarking RAG and GraphRAG for Agentic Search Systems](https://arxiv.org/html/2604.09666v1)
> 作者: Dongzhe Fan, Zheyi Xue, Siyuan Liu, Qiaoyu Tan
> 机构: 纽约大学上海分校 (NYU Shanghai)
> 日期: 2025-04

## 1. 研究动机

核心问题：**当搜索系统加入 Agent 能力后（迭代检索、推理引导），Graph 结构是否还有必要？Agent 的隐式结构组织能否替代 GraphRAG 的显式图构建？**

这是对"Is GraphRAG Needed?"（AWS+Cisco）的互补研究——后者关注已有 KG 的场景，本文关注**从头构建**的场景，并加入了 RL-based Agent 的维度。

## 2. RAGSearch 基准框架

### 2.1 统一评估设计

| 设计要素 | 说明 |
|----------|------|
| 检索预算匹配 | Dense RAG 和 GraphRAG 使用相同的检索预算 |
| LLM backbone 一致 | 所有方法使用相同的 LLM |
| 多维度指标 | 准确率 + 预处理成本 + 推理效率 + 稳定性 |

### 2.2 测试的检索方法（6种）

| 方法 | 类型 |
|------|------|
| Dense RAG | 向量检索基线 |
| GraphRAG | 微软社区摘要方案 |
| RAPTOR | 层次聚类+摘要 |
| HippoRAG2 | 实体中心 + PageRank |
| HyperGraphRAG | N元关系超图 |
| LinearRAG | spaCy NER + 三层图 |

### 2.3 测试的 Agent 推理范式（4种）

| 方法 | 类型 | 说明 |
|------|------|------|
| Search-o1 | Training-free | 无训练的搜索推理 |
| GraphSearch | Training-free | 图增强的搜索 |
| Search-R1 | RL-based | 强化学习训练的搜索 Agent |
| Graph-R1 | RL-based | RL + 图结构的搜索 Agent |

### 2.4 评估数据集（6个）

| 类型 | 数据集 |
|------|--------|
| 通用 QA | Natural Questions (NQ), PopQA, TriviaQA |
| 多跳 QA | HotpotQA, MuSiQue, 2WikiMultiHopQA |

## 3. 核心实验结果

### 3.1 最关键的发现：通用 vs 多跳的分水岭

| 任务类型 | Dense RAG vs GraphRAG 差距 | 结论 |
|----------|--------------------------|------|
| **通用 QA** | +0.47（Dense 略优） | **GraphRAG 不需要** |
| **多跳 QA** | **+27.23（GraphRAG 大幅领先）** | **GraphRAG 明确需要** |

**这是目前最清晰的量化答案**：GraphRAG 在通用 QA 上无优势甚至略差，但在多跳推理上带来 **+27.23** 的巨大提升。

### 3.2 Agent 能否弥补差距？

| Agent 方式 | 对 Dense-Graph 差距的影响 |
|-----------|--------------------------|
| Training-free (GraphSearch) | 缩小差距 **32.3%** |
| RL-based (Search-R1/Graph-R1) | 进一步提升，但设计良好的 training-free 仍有竞争力 |

**结论：** Agent 的迭代搜索能部分弥补（~1/3），但**不能消除** GraphRAG 在多跳任务上的优势。

### 3.3 稳定性分析

| 维度 | GraphRAG | Dense RAG |
|------|----------|-----------|
| 检索召回率 | 更高 | 较低 |
| 性能方差 | **更低（更稳定）** | 更高 |
| 最佳方法 | HippoRAG2（实体中心） | — |

GraphRAG 不仅准确率更高（多跳），而且**更稳定**——生产系统中这一点很重要。

## 4. 各方法表现

### 4.1 GraphRAG 方法排名（多跳任务）

| 排名 | 方法 | 特点 |
|------|------|------|
| 1 | **HippoRAG2** | 实体中心，跨多跳基准最强 |
| 2 | LinearRAG | 零 token 索引，性价比高 |
| 3 | HyperGraphRAG | N元关系，特定场景有优势 |
| 4 | GraphRAG | 社区摘要，全局查询擅长 |
| 5 | RAPTOR | 层次聚类 |

### 4.2 Agent 范式对比

- RL-based（Search-R1/Graph-R1）总体更好
- 但 training-free（GraphSearch）设计良好时仍有竞争力
- 工程成本差异大：RL 训练需要大量资源

## 5. 主要贡献

1. **RAGSearch 统一基准：** 首次在匹配预算和一致 backbone 下公平对比 Dense RAG vs 5种 GraphRAG
2. **Agent 维度引入：** 不仅对比检索后端，还对比推理范式（training-free vs RL-based）
3. **任务类型分水岭量化：** 明确给出通用 QA（不需要 Graph）vs 多跳 QA（需要 Graph）的量化证据
4. **稳定性指标：** 首次系统评估各方法的性能方差

## 6. 核心洞察

### 6.1 "何时需要 GraphRAG"的最清晰回答

综合本文和 AWS+Cisco 的论文，可以给出**决定性结论**：

```
你的查询是多跳推理吗？
├── 否（单跳/通用事实）→ Dense RAG 就够（GraphRAG 无收益甚至略差）
└── 是（需要跨文档推理链）→ GraphRAG 明确有效（+27.23 差距）
    ├── 加 Agent 能缩小约 1/3 差距但不能替代
    └── HippoRAG2 是当前最佳选择
```

### 6.2 Agent 是补充而非替代

Agent 的迭代搜索提供了"隐式结构组织"——通过多轮推理发现实体间的关联。但这只能弥补 ~32% 的差距，说明**显式图结构仍然有不可替代的价值**。

类比：Agent 像是"临时搭建脚手架"，GraphRAG 像是"预建的桥梁"。脚手架灵活但不稳固，桥梁固定但可靠。

### 6.3 与其他评估工作的互补

| 论文 | 核心问题 | 本文的补充 |
|------|----------|-----------|
| Is GraphRAG Needed? (AWS) | 已有 KG 时需要复杂方案吗？ | 从头构建时需要图吗？ |
| UnWeaver | 图谱复杂度必要吗？ | Agent 能替代图吗？ |
| **本文 (RAGSearch)** | **Agent + Graph 如何交互？** | **量化了任务类型分水岭** |

### 6.4 对生产系统的指导

1. **先分析查询分布：** 如果 80% 是单跳查询 → Dense RAG 足够
2. **多跳占比高时投资 Graph：** 回报明确（+27.23）
3. **Agent 作为增强而非替代：** 加 Agent 可以让 Dense RAG 缩小 1/3 差距，但不能替代 Graph
4. **首选 HippoRAG2：** 在多跳基准上最强且最稳定
5. **考虑成本：** RL-based Agent 训练成本高，training-free 方案（如 GraphSearch）可能是更务实的起点
