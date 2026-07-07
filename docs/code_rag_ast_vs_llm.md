# 代码 RAG：AST 确定性图谱 vs LLM 抽取图谱

> 论文: [Reliable Graph-RAG for Codebases: AST-Derived Graphs vs LLM-Extracted Knowledge Graphs](https://arxiv.org/html/2601.08773v1)
> 作者: Manideep Reddy Chinthareddy
> 日期: 2026-01
> 代码: https://github.com/Manideep-Reddy-Chinthareddy/graph-based-rag-ast-vs-llm

## 1. 研究动机

GraphRAG 在自然语言文档上效果显著，但代码有一个本质区别：**代码有确定性的结构（AST）可以直接解析，不需要 LLM 来"理解"依赖关系**。那么对代码仓库做 RAG 时：
- 需要用 LLM 抽取知识图谱吗？
- 还是直接用 AST 解析就够了？

## 2. 三种方案对比

| 方案 | 索引方式 | 图谱来源 |
|------|----------|----------|
| No-Graph | 向量相似度检索（chunk-based） | 无图谱 |
| LLM-KB | LLM 逐文件抽取依赖关系生成 JSON | LLM 生成 |
| DKB | Tree-sitter 解析 AST，确定性提取类型和边 | AST 确定性派生 |

### 2.1 DKB（Deterministic Knowledge Base）

- **Tree-sitter** 解析 Java AST
- 两遍扫描：(1) 类型发现 (2) 边插入
- 类型化边：`injects`, `extends`, `implements`
- 查询时：双向遍历 + 接口消费者扩展

### 2.2 LLM-KB

- 逐文件 LLM prompting + 批处理
- 文件截断（15,000 字符）适配上下文窗口
- 输出结构化 JSON：类名 + `depends_on` 关系
- 存在文件跳过问题（stochastic extraction failures）

## 3. 实验结果

### 3.1 索引时间

| 代码仓库 | No-Graph | DKB | LLM-KB |
|----------|----------|-----|--------|
| Shopizer | 18.41s | 22.09s | **215.09s** |
| ThingsBoard | 123.61s | 143.55s | **979.25s** |
| OpenMRS Core | 42.20s | 48.42s | **251.29s** |

DKB 仅比 No-Graph 增加约 20% 时间；LLM-KB 慢 10-12 倍。

### 3.2 成本（归一化到 No-Graph）

| 工作负载 | No-Graph | DKB | LLM-KB |
|----------|----------|-----|--------|
| Shopizer | 1.00× | 2.25× | **19.75×** |
| Multi-repo | 1.00× | 2.13× | **45.64×** |

LLM-KB 在大规模场景下成本爆炸（45x）。

### 3.3 索引覆盖率（Shopizer）

| 指标 | No-Graph | DKB | LLM-KB |
|------|----------|-----|--------|
| Chunks 嵌入 | 5,403 | 4,873 (0.902) | 3,465 (**0.641**) |
| 图节点数 | — | 1,158 | 842 (0.727) |
| 文件成功率 | — | ~100% | **0.688** (377文件被跳过) |

**关键发现：LLM-KB 跳过了 31% 的文件**，产生不可预测的盲区。

### 3.4 正确性（45 题跨 3 个仓库）

| 方案 | 正确 | 部分正确 | 错误 |
|------|------|----------|------|
| No-Graph | 31 | 9 | 5 |
| LLM-KB | 38 | 5 | 2 |
| **DKB** | **43** | **2** | **0** |

DKB 在所有仓库上正确率最高，且**零错误**。

### 3.5 查询延迟

| 仓库 | No-Graph | DKB | LLM-KB |
|------|----------|-----|--------|
| Shopizer | 9.52±2.98s | 10.51±4.17s | 13.36±7.87s |
| ThingsBoard | 10.92±1.43s | 11.17±1.97s | 15.29±4.94s |

DKB 查询延迟与 No-Graph 接近；LLM-KB 更慢且方差更大。

## 4. 关键技术细节

### 4.1 接口消费者扩展（Interface-Consumer Expansion）

DKB 的独特优势：当检索到一个实现了某接口的类时，自动扩展检索该接口的所有消费者。这对理解"Controller 调用了哪些 Service"这类架构问题至关重要。

### 4.2 LLM-KB 的失败模式

- 批处理截断导致文件被跳过
- 结构化输出要求导致抽取失败
- "不可预测的盲区"——不知道哪些文件会被遗漏
- 越大的仓库跳过率越高

## 5. 主要贡献

1. **首个代码 RAG 的图谱方案对比基准**（向量 vs LLM图谱 vs AST图谱）
2. **量化了 LLM 图谱构建的不可靠性**（31% 文件跳过率）
3. **证明 AST 确定性图谱在代码场景全面优于 LLM 图谱**（更快、更便宜、更完整、更准确）
4. **建议未来方向：混合图谱**（确定性 AST 骨架 + LLM 语义边增强）

## 6. 核心洞察

### 6.1 代码是"有确定性结构"的知识

自然语言文档没有确定性结构，必须靠 LLM "理解"后抽取。但代码天然有 AST、类型系统、import 关系——这些是**确定性的、完整的、零成本可解析的**。在这个场景下 LLM 图谱抽取不仅不必要，反而引入了不可靠性。

### 6.2 与自然语言 GraphRAG 的对比

| 维度 | 自然语言文档 | 代码仓库 |
|------|-------------|----------|
| 结构 | 无确定性结构 | AST/类型系统/import 是确定性的 |
| LLM 抽取 | 必须用（唯一选择） | 不必要（AST 更好） |
| 覆盖率 | LLM 可能遗漏 | AST 100% 覆盖 |
| 成本 | LLM 抽取有成本 | AST 解析几乎零成本 |
| 最佳方案 | LightRAG/FlowRAG 等 | **DKB (AST 确定性图谱)** |

### 6.3 实践启示

- 代码仓库的 RAG：**优先用 AST 解析构建图谱**，不要用 LLM 抽取
- 可以在 AST 骨架上叠加 LLM 语义边（如"这个类的设计意图是什么"）作为增强
- Tree-sitter 支持 40+ 语言，是通用的解析方案

---

# Graphify: 代码知识图谱生成工具

> 项目: [Graphify-Labs/graphify](https://github.com/Graphify-Labs/graphify)
> 类型: 开源工具（AI 编程助手 skill）
> 集成: Claude Code, Cursor, Gemini CLI, GitHub Copilot 等 20+ 平台
> 安装: `uv tool install graphifyy`

## 1. 产品定位

Graphify 将代码仓库、文档和媒体文件转化为**可查询的知识图谱**。支持 36+ 编程语言，核心代码解析完全本地化（零 API 调用），可选的语义增强使用 LLM。

## 2. 技术架构

### 2.1 代码处理（本地，无 LLM）

- **Tree-sitter AST 解析**，覆盖约 40 种语言
- 识别跨文件关系：imports, calls, inheritance
- 零 API 调用，完全离线处理

### 2.2 语义增强（可选 API）

- 文档、PDF、图片使用 LLM 进行语义关联
- 每条边标记为 `EXTRACTED`（源码中显式存在）或 `INFERRED`（推导得出）
- 内置置信度评分

### 2.3 社区检测

- **Leiden 算法**聚类相关概念为子系统
- 无需 LLM 的分区；可选语义标签

## 3. 核心能力

| 能力 | 说明 |
|------|------|
| 三种输出 | `graph.html`（交互可视化）, `GRAPH_REPORT.md`（关键发现）, `graph.json`（可查询数据） |
| 查询接口 | `graphify query`, `graphify path`, `graphify explain` |
| God Nodes | 识别最高连接度的概念（架构核心） |
| 设计意图提取 | 注释和设计文档作为一等节点 |
| 隐私优先 | 代码留本地；语义 pass 仅在配置时调用 API |

## 4. 性能数据

在 LOCOMO 数据集 (n=300) 上：

| 指标 | Graphify | mem0 | supermemory |
|------|----------|------|-------------|
| Recall@10 | **0.497** | 0.048 | 0.149 |
| QA 准确率 | 45.3% | 27.3% | 49.7% |
| LLM 成本 | **零**（AST 阶段） | 有 | 有 |

## 5. 支持范围

- 36+ 编程语言（Tree-sitter）
- 文档：.md, .pdf, .docx, .xlsx
- 数据：YAML, JSON, SQL, Terraform HCL
- 媒体：.mp4, .mp3, 图片
- 平台特定：Salesforce Apex

## 6. 与论文的互补关系

Graphify 是论文 DKB 方案的**产品化实现**——都基于 Tree-sitter AST 确定性解析，不依赖 LLM 构建代码图谱。但 Graphify 更进一步：
- 支持 40 种语言（论文仅 Java）
- 加入了可选的语义增强层（论文建议的"混合图谱"方向）
- 提供了交互可视化和查询接口
- 集成了 20+ AI 编码助手平台

## 7. 核心洞察

Graphify 验证了论文的核心结论在产品层面的可行性：**代码的知识图谱应该以 AST 确定性解析为骨架，LLM 语义增强为可选层**。这种"确定性底座 + 可选智能"的设计比纯 LLM 抽取更可靠、更快、更便宜，且天然隐私友好（代码不出本地）。
