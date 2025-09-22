# AIF-C01｜3.3.2 **基盤模型的微调方法（指示调优・领域适配・迁移学习・持续事前训练）**／**基盤モデルをファインチューニングするための方法**

*（以中文讲解；术语用 中・日〔カタカナ读法〕・英 三语展示）*

---

## 0) 本题你需要会什么（考试指向）

* **能“准确定义并区分”** 4 种定制方式：
  **指示调优**（シジ・チューニング / *Instruction tuning*）、**领域适配**（トクテイ・ブンヤ・テキオウ / *Domain adaptation*）、**迁移学习**（テンイ・ガクシュウ / *Transfer learning*）、**持续事前训练**（ケイゾクテキナ・ジゼン・トレーニング / *Continued pre-training*）。
* **能结合 AWS 官方**说明：在 **Amazon Bedrock** 中如何准备数据（`.jsonl` 格式）、如何**提交作业**与**监控指标**、何时优先 **RAG（ナレッジベース / Knowledge Bases）** 而非改参数。([AWS Documentation][1])

---

## 1) 术语速查（中・日〔读法〕・英）

| 概念     | 日文（读法）                               | English                      | 一句话定义                                                             |
| ------ | ------------------------------------ | ---------------------------- | ----------------------------------------------------------------- |
| 微调     | **ファインチューニング（ファインチューニング）**           | Fine-tuning                  | 用**标注**数据让 FM 在**具体任务**上更好。([AWS Documentation][2])               |
| 指示调优   | **指示のチューニング（シジ・チューニング）**             | Instruction tuning           | 用 *指令—响应* 样本，增强**遵循指令**能力（属于微调的一种）。                               |
| 领域适配   | **特定分野への適応（トクテイ・ブンヤ・テキオウ）**          | Domain adaptation            | 让模型在**特定行业/话术**下表现更佳；可走**微调**或**CPT**两条路。([AWS Documentation][3]) |
| 迁移学习   | **転移学習（テンイ・ガクシュウ）**                  | Transfer learning            | 以**已预训练的 FM**为起点，少量数据就能适应新任务（微调即为一类迁移）。                           |
| 持续事前训练 | **継続的な事前トレーニング（ケイゾクテキナ・ジゼン・トレーニング）** | Continued pre-training (CPT) | 用**未标注领域语料**继续“喂知识”，增强**领域知识与语体**。([AWS Documentation][2])        |
| 知识增强   | **ナレッジベース（ナレッジベース）**                 | Knowledge Bases (RAG)        | 不改参数，**检索+拼接**私有数据来回答问题的方案。([AWS Documentation][4])               |

---

## 2) 四种方法逐一详解（含 Bedrock 实操）

### 2.1 指示调优（Instruction tuning｜指示のチューニング）

**目标**：让模型**更好地遵循任务指令**（格式、风格、步骤）。典型任务：摘要按模板、结构化要点抽取、对话体 QA 等。
**数据形态**：**有标注**；在 Bedrock 中以 **`.jsonl`** 存放，一行一条记录。常见键：`prompt` 与 `completion`（非会话），或与 **Converse** 消息结构对齐（会话）。([AWS Documentation][1])

**示例（非会话模板）**：

```json
{"prompt":"请把句子分类：『我想退货，怎么走流程？』","completion":"类别=退款/售后"}
{"prompt":"将文本提炼为JSON，字段：问题、答案、出处。文本=『……』","completion":"{\"问题\":\"…\",\"答案\":\"…\",\"出处\":\"…\"}"}
```

**在 Bedrock 的流程**：

1. **准备数据**：按所选基础模型与方法的格式生成 `.jsonl` 并上传 **S3**；
2. **提交作业**：控制台或 API（`CreateModelCustomizationJob`）选择 **Fine-tuning**、指定 S3 数据、超参（epochs、batch size 等）、KMS 等；
3. **监控**：作业结束返回**验证损失**与**样例输出**，用于判断是否需要调整超参或数据。([AWS Documentation][1])

**何时优先**：

* 需要**稳定可复现**的结构化输出；
* 仅靠 Prompt 或 RAG 难以把控**格式与一致性**。
  **常见误区**：样本结构不一致、输出格式未锁定（可用 *模式/示例* 统一）。
  **实践参考**：AWS 针对不同模型族的微调经验与注意事项（如样本质量、超参调优）。([Amazon Web Services, Inc.][5])

---

### 2.2 领域适配（Domain adaptation｜特定分野への適応）

**目标**：让模型“说行业的话、按行规办事”。
**两条技术路径**：

* **监督式微调**（有标注）：如果你有“问→答/输入→输出”的**标注**数据（法规问答、工单意图、模板化摘要），走**Fine-tuning**。([AWS Documentation][2])
* **无标注语料**（知识补钙）：若你有大量**行业语料但未标注**（制度、手册、报告），走 **CPT**（见 2.4）。([AWS Documentation][3])

**小例子**：

* **医疗机构**：把“出院小结 → 结构化字段（诊断、药嘱、复诊）”做**监督式微调**；同时把**指南/术语表**做 **CPT**，让模型既会**按字段输出**又**懂医学表述**。

---

### 2.3 迁移学习（Transfer learning｜転移学習）

**本质**：以**预训练 FM**为底座，通过**少量**新数据快速“迁移”到新任务/新域；在大模型时代，**微调**就是典型的迁移学习形态。
**为什么重要**：成本与数据需求大幅低于从零训练；同时继承底座的语言/推理能力。
**在 Bedrock 的体现**：你**选定一个基础模型**（如 Amazon Nova/Titan/合作方模型），再通过 **Fine-tuning 或 CPT** 创建**自定义模型**并直接推理使用。([AWS Documentation][3])

---

### 2.4 持续事前训练（CPT｜継続的な事前トレーニング）

**目标**：**不要求标注**，用**海量领域文本**增强**背景知识与语体**；适合“想让模型更懂我们行业”的场景。
**数据形态**：`.jsonl`，每行只有 **`input`** 字段；**以 6 字符 ≈ 1 token** 粗估数据规模。示例：

```json
{"input":"贵行跨境结算-操作规程（2023版）第六章……"}
{"input":"证券业协会《投资研究行为准则》条款……"}
```

（官方明确 *CPT* 为**未标注数据**；每行仅 `input`。）([AWS Documentation][6])

**规模经验**：早期官方博客举例“最多 100,000 条记录；通常\*\*≥ 十亿 token\*\*能看到明显增益”（经验值，视模型/领域而定）。([Amazon Web Services, Inc.][7])

**何时优先**：

* 你有**大量未标注资料**，且目标是**提升行业知识/文风**而非特定打分任务；
* 搭配少量微调，可同时获得**会说**且**会做**。([AWS Documentation][3])

**提交流程与监控**：与微调相同（控制台/API、S3、KMS、超参）；作业完成后同样有**训练/验证指标**供分析。([AWS Documentation][3])

---

## 3) 与 RAG／混合方案的取舍（考试爱考）

| 诉求             | 首选                       | 理由/依据                                                                       |
| -------------- | ------------------------ | --------------------------------------------------------------------------- |
| **仅需“用上私有知识”** | **RAG（Knowledge Bases）** | **不改模型参数**、上线快、更新灵活、成本低；Bedrock KB 为 RAG 提供托管组件。([AWS Documentation][4])    |
| **需要稳定结构化输出**  | **微调（含指示调优）**            | 学到“输入→输出”映射，格式稳定。([AWS Documentation][2])                                   |
| **需要增强行业底座知识** | **CPT**                  | 大量未标注文本即可显著提升**领域语体/术语**。([AWS Documentation][3])                           |
| **更高质量**       | **微调 + RAG（或 + CPT）**    | 官方案例：**微调 与 RAG 组合显著提升**回答质量（以 Nova 为例的对比）。([Amazon Web Services, Inc.][8]) |

---

## 4) Bedrock 上的端到端操作要点

### 4.1 数据准备（官方要求）

* **统一用 `.jsonl`**：一行一条 JSON；**格式依方法/模型而异**（微调常见 `prompt`/`completion`；CPT 为 `input`）。([AWS Documentation][1])
* **放入 S3**（训练、验证、输出各自路径），记录大小/长度需符合所选模型限制。([AWS Documentation][1])

### 4.2 提交与监控

* **提交作业**：控制台或 API/CLI（`CreateModelCustomizationJob` / `create-model-customization-job`）。需指定 `baseModelIdentifier`、训练/验证数据 S3、超参、输出位置与 **KMS**。([AWS Documentation][9])
* **时长影响因子**：**记录数、输入/输出 token 数、epoch、batch size** 等。([AWS Documentation][10])
* **结果产物**：作业完成后，返回**验证损失与样例输出**并在输出桶可见，用于评估与调参。([AWS Documentation][9])

---

## 5) 评估与最佳实践（精简必记）

* **离线评估**：用独立验证集；对生成类看**一致性/格式合规**，对判定类看**准确率/F1**。
* **线上监控**：灰度 + A/B；关注延迟、费用、越狱/注入风险（建议绑定 **Guardrails**）。([Amazon Web Services, Inc.][11])
* **数据质量优先于数量**：AWS 微调经验强调**清洗、模式对齐与高质量样本**比堆量更关键。([Amazon Web Services, Inc.][5])

---

## 6) 三个“带感”的实战小例子

1. **客服意图 → 标签（任务型）**

   * 方法：**指示调优/微调**。
   * 数据：`prompt`=用户话术，`completion`=标准化标签。
   * 结果：推理稳定、延迟低，省去复杂 Prompt。([AWS Documentation][1])

2. **法律写作风格（知识+风格）**

   * 方法：**CPT** 喂法规/判例/合同语料 → 学会“法律腔”；
   * 若还要求**固定模板**（章、条、款），再加**指示调优**。([AWS Documentation][6])

3. **企业 FAQ 快速上线**

   * 方法：**RAG（Knowledge Bases）** 连接私有文档；
   * 若仍有**格式/稳定性**要求，在 RAG 之外再做**轻量微调**（混合方案）。([AWS Documentation][4])

---

## 7) “一眼选法”速记（应试友好）

* **能用 RAG 先 RAG**（上线快/可溯源/低成本）。([AWS Documentation][4])
* **要稳定格式/任务达标 → 微调（指示调优）**。([AWS Documentation][2])
* **要补行业底座 → CPT**（未标注语料、每行 `input`，6 字符≈1 token）。([AWS Documentation][6])
* **想要最强效果 → RAG +（微调 / CPT）**。([Amazon Web Services, Inc.][8])
* **提交作业三件套**：S3 `.jsonl` → `CreateModelCustomizationJob` → 看**验证损失/样例输出**。([AWS Documentation][1])

---

## 8) 参考（AWS 官方为主）

* **定制方式总览（Fine-tuning & CPT）**：Bedrock 用户指南。([AWS Documentation][2])
* **数据准备（`.jsonl` 规范）**：适配不同方法与模型；含格式示例。([AWS Documentation][1])
* **CPT 数据格式**：每行 `input`；6 字符≈1 token。([AWS Documentation][6])
* **CPT 规模经验**：早期官方博客的经验值与实践。([Amazon Web Services, Inc.][7])
* **提交/耗时因素/监控**：提交与作业时长说明；API/CLI 参考；返回验证损失与输出示例。([AWS Documentation][10])
* **RAG（Knowledge Bases）**：工作原理与“无需持续训练”的优势。([AWS Documentation][4])
* **混合增益案例**：微调 + RAG 组合带来更高质量（Nova 案例）。([Amazon Web Services, Inc.][8])
* **微调最佳实践**：数据清洗、超参与评估建议。([Amazon Web Services, Inc.][5])
* **Guardrails 安全护栏**：把安全与合规“平台化”。([Amazon Web Services, Inc.][11])

---

### 一句话收尾

> **在 AWS 上定制 FM 的“正确姿势”**：**RAG** 解决“会用知识”，**微调**解决“会做任务”，**CPT**解决“更懂行业”，三者可叠加；**数据按 `.jsonl` 规范入 S3**，用 `CreateModelCustomizationJob` 提交并依据**验证损失**迭代，**Guardrails** 兜底安全与合规。([AWS Documentation][1])

如果你愿意，我可以把以上内容再拆成**对比表+流程图**版，便于临考速记。

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-customization-prepare.html?utm_source=chatgpt.com "Prepare your training datasets for fine-tuning and ..."
[2]: https://docs.aws.amazon.com/bedrock/latest/userguide/custom-model-fine-tuning.html?utm_source=chatgpt.com "Customize a model with fine-tuning or continued pre- ..."
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/custom-models.html?utm_source=chatgpt.com "Customize your model to improve its performance for ..."
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-it-works.html?utm_source=chatgpt.com "How Amazon Bedrock knowledge bases work"
[5]: https://aws.amazon.com/blogs/machine-learning/best-practices-and-lessons-for-fine-tuning-anthropics-claude-3-haiku-on-amazon-bedrock/?utm_source=chatgpt.com "Best practices and lessons for fine-tuning Anthropic's ..."
[6]: https://docs.aws.amazon.com/bedrock/latest/userguide/dataset-prep-continued-pretraining.html?utm_source=chatgpt.com "Prepare datasets for continued pre-training"
[7]: https://aws.amazon.com/blogs/aws/customize-models-in-amazon-bedrock-with-your-own-data-using-fine-tuning-and-continued-pre-training/?utm_source=chatgpt.com "Customize models in Amazon Bedrock with your own data ..."
[8]: https://aws.amazon.com/blogs/machine-learning/model-customization-rag-or-both-a-case-study-with-amazon-nova/?utm_source=chatgpt.com "Model customization, RAG, or both: A case study with ..."
[9]: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_CreateModelCustomizationJob.html?utm_source=chatgpt.com "CreateModelCustomizationJob - Amazon Bedrock"
[10]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-customization-submit.html?utm_source=chatgpt.com "Submit a model customization job for fine-tuning or continued ..."
[11]: https://aws.amazon.com/blogs/security/safeguard-your-generative-ai-workloads-from-prompt-injections/?utm_source=chatgpt.com "Safeguard your generative AI workloads from prompt ..."
