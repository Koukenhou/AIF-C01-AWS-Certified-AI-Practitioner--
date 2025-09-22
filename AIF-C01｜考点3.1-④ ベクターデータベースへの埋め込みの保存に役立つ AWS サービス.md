# AIF-C01｜3.1.4 **ベクターデータベースへの埋め込みの保存に役立つ AWS サービス**

*AIF-C01 Domain 3, Task 3.1.4 — Identify AWS services for storing/querying embeddings (vector databases)*

> 目标：能**点名并区分**在 AWS 上可用于**向量（ベクトル）/嵌入（埋め込み【うめこみ】）**存取与相似度检索的托管服务，理解其**数据模型/检索算法/典型用法/取舍**，并知道\*\*与 Amazon Bedrock（ナレッジベース）\*\*如何衔接。
> 术语给出 **中・日（读法）・英**；讲解以中文为主，引用官方文档。

---

## 0) 一页速览｜Which service for vectors?

| 服务                                     | 日文（读法）                  | English                                | 核心用途（向量相关）                                                                                                       |
| -------------------------------------- | ----------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Amazon OpenSearch Service**          | アマゾン・オープンサーチ・サービス       | Amazon OpenSearch Service              | 原生 **k-NN/Approx k-NN** 向量检索；`knn_vector` 字段、Serverless 向量集合（コレクション）。([AWS ドキュメント][1])                           |
| **Amazon Aurora（PostgreSQL 互換）**       | アマゾン・オーロラ（ぽすとぐれす互換）     | Amazon Aurora PostgreSQL-compatible    | 通过 **pgvector** 存取/索引向量；亦可作为 **Bedrock Knowledge Bases** 的向量存储。([AWS ドキュメント][2], [Amazon Web Services, Inc.][3]) |
| **Amazon RDS for PostgreSQL**          | アマゾン・アールディーエス・フォー・ポストグレ | Amazon RDS for PostgreSQL              | 同样支持 **pgvector** 扩展；版本矩阵与安装见官方表。([Amazon Web Services, Inc.][4], [AWS ドキュメント][5])                               |
| **Amazon Neptune / Neptune Analytics** | アマゾン・ネプチューン             | Amazon Neptune / Neptune Analytics     | **图 + 向量** 一体：支持向量索引/相似度（VSS）用于知识图谱、RAG on graph。([AWS ドキュメント][6])                                               |
| **Amazon DocumentDB (MongoDB 互換)**     | アマゾン・ドキュメントDB（もんご互換）    | Amazon DocumentDB (MongoDB-compatible) | 提供 **Vector Search**（向量索引/查询运算子）结合文档型 JSON 查询。([AWS ドキュメント][7])                                                  |

> 备忘：**Bedrock Knowledge Bases（ナレッジベース）** 可直接管理分块/嵌入/向量检索，也支持 **BYO 向量库**（如 OpenSearch Serverless、Aurora pgvector、RDS pgvector）对接。([AWS ドキュメント][8])

---

## 1) Amazon OpenSearch Service（向量检索的“通用盘”）

* **日**：アマゾン・オープンサーチ・サービス
* **EN**：Amazon OpenSearch Service
* **向量能力**：

  * 原生 **`knn_vector` 字段**，支持向量维度至 **16,000**；相似度度量含 **cosine / Euclidean 等**。([AWS ドキュメント][1])
  * **k-NN / Approx k-NN（HNSW 等）** 实现高维近似最近邻检索；适合大规模语义检索。([AWS ドキュメント][1])
  * **Serverless 向量集合**：简化“建集合→写入→检索”的流程，便于实时存取嵌入。([AWS ドキュメント][9])
* **与 Bedrock 的关系**：作为 **Knowledge Bases** 的后端选项之一（自带向量库/BYO），常见于企业 RAG。([AWS ドキュメント][8])
* **何时选**：需要**高并发/低延迟**的语义检索、混合检索（BM25 + 向量）、或**接入生态广**（可观测/可扩展）的场景。
* **补充（开源文档）**：OpenSearch OSS 对 `knn_vector`、embedding pipeline 有更细的参数与示例。([OpenSearch Docs][10])

---

## 2) Amazon Aurora（PostgreSQL 互換）× pgvector（关系型 + 向量）

* **日**：アマゾン・オーロラ（ポストグレ互換）
* **EN**：Amazon Aurora PostgreSQL-compatible
* **向量能力**：

  * 通过 **pgvector 扩展** 将向量作为 **列类型 `vector(n)`** 存储，并支持 **HNSW/IVFFlat** 等索引策略（随扩展版本而异）。([Amazon Web Services, Inc.][3])
  * **作为 Bedrock KB 的向量存储**：官方文档给出版本要求与一键接入 KB 的步骤。([AWS ドキュメント][2])
* **何时选**：**结构化（表格）数据 + 语义查询**并存；需要事务能力（ACID）、SQL 生态与向量一库管理。

---

## 3) Amazon RDS for PostgreSQL × pgvector（标准托管 PG + 向量）

* **日**：アマゾン・アールディーエス・フォー・ポストグレ
* **EN**：Amazon RDS for PostgreSQL
* **向量能力**：

  * **pgvector** 在 **RDS PostgreSQL 15.2+** 即可使用（各小版本支持见“扩展版本矩阵”与“扩展安装”文档）。([Amazon Web Services, Inc.][4], [AWS ドキュメント][5])
* **何时选**：已有 **RDS PG** 工作负载，希望**低代价**引入语义检索；或与现有 BI/ETL/SQL 工具无缝结合。

---

## 4) Amazon Neptune / Neptune Analytics（图数据 + 向量）

* **日**：アマゾン・ネプチューン
* **EN**：Amazon Neptune / Neptune Analytics
* **向量能力**：

  * **Neptune Analytics** 支持**向量索引**与**向量相似度（VSS）** 算法（kNN 等），用于在图上进行上下文/相似性问答。([AWS ドキュメント][6])
  * 注意：向量索引的更新**不具备 ACID 事务**（更新的原子性保证不同于基础图存储）。([AWS ドキュメント][11])
* **何时选**：你有**实体-关系（知识图谱）**与**语义检索**的组合诉求（例如“关联 + 语义”混合问答、合规追溯）。

---

## 5) Amazon DocumentDB (MongoDB 互換)（文档型 + 向量）

* **日**：アマゾン・ドキュメントDB（モンゴ互換）
* **EN**：Amazon DocumentDB (with MongoDB compatibility)
* **向量能力**：

  * **Vector Search** 将文档型 JSON 查询与向量检索结合（创建向量索引并用向量查询操作符检索）。([AWS ドキュメント][7])
* **何时选**：已有 **DocumentDB/MongoDB** 生态，需在**文档查询 + 语义检索**间无缝切换（如 FAQ/商品目录）。

---

## 6) 和 Amazon Bedrock（ナレッジベース）の集成（RAG 全托管 / BYO）

* **托管（推荐起步）**：**Knowledge Bases for Amazon Bedrock** 自动完成 **分块（チャンク化）→ 嵌入（埋め込み）→ 向量入库 → 检索拼接**；适合快速上线企业 RAG。([AWS ドキュメント][8])
* **BYO（自带向量库）**：KB 允许接入 **OpenSearch Serverless**、**Aurora/RDS (pgvector)** 等，保留你既有数据平台的一致性与成本弹性。([AWS ドキュメント][8])

---

## 7) 选型与取舍（考试可直接套）

| 关注点               | 建议                                        | 理由/依据                                                                                |
| ----------------- | ----------------------------------------- | ------------------------------------------------------------------------------------ |
| **超大规模/低延迟语义检索**  | **OpenSearch Service（含 Serverless 向量集合）** | 专为搜索场景优化，`knn_vector`、Approx k-NN（HNSW），工程生态成熟。([AWS ドキュメント][1])                     |
| **关系型 + 向量同库**    | **Aurora PG / RDS PG + pgvector**         | 保留 SQL/事务/外键/BI 生态，同时做语义检索；Aurora/RDS 官方支持 pgvector。([Amazon Web Services, Inc.][4]) |
| **图谱 + 向量**       | **Neptune / Neptune Analytics**           | 在实体关系上叠加向量相似度；但注意向量索引**事务特性差异**。([AWS ドキュメント][6])                                    |
| **文档型 + 向量**      | **DocumentDB (MongoDB 互換) Vector Search** | JSON 文档查询 + 语义检索合一，适合内容库/目录/FAQ。([AWS ドキュメント][7])                                    |
| **最少工程、快速落地 RAG** | **Bedrock Knowledge Bases（托管）**           | 一站式分块/嵌入/检索/拼接，亦可接 BYO 向量库。([AWS ドキュメント][8])                                         |

---

## 8) 小抄：各服务“向量关键字”（中・日・英）

* **向量字段**：`knn_vector`（**ベクトル フィールド**）/ OpenSearch。([AWS ドキュメント][1])
* **近似最近邻**：**近似 k-NN（えぬえぬ）** / Approx k-NN（HNSW）。([AWS ドキュメント][1])
* **pgvector**：**ピージーベクター** / PostgreSQL `vector(n)` 类型与 HNSW/IVF 索引（Aurora/RDS）。([Amazon Web Services, Inc.][3])
* **Neptune 向量索引**：**ベクトル索引（そくいん）** / Vector index；**VSS** アルゴリズム（Neptune Analytics）。([AWS ドキュメント][6])
* **DocumentDB Vector Search**：**ベクトル検索** / Vector search（索引 + 向量查询算子）。([AWS ドキュメント][7])
* **Bedrock KB（RAG）**：**ナレッジベース** / Knowledge Bases（托管向量流程，支持 BYO）。([AWS ドキュメント][8])

---

## 9) 简要“起步配置”示例（只看思路）

* **OpenSearch（Serverless 向量集合）**：创建集合 → 定义 `knn_vector` 映射 → 写入向量 → `kNN`/近似 `kNN` 查询。([AWS ドキュメント][9])
* **Aurora/RDS PG（pgvector）**：`CREATE EXTENSION vector;` → `CREATE TABLE ... embedding vector(768);` → 建 HNSW/IVF 索引（随版本）→ 语义相似度 SQL。([Amazon Web Services, Inc.][3], [AWS ドキュメント][5])
* **Neptune Analytics**：为图属性生成向量 → 建立 **vector index** → 运行**向量相似度**查询/算法（VSS）。([AWS ドキュメント][6])
* **DocumentDB**：创建向量索引 → 使用官方**向量搜索**操作符检索。([AWS ドキュメント][7])
* **Bedrock KB**：配置数据源（S3 等）→ 选择向量后端（托管/OpenSearch/Aurora/RDS）→ KB 自动**分块/嵌入/入库**。([AWS ドキュメント][8])

---

### 一句话总括

> **在 AWS 存取嵌入**：
>
> * **搜索为先** → **OpenSearch**（含 Serverless 向量集合）；
> * **SQL/事务为先** → **Aurora/RDS（pgvector）**；
> * **图谱关系为先** → **Neptune/Neptune Analytics**；
> * **文档为先** → **DocumentDB Vector Search**；
> * **要最快做 RAG** → **Bedrock Knowledge Bases（托管）**，也能接 **BYO 向量库**。上述能力与边界以官方文档为准与更新为准。([AWS ドキュメント][1], [Amazon Web Services, Inc.][3])

[1]: https://docs.aws.amazon.com/opensearch-service/latest/developerguide/vector-search.html?utm_source=chatgpt.com "Vector search - Amazon OpenSearch Service"
[2]: https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.VectorDB.html?utm_source=chatgpt.com "Using Aurora PostgreSQL as a Knowledge Base ..."
[3]: https://aws.amazon.com/blogs/database/accelerate-hnsw-indexing-and-searching-with-pgvector-on-amazon-aurora-postgresql-compatible-edition-and-amazon-rds-for-postgresql/?utm_source=chatgpt.com "Accelerate HNSW indexing and searching with pgvector on ..."
[4]: https://aws.amazon.com/about-aws/whats-new/2023/05/amazon-rds-postgresql-pgvector-ml-model-integration/?utm_source=chatgpt.com "pgvector for simplified ML model integration"
[5]: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.PostgreSQL.CommonDBATasks.Extensions.html?utm_source=chatgpt.com "Using PostgreSQL extensions with Amazon RDS for ..."
[6]: https://docs.aws.amazon.com/neptune-analytics/latest/userguide/vector-similarity.html?utm_source=chatgpt.com "Working with vector similarity in Neptune Analytics"
[7]: https://docs.aws.amazon.com/documentdb/latest/developerguide/vector-search.html?utm_source=chatgpt.com "Vector search for Amazon DocumentDB"
[8]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-setup.html?utm_source=chatgpt.com "Prerequisites for using a vector store you created for a ..."
[9]: https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless-vector-search.html?utm_source=chatgpt.com "Working with vector search collections - Amazon OpenSearch ..."
[10]: https://docs.opensearch.org/latest/vector-search/?utm_source=chatgpt.com "Vector search - OpenSearch Documentation"
[11]: https://docs.aws.amazon.com/neptune-analytics/latest/userguide/vector-index.html?utm_source=chatgpt.com "Vector indexing in Neptune Analytics"
