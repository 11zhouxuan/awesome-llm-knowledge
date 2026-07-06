# Google (Vertex AI Search and Conversation)：搜索级底蕴与开箱即用

> 机构: Google
> 产品: Vertex AI Search and Conversation（绑定 Gemini）
> 类型: 托管云服务（非论文）

## 1. 官方定位

谷歌的方案将大厂的复杂技术封装到极致，主打**搜索级底蕴、多模态融合与开箱即用的极简开发者体验**。

## 2. 技术架构与数据联动

- **继承世界顶级搜索引擎算法**：底层通过 Vertex AI 与 **Gemini 大模型**深度绑定。
- **全托管（Managed）工作流**：企业只需给出一个 Google Cloud Storage（GCS）桶地址或公司内网 URL，系统自动开始工作。
- **多模态解析能力**：同时解析文档中的文字、复杂财务表格以及统计图表（折线图、柱状图），统一转化为高维语义上下文。

## 3. 安全治理与推理特性

- **消费级体验**：非常接近消费级互联网搜索。
- **企业级隔离**：依托 Google Cloud 企业级架构，保证客户数据在多租户环境下的绝对隔离。
- **自动溯源（Citations）与低代码**：生成段落后自动标注维基百科式的角标链接，点击即可预览原始 PDF 页面及对应图表位置。

## 4. 适用场景

拥有大量半结构化/非结构化文档（工程图纸、海量合同、技术手册、产品图片）的企业，希望**无需组装复杂 IT 管道**，短时间内上线一套高质量、高准确度的多模态企业知识问答系统。

## 5. 定位对比

Google 走的是"搜索引擎底蕴 + 多模态 + 低代码开箱即用"的路线，弱化图谱构建、强调自动溯源体验。与 AWS Context 的组织级知识图谱、Microsoft 的 GraphRAG 推理相比，Google 更偏向文档问答的即用性。参见 [AWS Context](aws_context.md)、[Microsoft Fabric + GraphRAG](microsoft_fabric_graphrag.md)。
