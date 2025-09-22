# AIF-C01｜考点 2.1：生成 AI の基本概念を説明する（中・日・英对照，日语含读法）

> 对齐 **AWS Certified AI Practitioner Exam Guide – Domain 2, Task 2.1**。本节聚焦：基础概念（token、チャンク化、埋め込み/ベクトル、プロンプトエンジニアリング、Transformer/LLM、基盤モデル、マルチモーダル、拡散モデル等）、常见用例、基础模型生命周期（数据/模型选择、事前学習、微调、评估、部署、反馈）。([AWS Static][1])

---

## 1) 生成式 AI 的核心概念（中・日【读法】・英｜考试用速记+实战提示）

| 中文              | 日文（读法）                           | English                        | 一句话理解 / 备考要点                                                                                                                       |
| --------------- | -------------------------------- | ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| 令牌              | トークン（トークン）                       | Token                          | 模型处理的最小单位（近似“字/词片”）。控制**上下文窗口**与**计费**，与**max tokens**、**temperature/top\_p**等推理参数一起影响输出风格与长度。([AWS ドキュメント][2])                    |
| 分块              | チャンク化（チャンクか）                     | Chunking                       | 把长文档切成段落等**逻辑片**，便于嵌入检索与长文本处理（RAG 的第一步）。([AWS ドキュメント][3])                                                                          |
| 嵌入向量            | 埋め込み（うめこみ）/ エンベディング、ベクトル         | Embeddings / Vectors           | 把文本/图像转换为**向量**以便相似度搜索；RAG/搜索/去重都用它。([AWS ドキュメント][3])                                                                              |
| 提示工程            | プロンプトエンジニアリング                    | Prompt Engineering             | 通过**清晰结构化**的提示+示例+约束+参数，让模型按你想法回答；“问题放在结尾”“用分隔符/格式指示”等是官方最佳实践。([AWS ドキュメント][4])                                                    |
| 推理参数            | 推論パラメータ（すいろん…）                   | Inference Parameters           | **temperature**（随机性）、**top\_p**（概率截断）、**max\_tokens**（输出长度）等，直接影响**创意度/确定性**。([AWS ドキュメント][2])                                     |
| Transformer/LLM | トランスフォーマー / 大規模言語モデル【だいきぼげんごモデル】 | Transformer / LLM              | 基于注意力机制的生成式主流架构与模型家族，处理/生成文本能力强。([Amazon Web Services, Inc.][5])                                                                   |
| 基础模型            | 基盤モデル【きばんモデル】/ ファウンデーションモデル      | Foundation Model (FM)          | 在海量数据上**事前学習**的通用模型，可直接调用或**微调/持续预训练**进行定制。([Amazon Web Services, Inc.][5], [AWS ドキュメント][6])                                       |
| 多模态模型           | マルチモーダルモデル                       | Multimodal Model               | 同时处理**文本+图像/视频/音频**等多种模态（如部分 Amazon Nova / Titan 家族）。([The Verge][7])                                                              |
| 扩散模型            | 拡散モデル（かくさんモデル）                   | Diffusion Model                | 以“逐步去噪”生成图像/视频的一类模型（生成图像常见）。([Amazon Web Services, Inc.][8])                                                                       |
| RAG             | 検索拡張生成【けんさくかくちょうせいせい】            | Retrieval-Augmented Generation | **先检索、后生成**：把你私有/最新资料检索为上下文再喂给模型，显著提升正确性与可引用性。Bedrock **Knowledge Bases**提供托管实现。([AWS ドキュメント][9], [Amazon Web Services, Inc.][10]) |
| 代理/智能体          | エージェント                           | Agent                          | 基于模型+工具编排的“能干活”的组件：可读知识、调用API执行任务（如创建工单），Bedrock **Agents**提供。([AWS ドキュメント][11])                                                   |
| 守护栏             | ガードレール                           | Guardrails                     | 负责安全/合规：过滤**有害内容**、**话题限制**、**敏感信息(PII)遮罩**等；可跨模型复用。([AWS ドキュメント][12])                                                             |

---

## 2) 生成式 AI 的代表用例（看到关键词即可归类）

* **文本类**：摘要（サマリー）、改写（リライト）、问答（QA）、对话机器人（チャットボット）、翻译（翻訳）、代码生成（コード生成）。
* **媒体类**：图像/视频/音频生成与编辑（生成・編集）。
* **企业场景**：**客服 Agent**、**文档/FAQ 搜索 + RAG**、报告自动化、推荐与搜索增强。
* **在 AWS 上**：统一入口 **Amazon Bedrock**（模型库+评估+KB+Agents+Guardrails）。([Amazon Web Services, Inc.][10], [AWS ドキュメント][13])

---

## 3) 基础模型（FM）的生命周期（考试高频）

> 记忆：**“选数据→选模型→事前学習/利用→定制→评估→部署→反馈/再训练”**。

1. **数据选择（データ選択）** & **模型选择（モデル選択）**：根据任务/成本/延迟/安全选择合适 FM（文本/多模态/规模）。([Amazon Web Services, Inc.][5])
2. **事前学習 / 利用（Pre-training / Use as-is）**：直接用通用知识进行零样本/小样本推理。
3. **定制（カスタマイズ）**：

   * **Fine-tuning（ファインチューニング）**：让模型在特定任务上学客户风格/术语；
   * **Continued Pre-training（継続事前学習）**：让模型熟悉某类输入分布；
   * **蒸馏/指令微调** 等（Bedrock 提供多种方式）。([AWS ドキュメント][6])
4. **评估（評価）**：自动/人工评测任务正确性、鲁棒性、RAG 检索正确率等（Bedrock Evaluations 支持）。([AWS ドキュメント][13])
5. **部署（デプロイ）**：以 **托管 API**（Bedrock 调用）或自建推理端点上线；对延迟/成本进行权衡。
6. **反馈 & 持续改进**：收集用户反馈、对话评分、知识更新；触发再训练/微调或 **KB 重建索引**。([AWS ドキュメント][9])

---

## 4) 实战：把“提示 + 参数 + 检索”用好

* **写好提示（プロンプト）**：

  * **清晰完整**说明任务；
  * **把问题放在提示末尾**；
  * 用**分隔符**组织上下文；
  * 告诉模型**输出格式**（如 JSON）；
  * 在 Bedrock Playground 反复试验。([AWS ドキュメント][14], [AWS Static][15])
* **调推理参数**：

  * **temperature** 低 → 更**稳定**；高 → 更**发散/创意**；
  * **top\_p** 控制采样范围；
  * **max\_tokens** 控制输出长度/成本。([AWS ドキュメント][2])
* **接入 RAG**：文档→**分块**→**嵌入**→**向量检索**→把命中内容连同问题送入模型（Bedrock **Knowledge Bases** 托管管道）。([AWS ドキュメント][9], [Amazon Web Services, Inc.][10])
* **加安全守护**：Bedrock **Guardrails** 统一配置**有害内容过滤/话题限制/PII 遮罩**；对话/Agent/RAG 统一生效。([AWS ドキュメント][12])

---

## 5) 生成式 AI 的优势与局限（考场排雷）

* **优势**：泛化强、可迁移、小样本即用；通过 **RAG/微调** 快速贴合业务。
* **局限**：**幻觉（ハルシネーション）**、知识时效性、可解释性、合规与隐私；需用 **Knowledge Bases + Guardrails + 评估** 组合治理。([AWS ドキュメント][13], [Amazon Web Services, Inc.][10])

---

## 6) 术语小词典（高频专有名词｜中・日【读法】・英）

* **上下文窗口**：コンテキストウィンドウ / Context Window
* **示例学习**：少数ショット学習 / Few-shot Learning
* **零样本**：ゼロショット / Zero-shot
* **指令微调**：指示チューニング / Instruction Tuning
* **评估基准**：ベンチマーク / Benchmark
* **企业搜索**：エンタープライズサーチ / Enterprise Search（常与 RAG 结合）
* **多模态**：マルチモーダル / Multimodal（文本+图像/视频/音频）
* **代理/工具调用**：エージェント / Agent（API 呼叫、工具编排）([AWS ドキュメント][11])

---

## 7) 备考速记 & 选择题技巧

1. **看到“最新/私有知识”** → 想到 **RAG/Knowledge Bases**；
2. **看到“过滤有害/PII/禁题”** → **Guardrails**；
3. **“结果不稳定/太发散”** → 降 **temperature** / 提示更结构化；
4. **“长文档效果差”** → **分块 + 嵌入**，问题放提示**末尾**；
5. **“业务要落地”** → 说清 **生命周期**：选择→定制→评估→部署→监控/反馈。([AWS ドキュメント][14])

---

### 参考（官方文档节选）

* AIF-C01 Exam Guide – Domain 2 / Task 2.1（考试要求条目）([AWS Static][1])
* What is Generative AI? / Foundation Models explained（概念）([Amazon Web Services, Inc.][8])
* Bedrock Prompt Engineering（写提示与参数）([AWS ドキュメント][4])
* Inference Parameters（temperature、top\_p、max tokens）([AWS ドキュメント][2])
* Knowledge Bases & RAG（检索增强）([AWS ドキュメント][9], [Amazon Web Services, Inc.][10])
* Model customization（Fine-tuning / Continued pre-training）([AWS ドキュメント][6])
* Model evaluation（自动评估与任务类型）([AWS ドキュメント][13])
* Agents（API 调用与任务编排）([AWS ドキュメント][11])
* Guardrails（安全与合规）([AWS ドキュメント][12])

---

**一句话总括**：

> **生成式 AI = 基础模型（FM） + 好提示 + 合适参数 + 知识增强（RAG） + 安全守护（Guardrails） + 全生命周期（定制/评估/部署/反馈）。**
> 按此“五件套”去定位考题，大多数 2.1 题都能稳稳拿分。

[1]: https://d1.awsstatic.com/training-and-certification/docs-ai-practitioner/AWS-Certified-AI-Practitioner_Exam-Guide.pdf?utm_source=chatgpt.com "AWS Certified AI Practitioner (AIF-C01) Exam Guide"
[2]: https://docs.aws.amazon.com/bedrock/latest/userguide/inference-parameters.html?utm_source=chatgpt.com "Influence response generation with inference parameters"
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html?utm_source=chatgpt.com "Amazon Titan Text Embeddings models - Amazon Bedrock"
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html?utm_source=chatgpt.com "Prompt engineering concepts - Amazon Bedrock"
[5]: https://aws.amazon.com/what-is/foundation-models/?utm_source=chatgpt.com "Foundation Models in Generative AI Explained"
[6]: https://docs.aws.amazon.com/bedrock/latest/userguide/custom-model-fine-tuning.html?utm_source=chatgpt.com "Customize a model with fine-tuning or continued pre- ..."
[7]: https://www.theverge.com/2024/12/3/24312260/amazon-nova-foundation-ai-models-anthropic?utm_source=chatgpt.com "Amazon announces its own set of Nova AI models"
[8]: https://aws.amazon.com/what-is/generative-ai/?utm_source=chatgpt.com "What is Generative AI? - Gen AI Explained"
[9]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html?utm_source=chatgpt.com "Retrieve data and generate AI responses with ..."
[10]: https://aws.amazon.com/bedrock/knowledge-bases/?utm_source=chatgpt.com "Amazon Bedrock Knowledge Bases"
[11]: https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html?utm_source=chatgpt.com "Automate tasks in your application using AI agents"
[12]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[13]: https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation.html?utm_source=chatgpt.com "Evaluate the performance of Amazon Bedrock resources"
[14]: https://docs.aws.amazon.com/bedrock/latest/userguide/design-a-prompt.html?utm_source=chatgpt.com "Design a prompt - Amazon Bedrock"
[15]: https://d1.awsstatic.com/events/Summits/reinvent2023/AIM377_Prompt-engineering-best-practices-for-LLMs-on-Amazon-Bedrock.pdf?utm_source=chatgpt.com "Prompt engineering best practices for LLMs on Amazon Bedrock"
