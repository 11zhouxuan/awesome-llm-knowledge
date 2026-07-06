# Atlan (Context Graph)：跨云中立的上下文总线

> 机构: Atlan（数据治理领域独角兽）
> 产品: Atlan Context Graph
> 类型: 托管服务 / 数据治理平台（非论文）

## 1. 官方定位

与云厂商巨头不同，Atlan 提供的是一套**跨云跨平台的、中立的上下文总线（Context Layer）**。它不绑定任何单一云生态，定位为多云企业的"数据治理大脑"。

## 2. 技术架构与数据联动

- **只管元数据，不存原始数据**：Atlan 的 **Context Graph（上下文图谱）** 不存储用户原始数据，只收集和管理"关于数据的属性和故事"，即**活跃元数据（Active Metadata）**。
- **图谱内容**：不仅包含传统的数据库血缘（Lineage），还深度整合：
  - **语义字典（Business Glossary）**——定义"利润"到底怎么算
  - **数据所有权**——谁是这张表的专家
  - **日常使用规范**——哪些是废弃表
  - **权限策略**

## 3. 安全治理与推理特性

- **业内率先原生支持 MCP（Model Context Protocol，模型上下文协议）**。
- **标准化接口**：无论前端使用 Anthropic Claude、OpenAI 还是 LangChain 框架搭建的 AI 代理，都可以在发起查询前，先通过 MCP 协议向 Atlan 的 Context Graph 调取"长期记忆与治理规则"。
- 相当于一个 AI 专属的"数据法律指南"，规范代理的每一次查询行为。

## 4. 适用场景

多云（Multi-cloud）或混合云企业，数据分散在 AWS、Azure、Databricks 等多个不同阵营的平台中，需要一个**不被任何巨头锁定的、统一的企业级 AI 数据治理大脑**。

## 5. 定位对比

Atlan 与 [Google OKF](google_okf.md) 都主打"供应商中立"，但 Atlan 是托管的活跃元数据治理平台 + MCP 接口，OKF 是开放的知识文件格式规范。相较 [AWS Context](aws_context.md) 的单云深度整合，Atlan 押注跨云中立的治理层。
