# AIF-C01｜3.1.3 検索拡張生成（RAG）の定義とビジネス活用

*AIF-C01 Domain 3, Task 3.1.3 — Define RAG and explain business usage on AWS (Amazon Bedrock, Knowledge Bases, etc.)*

> 目标：准确**定义 RAG**，并能说出在 **AWS（Amazon Bedrock / Knowledge Bases / Kendra / 向量存储）**上的**实现方式**、**业务价值**、**架构选型**与**治理要点**。术语提供**中・日（片假名读法）・英**三语，讲解以中文为主。

---

## 1) RAG 是什么？为何需要？

* **定义（中文）**：**检索增强生成**（RAG）是在**调用大模型之前**，先从你自己的数据源中**检索**最相关的内容，再把这些内容与问题一起**增强到提示**中，让模型生成**更相关、可引用**的回答。
* **日**：検索拡張生成【けんさく かくちょう せいせい】
* **EN**：Retrieval-Augmented Generation (RAG)
* **为什么（要点）**：

  1. 让模型“知道你的企业知识/最新变化”；
  2. 降低“幻觉”，支持**带引用**的可审计回答；
  3. 以**更小模型 + RAG**替代“巨长上下文+超大模型”，**降本提效**。
* **官方依据**：AWS 明确将 RAG 定义为“从数据源提取信息以提升回答的相关性与准确性”；**Knowledge Bases for Amazon Bedrock**是 RAG 的托管实现。([AWS ドキュメント][1])

---

## 2) 标准 RAG 管线（从数据到答案）

**文字图**：

```
数据源（S3/文件/数据库/业务系统）
   └─► 分块（チャンク化 / Chunking）
           └─► 嵌入（埋め込み / Embeddings）
                   └─► 向量库（ベクトルストア / Vector Store）
                          └─► 检索（Top-K + 可选重排序）
                                 └─► 生成（LLM，把命中文本拼入提示）
                                        └─► 带引用的回答 + 安全过滤（Guardrails）
```

**关键术语（中・日・英）**

* 分块：**チャンク化**（チャンクか）/ Chunking
* 嵌入：**埋め込み表現**（うめこみ）/ Embeddings
* 向量库：**ベクトルストア** / Vector store
* Top-K：**トップK** / Top-K（取前K段）
* 引用：**出典表示**（しゅってん ひょうじ）/ Citations

---

## 3) 在 AWS 上如何做 RAG（三条主线）

### 3.1 **托管一条龙：Knowledge Bases for Amazon Bedrock**

* **日**：ナレッジベース（for Bedrock）
* **EN**：Knowledge Bases for Amazon Bedrock
* **特点**：**自动完成**“分块→嵌入→向量入库→检索拼接→返回引用”，直接给应用**带引用**的答案，**最小工程量**即可落地。([AWS ドキュメント][2])
* **数据接入**：支持连到**非结构化数据源**或**经由查询引擎访问结构化数据**（控制台或 API 配置**数据源连接器**）。([AWS ドキュメント][3])
* **向量库选项**：

  * 托管向量存储（由 KB 管理）；
  * **自带向量库**（BYO）：如 **Amazon OpenSearch Serverless**（向量检索集合）或 **Aurora PostgreSQL（pgvector）** 等。([AWS ドキュメント][4])
* **使用方式**：你的应用向**知识库 API**发问，它会检索并把结果与问题一起传给所选的\*\*Foundation Model（FM）\*\*生成答案。([AWS ドキュメント][1])

### 3.2 **企业搜索加速器：Amazon Kendra**

* **日**：アマゾン・ケンドラ
* **EN**：Amazon Kendra（Enterprise Search）
* **特点**：企业级语义搜索（多连接器、权限同步、FAQ、问答）；可与 **Bedrock/Knowledge Bases**集成，构建**RAG 助手**。([Amazon Web Services, Inc.][5])
* **何时选**：大规模企业文档、**权限敏感**的内部搜索、或需要**Kendra GenAI Index**简化检索器构建时。([Amazon Web Services, Inc.][5])

### 3.3 **自建/可控栈：OpenSearch / Aurora pgvector**

* **日**：オープンサーチ／オーロラ PostgreSQL（ピージーベクター）
* **EN**：Amazon OpenSearch (Serverless) / Amazon Aurora PostgreSQL with pgvector
* **特点**：你自己管理**向量库**、索引参数、检索策略、重排序器；适合需要**高可控/高可扩**的团队。([AWS ドキュメント][4])
* **与 KB 的关系**：可作为 **Knowledge Bases 的后端向量存储**接入；Aurora pgvector 亦可直接被配置为 KB 的向量库。([AWS ドキュメント][6])

> **补充能力**：
>
> * **Guardrails（ガードレール）**：跨模型复用的**内容安全/PII**护栏，保护**提示与回答**。([AWS ドキュメント][7], [Amazon Web Services, Inc.][8])
> * **Agents（エージェント）/Tool Use**：回答之外还能**调用工具/API**去“办事”，把 RAG 升级为**会执行**的助手。([Amazon Web Services, Inc.][9], [AWS ドキュメント][10])
> * **Evaluations（エバリュエーション）**：对**模型**与**知识库检索**进行自动/对比评估（正确性、鲁棒性、Groundedness 等），用于**选型与回归**。([AWS ドキュメント][11])

---

## 4) 业务场景怎么用（看到关键词就能归类）

| 场景              | 日文（读法）      | English               | 目标 & AWS 组合                                                                                  |
| --------------- | ----------- | --------------------- | -------------------------------------------------------------------------------------------- |
| 内部知识问答/政策检索     | 社内ナレッジQA    | Internal Knowledge QA | **KB（RAG）+ Bedrock** 带引用回答；权限敏感可**Kendra**。([AWS ドキュメント][2], [Amazon Web Services, Inc.][5]) |
| 客服/帮助中心 Copilot | サポート コパイロット | Support Copilot       | **Kendra/KB** 检索 + **Bedrock** 生成；复杂流程加 **Agents** 执行动作。([Amazon Web Services, Inc.][9])     |
| 法规/合规查找         | コンプライアンス検索  | Compliance Search     | RAG + **Guardrails**（话题/PII）+ 带引用回答，便于审计。([AWS ドキュメント][7])                                   |
| 报告/摘要自动化        | レポート要約      | Summarization         | **批量 RAG**（夜间跑）+ **Evaluations** 做质量回归。([AWS ドキュメント][11])                                    |
| 分析问答/数据洞察       | データインサイトQA  | Analytics Q\&A        | **Kendra GenAI Index** + Bedrock；必要时接 BI/数据 API 做 Agent。([Amazon Web Services, Inc.][5])     |

> 参考：Amazon Finance 将 **Bedrock + Kendra + Claude** 组合打造分析助手，结合检索与生成提升情境化回答质量。([Amazon Web Services, Inc.][12])

---

## 5) 设计清单（Design Checklist｜考试高频）

1. **数据源与权限**（データソース/アクセス制御）

   * 数据位置（S3/SharePoint/Confluence/DB 等）与**权限同步**策略（Kendra/KB 连接器）。([AWS ドキュメント][3], [Amazon Web Services, Inc.][5])
2. **分块与嵌入**（チャンク化/埋め込み）

   * Chunk 大小（\~300–800 tokens 作为起点）、重叠、文档分层。
3. **向量库**（ベクトルストア）

   * KB 托管 vs **OpenSearch Serverless/Aurora pgvector**；选择**相似度度量**与索引参数。([AWS ドキュメント][4])
4. **检索策略**（検索戦略）

   * Top-K、过滤（元数据/时间）、可选重排序（rerank），避免**把整篇文档拼接**进上下文。
5. **提示工程 & 模型参数**（プロンプト/推論パラメータ）

   * 结构化模板、引用位置、`max_tokens` 控制长度；温度/Top-P 二选一小幅调。
6. **安全与合规**（セキュリティ/コンプライアンス）

   * **Guardrails**（PII/话题/提示注入）+ VPC Endpoint/KMS/IAM/CloudTrail。([AWS ドキュメント][7], [Amazon Web Services, Inc.][8])
7. **评估与监控**（評価/モニタリング）

   * **Evaluations**（召回/正确性/有引据） + CloudWatch 指标；AB 对比不同检索/提示方案。([AWS ドキュメント][11])
8. **成本与延迟**（コスト/レイテンシ）

   * 控上下文长度、限制 Top-K、批/异步处理、必要时 **Provisioned Throughput** 保峰值。

---

## 6) 常见陷阱与改进（Pitfalls → Fix）

* **只“堆上下文”不检索** → 上下文溢出、成本激增 → **RAG + 精准 Top-K**。
* **Chunk 太大或太小** → 命中差/拼接冗余 → 从**中等粒度**起，视召回与读者体验调优。
* **无引用** → 难审计/信任低 → **KB/Kendra 提供引用**；在回答中**内嵌出典**。([AWS ドキュメント][2])
* **忽略安全** → 可能泄露 PII/生成不当内容 → **Guardrails** + 私网/VPC/KMS。([AWS ドキュメント][7])
* **缺评估反馈** → 质量难迭代 → **Evaluations** 持续回归，记录**业务指标**（解决率、CSAT、成本/会话）。([AWS ドキュメント][11])

---

## 7) 术语小词典（中・日・英）

* 检索增强生成：**検索拡張生成**（けんさく かくちょう せいせい）/ Retrieval-Augmented Generation (RAG)
* 知识库：**ナレッジベース** / Knowledge Bases（for Amazon Bedrock）
* 企业搜索：**エンタープライズサーチ** / Amazon Kendra
* 向量检索：**ベクトル検索** / Vector Search（OpenSearch Serverless / pgvector）([AWS ドキュメント][4])
* 代理/工具调用：**エージェント／ツールユース** / Agents / Tool Use（Function calling）([Amazon Web Services, Inc.][9], [AWS ドキュメント][10])
* 守护栏：**ガードレール** / Guardrails（PII/安全/话题限制）([AWS ドキュメント][7])
* 评估：**エバリュエーション** / Evaluations（模型与 KB 的质量评估）([AWS ドキュメント][11])
* 引用：**出典表示** / Citations（源文段落的可点击出处）

---

## 8) 一句话总括

> **RAG = 检索 + 生成 + 治理**：
> 在 AWS，优先用 **Knowledge Bases for Bedrock** 快速把你数据“接进模型”，必要时配 **Kendra** 做企业级检索，或用 **OpenSearch/Aurora pgvector** 自带向量库。再叠加 **Guardrails**（安全）与 **Evaluations**（质量回归）形成闭环，既能**降本提效**，又能**可审计、可落地**。([AWS ドキュメント][2], [Amazon Web Services, Inc.][5])

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html?utm_source=chatgpt.com "Retrieve data and generate AI responses with ..."
[2]: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-it-works.html?utm_source=chatgpt.com "How Amazon Bedrock knowledge bases work"
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/data-source-connectors.html?utm_source=chatgpt.com "Connect a data source to your knowledge base"
[4]: https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless-vector-search.html?utm_source=chatgpt.com "Working with vector search collections - Amazon OpenSearch ..."
[5]: https://aws.amazon.com/kendra/?utm_source=chatgpt.com "Enterprise Search Engine - Amazon Kendra - AWS"
[6]: https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.VectorDB.html?utm_source=chatgpt.com "Using Aurora PostgreSQL as a Knowledge Base ..."
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[8]: https://aws.amazon.com/bedrock/guardrails/?utm_source=chatgpt.com "Amazon Bedrock Guardrails"
[9]: https://aws.amazon.com/bedrock/agents/?utm_source=chatgpt.com "AI Agents - Amazon Bedrock"
[10]: https://docs.aws.amazon.com/bedrock/latest/userguide/tool-use.html?utm_source=chatgpt.com "Use a tool to complete an Amazon Bedrock model response"
[11]: https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation.html?utm_source=chatgpt.com "Evaluate the performance of Amazon Bedrock resources"
[12]: https://aws.amazon.com/blogs/machine-learning/how-amazon-finance-built-an-ai-assistant-using-amazon-bedrock-and-amazon-kendra-to-support-analysts-for-data-discovery-and-business-insights/?utm_source=chatgpt.com "How Amazon Finance built an AI assistant using ..."
