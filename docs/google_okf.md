# Google Open Knowledge Format (OKF): AI 知识交换的开放标准

> 来源: [Introducing the Open Knowledge Format](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing)
> 作者: Sam McVeety (Tech Lead, Data Analytics), Amir Hormati (Tech Lead, BigQuery)
> 机构: Google Cloud
> 发布日期: 2026-06-12

## 1. 背景与动机

### 1.1 问题：知识碎片化

组织的内部知识分散在多个系统中：
- 元数据目录（专有 API）
- Wiki 和共享文档
- 代码注释和文档字符串
- 个人工程师的"脑中知识"

这种碎片化导致：
- 知识难以共享和复用
- AI 系统无法统一访问组织知识
- 系统迁移时知识流失
- 重复的文档化工作

### 1.2 解决方案：标准化 LLM-Wiki 模式

Open Knowledge Format (OKF) 将 LLM-Wiki 模式形式化为一个**供应商中立、可互操作的开放标准**，用于表示 AI 系统所需的元数据、上下文和策划知识。

## 2. OKF v0.1 规范

### 2.1 核心设计：极简主义

| 原则 | 实现 |
|------|------|
| Just Markdown | 任何编辑器可读，GitHub 可渲染，搜索工具可索引 |
| Just Files | 可作为 tarball 分发，可存在 Git 仓库，可挂载文件系统 |
| Just YAML Frontmatter | 可查询字段: type, title, description, resource, tags, timestamp |

**没有：** 复杂压缩、运行时依赖、必需的 SDK。

### 2.2 目录结构

```
sales/
├── index.md           # 层次导航
├── datasets/
│   └── orders_db.md   # 数据集描述
├── tables/
│   ├── orders.md      # 表定义
│   └── customers.md
└── metrics/
    └── weekly_active_users.md  # 业务指标
```

### 2.3 文档格式

```markdown
---
type: table
title: Orders
description: Core transaction table
resource: bigquery://project.dataset.orders
tags: [commerce, transactions]
timestamp: 2026-06-10T14:30:00Z
---

## Schema
| Column | Type | Description |
|--------|------|-------------|
| order_id | STRING | Primary key |
| customer_id | STRING | FK to [customers](./customers.md) |

## Business Context
This table captures all completed transactions...
```

### 2.4 关键设计决策

| 设计 | 意义 |
|------|------|
| Markdown links = 概念关系 | 无需专用关系数据库 |
| 可选 index.md | 提供层次导航 |
| 可选 log.md | 时间序列历史 |
| 仅 "type" 字段必需 | 最低准入门槛 |

## 3. 三大设计原则

### 3.1 最小化约束（Minimally Opinionated）

- 仅要求 "type" 字段
- 其余所有字段由生产者自定义
- 不强制特定的知识组织方式

### 3.2 生产者/消费者独立（Producer/Consumer Independence）

- 知识创建与消费完全分离
- 不需要了解消费者的实现细节即可生产知识
- 不需要了解生产者的系统即可消费知识

### 3.3 格式而非平台（Format, not Platform）

- 供应商中立
- 不绑定特定云、数据库或框架
- 可在任何系统间迁移

## 4. 参考实现

Google 提供了三个参考实现：

| 实现 | 功能 |
|------|------|
| Enrichment Agent | 遍历 BigQuery 数据集，生成 OKF 文档，丰富引用和 schema |
| Static HTML Visualizer | 将 OKF 包转换为交互式图可视化 |
| 示例包（3个） | GA4 电商、Stack Overflow、Bitcoin 数据集 |

## 5. 使用场景

| 场景 | 说明 |
|------|------|
| Schema 文档 | 数据表结构和字段含义 |
| 业务指标定义 | KPI 的精确定义和计算方式 |
| 事件运行手册 | 故障响应的标准操作流程 |
| 系统关联路径 | 表之间的 Join 关系 |
| API 废弃通知 | 版本变化和迁移指南 |
| Agent 驱动的知识管理 | AI 智能体自动维护和交叉引用 |
| 跨组织知识共享 | 供应商-客户之间的知识传递 |

## 6. 生态系统集成

- Google Cloud Knowledge Catalog 已更新支持 OKF 格式摄入
- GitHub 仓库提供完整规范、工具和示例
- 社区开放参与

## 7. 与 LLM Wiki 模式的关系

OKF 是 LLM Wiki 模式的**标准化层**：

```
LLM Wiki 概念 (Karpathy)
    ↓ 具体实现
nashsu/llm_wiki (桌面应用)
    ↓ 标准化
Google OKF (开放规范)
    ↓ 企业采纳
Google Cloud Knowledge Catalog, 其他厂商...
```

## 8. 核心洞察

Google OKF 的发布标志着 LLM-Wiki 模式从**个人工具走向行业标准**：

### 8.1 为什么需要标准化？

1. **可互操作性:** 不同 LLM 系统、不同工具产生的知识可以无缝集成。没有标准时，每个系统都是信息孤岛。

2. **可迁移性:** 知识跟随组织，不被锁定在特定供应商。Markdown + Git 是最长寿的技术组合之一。

3. **可审计性:** 人类可读格式使知识审计和合规检查成为可能。对比向量数据库——你无法"审计"一个嵌入向量。

4. **低摩擦采纳:** "Just files" 的设计意味着零基础设施投入。任何团队都可以从一个 Git 仓库开始。

### 8.2 战略意义

- 从 RAG 的"运行时知识"转向"编译时知识"
- 知识成为可版本控制、可 diff、可 review 的一等公民
- 为 AI Agent 生态提供统一的知识接口
- 可能成为"企业知识交换"的事实标准

### 8.3 与 RAG 的关系

OKF 不是替代 RAG，而是为 RAG 提供**更好的知识源**：
- OKF 文档本身可以作为 RAG 的检索对象
- 但其结构化特性（type, links, hierarchy）为检索提供了额外信号
- 长期看，OKF 可能成为 AI 系统间知识共享的通用协议

### 8.4 对企业 AI 架构的启示

1. **知识基础设施的标准化** 优先级应高于 RAG 系统的优化
2. **Markdown + Git** 是知识管理的最佳载体（简单、持久、通用）
3. **AI 生产知识，人类审核知识** 的协作模式正在成型
4. **供应商中立** 是企业知识策略的核心要求
