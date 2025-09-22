# AIF-C01｜3.4.1 **基盤模型性能的评估方法（人工评估・基准数据集 等）**／**基盤モデルのパフォーマンスを評価する手法**

*（中文为主；术语三语：中・日〔カタカナ读法〕・英）*

---

## 0) 本题你要掌握什么（考试指向）

* 能**区分与应用**两大评估路径：**自动/程序化评估**（オートマチック・エバリュエーション｜*Automatic/Programmatic evaluation*）与**人工评估**（ヒューマン・エバリュエーション｜*Human evaluation*）。
* 知道 **Amazon Bedrock Evaluations** 的**评估对象**（FM、KB/RAG、外部模型）、**数据输入（Prompt Dataset）**、**内置指标**（正確度/ロバスト性/有害性｜*Accuracy/Robustness/Toxicity*）与**RAG 专用评估**（检索/生成分开测）。([Amazon Web Services, Inc.][1])
* 会按**官方流程**在**控制台/CLI/API**创建评估作业、查看报告，并知道**可比较两个推理来源**（两种模型或一个模型+自带推理结果）。([AWS Documentation][2])

---

## 1) 关键术语（中・日〔读法〕・英）

| 概念      | 日文（读法）                  | English                           | 一句话理解                                      |
| ------- | ----------------------- | --------------------------------- | ------------------------------------------ |
| 评估      | **評価（ヒョウカ）**            | Evaluation                        | 用数据与指标系统衡量模型/KB 的好坏                        |
| 自动评估    | **自動評価（ジドウ・ヒョウカ）**      | Automatic/Programmatic evaluation | 由系统用**内置/自定义指标**自动打分                       |
| 人工评估    | **人手評価（ニンテ・ヒョウカ）**      | Human evaluation                  | 人类审阅员对回答**打分/比较**                          |
| 评估作业    | **評価ジョブ（ヒョウカ・ジョブ）**     | Evaluation job                    | 在 Bedrock 发起的一次评估运行                        |
| 评估数据集   | **プロンプト・データセット**        | Prompt dataset                    | 一组**提示/真值/检索片段/推理结果**等条目                   |
| 内置指标    | **内蔵メトリクス（ナイゾウ・メトリクス）** | Built-in metrics                  | *Accuracy / Robustness / Toxicity* 等官方预置指标 |
| 评审模型    | **ジャッジ・モデル**            | LLM-as-a-Judge                    | 由模型对输出**按指标评分**的评估方式                       |
| 基准数据集   | **ベンチマーク・データセット**       | Benchmark dataset                 | 官方或自定义的**标准测试集**                           |
| 知识库/RAG | **ナレッジベース（RAG）**        | Knowledge Bases (RAG)             | 可做**检索专测/检索+生成联测**                         |

> 以上术语与能力均来自 **Amazon Bedrock Evaluations** 官方说明与 API 参考。([Amazon Web Services, Inc.][1])

---

## 2) 评估全景：你可以评什么、怎么评

### 2.1 评估对象（What）

* **基础模型（FM）**：任何 Bedrock 中的模型；也可把**外部模型的推理结果**放进数据集来评。([Amazon Web Services, Inc.][1])
* **RAG/知识库（KB）**：既可评**检索正确性**，也可评**检索+生成**整体表现。([AWS Documentation][3])

### 2.2 评估方式（How）

* **自动评估（程序化）**：用**内置数据集或自带数据集**，系统计算指标；常见内置指标：**Accuracy / Robustness / Toxicity**。([Amazon Web Services, Inc.][1])
* **人工评估**：配置审阅工作流，在 Web UI 中由审阅员对回答**打分、二选一、排序**等；结果以**报告卡**形式汇总并存 S3。([AWS Documentation][4])

---

## 3) 数据：Prompt Dataset（プロンプト・データセット）的组成

### 3.1 通用结构（FM 评估）

* 自动评估需要**Prompt Dataset**（可用官方内置或自带）。内容包含\*\*提示（prompt）\*\*与（可选）**真值答案（ground truth）**；系统用它对选定模型跑推理并计算指标。([AWS Documentation][5])

**示例（JSONL，节选）**

```json
{"prompt":"請總結下文重點：……","ground_truth":"三條要點：A/B/C"}
{"prompt":"將問題分類為{退款,發票,投訴}：……","ground_truth":"退款"}
```

### 3.2 RAG 专用结构

* \*\*Retrieve-only（只评检索）\*\*与 **Retrieve-and-Generate（评检索+生成）**两类；数据集**JSONL**，**最多 1000 条**。([AWS Documentation][6])

**Retrieve-only 必备键（示意）**：`prompt`、`ground_truth`、`knowledgeBaseId`（或目标 KB 信息）等。
**Retrieve-and-Generate 额外键**：可包含**模型回答/自带推理结果**、**被检索到的片段**等，便于对\*\*有据可依（grounding）\*\*打分。([AWS Documentation][7])

---

## 4) 指标（メトリクス）：看什么才算好

### 4.1 官方内置指标（Built-in）

* **正確度（セイカクド）｜Accuracy**：回答与真值吻合度（分类/抽取/QA 常用）。
* **ロバスト性｜Robustness**：对**对抗/扰动**输入的稳定性。
* **有害性（ユウガイセイ）｜Toxicity**：内容安全相关的有害度检测。

> 以上为 Bedrock Evaluations 的**内置可选指标**，用于自动评估作业。([Amazon Web Services, Inc.][1])

### 4.2 评审模型指标（Judge-based）

* 在 **Judge-based** 评估中，你可选择评审模型与一组指标（也可自定义标准/打分规则），如**有用性/真实性/遵循性/格式合规**等。官方文档说明可**选择内置或自定义指标**。([AWS Documentation][8])

### 4.3 RAG 专项指标

* Bedrock 能计算 **检索正确性**（是否命中应有片段）与**生成正确性/有据可依**等，以评估**KB 与 RAG**的有效性。([AWS Documentation][3])

---

## 5) 人工评估（Human Evaluation｜人手評価）

### 5.1 能力与场景

* 适合**主观/细腻**标准（语气、礼貌、合规、可读性）或**多目标权衡**（例如“有用但潜在风险”）。
* 可一次对**两个推理来源**进行**并排对比**（例如 A 模型 vs B 模型）。([AWS Documentation][4])

### 5.2 工作流与产出

* 控制台创建**人手评估作业**，配置**评审问题/评分方法**；评审员在 UI 中完成打分；系统生成**报告卡**（包含指标描述、评分方法、分布可视化），并把原始结果写入 **S3**。([AWS Documentation][4])

---

## 6) 如何在 Bedrock 发起评估（控制台/CLI/API）

### 6.1 自动评估（FM 或 RAG）

1. **准备数据集**：选择内置或自制 **Prompt dataset**（RAG 评估遵从对应键，最多 1000 条）。([AWS Documentation][5])
2. **创建作业**：控制台或 **CLI/SDK**（`CreateEvaluationJob` / `create_evaluation_job`），指定**模型/KB**、**指标**、**数据集 S3**、**服务角色**等。([AWS Documentation][2])
3. **查看报告**：在控制台查看**自动评估报告**（分数、分布、样例）；或编程方式读取。([AWS Documentation][9])

### 6.2 人工评估

1. **创建人手评估作业**：设定**评审问题/评分方式**与**推理来源**（可最多 2 个来源，包括**外部模型**的结果）。([AWS Documentation][4])
2. **收集结果**：报告卡汇总图表+每题原始评分（S3 JSON）。([AWS Documentation][10])

> **权限**：自动评估需要**服务角色（IAM）**；详见“创建自动评估作业”的 IAM 前提。([AWS Documentation][2])

---

## 7) 三个“手把手”小例子

### 例 1：**选择客服意图分类的最佳 FM（自动评估）**

* **数据**：自制 5,000 条 `prompt/ground_truth` 分类样本。
* **操作**：用内置指标 **Accuracy/Robustness/Toxicity**，对比两款模型。
* **结论**：A 模型 Accuracy 高但 Toxicity 略升；结合业务更看重安全，可选 B。([Amazon Web Services, Inc.][1])

### 例 2：**评估企业 KB 的“检索是否靠谱”（RAG：Retrieve-only）**

* **数据**：构建 **Retrieve-only** Prompt dataset（最多 1000 条），包含问题、期望出处与 KB 标识。
* **操作**：发起 RAG 检索评估作业；查看**命中率/相关性**。
* **收获**：发现某类产品手册命中低 → 调整**切片/嵌入/Top-k**。([AWS Documentation][6])

### 例 3：**当标准很主观：人工评估“法律文书的合规语感”**

* **数据**：挑选 300 条真实问题；让两种方案（A=RAG、B=RAG+微调）分别生成答案。
* **操作**：创建**人手评估作业**，设置“合规性/完整性/措辞”3 个问项；并排比较 A/B。
* **收获**：B 在“措辞”得分显著更高，支持上线 B 方案。([AWS Documentation][4])

---

## 8) 与“基准数据集”的关系（考试常问）

* Bedrock 提供**内置 Prompt Datasets**（等同“基准数据集”思路）可直接用于自动评估；你也可以**自带数据**贴合业务。([AWS Documentation][5])
* 对 **RAG**，官方提供了**检索/检索+生成**两类评估模版与**数据字段规范**（JSONL、≤1000 条）。([AWS Documentation][6])

---

## 9) 实务与治理提示

* **与安全联动**：评估数据同样受**数据保护/加密/最小权限**约束（S3+KMS+IAM）。([AWS Documentation][11])
* **外部模型也能评**：把外部推理结果写进数据集字段即可在 Bedrock 统一比较。([AWS Documentation][4])
* **自定义指标**：当内置指标不足时，可在 Evaluations 中**定义自定义指标**（扩展 LLM-as-a-Judge 逻辑），以业务语言做评分。([Amazon Web Services, Inc.][12])

---

## 10) “拿分速记”（问→答）

* **问：Bedrock 能评什么？**
  **答**：评 **FM、KB/RAG**，甚至**外部模型**（自带推理结果）。([Amazon Web Services, Inc.][1])
* **问：自动评估有哪些内置指标？**
  **答**：**Accuracy / Robustness / Toxicity**。([Amazon Web Services, Inc.][1])
* **问：RAG 如何评？**
  **答**：有 **Retrieve-only** 与 **Retrieve-and-Generate** 两种；**JSONL 数据集，≤1000 条**。([AWS Documentation][6])
* **问：什么时候用人工评估？**
  **答**：主观/细腻维度（语气、礼貌、合规、偏好）或需要**A/B 并排比较**时。([AWS Documentation][10])
* **问：如何开始？**
  **答**：准备 **Prompt dataset** → 控制台/CLI **CreateEvaluationJob** → 查看报告；需要**IAM 服务角色**。([AWS Documentation][5])

---

## 参考（AWS 官方优先）

* **Bedrock Evaluations 总览**：用途、对象、RAG/模型评估能力。([Amazon Web Services, Inc.][1])
* **自动评估：创建作业（控制台/CLI/SDK）** 与 IAM 前提。([AWS Documentation][2])
* **人手评估：创建与对比两个推理来源**；**报告卡**与 S3 结果。([AWS Documentation][4])
* **Prompt Dataset：内置与自定义**。([AWS Documentation][5])
* **RAG 评估数据：Retrieve-only / Retrieve-and-Generate、≤1000 条**。([AWS Documentation][6])
* **内置指标与自定义指标**。([AWS Documentation][13])
* **外部模型也能评**（自带推理结果）。([AWS Documentation][4])
* **数据保护与加密（通用）**。([AWS Documentation][11])

---

### 一句话总结

> **评估 = 数据集 + 指标 + 流程**。在 **Amazon Bedrock Evaluations** 中，用**内置或自带数据集**对 **FM 与 KB/RAG** 做**自动评估**（Accuracy/Robustness/Toxicity 等）或**人工评估**（主观质量与 A/B 比较），必要时**自定义指标**；这样你就能**可重复、可审计、可比较**地为生产环境选型与持续改进提供依据。([Amazon Web Services, Inc.][1])

如果你愿意，我可以把 RAG 两类评估的数据集**JSONL 模板**和**CLI 脚本骨架**也整理给你，直接套用即可。

[1]: https://aws.amazon.com/bedrock/evaluations/?utm_source=chatgpt.com "Evaluate Foundation Models - Amazon Bedrock Evaluations"
[2]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-evaluation-jobs-management-create.html?utm_source=chatgpt.com "Starting an automatic model evaluation job in Amazon Bedrock"
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation.html?utm_source=chatgpt.com "Evaluate the performance of Amazon Bedrock resources"
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-evaluation-jobs-management-create-human.html?utm_source=chatgpt.com "Create a human-based model evaluation job"
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-evaluation-prompt-datasets.html?utm_source=chatgpt.com "Use prompt datasets for model evaluation in ..."
[6]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-evaluation-prompt-retrieve.html?utm_source=chatgpt.com "Create a prompt dataset for retrieve-only RAG evaluation ..."
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-evaluation-prompt.html?utm_source=chatgpt.com "Create a prompt dataset for a RAG evaluation in ..."
[8]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-evaluation-metrics.html?utm_source=chatgpt.com "Use metrics to understand model performance"
[9]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-evaluation-report-programmatic.html?utm_source=chatgpt.com "Review metrics for an automated model evaluation job in ..."
[10]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-evaluation-report-human-customer.html?utm_source=chatgpt.com "Review a human-based model evaluation job in ..."
[11]: https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html?utm_source=chatgpt.com "Data protection - Amazon Bedrock"
[12]: https://aws.amazon.com/blogs/machine-learning/use-custom-metrics-to-evaluate-your-generative-ai-application-with-amazon-bedrock/?utm_source=chatgpt.com "Use custom metrics to evaluate your generative AI ..."
[13]: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_EvaluationDatasetMetricConfig.html?utm_source=chatgpt.com "EvaluationDatasetMetricConfig - Amazon Bedrock"
