# AWS Context: 企业级 AI Agent 的"大规模上下文智能"服务

> 来源: [Context intelligence for your data and AI agents at scale](https://aws.amazon.com/cn/blogs/machine-learning/context-intelligence-for-your-data-and-ai-agents-at-scale/)
> 作者: Mai-Lan Tomsen Bukovec (VP, Technology, AWS)
> 机构: AWS
> 发布日期: 2026-06-17（AWS Summit New York City）
> 状态: **Coming Soon**（发布公告，产品尚未 GA）
> 类型: 托管云服务（非论文）

## 1. 官方定位

> "AWS Context, a new service that **automatically maps the relationships across your existing data into a knowledge graph** and provides **agentic search**."

AWS Context 是一项面向企业 AI Agent 的**托管知识图谱服务**，把散布在数据湖、数仓、lakehouse、数据库、流数据中的关系、业务规则与领域知识汇聚为一张**组织级动态图谱**，通过 agentic search APIs 与 MCP tools 供 Agent 在运行时检索。

由 **Amazon Quick 背后的个人知识图谱技术**扩展而来——"extending what was a personal knowledge graph into an organizational one"。

## 2. 官方核心能力

| 能力 | 官方措辞 / 说明 |
|------|------------------|
| 图谱自动构建 | 自动映射跨数据源的关系，形成 knowledge graph |
| Agentic Search | 为 Agent 提供 agentic search APIs 与 MCP tools |
| 元数据存储 | 关键元数据以 **Apache Iceberg 格式**发布至 **Amazon S3** |
| 数据管理员审查 | 通过控制台审查、修正、提升推断关系 |
| 零基础设施 | 无需预置基础设施或构建检索管道 |
| 与 Bedrock AgentCore / EKS 兼容 | Agent 可基于 Bedrock AgentCore 或 EKS 构建 |

## 3. 关键机制（两个官方章节标题）

### 3.1 "Context that learns from how your agents work"（自学习机制）

> "it **observes which sources produce correct results, which join paths agents rely on, and which curated rules get applied**"
>
> "It **ranks sources by actual usage** and shares what it learns across your organization."

AWS Context 从 Agent 与人类专家的实际使用中累积三类信号：
1. **哪些数据源产出了正确结果**
2. **Agent 依赖了哪些 join paths**
3. **哪些人工策划的规则被采用**

再据此**按实际使用为数据源排名**，形成"越用越准"的图谱。这是它与传统静态元数据目录（Glue Data Catalog、DataHub）本质不同的地方——**使用行为本身是训练信号**。

### 3.2 "Identity-aware and governed by default"（权限透传）

> "Each call is **designed to inherit the calling user's IAM and Lake Formation permissions**."

- 每次 Agent 对 AWS Context 的调用都以**调用者身份**执行，天然继承其 IAM 与 Lake Formation 权限（含行/列级）
- 每次交互均可审计（auditable）
- 从设计上避免"权限越界的检索命中"污染 Agent 上下文——这是企业合规场景（金融、医疗、政府）的硬性要求

## 4. 生态集成关系

> "AWS Glue Data Catalog, Amazon SageMaker Unified Studio, and AWS Lake Formation **integrate with** the knowledge graph."

AWS Context 不替代 AWS 现有数据栈，而是作为**跨栈的治理与知识层**：

```
        ┌───────────────────────────────────────┐
        │           AWS Context                 │
        │  (organizational knowledge graph +    │
        │   agentic search APIs + MCP tools)    │
        └───┬─────────────┬──────────────┬──────┘
            │             │              │
            ▼             ▼              ▼
   Glue Data Catalog  SageMaker    Lake Formation
   (schemas)          Unified      (permissions)
                      Studio
            ▲             ▲              ▲
            │             │              │
        ┌───┴─────────────┴──────────────┴───┐
        │  Data Lakes / Warehouses /         │
        │  Lakehouses / Databases / Streams  │
        └────────────────────────────────────┘

  Consumers: Bedrock AgentCore agents · EKS 上自建 Agent · Amazon Quick
```

同期发布的配套能力：
- **AWS Glue Data Catalog Business Context and Semantic Search** (Preview)
- **Amazon S3 Annotations** (GA)

## 5. 定位：工业界知识库产品

本仓库将 AWS Context 单列在 **"工业界知识库产品"** 章节，与学术论文分开。与范式五（持久化知识构建）中的 LLM Wiki、Google OKF 相比，AWS Context 属于同一价值链的不同层：

| 条目 | 归属章节 | 层级 | 载体 | 目标用户 | 学习/演化机制 |
|------|----------|------|------|----------|--------------|
| **LLM Wiki** | 范式五 | 个人级 | 桌面应用（本地 Markdown + 图库） | 个人 / 小团队 | 人工编辑 + LLM 摄入 |
| **Google OKF** | 范式五 | 标准级 | Markdown + YAML + Git（开放规范） | 跨组织、跨供应商 | 生产者/消费者独立，格式驱动 |
| **AWS Context** | 工业界产品 | 企业级 | 托管云服务（图谱 + 权限层 + 反馈闭环） | 企业 Bedrock Agent / EKS Agent | 观察使用行为，按实际使用排名数据源 |

两点关键差异：
1. **AWS Context 是唯一自带"反馈学习闭环"的一档**——LLM Wiki 与 OKF 都是"人写机器读"的静态知识载体，而 AWS Context 会因使用而自我强化
2. **AWS Context 是商业托管服务而非可复现方法**——因此归入独立的"工业界知识库产品"章节，该章节未来将容纳其他云厂商 / 商业公司的同类产品

## 6. 典型使用场景（官方）

- 为 AI Agent 提供跨 data lakes / warehouses / lakehouses / databases / streams 的**可信上下文**
- 跨系统关系、业务规则、领域知识的**共享治理层**
- 运行时供组织内 Agent 与应用检索**受治理的数据关系**

## 7. 局限与开放问题

- **供应商绑定**：与 AWS 数据生态深度耦合，跨云组织需评估锁定成本；OKF 是相对的开放标准解法
- **冷启动**：自学习图谱需要一定量的成功查询累积，早期收益不如稳态
- **"成功"的定义**：博客未详述如何判定"which sources produce correct results"——依赖显式反馈、结果被采用、还是无追问信号？会直接影响图谱质量
- **端到端准确性仍受限于 Agent 生成能力**：AWS Context 提供"路径与规则"，最终 SQL 与洞察仍由 Bedrock / 自建 LLM 生成

## 8. 核心洞察

1. **Agent 的瓶颈从"生成 SQL"上移到"选择正确的表和 join path"**：AWS Context 押注的正是这个更上游的问题
2. **知识基础设施的商业化临界点**：LLM Wiki（免费桌面工具）→ OKF（开放规范）→ AWS Context（付费托管服务），持久化知识层正在形成完整市场
3. **"使用即训练"的图谱**：与静态元数据目录（Glue Data Catalog、DataHub）的本质区别，是把 Agent 与人类的日常查询变成图谱演化的燃料
4. **权限即上下文**：企业 RAG 的下一个战场不是检索精度，而是**在权限边界内推理**的能力（Identity-aware by default）
5. **Iceberg-on-S3 作为知识底座**：AWS Context 把图谱元数据固化为 Iceberg 表——知识本身也成了可版本化、可查询、可对接 lakehouse 生态的一等数据资产

## 9. 与本仓库其他范式的关联

- **范式一（LLM 索引构建）**：AWS Context 内部的图谱构建技术可能借鉴 GraphRAG / LightRAG 类方法，但作为托管服务不发表构建算法本身
- **范式三（Agentic RAG）**：Bedrock AgentCore 上的 Agent 是消费方，AWS Context 是被消费方。参考 [Is Grep All You Need?](grep_vs_vector.md) 的结论："Agent Harness 与检索器同等重要"——好的知识层能让相同 Agent 表现大幅提升
- **[Is GraphRAG Needed?](is_graphrag_needed.md)** 指出"检索-生成鸿沟"：检索覆盖 83.5% 但 LLM 仅利用 47.9%。AWS Context 按"actual usage"排名数据源，本质是在缩小这个鸿沟——不给 Agent 更多信息，而是给它**更精准的一条路径**