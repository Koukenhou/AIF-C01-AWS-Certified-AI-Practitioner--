# AIF-C01｜4.1.5 **识别数据集的关键特征（包容性・多样性・策展来源・均衡性）**

**データセットの特徴（包括性・多様性・キュレートされたデータソース・バランスの取れたデータセットなど）を特定する**

*中文为主；术语三语：中・日〔カタカナ读法〕・英；结合 AWS 官方文档与工具。*

---

## 0) 本题你要做到什么

* 能**准确定义** 4 个数据特性：**包容性**、**多样性**、**策展（可追溯）来源**、**均衡性**，并能**识别如何在 AWS 上量化与实现**。
* 把“责任 AI”的数据要求落到 **SageMaker Clarify（偏差/公平）**、**Data Wrangler（清洗/变换）**、**Ground Truth/GT Plus（高质量标注）**、**Macie & Comprehend（敏感数据发现/PII）**、**Bedrock Guardrails（敏感信息过滤/策略）**、**Bedrock Knowledge Bases（RAG 数据治理）** 等具体能力上。([AWS ドキュメント][1])

---

## 1) 术语速查（中・日〔读法〕・英）

| 概念       | 日文（读法）             | English              | 一句话识别                         |
| -------- | ------------------ | -------------------- | ----------------------------- |
| **包容性**  | **包括性（ホウカツセイ）**    | Inclusiveness        | 覆盖多语言/多文化/可访问性与弱势场景的**全面覆盖**  |
| **多样性**  | **多様性（タヨウセイ）**     | Diversity            | 数据来源/场景/风格/时间段/设备等**多维度异质性**  |
| **策展来源** | **キュレートされたデータソース** | Curated data sources | **可溯源、可验证**、经筛查清洗与治理的**可信来源** |
| **均衡性**  | **バランスの取れたデータセット** | Balanced dataset     | 关键群体/类别**比例适当**，避免训练/评测偏差     |

---

## 2) 为什么“数据特性”对 Responsible AI 至关重要

* **不包容/不多样** → 上线后出现**群体性能断崖**；
* **来源未策展** → **RAG 污染/注入**或**合规风险**（PII/版权）；
* **不均衡** → **偏差/不公平**指标恶化。
  AWS 官方将这些内容写进**生成式 AI 安全/治理与 RAG 架构**指南，并提供可执行工具链（Clarify、KBs、Macie、Guardrails 等）。([AWS ドキュメント][2])

---

## 3) 四大特性逐点详解（定义→识别→AWS 落地→小例子）

### 3.1 包容性（Inclusiveness｜包括性）

**定义**：数据覆盖**多语言、多文化、多设备与可访问性**场景，尽量贴近真实用户分布。
**如何识别**：

* 按语言/地区/渠道对**训练与验证**分桶，观察**每桶样本量**与**质量指标**（准确/一致性/人工评分）。
* 在 **SageMaker Clarify** 中进行**分组对比**（群体指标对齐）。([AWS ドキュメント][1])
  **AWS 落地**：
* **Data Wrangler** 聚合多源数据，做语言/渠道字段的质量检查与转换；建立“语言=ja/zh/en”等分桶。([AWS ドキュメント][3])
* **Ground Truth / GT Plus** 增补少数语种标注；专家队伍“交钥匙”。([AWS ドキュメント][4])
* **Bedrock Knowledge Bases（RAG）** 接入**多语文档**并保留来源元数据，减少“只会英文资料”的覆盖缺口。([AWS ドキュメント][5])
  **小例**：客服 QA 中，英文指标 AUC=0.95、日文=0.82 → 增补日文语料与标注，RAG 引入日文手册，复评后群体差距收敛。

---

### 3.2 多样性（Diversity｜多様性）

**定义**：**来源/场景/风格/时期/设备/通道**等维度的**异质性**足够，避免“只在某一种条件下学会”。
**如何识别**：

* 计算**来源多样度**（域名/仓库/系统/时期）；
* 观察**文本体裁/风格**与**问题类型**覆盖；
* 在 **RAG** 中，评估“**检索命中**是否集中在少量源”。([AWS ドキュメント][6])
  **AWS 落地**：
* **Prescriptive Guidance（RAG 选型/架构）**建议通过**多源接入**与**元数据过滤**提升覆盖；**自定义切片（Chunking）**与**元数据**能显著改善检索质量。([AWS ドキュメント][6])
* **Data Wrangler** 统一格式、去重与特征化（例如增加“来源域/时间戳/品类”字段）。([AWS ドキュメント][3])
  **小例**：知识库只有“官网手册”，遇到“论坛常见坑”失效 → 加入**FAQ/社区 Q\&A/变更日志**通道，RAG 命中分散化，召回显著提升。

---

### 3.3 策展来源（Curated data sources｜キュレートされたデータソース）

**定义**：可追溯、可验证、**经清洗/检测/治理**的来源；对**敏感信息**做发现与去除；对**恶意/注入**做过滤。
**如何识别**：

* 每条数据**记录“来源、时间、许可、哈希”**；
* 在摄取前运行**敏感数据发现**与**PII 检测**；对外部网页/上传内容执行**注入防护**策略。
  **AWS 落地**：
* **Amazon Macie** 扫描 S3 中的敏感数据（自动发现或任务），输出**发现报告**与位置细节；支持**自定义识别器**。([AWS ドキュメント][7])
* **Amazon Comprehend DetectPiiEntities** 或 **Bedrock Guardrails 的 Sensitive Information Filters** 对文本做**PII 定位与遮罩/阻断**。([AWS ドキュメント][8])
* **安全与注入防护**：依 **AWS 安全博客/PG 指南**，在**摄取/RAG 拼接**前进行**提示注入**防护与规则化提示处理。([Amazon Web Services, Inc.][9])
* **Bedrock Knowledge Bases**：把策展后资料接入 KB，**以 RAG 替代“反复再训练”**，降低合规与维护风险。([AWS ドキュメント][5])
  **小例**：用户上传含邮箱/卡号文档 → **Macie** 发现 S3 敏感对象，**Comprehend/Guardrails** 在入库或拼接时遮罩 → KB 仅使用“净化后”内容。

---

### 3.4 均衡性（Balanced｜バランス）

**定义**：关键**类别/群体**在训练/验证中**比例与权重合理**，避免单侧学习。
**如何识别**：

* 用 **SageMaker Clarify** 的\*\*“训练前偏差（Pre-training Data Bias）”**度量（如**Jensen–Shannon 散度**等指标），或**训练后偏差**比较各群体**接受/拒绝率**与**准确率\*\*差异。([AWS ドキュメント][10])
  **AWS 落地**：
* **Data Wrangler** 做**重采样/重加权**与数据分层；
* **Ground Truth/GT Plus** 为长尾类采集**更多标注**；
* 上线后以 **Model Monitor** 做\*\*偏差漂移（Bias drift）\*\*基线+调度+告警，防止数据分布变化导致再度失衡。([AWS ドキュメント][3])
  **小例**：退款工单标签“发票/投诉/退货”严重失衡 → 重采样+补标注，Clarify 复测偏差下降，线上通过 Monitor 持续观测。

---

## 4) 把“数据特性”融入 RAG/KB 的工程化做法

* **多源/多样**接入：KB 同时挂接**知识库、FAQ、变更日志、政策 PDF**，并在**元数据**中记录来源/日期/版本。([AWS ドキュメント][5])
* **自定义切片（Chunking）与元数据过滤**：利用**变长切片**与**元数据过滤**提升检索质量，减少长段噪声；这是 AWS 实践中提升准确率的关键手段。([Amazon Web Services, Inc.][11])
* **从“再训练”迁移到“策展+检索”**：官方 PG 强调**RAG 是用私有知识的首选**，既能减少成本也更利于合规与更新。([AWS ドキュメント][6])

---

## 5) 一条龙流程（可直接套改）

1. **发现与合规**：

   * **Macie** 扫描 S3 → 敏感数据发现报告；
   * **Comprehend DetectPiiEntities / Guardrails PII** → 文本级遮罩。([AWS ドキュメント][7])
2. **清洗与特征化**：

   * **Data Wrangler** 去重、统一编码、打“来源/日期/许可”标签，做分桶字段。([AWS ドキュメント][3])
3. **补齐与标注**：

   * **Ground Truth / GT Plus** 定义标注指南与质检流程，聚焦少数类与少数语言。([AWS ドキュメント][4])
4. **偏差与均衡评估**：

   * **Clarify** 跑**训练前/后偏差**与**解释**；对“群体/类别”进行差异分析与调参。([AWS ドキュメント][1])
5. **RAG 构建与检索评估**：

   * **Knowledge Bases** 接入策展数据；按**自定义 Chunk**与**元数据过滤**优化召回；对**Retrieve-only / Retrieve-and-Generate**做评估（命中/有据）。([AWS ドキュメント][5])
6. **上线监控与漂移**：

   * **Model Monitor** 建**Bias drift**基线与调度；对 KB 监视数据新旧/来源分布。([AWS ドキュメント][8])

---

## 6) 常见误区（考点 + 实战雷区）

* **只看总体指标，不做分组** → 包容性问题被掩盖（用 Clarify 分组分析）。([AWS ドキュメント][1])
* **把脏数据直接接入 KB** → RAG 中毒与注入风险（Macie/Comprehend/Guardrails 预过滤）。([AWS ドキュメント][7])
* **过度追求大模型/再训练** → 忽视“策展 + RAG + 元数据过滤”的**低成本高可控**路径。([AWS ドキュメント][6])
* **均衡只在训练集做** → 上线后分布变化无人管（用 Model Monitor 做 Bias drift）。([AWS ドキュメント][8])

---

## 7) 三个微型案例（读完即可迁移）

1. **多语言客服**：

   * 目标：提升日/中/英一致性。
   * 动作：Data Wrangler 分桶；GT Plus 补标注；Clarify 分组评估；KB 接入多语文档；命中差距缩小。([AWS ドキュメント][3])

2. **合规 RAG（金融）**：

   * 目标：只引用“可信来源”。
   * 动作：Macie & Comprehend 先净化；Guardrails PII 过滤；KB 仅接入有许可与版本元数据的文档；检索评估达标再上线。([AWS ドキュメント][7])

3. **不均衡意图分类**：

   * 目标：长尾“发票”类召回提升。
   * 动作：GT 补样本；Wrangler 重采样；Clarify 复检偏差；Monitor 建 Bias drift；上线后保持稳定。([AWS ドキュメント][4])

---

## 8) 口袋清单（考试速记）

* **包容性**=多语言/多设备/多渠道分桶覆盖 → **Data Wrangler + Clarify（分组分析）**。([AWS ドキュメント][3])
* **多样性**=来源/体裁/时期/渠道异质化 → **RAG 多源 + 元数据过滤 + 自定义切片**。([AWS ドキュメント][6])
* **策展来源**=可追溯 + 去敏 + 反注入 → **Macie/Comprehend/Guardrails** + **KB**。([AWS ドキュメント][7])
* **均衡性**=类/群体配比合适 → **Clarify（训练前/后偏差）** + **GT/GT Plus 补样** + **Model Monitor（Bias drift）**。([AWS ドキュメント][10])

---

## 参考（AWS 官方优先）

* **SageMaker Clarify**：偏差与解释、偏差度量与配置。([AWS ドキュメント][1])
* **SageMaker Data Wrangler**：数据导入/清洗/转换/特征化。([AWS ドキュメント][3])
* **SageMaker Ground Truth / GT Plus**：高质量标注与托管团队。([AWS ドキュメント][4])
* **Amazon Macie**：S3 敏感数据发现与调查。([AWS ドキュメント][7])
* **Amazon Comprehend（Detect PII）** 与 **Bedrock Guardrails（敏感过滤）**。([AWS ドキュメント][8])
* **Bedrock Knowledge Bases（RAG）**：工作原理与“用 RAG 代替再训练”。([AWS ドキュメント][5])
* **RAG 选型与架构 / 基础指南**；**自定义切片与元数据过滤**实战。([AWS ドキュメント][6])
* **Prompt 注入防护（安全博客/PG）**：把源数据“带毒”挡在外面。([Amazon Web Services, Inc.][9])

---

### 一句话总结

> **好模型=好数据。**在 AWS 上，用 **Wrangler 清洗/分桶**、**GT/GT Plus 补标**、**Clarify 衡量偏差/公平**、**Macie/Comprehend/Guardrails 净化与防注入**、**KBs + RAG 实现多源且可追溯的知识接入**、**Model Monitor 守住上线均衡**，即可把“包容性・多样性・策展来源・均衡性”变成**可度量、可执行**的数据标准。

[1]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-configure-processing-jobs.html?utm_source=chatgpt.com "Fairness, model explainability and bias detection with ..."
[2]: https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/gen-ai-sra.html?utm_source=chatgpt.com "Generative AI for the AWS SRA - AWS Prescriptive Guidance"
[3]: https://docs.aws.amazon.com/sagemaker/latest/dg/data-wrangler.html?utm_source=chatgpt.com "Prepare ML Data with Amazon SageMaker Data Wrangler"
[4]: https://docs.aws.amazon.com/sagemaker/latest/dg/sms.html?utm_source=chatgpt.com "Training data labeling using humans with ..."
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-it-works.html?utm_source=chatgpt.com "How Amazon Bedrock knowledge bases work"
[6]: https://docs.aws.amazon.com/prescriptive-guidance/latest/retrieval-augmented-generation-options/introduction.html?utm_source=chatgpt.com "Retrieval Augmented Generation options and architectures on AWS"
[7]: https://docs.aws.amazon.com/macie/latest/user/data-classification.html?utm_source=chatgpt.com "Discovering sensitive data with Macie"
[8]: https://docs.aws.amazon.com/comprehend/latest/APIReference/API_DetectPiiEntities.html?utm_source=chatgpt.com "DetectPiiEntities - Amazon Comprehend API Reference"
[9]: https://aws.amazon.com/blogs/security/safeguard-your-generative-ai-workloads-from-prompt-injections/?utm_source=chatgpt.com "Safeguard your generative AI workloads from prompt injections - AWS"
[10]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-measure-data-bias.html?utm_source=chatgpt.com "Pre-training Bias Metrics - Amazon SageMaker AI"
[11]: https://aws.amazon.com/blogs/machine-learning/accelerate-performance-using-a-custom-chunking-mechanism-with-amazon-bedrock/?utm_source=chatgpt.com "Accelerate performance using a custom chunking mechanism ... - AWS"
