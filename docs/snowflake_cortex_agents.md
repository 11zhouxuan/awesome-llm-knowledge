# Snowflake (CoWork / Cortex Agents)：数据重力与计算下沉

> 机构: Snowflake
> 产品: Snowflake CoWork（内置 Cortex Agents）
> 类型: 托管云服务（非论文）

## 1. 官方定位

Snowflake 的方案主打**极高的数据重力（Data Gravity）与计算下沉**。核心逻辑：既然企业最核心、最干净的结构化业务数据都在数仓里，那就让 AI 助理**直接进入数仓内部工作**。

## 2. 技术架构与数据联动

- **摒弃传统 RAG 的导出-向量化-外部向量库流程**：不需要把数据导出、向量化再存入外部向量库。
- **数据存储层内运行 AI 计算**：通过 Snowflake CoWork（内置 Cortex Agents）平台，直接在数据层（Data Layer）内运行 AI 计算。
- **混合搜索（Hybrid Search）**：无缝结合高维向量检索与数仓原生的结构化 SQL 查询。
- **深度研究（Deep Research）工作流**：允许一个主代理调度多个专项领域子代理，并行处理庞大的财务表与 PDF 年度报告。

## 3. 安全治理与推理特性

- **受信任的边界（Trusted Boundary）内进行一切计算**：数据从未离开 Snowflake 的安全控制网，无需担心数据外泄或被用于公共模型训练。
- **智能路由（Agent Router）**：根据用户自然语言，精准将任务分发给对应数仓节点，提供高并发、低延迟、绝对安全的数据分析体验。

## 4. 适用场景

核心业务数据（交易、财务、ERP 报表）大量托管在 Snowflake 中，希望 AI 代替人工执行**高强度数据分析与自动化报表生成**的企业。

## 5. 定位对比

Snowflake 与 AWS Context 都强调"数据不出安全边界"，但 Snowflake 把计算下沉到数仓内部、以结构化 SQL + 向量的混合搜索为核心；AWS Context 则跨数据湖/数仓/流数据构建组织级知识图谱。参见 [AWS Context](aws_context.md)。
