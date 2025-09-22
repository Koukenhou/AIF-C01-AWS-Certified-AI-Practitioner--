# AIF-C01｜5.1.2 **ソース用途とデータ出典の文書化の概念**

*AIF-C01 Domain 5, Task 5.1.2 — Documenting source usage & data provenance (Data lineage, Data cataloging, SageMaker Model Cards, etc.)*

> 目标：系统理解**数据来源与去向如何被工程化记录**，包括
>
> * **データリネージ【でーたりねーじ】/Data lineage（数据血统）**
> * **データのカタログ化【かたろぐか】/Data cataloging（数据目录与可发现性）**
> * **SageMaker モデルカード【もでるかーど】/SageMaker Model Cards（模型“说明书”）**
>   并会把这些能力落到 **AWS Glue / Amazon DataZone / Amazon SageMaker / Amazon Bedrock** 的实际流程里（带例子与清单）。

---

## 1) 为什么要“记录数据出典”？（考试 & 实战总览）

* **合规与问责**：能回答“这条结果来自哪里的数据？中间做了哪些处理？谁在用？”
* **质量与重现**：当出现异常（质量下降/偏差）时，能**追溯并重跑**相同流程。
* **协作与复用**：团队成员能**按目录**搜索到可用数据与模型，并理解**用途/限制**。
* **RAG/生成式应用**：给最终回答**附上引用（citations）**，可核验可靠性（尤其面向政务/金融等审计场景）。([AWS ドキュメント][1])

---

## 2) 核心概念｜中・日（读法）・英

### 2.1 データリネージ【でーたりねーじ】/ **Data lineage（数据血统）**

记录**从源到目的地**的**端到端流转**（采集→清洗→特征→训练→部署→推理/消费），含**谁/何时/何地/如何**加工的元信息。

* **SageMaker ML Lineage Tracking**：自动/手动记录**工件（Artifacts）**、**步骤（Trial components/Jobs）**、**关系（Associations）**，帮助**重现流程、治理与审计**。([AWS ドキュメント][2])
* **Amazon DataZone Lineage**（OpenLineage 兼容）：在**业务数据目录**层面**捕获与可视化**跨系统的血统事件，串起 Glue/Redshift 等数据资产与消费关系（现已 GA）。([AWS ドキュメント][3])

### 2.2 データのカタログ化【かたろぐか】/ **Data cataloging（数据目录）**

为数据创建**统一元数据目录**（库/表/位置/schema/分区/更新频次/权限），提升**可发现性**与**一致治理**。

* **AWS Glue Data Catalog**：集中式元数据存储；用 **Crawler** 自动发现 S3/关系库数据，按**数据库/表**组织，支持借助 IAM 做细粒度权限控制。([AWS ドキュメント][4])
* **快速入门**：创建数据库/表、指向 S3 数据源的教程。([AWS ドキュメント][5])
* **Amazon DataZone**：在**业务语义层**编目数据资产（项目库存、发布/订阅），让数据**可搜索、可治理**。([AWS ドキュメント][6])

### 2.3 SageMaker モデルカード【もでるかーど】/ **SageMaker Model Cards**

把模型的**关键信息**集中记录在一张“**模型卡**”里：**预期用途/限制、数据来源、评估指标、风险与合规说明**，支持**控制台/Python SDK/CLI** 管理与导出 PDF。([AWS ドキュメント][7])

### 2.4 生成式应用中的“引用”/ **Citations**（与 RAG 相关）

**Amazon Bedrock Knowledge Bases** 在生成回答时可**附带引用**（源文档片段与标识），`RetrieveAndGenerate` 与 `InvokeAgent` 的响应里有 `citations` 字段，便于**核验来源**。([AWS ドキュメント][1])

---

## 3) 典型落地架构（以“道路影像识别 + RAG 报告”示例）

> 目标：把**数据出典**和**用途文档**贯穿**数据→模型→应用**的全生命周期，适合考试作答与项目实践。

**(A) 数据域（Data Lake/Lakehouse）**

1. **S3 原始区**存放拍摄原片；**Glue Data Catalog** 建库/表，由 Crawler 自动抓取 schema/分区；表项记录**位置/格式/分区键**（如日期/区域）。([AWS ドキュメント][4])
2. **数据处理作业**（Glue/SageMaker Processing）产出**清洗集/特征集**；作业将**输入-输出**与**代码版本**写入 **SageMaker ML Lineage**（自动+手动补录结合）。([AWS ドキュメント][2])
3. 在 **Amazon DataZone** 将这些**数据资产发布到业务目录**，治理谁能发现/订阅，自动捕获**血统事件**以可视化上下游。([AWS ドキュメント][6])

**(B) 模型域（SageMaker）**
4\) **训练/评估/批推理作业**：SageMaker 自动创建 lineage 实体（trial components），串联**训练作业→模型工件→端点**；你可补录**自定义步骤**（如第三方特征工程）。([AWS ドキュメント][8])
5\) **创建 Model Card**：写明**训练数据来源（对接 Glue/DataZone 条目）**、**用途/限制**、**指标**、**伦理与合规考量**，并**导出 PDF**归档。([AWS ドキュメント][7])

**(C) 应用域（生成式/RAG）**
6\) 将**道路法规/历史巡检报告**同步到 **Bedrock Knowledge Bases**。当用户问“这段路面为何判为 D 级？”时，KB 返回答案**连同引用**（源 PDF/条款编号）。([AWS ドキュメント][1])
7\) 运维/审计：CloudWatch/CloudTrail 监控 API 调用与指标（此处与 5.1.1 主题相关），**引用 + 模型卡 + 血统图**三者为审计闭环。

---

## 4) 逐项深入 & 操作要点

### 4.1 Glue Data Catalog（“数据黄页”）

* **你拥有什么**：**数据库（Database）**与**表（Table）**两级结构；表保存**位置、格式、schema、分区**元数据，充当查询/ETL 的**统一索引**。([AWS ドキュメント][9])
* **怎么来**：

  * 用 **Crawler** 指向 S3 路径或 JDBC 源，自动发现并注册到 Catalog。
  * 教程级路径：**建数据库→建表→指向 S3**。([AWS ドキュメント][5])
* **安全/治理**：结合 IAM/资源策略对**库/表**做细粒度控制（与 Lake Formation 搭配更强）。([AWS ドキュメント][9])
* **在出典上的作用**：为**数据来源与最新版本**提供**权威入口**；配合 **DataZone** 做**业务语义**与**可见范围**管理。([AWS ドキュメント][6])

### 4.2 Data lineage（两条主线：SageMaker & DataZone）

* **SageMaker ML Lineage Tracking**：

  * 自动追踪**处理/训练/批推理作业**→ 产出**Artifacts/Associations**；
  * 你也可**手动创建实体**代表自定义步骤（如 OpenCV 预处理），保证链路完整。([AWS ドキュメント][2])
* **Amazon DataZone Lineage（OpenLineage 兼容）**：

  * **API 驱动**捕获/可视化事件，贯通**业务数据目录**内外活动（发布/订阅/ETL/分析/消费）。
  * 最新博客与文档：**预览引入 → 现已 GA**；可追溯**跨组织**数据消费。([Amazon Web Services, Inc.][10])
* **搭配方式**：

  * **底层算子/ML 细节** → 用 **SageMaker Lineage** 记录；
  * **跨域/跨团队/业务视角** → 用 **DataZone** 汇总与可视化。

### 4.3 SageMaker Model Cards（“模型说明书”）

* **记录什么**：**预期用途、限制、数据来源/比例、标注策略、评估指标、风险与偏差分析、部署信息**；支持**SDK/CLI/Console** 创建与**PDF 导出**。([AWS ドキュメント][7])
* **工程要点**：

  * 把**数据来源**直接**引用到 Glue/DataZone 条目**（库/表/资产 ID），避免“口口相传”。
  * 将**评估结果**与**线上的 A/B 指标**定期更新到 Model Card，形成**活文档**。
* **SDK 示例参考**：官方示例与 SDK 说明展示了如何**创建/更新/导出**模型卡。([sagemaker-examples.readthedocs.io][11])

### 4.4 Bedrock Knowledge Bases 的“引用”（RAG 可信度）

* **功能**：在回答中**附带 citations**（引用对象与命中的源片段），便于核实**原始依据**与**文档版本**。
* **接口**：`RetrieveAndGenerate` 与 `InvokeAgent` 的响应里包含 `citations` 字段。([AWS ドキュメント][1])
* **注意**：引用的**数量与粒度**取决于检索配置与召回，设计上是**辅助核对**而非替代**数据目录/血统记录**。([AWS ドキュメント][1])

---

## 5) “一条龙”实操清单（可直接贴进项目 Wiki）

**阶段 A｜编目（Cataloging）**

* [ ] 为 `s3://city-roads/raw/`、`…/clean/`、`…/features/` 建 **Glue Data Catalog** 数据库/表，启用 Crawler **每日**更新。([AWS ドキュメント][4])
* [ ] 在 **DataZone** 将关键资产**加入项目库存**，按团队发布/订阅策略管理可见性。([AWS ドキュメント][6])

**阶段 B｜血统（Lineage）**

* [ ] 所有 **SageMaker 作业**（Processing/Training/BatchTransform）均被 Lineage 自动追踪；对**自定义脚本步骤**调用 SDK **手动创建实体**并建立关联。([AWS ドキュメント][8])
* [ ] 打通 **DataZone Lineage**，从 Glue/Redshift 采集 lineage 事件，**可视化端到端**路径。([Amazon Web Services, Inc.][12])

**阶段 C｜模型文档（Model Cards）**

* [ ] 在**初次部署前**创建 Model Card：写明**用途/限制、训练集来源（对接 Catalog/DataZone 条目）**、**指标/基线**、**风险评估**。
* [ ] 通过 **SDK/Console** 在每次 **再训练/再评估**后更新 Model Card，并**导出 PDF**归档到 S3（只读桶）。([AWS ドキュメント][13])

**阶段 D｜RAG 引用（Citations）**

* [ ] 把**法规/检修制度/历史病害报告**上到 **Knowledge Bases**；
* [ ] 在前端显示**答案 + 引用列表**（文档名/页码/片段），用户可点击核验。([AWS ドキュメント][1])

---

## 6) 现实示例（道路影像场景）

> **问题**：“为什么系统把××路段评为 D 级？”
> **回答链路**：
>
> 1. **RAG**：Bedrock KB 检索到《城市道路技术标准》第 5.2 节和**2024-07 巡检报告**第 12 页，模型生成答案**附带引用**。([AWS ドキュメント][1])
> 2. **追问“这张模型是用什么数据训练的？”**：打开 **SageMaker Model Card**，看到**训练集来自 Glue 表** `roads_train_2021_2024`（链接到 DataZone 资产），**限制**注明“夜间雨天识别能力下降”。([AWS ドキュメント][7])
> 3. **再追问“这份训练集怎么来的？”**：在 **SageMaker Lineage** 与 **DataZone Lineage** 中查看**从原片→清洗→特征→训练→模型→端点**的**端到端血统图**。([AWS ドキュメント][2])

---

## 7) 与 5.1.1 的衔接（安全与治理）

* **目录/血统/模型卡**是“**记录**与**可解释**”面；
* **IAM/KMS/PrivateLink/Macie**是“**保护与合规**”面；
* 两者**互补**：先**记录清楚**，再**保护到位**，并用 CloudWatch/CloudTrail 做**可审计**闭环。

---

## 8) 术语小词典（中・日・英，带读法）

* 数据血统：**データリネージ【でーたりねーじ】** / Data lineage（SageMaker Lineage, DataZone Lineage）。([AWS ドキュメント][2])
* 数据目录化：**データのカタログ化【かたろぐか】** / Data cataloging（Glue Data Catalog, DataZone Business Catalog）。([AWS ドキュメント][4])
* 模型卡：**SageMaker モデルカード【もでるかーど】** / SageMaker Model Cards（治理与报告，可导出 PDF）。([AWS ドキュメント][7])
* OpenLineage：**オープンリネージ** / OpenLineage（DataZone/SageMaker 兼容的 lineage 规范）。([Amazon Web Services, Inc.][10])
* 知识库引用：**引用【いんよう】/ サイテーション** / Citations（Bedrock KB 的 `citations` 字段）。([AWS ドキュメント][14])
* 元数据目录：**メタデータカタログ** / Metadata catalog（Glue 的数据库/表，支持 Crawler 自动发现）。([AWS ドキュメント][4])

---

## 9) 备考小抄（速答版）

* **Lineage = 追踪流向**：SageMaker（作业/工件级）＋ DataZone（跨域/可视化）。([AWS ドキュメント][2])
* **Catalog = 可发现**：Glue Data Catalog（数据库/表/Crawler）＋ DataZone（业务目录/发布-订阅）。([AWS ドキュメント][4])
* **Model Cards = 透明**：用途/限制/来源/指标/风险，支持 SDK/CLI/Console 与 PDF 导出。([AWS ドキュメント][7])
* **RAG 引用**：Bedrock KB 在回答中附**citations**，便于核验来源。([AWS ドキュメント][1])

---

### ✅ 一句话总括

> **记录出典 = Catalog（找得到）＋ Lineage（看得清）＋ Model Card（说得明）＋ RAG Citations（查得到）**。在 AWS 上，分别用 **Glue/DataZone** 编目与可视化血统，**SageMaker** 追踪 ML 流程并生成**模型卡**，**Bedrock KB** 把引用带到用户答案中，形成**从数据到回答的“端到端透明链”**。 ([AWS ドキュメント][4])

如果你愿意，我可以把**示例 Model Card 模板（中文注释版）**和**DataZone/Glue 字段命名规范**整理成一页速查表，方便你团队直接沿用。

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html?utm_source=chatgpt.com "Retrieve data and generate AI responses with ..."
[2]: https://docs.aws.amazon.com/sagemaker/latest/dg/lineage-tracking.html?utm_source=chatgpt.com "Amazon SageMaker ML Lineage Tracking"
[3]: https://docs.aws.amazon.com/datazone/latest/userguide/datazone-data-lineage.html?utm_source=chatgpt.com "Data lineage in Amazon DataZone - AWS Documentation"
[4]: https://docs.aws.amazon.com/glue/latest/dg/catalog-and-crawler.html?utm_source=chatgpt.com "Data discovery and cataloging in AWS Glue"
[5]: https://docs.aws.amazon.com/glue/latest/dg/start-data-catalog.html?utm_source=chatgpt.com "Getting started with the AWS Glue Data Catalog"
[6]: https://docs.aws.amazon.com/datazone/latest/userguide/working-with-business-catalog.html?utm_source=chatgpt.com "Amazon DataZone data catalog"
[7]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-cards.html?utm_source=chatgpt.com "Amazon SageMaker Model Cards"
[8]: https://docs.aws.amazon.com/sagemaker/latest/dg/lineage-tracking-entities.html?utm_source=chatgpt.com "Lineage Tracking Entities - Amazon SageMaker AI"
[9]: https://docs.aws.amazon.com/prescriptive-guidance/latest/serverless-etl-aws-glue/aws-glue-data-catalog.html?utm_source=chatgpt.com "AWS Glue Data Catalog - AWS Prescriptive Guidance"
[10]: https://aws.amazon.com/blogs/aws/introducing-end-to-end-data-lineage-preview-visualization-in-amazon-datazone/?utm_source=chatgpt.com "Introducing end-to-end data lineage (preview) visualization ..."
[11]: https://sagemaker-examples.readthedocs.io/en/latest/sagemaker_model_governance/model_card.html?utm_source=chatgpt.com "Amazon SageMaker Model Governance - Model Cards"
[12]: https://aws.amazon.com/blogs/aws/announcing-the-general-availability-of-data-lineage-in-the-next-generation-of-amazon-sagemaker-and-amazon-datazone/?utm_source=chatgpt.com "Announcing the general availability of data lineage in ..."
[13]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-cards-manage.html?utm_source=chatgpt.com "Model cards actions - Amazon SageMaker AI"
[14]: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_agent-runtime_Citation.html?utm_source=chatgpt.com "Citation - Amazon Bedrock"
