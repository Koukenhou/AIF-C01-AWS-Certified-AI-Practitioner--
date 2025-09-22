# AIF-C01｜3.3.3 **微调用数据的准备方法（数据策展・治理・规模・标注・代表性・RLHF）**／**基盤モデルをファインチューニングするためのデータの準備方法**

*（中文为主讲解；术语三语：中・日〔カタカナ读法〕・英）*

---

## 0) 考点定位（你需要会什么）

* 能**系统阐述**微调前的数据准备六要素：
  **数据策展**（データ・キュレーション｜*Data curation*）、**治理/合规**（ガバナンス｜*Governance*）、**规模**（サイズ｜*Size*）、**标注**（ラベル付け｜*Labeling*）、**代表性**（代表性｜*Representativeness*）、**RLHF**（人間からのフィードバックによる強化学習｜*Reinforcement Learning from Human Feedback*）。
* 会把上述要素**落地到 AWS**（Amazon Bedrock + S3 + Macie/Comprehend + SageMaker Ground Truth + Guardrails + Bedrock Evaluations 等）并能给出**格式示例/流程**。
* 牢记平台侧**数据保护**承诺：**Bedrock 不会**用你的输入/输出训练任何模型，也**不共享给第三方**。([AWS Documentation][1])

---

## 1) 术语速查表（中・日〔读法〕・英）

| 概念    | 日文（读法）                              | English            | 关键理解                      |
| ----- | ----------------------------------- | ------------------ | ------------------------- |
| 数据策展  | データ・キュレーション（データ・キュレーション）            | Data curation      | 清洗、去重、拆分、格式化、质量检查         |
| 治理/合规 | ガバナンス（ガバナンス）                        | Governance         | 加密、访问控制、合规与审计             |
| 规模    | サイズ（サイズ）                            | Size               | 记录数/Token 数、配额、训练/验证/测试拆分 |
| 标注    | ラベル付け（ラベルづけ）                        | Labeling           | 人工/半自动/主动学习、质检            |
| 代表性   | 代表性（ダイヒョウセイ）                        | Representativeness | 覆盖真实分布与关键场景、避免偏倚          |
| RLHF  | 人間からのフィードバックによる強化学習（ニンゲン…キョウカガクシュウ） | RLHF               | 收集**偏好/排序/评分**做对齐与偏好学习    |

---

## 2) 数据策展（Data Curation｜データ・キュレーション）

### 2.1 基础格式（以 Bedrock 为准）

* **微调（Fine-tuning）**：常见是 **`.jsonl`**（每行一个样本）。**非会话**任务多用 `{"prompt": "...","completion":"..."}`；**会话**/指令类可按所选模型的消息格式对齐（与 **Converse** 结构一致）。**不同模型格式可能不同**，以官方“Prepare your training datasets”页为准。([AWS Documentation][2])
* **持续事前训练（CPT）**：**未标注**数据；`.jsonl` 每行 **只含 `input` 字段**；**估算 Token**可用“**6 字符 ≈ 1 token**”。([AWS Documentation][3])

**示例（分类/抽取类微调）**

```json
{"prompt":"将用户话术分类：『我想退货，怎么走流程？』","completion":"退款/售后"}
{"prompt":"把文本提取为JSON(字段: 疾病,药物)。文本=『…』","completion":"{\"疾病\":\"高血压\",\"药物\":[\"缬沙坦\"]}"}
```

**示例（CPT）**

```json
{"input":"《跨境结算操作规程（2023）》第六章……"}
{"input":"证券业协会《投资研究行为准则》条款……"}
```

> Bedrock 支持**自定义模型作业**：通过 **CreateModelCustomizationJob**（控制台/CLI/SDK）提交，并在完成后返回**验证损失**与**样例输出**。([AWS Documentation][4])

### 2.2 清洗与去重

* **去重**（对近似重复段落/网页快照执行 hash/相似度去重），**剔除低质/有害/乱码**内容。
* **敏感信息处理**：

  * 训练前在 S3 上用 **Macie** 扫描并出具**敏感数据发现报告**（自动/按需 Job）。([AWS Documentation][5])
  * 对文本数据可用 **Comprehend** 的 **Detect PII** API/CLI 进行**PII 定位与涂黑**。([AWS Documentation][6])
  * 推理链路上再用 **Bedrock Guardrails** 的**敏感信息过滤**进行**输入/输出**双向掩码或阻断。([AWS Documentation][7])
  * **（PII = Personally Identifiable Information，个人可识别信息）**

### 2.3 目录与版本化（建议）

* S3 目录结构建议：

  ```
  s3://<bucket>/customization/<model-id>/v1/
    ├── train/part-*.jsonl
    ├── validation/part-*.jsonl
    └── test/part-*.jsonl
  ```
* 打上 **数据版本/来源/许可**元数据（S3 Object Tagging），便于溯源与审计。
* 训练/验证数据的**配额**与**记录数限制**以官方“Endpoints and quotas”说明为准（不同模型不同）。([AWS Documentation][2])

---

## 3) 治理/合规（Governance｜ガバナンス）

### 3.1 平台侧承诺与加密

* **不用于训练/不共享**：Bedrock **不会**用你的输入/输出训练任何 Amazon 或第三方模型，也**不与第三方共享**。([AWS Documentation][1])
* **加密与访问控制**：数据与产物存 S3，启用 **SSE-KMS**（自管 KMS Key），结合 **最小权限 IAM** 与 **CloudTrail** 审计。Bedrock 定制作业也支持你指定 **KMS Key**。([AWS Documentation][1])
* SSE-KMS=Server-Side Encryption with AWS Key Management Service keys
* KMS = AWS Key Management Service（密钥管理服务）
* CMK = Customer Managed Key（客户自管密钥）
* IAM = Identity and Access Management（身份与访问管理）

### 3.2 数据发现与隔离

* **Macie**：自动化扫描 S3 中的 PII/敏感数据，生成发现报告，支持**托管识别器**（Managed data identifiers）。([AWS Documentation][5])
* **Guardrails**：在应用层**阻断/遮罩**PII 与敏感实体，补齐**推理时**的安全。([AWS Documentation][8])

### 3.3 合规清单（微调前）

1. 数据许可与版权确认（爬取/内部资料的授权范围）。
2. PII 识别与处理（Macie/Comprehend/正则 + Guardrails）。([AWS Documentation][5])
3. 加密、访问边界与审计（KMS + IAM + CloudTrail；Bedrock 作业指定 KMS）。([AWS Documentation][1])

---

## 4) 规模（Size｜サイズ）

### 4.1 多少数据算“够”？

* **微调（标注数据）**：没有“一刀切”的硬指标；经验上**几千～数十万**样本取决于任务复杂度与基础模型大小。重点是**质量一致性**。
* **CPT（未标注）**：官方博客给的经验是\*\*≥ 10 亿 token\*\*常见到明显增益；每行 `input`；**上限示例**：一次最多 **100,000** 训练样本（记录数）可提交（示例值，随服务演进可能变动，请以当前文档/配额为准）。([Amazon Web Services, Inc.][9])
* **配额**：训练/验证记录数上限、作业时长等受**Bedrock 配额**限制，按“准备数据集”页面与 General Reference 的**Endpoints and quotas**表执行。([AWS Documentation][2])

### 4.2 拆分与度量

* 建议 **80/10/10** 或 **90/5/5**（Train/Val/Test）。
* 以**验证集**为主调参，关注 **验证损失**与**样例输出**（Bedrock 作业会返回）。([AWS Documentation][4])

---

## 5) 标注（Labeling｜ラベル付け）

### 5.1 工具与流程

* **SageMaker Ground Truth**：托管数据标注服务，支持**公有众包/供应商/私有团队**，可与\*\*主动学习（Active learning）\*\*结合降低成本。([AWS Documentation][10])
* **Ground Truth Plus**：**交钥匙**标注服务，由 AWS 运营专家队伍执行并产出高质量标注。([AWS Documentation][11])
* 支持**流式标注**（新增数据持续进入的场景）。([AWS Documentation][12])

### 5.2 标注规范与质检

* 明确**任务定义/标签体系**（例如：意图=〔退款/查询/发票/投诉〕）。
* 设计**指南与示例**，包含**正负例/边界例**。
* **双标/仲裁**：两人以上标注 + 冲突仲裁，抽样复检 5–10%。
* 将**标注输出**转为 Bedrock 需求的 **`.jsonl`** 格式（`prompt`/`completion` 或对话消息格式）。([AWS Documentation][2])

---

## 6) 代表性（Representativeness｜代表性）

### 6.1 为什么重要

若训练集的用户群/场景与**线上真实分布**差距大，模型会在**少数群体或边缘用例**上失衡。

### 6.2 怎样度量与改进

* 用 **SageMaker Clarify** 在**训练前**做**数据偏倚指标**（facet imbalance 等），并在训练后与上线期继续监测。([AWS Documentation][13])
* 与 **Data Wrangler**/Clarify 的再采样（下采样/上采样/SMOTE）联动，**重平衡**数据。([Amazon Web Services, Inc.][14])
* 在**验证/测试集**中，确保**关键人群/场景覆盖**，并对敏感属性（如年龄/地区）观察**分组性能**。([AWS Documentation][15])

---

## 7) RLHF（Reinforcement Learning with Human Feedback｜人間からのフィードバックによる強化学習）

### 7.1 概念与与数据形态

* RLHF（Reinforcement Learning with Human Feedback，人类反馈强化学习）
* **目标**：对齐模型偏好与人类价值/任务要求。
* **数据形态**（常见两种）：

  1. **成对偏好**（ペアワイズ・センコウ｜*pairwise preference*）：给定同一提示的两个回答 A/B，标注**更优者**；
  2. **标度评分**（レイティング｜*ratings*）：对单个回答按维度打分（有用性/事实性/安全性等）。
* 这些**人类偏好数据**既可用于**离线评估**，也可构建**奖励模型/对齐训练**的数据基础。**Bedrock Model Evaluation**支持**自动**与**人工评估**，可用来**收集人工评分/偏好**（存入 S3 的评估报告），为偏好数据集打基础。([Amazon Web Services, Inc.][16])

> 说明：**Bedrock 侧并未对外提供完整 RLHF 训练流水线**；考试关注你是否**理解数据如何准备**与**如何借助 Bedrock 的“人工评估”收集偏好**，再用于你自建或第三方流程的对齐训练/选择模型。([Amazon Web Services, Inc.][16])

### 7.2 偏好数据收集小模版

* 维度（カテゴリ｜*criteria*）：**有用性**、**真实性**、**安全性**、**格式合规**
* 评分（スコアリング｜*scoring*）：Likert 1–5 或 A/B 偏好
* 指南（ガイドライン｜*guidelines*）：提供**正反例**与**边界例**
* 产出：

  * A/B 偏好：`{"prompt": "...","response_A":"...","response_B":"...","preferred":"A"}`
  * 评分：`{"prompt":"...","response":"...","helpfulness":4,"safety":5,"factuality":3}`

---

## 8) 端到端范例（把 3.3.1/3.3.2 与本节串起来）

### 场景 A：**客服意图分类（任务型）**

1. **收集**真实客服语料 → 去重/去噪；
2. **PII 处理**：Macie 扫描库、Comprehend 涂黑，部署 Guardrails 做推理期防护。([AWS Documentation][5])
3. **标注**：Ground Truth 建标签体系与指南；质检抽样；导出 `.jsonl`（`prompt`/`completion`）。([AWS Documentation][10])
4. **训练**：`CreateModelCustomizationJob` 提交微调；关注**验证损失/样例输出**；
5. **评估**：用 Bedrock Evaluations（自动 + 人工）做离线评估；线上 A/B 与监控。([AWS Documentation][17])

### 场景 B：**法律写作风格（知识+风格）**

1. **汇集**判例/合同/条例（授权确认）→ 清洗去重；
2. **CPT**：按 `{"input":"…"}` 组织，估算 Token（6 字符≈1 Token），量级尽量大（≥10 亿 Token 常见到增益）。([AWS Documentation][3])
3. **微调**（可选）：少量指令样本把**输出模板**学牢；
4. **评估**：人工评分“合规性/措辞/完整性”（可用 Bedrock 人工评估收集评分）。([AWS Documentation][18])

### 场景 C：**企业 FAQ 快速上线**

* **优先 RAG（Knowledge Bases）**：把私有文档接入 KB，**不改模型参数**即可用私有知识；减少“持续训练”的需求与成本。([AWS Documentation][19])
* 若需要**固定格式**或**更高稳定性**，再补**微调**少量样本（混合方案）。

---

## 9) “拿分要点清单”（可背）

* **格式**：Fine-tune 常用 `prompt/completion`；CPT 用 **`input`**；`.jsonl` 一行一条；不同模型格式见官方准备页。([AWS Documentation][2])
* **治理**：**不用于训练/不共享**；KMS 加密 + 最小权限 + 审计。([AWS Documentation][1])
* **PII**：S3 静态扫描用 **Macie**，文本级涂黑用 **Comprehend**，推理期拦截用 **Guardrails**。([AWS Documentation][5])
* **规模**：CPT 经验值\*\*≥10 亿 token\*\*（示例自官方博客）；配额以文档为准。([Amazon Web Services, Inc.][9])
* **代表性**：用 **SageMaker Clarify** 度量并治理偏倚（数据前/后/线上）。([AWS Documentation][13])
* **RLHF 数据**：用 **Bedrock 人工评估**收集**评分/偏好**，沉淀到 S3，为对齐/选择模型提供依据。([AWS Documentation][18])

---

## 10) 参考（官方优先）

* **Bedrock 数据保护**（不存储/不用于训练/不共享）与加密/权限要点。([AWS Documentation][1])
* **准备训练/验证数据集（格式/配额/差异）**。([AWS Documentation][2])
* **CPT 数据格式与 Token 估算**。([AWS Documentation][3])
* **自定义模型作业 API/CLI（返回验证损失与样例输出）**。([AWS Documentation][4])
* **Guardrails（PII/敏感信息过滤）**。([AWS Documentation][8])
* **Macie（S3 敏感数据发现）**。([AWS Documentation][5])
* **Comprehend（Detect PII）**。([AWS Documentation][6])
* **SageMaker Ground Truth/Plus（标注/主动学习/流式）**。([AWS Documentation][10])
* **SageMaker Clarify（数据偏倚度量/可解释）**。([AWS Documentation][13])
* **Bedrock Evaluations（自动+人工评估）**。([Amazon Web Services, Inc.][16])
* **Knowledge Bases（RAG 工作原理与优势）**。([AWS Documentation][19])

---

### 一句话总结

> **微调成败，数据为王。**在 AWS 上，把“**策展**干净标准化、把**治理**落到 KMS/Guardrails/Macie/Comprehend、把**规模**与**代表性**校准到任务、用 **Ground Truth** 做**高质量标注**、用 **Clarify** 度量偏倚、借 **Bedrock Evaluations** 收集**人类偏好**（RLHF 基料）”，你就具备了**可考、可复用、可上线**的数据准备全链路。

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html?utm_source=chatgpt.com "Data protection - Amazon Bedrock"
[2]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-customization-prepare.html?utm_source=chatgpt.com "Prepare your training datasets for fine-tuning and ..."
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/dataset-prep-continued-pretraining.html?utm_source=chatgpt.com "Prepare datasets for continued pre-training"
[4]: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_CreateModelCustomizationJob.html?utm_source=chatgpt.com "CreateModelCustomizationJob - Amazon Bedrock"
[5]: https://docs.aws.amazon.com/macie/latest/user/data-classification.html?utm_source=chatgpt.com "Discovering sensitive data with Macie"
[6]: https://docs.aws.amazon.com/comprehend/latest/dg/how-pii.html?utm_source=chatgpt.com "Detecting PII entities - Amazon Comprehend"
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-sensitive-filters.html?utm_source=chatgpt.com "Remove PII from conversations by using sensitive information ..."
[8]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[9]: https://aws.amazon.com/blogs/aws/customize-models-in-amazon-bedrock-with-your-own-data-using-fine-tuning-and-continued-pre-training/?utm_source=chatgpt.com "Customize models in Amazon Bedrock with your own data ..."
[10]: https://docs.aws.amazon.com/sagemaker/latest/dg/sms.html?utm_source=chatgpt.com "Training data labeling using humans with Amazon SageMaker Ground Truth"
[11]: https://docs.aws.amazon.com/sagemaker/latest/dg/gtp.html?utm_source=chatgpt.com "Use Amazon SageMaker Ground Truth Plus to Label Data"
[12]: https://docs.aws.amazon.com/sagemaker/latest/dg/sms-streaming-labeling-job.html?utm_source=chatgpt.com "Ground Truth streaming labeling jobs - Amazon SageMaker AI"
[13]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-measure-data-bias.html?utm_source=chatgpt.com "Pre-training Bias Metrics - Amazon SageMaker AI"
[14]: https://aws.amazon.com/sagemaker/ai/clarify/?utm_source=chatgpt.com "Amazon SageMaker Clarify - AWS"
[15]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-explainability.html?utm_source=chatgpt.com "Evaluate, explain, and detect bias in models"
[16]: https://aws.amazon.com/bedrock/evaluations/?utm_source=chatgpt.com "Evaluate Foundation Models - Amazon Bedrock Evaluations - AWS"
[17]: https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation-automatic.html?utm_source=chatgpt.com "Creating an automatic model evaluation job in Amazon Bedrock"
[18]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-evaluation-jobs-management-create-human.html?utm_source=chatgpt.com "Create a human-based model evaluation job - Amazon Bedrock"
[19]: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-it-works.html?utm_source=chatgpt.com "How Amazon Bedrock knowledge bases work"
