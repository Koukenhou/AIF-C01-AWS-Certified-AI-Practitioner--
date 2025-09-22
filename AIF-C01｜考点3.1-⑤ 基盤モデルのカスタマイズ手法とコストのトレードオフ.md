# AIF-C01｜3.1.5 基盤モデルのカスタマイズ手法と**コストのトレードオフ**

*AIF-C01 Domain 3, Task 3.1.5 — Cost trade-offs across FM customization approaches (pre-training, fine-tuning, in-context learning, RAG, etc.)*

> 目标：能比较 **文脉内/提示工程（In-context）**、**RAG**、**ファインチューニング（微调）**、**継続事前学習（Continued pre-training）**（以及**蒸馏 Distillation**）在**成本、开发难度、上线速度、运维负担**上的取舍，并能给出**AWS 上的推荐做法**。术语均给出 **中・日（含读法）・英**。

---

## 0) 总览：四（+一）种“定制”思路

| 手法       | 日文（读法）                             | English                                  | 改变了什么                        | 典型成本构成（粗略优先级）                          |
| -------- | ---------------------------------- | ---------------------------------------- | ---------------------------- | -------------------------------------- |
| 文脉内/提示工程 | 文脈内学習【ぶんみゃくない がくしゅう】／プロンプトエンジニアリング | In-context learning / Prompt engineering | **推理阶段**把任务说明、示例、规则放进**上下文** | 仅**推理 token 费**；几乎无训练/托管费；人力调参         |
| RAG      | 検索拡張生成【けんさく かくちょう せいせい】            | Retrieval-Augmented Generation           | 在**推理前**检索你私有数据并拼接进提示        | **推理 token 费 + 检索/向量存储 + 数据接入**；无需反复训练 |
| 微调       | ファインチューニング                         | Fine-tuning                              | 用小规模标注数据**更新部分权重**           | **训练作业 + 数据准备 +（在 Bedrock 上）推理需预留吞吐**  |
| 继续预训练    | 継続事前学習【けいぞく じぜん がくしゅう】             | Continued pre-training                   | 用大批无标注领域语料**再学习**            | **长时训练 + 数据治理/存储 + 评估**；推理形态同上         |
| 蒸馏（加分）   | 蒸留【じょうりゅう】                         | Distillation                             | 大模型知识→小模型                    | **训练 + 目标端点托管**；换取推理成本/延迟下降            |

> AWS 官方强调：**Bedrock 推理按输入/输出 token 计费；自定义模型（微调/继续预训/蒸馏或导入的自定义权重）在 Bedrock 上通常需要购买 Provisioned Throughput 才能上线推理**（换来稳定吞吐/延迟，代价是按时计费）。([AWS ドキュメント][1], [Amazon Web Services, Inc.][2])

---

## 1) 文脉内学习／提示工程（In-context / Prompting）

* **定义**：通过**高质量提示**（任务指令、格式约束、少样例）引导基础模型完成任务，无需训练。
  日：**プロンプトエンジニアリング**／文脈内学習【ぶんみゃくない】；EN：Prompt engineering / In-context learning。
* **成本画像**：

  * 仅付 **token 费**（输入+输出）；可用**短提示/结构化模板/流式**降低感知延迟与成本。([AWS ドキュメント][1], [Amazon Web Services, Inc.][3])
* **优点**：**上线快、零训练成本**、维护简单；适合需求快速迭代。([AWS ドキュメント][4])
* **局限**：当**企业知识/术语**多、回答需**可引用**或**稳定性**要求高时，单靠提示往往不够。
* **AWS 用法**：**Bedrock Playground** 快速试参；在 `converse` / `inferenceConfig` 里设置 `temperature/topP/maxTokens/stopSequences`。([AWS ドキュメント][4])

> 何时选：**题目清晰+数据少+想快速起步** → 先用 Prompt；若缺企业知识/引用 → 叠加 **RAG**。

---

## 2) RAG（检索增强生成）

* **定义**：先从数据源检索相关片段（向量/语义），再把命中内容拼入提示，由 FM 生成**带引用**的回答。
  日：**検索拡張生成**；EN：Retrieval-Augmented Generation。
* **成本画像**：

  * **推理 token 费**（通常**小模型+短上下文**即可）；
  * **RAG 管道**（KB 托管或自建向量库/OpenSearch/pgvector）；
  * **数据接入/同步**（连接器/解析）。
  * 官方指出：**引入 Knowledge Bases 能提升性价比，因为避免为了利用私有数据而“持续训练模型”**。([AWS ドキュメント][5])
* **优点**：可审计（**引用**）、知识**易更新**（分钟级）、**无需反复训练**；对降低**幻觉**有帮助。([AWS ドキュメント][6])
* **局限**：**长文全局摘要**或**复杂风格/格式**可能仍需微调辅助。([AWS ドキュメント][7])
* **AWS 用法**：

  * **Knowledge Bases for Amazon Bedrock**：托管分块/嵌入/向量/检索/拼接，少工程即可上线；可 BYO 向量库（OpenSearch Serverless / Aurora pgvector）。([Amazon Web Services, Inc.][8])

> 何时选：**你有大量私有文档/需要引用/需快速更新** → **RAG 优先**；若还要风格统一/结构化强 → 可与**微调**组合为混合方案。([AWS ドキュメント][7])

---

## 3) 微调（Fine-tuning）

* **定义**：用**带标签**的小规模数据，对 FM 进行任务/风格/格式的定向学习（SFT/LoRA/QLoRA 等）。
  日：**ファインチューニング**；EN：Fine-tuning。
* **成本画像**（Bedrock 官方维度）：

  * **训练作业**：时长受**数据量/输入输出 token/epoch/batch**等影响；需 S3 数据准备（`.jsonl`）。([AWS ドキュメント][9])
  * **部署推理**：**自定义模型在 Bedrock 上通常需购买 Provisioned Throughput** 才能被调用（稳定容量/低抖动，**按时计费**）。([Amazon Web Services, Inc.][2])
* **优点**：提升**格式稳定性**、**话术/风格一致性**，可缩短提示、减少重试（间接降 token 成本）。
* **局限**：**前期投入**（标注/训练/评估/治理）明显高于 RAG；**上线前需评估与监控**。
* **AWS 用法**：**Bedrock Model Customization**（创建微调作业），或在 SageMaker 训练后用 **Custom Model Import** 导入到 Bedrock 推理。([AWS ドキュメント][10])

> 何时选：RAG+Prompt 已“够用”但**输出不稳/风格不统一/格式要求极严** → 考虑**微调**。

---

## 4) 继续预训练（Continued pre-training, CPT）

* **定义**：用**大规模无标注**领域语料让 FM 学到**更深的领域分布与术语**（不针对具体指令）。
  日：**継続事前学習【けいぞく じぜん がくしゅう】**；EN：Continued pre-training。
* **成本画像**：**训练时长/算力/数据治理**远高于微调；推理同为**自定义模型**（多采用预留吞吐）。([AWS ドキュメント][10])
* **优点**：长期积累的**领域知识/语域**适配；对 RAG 的“阅读理解能力”也可能有加成。
* **局限**：投入大、周期长、需要严格的数据/安全/评估流程；**AIF-C01 场景一般不首选**。

> 何时选：行业极专业、语域很特殊、**RAG + 微调**仍不够，且你有**大量高质量无标注语料**与预算时。

---

## 5) 蒸馏（Distillation，补充）

* **定义**：让小模型学习大模型的行为/知识（学生-教师范式），以减少**推理成本与延迟**。
  日：**蒸留【じょうりゅう】**；EN：Model Distillation。
* **成本画像**：一次性**训练成本**换得长期**推理成本**下降；Bedrock 支持开启**Distillation 作业**与自定义模型在线推理。([AWS ドキュメント][11], [Amazon Web Services, Inc.][12])
* **适用**：流量大、对延迟/单次成本敏感，且对顶尖质量要求稍低的场景。

---

## 6) “钱花在哪”：分解成本与可控杠杆

| 成本分类     | 主要来源                           | 可控杠杆（中・日・英）                                                                                                                                  |
| -------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **推理**   | **输入/输出 token 费；预留吞吐（若自定义模型）** | 缩提示/限制 `max_tokens`；**小模型+RAG**；**Provisioned Throughput**只为**高峰/核心**开；批/异步/Serverless 混合。([AWS ドキュメント][1], [Amazon Web Services, Inc.][13]) |
| **训练**   | 微调/继续预训/蒸馏的 GPU 小时、S3 存储与出入    | 数据抽样/清洗；**LoRA** 等参数高效微调；作业并行/epoch 调优。([AWS ドキュメント][9])                                                                                     |
| **数据接入** | RAG 的连接器、分块/嵌入/向量库             | **Knowledge Bases** 托管化；**OpenSearch Serverless/pgvector**按需计费。([Amazon Web Services, Inc.][8])                                              |
| **运维**   | 评估、监控、审计、安全                    | **Bedrock Evaluations**；CloudWatch/CloudTrail；Guardrails（跨模型复用）。([Amazon Web Services, Inc.][13])                                            |

---

## 7) 选择建议：以成本为中心的“三步法”

1. **先 Prompt（文脉内）→ 再 RAG**

   * 需求能用提示解决、数据不多：直接 Prompt；
   * 有私有文档/要引用/频繁更新：**RAG 优先**，官方也建议**问答就先上 RAG**；需要摘要等额外能力再考虑微调。([AWS ドキュメント][7])
2. **确需稳定风格/格式** → **微调**

   * 评估微调后**能否缩短提示/减少重试**（间接省 token 费）；但要记得**自定义模型通常需预留吞吐**才能在 Bedrock 调用。([Amazon Web Services, Inc.][2])
3. **领域极深/语域独特/预算充足** → **继续预训练或蒸馏**

   * CPT 投入大但知识沉淀深；蒸馏面向**长期降本**（高流量）。

---

## 8) “哪种更便宜？”——场景化对照

| 场景                  | 成本最优解                                                   | 说明                                                      |
| ------------------- | ------------------------------------------------------- | ------------------------------------------------------- |
| **内部文档问答（要引用/更新快）** | **RAG（Knowledge Bases）+ 小模型 + 结构化提示**                   | 无需持续训练；引用可审计；KB 官方称**更具成本效益**。([AWS ドキュメント][5])         |
| **客服对话（高并发/日高峰）**   | 常态 On-demand + 高峰 **Provisioned Throughput**；必要时**小微调** | 既控成本又保 SLA；微调用来稳话术/格式。([Amazon Web Services, Inc.][13]) |
| **法务/财务长文摘要**       | **RAG + 批/异步**；如需固定模板→小规模微调                             | 批处理/异步降价；微调稳结构化输出。([Amazon Web Services, Inc.][13])     |
| **移动端低延迟+大规模调用**    | **蒸馏**或选择更小的 FM；结合 RAG                                  | 以训练换推理费与延迟。([Amazon Web Services, Inc.][12])            |

---

## 9) 关键术语（中・日・英）

* 文脉内学习：**文脈内学習【ぶんみゃくない がくしゅう】** / In-context learning（Prompt engineering） ([AWS ドキュメント][4])
* 检索增强生成：**検索拡張生成** / Retrieval-Augmented Generation (RAG) ([AWS ドキュメント][6])
* 微调：**ファインチューニング** / Fine-tuning ([AWS ドキュメント][10])
* 继续预训练：**継続事前学習** / Continued pre-training ([AWS ドキュメント][10])
* 蒸馏：**蒸留** / Model distillation（Bedrock 支持） ([Amazon Web Services, Inc.][12])
* 预留吞吐：**プロビジョンドスループット** / Provisioned Throughput（自定义模型常用） ([Amazon Web Services, Inc.][2])
* 知识库：**ナレッジベース** / Knowledge Bases（RAG 托管） ([Amazon Web Services, Inc.][8])
* Token 计费：**トークン課金** / Token-based pricing（输入+输出） ([AWS ドキュメント][1])

---

## 10) 一句话总括

> **性价比路径**：**Prompt → RAG →（必要时）微调 →（高门槛）继续预训练/蒸馏**。
> 在 AWS 上：**RAG 用 Knowledge Bases** 省工程且**更具成本效益**；**微调/继续预训**要把**训练 + 预留吞吐**一起算清；想降长期推理成本/延迟，可评估**蒸馏**。([AWS ドキュメント][5], [Amazon Web Services, Inc.][2])

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/bedrock-pricing.html?utm_source=chatgpt.com "Amazon Bedrock pricing - AWS Documentation"
[2]: https://aws.amazon.com/blogs/machine-learning/security-best-practices-to-consider-while-fine-tuning-models-in-amazon-bedrock/?utm_source=chatgpt.com "Security best practices to consider while fine-tuning models ..."
[3]: https://aws.amazon.com/blogs/machine-learning/implementing-advanced-prompt-engineering-with-amazon-bedrock/?utm_source=chatgpt.com "Implementing advanced prompt engineering with Amazon ..."
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html?utm_source=chatgpt.com "Prompt engineering concepts - Amazon Bedrock"
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-it-works.html?utm_source=chatgpt.com "How Amazon Bedrock knowledge bases work"
[6]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html?utm_source=chatgpt.com "Retrieve data and generate AI responses with ..."
[7]: https://docs.aws.amazon.com/prescriptive-guidance/latest/retrieval-augmented-generation-options/rag-vs-fine-tuning.html?utm_source=chatgpt.com "Comparing Retrieval Augmented Generation and fine-tuning"
[8]: https://aws.amazon.com/bedrock/knowledge-bases/?utm_source=chatgpt.com "Amazon Bedrock Knowledge Bases"
[9]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-customization-submit.html?utm_source=chatgpt.com "Submit a model customization job for fine-tuning or continued ..."
[10]: https://docs.aws.amazon.com/bedrock/latest/userguide/custom-model-fine-tuning.html?utm_source=chatgpt.com "Customize a model with fine-tuning or continued pre- ..."
[11]: https://docs.aws.amazon.com/bedrock/latest/userguide/custom-models.html?utm_source=chatgpt.com "Customize your model to improve its performance for your use case"
[12]: https://aws.amazon.com/blogs/machine-learning/a-guide-to-amazon-bedrock-model-distillation-preview/?utm_source=chatgpt.com "A guide to Amazon Bedrock Model Distillation (preview)"
[13]: https://aws.amazon.com/blogs/machine-learning/effective-cost-optimization-strategies-for-amazon-bedrock/?utm_source=chatgpt.com "Effective cost optimization strategies for Amazon Bedrock - AWS"
