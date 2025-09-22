# AIF-C01｜3.3.1 **基盤モデル训练的重要要素（事前训练・微调・持续事前训练）/ 基盤モデルのトレーニングの重要な要素**

*（以中文讲解｜中-日〔カタカナ〕-英 三语术语）*

---

## 0) 这题要你会什么（考试瞄准）

* 能**区分**三种训练方式：**事前训练**（ジゼン・トレーニング / Pre-training）、**微调**（ファインチューニング / Fine-tuning）、**持续事前训练**（ケイゾクテキナ・ジゼン・トレーニング / Continued Pre-training, CPT）。
* 能**说清各自目的、数据要求、何时选用、在 AWS Bedrock 上如何落地**（含数据格式、提交流程、加密与权限）。([AWS Documentation][1])
* 知道**RAG**（ナレッジベース / Knowledge Bases）何时可替代训练，降低成本和风险。([AWS Documentation][2])

---

## 1) 术语速查（中・日〔读法〕・英）

| 概念         | 日文（读法）                               | English                      | 一句话理解                                                       |
| ---------- | ------------------------------------ | ---------------------------- | ----------------------------------------------------------- |
| 事前训练       | **事前トレーニング（ジゼン・トレーニング）**             | Pre-training                 | 用超大规模通用语料学“基础能力”，通常由模型提供方完成。                                |
| 微调         | **ファインチューニング（ファインチューニング）**           | Fine-tuning                  | 用**标注**数据让模型在**具体任务**上表现更好。([AWS Documentation][3])         |
| 持续事前训练     | **継続的な事前トレーニング（ケイゾクテキナ・ジゼン・トレーニング）** | Continued Pre-training (CPT) | 用**未标注的领域语料**继续“喂知识”，增强**领域知识**与风格。([AWS Documentation][1]) |
| 知识增强       | **ナレッジベース（ナレッジベース）**                 | Knowledge Bases (RAG)        | 不改参数，**检索+拼接上下文**提升回答的“有据可依”。([AWS Documentation][4])       |
| Guardrails | **ガードレール（ガードレール）**                   | Guardrails                   | 输入/输出安全护栏，含**提示攻击**检测、PII 过滤等。([AWS Documentation][5])      |

---

## 2) 三种训练方式：目的、数据、何时用

### 2.1 事前训练（Pre-training｜事前トレーニング）

* **谁来做**：一般由模型提供方完成（如 Amazon、合作伙伴）。作为客户，你直接使用已训练好的 **FM（Foundation Model）**。([AWS Documentation][6])
* **作用**：让模型拥有通用语言/推理/知识表征能力。
* **考试说法**：**客户通常不做**从零训练；在 Bedrock 里，你做的是**微调**或**持续事前训练**，或干脆用 **RAG**。([AWS Documentation][1])

### 2.2 微调（Fine-tuning｜ファインチューニング）

* **目的**：在**明确任务**（如分类、抽取、指令遵循）上提升效果。
* **数据**：**有标注**（prompt → completion），**JSONL** 每行一条记录。常见键：`prompt` 与 `completion`（非会话任务），或按 **Message/Converse 格式**（会话任务）。([AWS Documentation][7])
* **何时用**：

  * 需要**稳定、可复现**的任务性能；
  * Prompt 工程与 RAG 不足以达标。
* **Bedrock 支持**：控制台或 API 通过 **CreateModelCustomizationJob** 提交；可调 **epochs、batch size** 等超参；作业完成返回**验证损失与样例输出**。([AWS Documentation][8])
* **模型可选**：不同时间支持的 FM 会更新（如 Amazon Nova/Titan 等）；考试记“**Bedrock 支持对若干 FM 微调**”。([AWS Documentation][9])

**小例子**

* 你要做**中文客服意图分类**：准备 5–10 万条 `{"prompt": "用户: …", "completion": "意图=退款"}`，微调后接口直接输出意图标签，更稳更快；RAG 在此类**判定任务**帮助有限。

### 2.3 持续事前训练（CPT｜継続的な事前トレーニング）

* **目的**：把模型“继续喂养”你行业的**未标注**语料（法规、手册、代码库等），提升**领域知识/术语**与**语体**。
* **数据**：**无标注**，**JSONL** 每行仅含 `input`：

  ```json
  {"input":"贵行跨境结算—操作规程…"}
  {"input":"第十四条 风险管理…"}
  ```

  官方建议按字符粗估 token（约 **6 字符 ≈ 1 token**），便于预估规模。([AWS Documentation][10])
* **何时用**：

  * 你希望**整体提升**领域知识，而不仅是完成某个任务；
  * 你有大量**未标注**内部材料。
* **安全与托管**：在 **Bedrock** 的**托管环境**中运行，可用**自管 KMS 密钥**保障数据与产物加密。([Amazon Web Services, Inc.][11])

**小例子**

* 你是一家**证券公司**：把招股书、研报、内部风控手册做 CPT，模型更懂“券商话术”；再配合少量微调数据实现**问答风格/表格抽取**的稳定输出。

---

## 3) “选型矩阵”：微调 vs 持续事前训练 vs RAG

| 诉求                      | 推荐选择                  | 理由                                                              |
| ----------------------- | --------------------- | --------------------------------------------------------------- |
| **任务型**（抽取/分类/指令遵循）     | **微调**                | 有标注示例 → 学到**输入→输出**映射；上线后**免长 Prompt**。([AWS Documentation][3]) |
| **知识型**（行业术语/写作风格/专业背景） | **CPT**               | 未标注大语料，增强**领域知识与语体**，保持通用性较好。([AWS Documentation][1])           |
| **仅需要“用到私有知识”**         | **RAG（知识库）**          | **不改模型参数**，按需检索拼接，**更经济**且更新灵活。([AWS Documentation][2])         |
| **安全与合规**               | **Guardrails + 数据加密** | 统一输入/输出安全策略；训练/产物支持 **KMS** 加密。([AWS Documentation][5])         |

> 小结：能用 **RAG** 先用 RAG；任务稳定性不够再**微调**；要补“行业知识底座”做 **CPT**。这三者经常**组合**使用。

---

## 4) 在 Amazon Bedrock 上如何落地（从数据到作业）

### 4.1 数据准备（格式与示例）

**微调（非会话任务）**

* `.jsonl`：每行一条 `prompt` / `completion`

```json
{"prompt":"把句子归类：『我想退货』","completion":"退款/售后"}
{"prompt":"把句子归类：『怎么开电子发票』","completion":"发票/索取"}
```

> 官方说明：非会话任务使用 `prompt`+`completion`；会话类可用消息格式（与 Converse API 对齐）。([AWS Documentation][7])

**持续事前训练（CPT）**

* `.jsonl`：每行仅 `input`；用于**无标注**领域语料。([AWS Documentation][10])

**通用要求**

* 训练/验证集放在 **S3**；**一行一个 JSON**；不同模型/方法格式略有差异，以官方“准备数据集”文档为准。([AWS Documentation][12])

### 4.2 提交流程（控制台 & API）

**控制台**（概要）

1. 进入 **Bedrock → Custom models**；选择 **Fine-tuning** 或 **Continued Pre-training**。
2. 指定 **S3 数据位置**、**超参**（epochs、batch size 等）、**输出位置**与 **KMS**。
3. 选择/创建 **服务角色**（有 S3/KMS/VPC 权限）。提交作业。([AWS Documentation][8])

**API（Boto3）**

* 关键调用：`CreateModelCustomizationJob`，字段包含 `baseModelIdentifier`、`trainingDataConfig`（S3 URI）、`hyperParameters`、`outputDataConfig`、`customModelKmsKeyId` 等。([AWS Documentation][13])
* 监控：`GetModelCustomizationJob` / `ListModelCustomizationJobs` 查看状态与指标；作业完成会提供**验证损失**与样例输出。([Boto3][14])

> 作业时长与**记录数/输入输出 token 数、epoch、batch size**有关。([AWS Documentation][8])

### 4.3 安全与合规（必答点）

* **数据与产物加密**：训练/验证/输出、定制模型支持 **SSE-S3 或 SSE-KMS**，可指定**自管 KMS Key**；需要在 **Key Policy** 中允许 Bedrock 解密/产生日志所需权限（`kms:Decrypt`, `kms:GenerateDataKey` 等）。([AWS Documentation][15])
* **最小权限**：作业的 **IAM Role** 需最小读写 **S3**、必要的 **KMS** 与（可选）**VPC** 权限；调用者需有 `iam:PassRole`。([AWS CLI Command Reference][16])
* **平台侧隐私**：Bedrock 提供**安全与合规**能力（监控、审计），数据保护遵循共享责任模型。([Amazon Web Services, Inc.][17])
* **生成安全**：在训练前后都可为应用**绑定 Guardrails**（含 Prompt Attack 过滤）以抵御越狱/注入。([AWS Documentation][5])

---

## 5) 超参与评估：怎么“调得更好”

* **epochs / batch size / learning rate**：数据越小越要**少 epoch**，避免过拟合；CPT 用未标注数据规模可更大，但仍要监控验证损失。([AWS re\:Invent 2025][18])
* **验证集**：务必准备**独立验证集**，使用 **Bedrock 返回的验证损失**与样例输出来判定是否早停或回滚。([AWS Documentation][13])
* **离线评估 + 线上监控**：

  * 先用离线集（准确率、F1、BERTScore/ROUGE 等）评估；
  * 上线用 A/B + 质量抽检；**CloudWatch/CloudTrail** 记录与审计。([Amazon Web Services, Inc.][17])
* **搭配 Guardrails/Evals**：在推理前后过滤不当内容与注入尝试，维持质量与合规。([AWS Documentation][5])

---

## 6) 场景化理解（三个典型案例）

1. **金融意图识别（任务型）**

   * **做法**：微调。几万到十万级标注样本，`prompt`=用户话术，`completion`=类别。
   * **收益**：比长 Prompt 或少样本示例更稳、时延更低。([AWS Documentation][3])

2. **法律行业写作风格（知识型）**

   * **做法**：CPT。无标注的判例/合同/条例若干 GB；再配少量指令微调，获得“既懂法又会按模板写”的效果。([AWS Documentation][10])

3. **企业 FAQ/手册快速上线（知识驱动）**

   * **做法**：先上 **Knowledge Bases（RAG）**，把私有文档接入检索；按需再做微调或 CPT。**优势**：无需持续训练就能利用私有数据，**性价比高**。([AWS Documentation][2])

---

## 7) “一眼选法”口诀（应试友好）

* **能用 RAG 先 RAG**，更新快、便宜、可溯源。([AWS Documentation][2])
* **精任务 → 微调**；**补底座知识 → CPT**。([AWS Documentation][1])
* 提交作业要 **S3 数据（JSONL）+ 超参 + 角色/KMS**；完结看**验证损失**。([AWS Documentation][12])
* 上线别忘 **Guardrails**（含 Prompt Attack）与**加密/审计**。([AWS Documentation][5])

---

## 8) 参考（AWS 官方文档为主）

* **Bedrock 定制方式总览**（微调 & CPT）([AWS Documentation][1])
* **准备数据（JSONL）**：微调与 CPT、文本任务格式示例([AWS Documentation][12])
* **提交作业（控制台 & API）**：耗时因素、必要字段、监控与返回指标([AWS Documentation][8])
* **RAG/知识库**：如何工作、为什么可减少“重新训练”的需要([AWS Documentation][2])
* **安全**：Guardrails（含提示攻击过滤）、数据保护与加密、监控与审计([AWS Documentation][5])
* **模型家族示例**：Nova 模型微调指引（了解支持面）([AWS Documentation][9])

---

### 一句话收尾

> \*\*在 AWS 上定制 FM 的关键是“选对手段 + 准备好数据 + 管好安全”。\*\*任务型选微调，知识型选 CPT，能用 RAG 先 RAG；数据按 JSONL 规范放 S3，KMS 加密、最小权限与 Guardrails 贯穿全程。([AWS Documentation][12])

需要我把这份讲义再拆成\*\*“思维导图 + 10 题单选”\*\*的复习包吗？我可以直接按这一版出题和给标准答案。

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/custom-model-fine-tuning.html?utm_source=chatgpt.com "Customize a model with fine-tuning or continued pre- ..."
[2]: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-it-works.html?utm_source=chatgpt.com "How Amazon Bedrock knowledge bases work"
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/custom-models.html?utm_source=chatgpt.com "Customize your model to improve its performance for ..."
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html?utm_source=chatgpt.com "Retrieve data and generate AI responses with ..."
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[6]: https://docs.aws.amazon.com/bedrock/?utm_source=chatgpt.com "Amazon Bedrock Documentation"
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/preparing-text-data.html?utm_source=chatgpt.com "Prepare data for fine-tuning text-to-text models"
[8]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-customization-submit.html?utm_source=chatgpt.com "Submit a model customization job for fine-tuning or continued ..."
[9]: https://docs.aws.amazon.com/nova/latest/userguide/customize-fine-tune.html?utm_source=chatgpt.com "Fine-tuning Amazon Nova models"
[10]: https://docs.aws.amazon.com/bedrock/latest/userguide/dataset-prep-continued-pretraining.html?utm_source=chatgpt.com "Prepare datasets for continued pre-training"
[11]: https://aws.amazon.com/blogs/aws/customize-models-in-amazon-bedrock-with-your-own-data-using-fine-tuning-and-continued-pre-training/?utm_source=chatgpt.com "Customize models in Amazon Bedrock with your own data ..."
[12]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-customization-prepare.html?utm_source=chatgpt.com "Prepare your training datasets for fine-tuning and ..."
[13]: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_CreateModelCustomizationJob.html?utm_source=chatgpt.com "CreateModelCustomizationJob - Amazon Bedrock"
[14]: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/bedrock/client/get_model_customization_job.html?utm_source=chatgpt.com "get_model_customization_job - Boto3 1.40.25 documentation"
[15]: https://docs.aws.amazon.com/bedrock/latest/userguide/encryption-custom-job.html?utm_source=chatgpt.com "Encryption of custom models - Amazon Bedrock"
[16]: https://awscli.amazonaws.com/v2/documentation/api/2.14.0/reference/bedrock/create-model-customization-job.html?utm_source=chatgpt.com "create-model-customization-job - bedrock"
[17]: https://aws.amazon.com/bedrock/security-compliance/?utm_source=chatgpt.com "Amazon Bedrock Security and Privacy - AWS"
[18]: https://reinvent.awsevents.com/content/dam/reinvent/2024/slides/aim/AIM357_Customizing-models-for-enhanced-results-Fine-tuning-in-Amazon-Bedrock.pdf?utm_source=chatgpt.com "Fine-tuning in Amazon Bedrock"
