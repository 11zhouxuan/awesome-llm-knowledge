# Microsoft (Fabric + Purview + GraphRAG)：全家桶级智能治理体系

> 机构: Microsoft
> 产品: Microsoft Fabric (OneLake) + Microsoft Purview + GraphRAG
> 类型: 托管云服务（非论文）
> 关联论文: [GraphRAG](graphrag_vs_hipporag_vs_pathrag_vs_og_rag.md)（Microsoft Research, 2024）

## 1. 官方定位

微软的方案是典型的"全家桶级智能治理体系"，核心是将企业的**数据湖仓、安全合规屏障与知识图谱**三位一体融合——把 Microsoft Research 的 GraphRAG 技术嵌入 Fabric 数据底座，并以 Purview 提供原生级安全治理。

## 2. 技术架构与数据联动

- **单一数据源（Single Source of Truth）**：以 Microsoft Fabric (OneLake) 作为统一数据底座，企业数据无需搬迁，通过**快捷方式（Shortcuts）**引入。
- **GraphRAG（图检索增强生成）**：不同于传统的文本切块 + 向量相似度匹配，GraphRAG 先用大模型对全量数据做预处理，抽取**实体（人、组织、概念）与关系**，构建全局"语义网络"。参见 [GraphRAG 详细解读](graphrag_vs_hipporag_vs_pathrag_vs_og_rag.md)。

## 3. 安全治理与推理特性

- **DSPM for AI（Purview 的原生数据安全态势管理）** 是杀手锏。
- **多跳推理下的运行时权限审计**：当 AI 代理在图谱中做多跳推理（如"华东区销售额" → "供应商停电事件" → "受影响的合同条文"）时，Purview 在运行时实时审计权限。
- **敏感度标签继承**：代理输出自动继承原始文件的敏感度标签（机密、限阅）。
- **推理路径透明可审计**：解决 AI 代理"无法追溯逻辑、无法证明合规"的痛点。

## 4. 适用场景

重度依赖 Azure / Office 生态，且身处金融、医疗等需要**严格合规审计与复杂逻辑推理**的行业。

## 5. 定位对比

微软走的是"生态内深度整合 + 图谱推理 + 合规审计"的路线。与 AWS Context 相比，两者都强调 identity-aware 的权限透传，但微软把 GraphRAG 学术成果直接产品化，且以 Purview 的敏感度标签体系为治理核心。参见 [AWS Context](aws_context.md)。
