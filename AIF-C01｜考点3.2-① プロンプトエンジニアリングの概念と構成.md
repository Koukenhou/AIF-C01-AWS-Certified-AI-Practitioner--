# AIF-C01｜3.2.1 **プロンプトエンジニアリングの概念と構成**

*AIF-C01 Domain 3, Task 3.2.1 — Prompt engineering concepts & components (context, instructions, negative prompts, latent space, etc.)*

> 目标：系统理解**提示工程**是什么、由哪些“块”组成、如何影响模型输出，并能在 AWS（Amazon Bedrock）里**落地与调参**。术语提供**中・日（读法）・英**三语；讲解以中文为主，结合官方文档与最佳实践。

---

## 1) 什么是“提示工程”（Prompt Engineering）

* **定义（中）**：通过**设计与优化输入**（提示），让模型更稳定地按预期完成任务（分类、问答、要约、代码等）。提示的质量直接影响输出质量。**（考试要点：知道“提示工程=优化输入以获得期望输出”。）** ([AWS ドキュメント][1])
* **日**：**プロンプトエンジニアリング**（ぷろんぷと えんじにありんぐ）
* **EN**：Prompt engineering
* **在 Bedrock 中**：你可用 **Converse API** 统一给“消息”（system/user/assistant 等），并通过 `inferenceConfig` 传递温度、top\_p、maxTokens、stopSequences 等参数，跨多家模型保持一致用法。([AWS ドキュメント][2])

---

## 2) 提示的四大“构成块”（Components）

### 2.1 コンテキスト【こんてきすと】/ **Context**（语境与材料）

* **是什么**：为任务提供的**背景与素材**——包括对话历史、RAG 检索到的**证据片段（chunks）**、样例（few-shot）、角色设定、约束条件、术语表等。
* **在 RAG 中**：**Knowledge Bases for Bedrock** 会把文档**分块→嵌入→向量检索**，把**命中块**附加到提示中（上下文增强），再一起发给模型。**这是“把数据接进模型”的标准方法**。([AWS ドキュメント][3])
* **要点**：上下文要**短而准**（避免整篇贴上来），并**保留引用**以便审计。([AWS ドキュメント][4])

### 2.2 指示（インストラクション）/ **Instructions**

* **是什么**：告诉模型**要做什么、如何做、输出格式**。应**清晰可检验**（如“以 JSON 输出、键名固定、不要多余文本”）。
* **Bedrock 模板**：官方建议在提示中**显式分隔**“任务/上下文/示例/输出要求”等区块（常用分隔符或 XML 标签），并给出规范的**模版与示例**。([AWS ドキュメント][5], [Amazon Web Services, Inc.][6])
* **在 Agent 中**：还可通过**高级提示模板**分别控制“前处理/编排/KB生成/后处理”等阶段的指令（托管多步任务）。([AWS ドキュメント][7])

### 2.3 ネガティブ・プロンプト / **Negative Prompt**（告诉模型**不要**什么）

* **文本生成**：较少单独设负向字段；通常在**指示里**描述禁止内容，并用 **Guardrails** 统一过滤。
* **图像生成**：在 **Amazon Titan Image Generator** 等模型中有**专门字段**（`negativeText`），用于列出不希望出现的概念（注意：**写要排除的词本体，不要写“no …”之类否定短语**）。([AWS ドキュメント][8], [Amazon Web Services, Inc.][9])
* **日**：ネガティブ・プロンプト（ねがてぃぶ）
* **EN**：Negative prompt

### 2.4 モデルの潜在空間【せんざい くうかん】/ **Latent Space**

* **是什么**：模型把概念（词义、风格、图像构图等）映射到的**高维“隐空间”**；相近语义在该空间中彼此更“近”。**提示工程**的本质是在**隐空间**中**定位/约束**输出区域（通过上下文、示例、负向约束等）。([Amazon Web Services, Inc.][10], [Amazon Science][11])
* **意义**：理解隐空间有助于知道**为什么**“更清晰的概念、更结构化的样例、更合适的检索片段”会带来**更稳定**的输出。

---

## 3) 提示工程 × 推理参数（一起调，效果更稳）

| 维度    | 日文（读法）  | English        | 作用             | 典型用法                         |
| ----- | ------- | -------------- | -------------- | ---------------------------- |
| 温度    | 温度【おんど】 | Temperature    | 控制**随机性**      | 事实/代码：低（0–0.2）；创意：高（0.7–1.0） |
| Top-P | トップP    | Top-P          | 在**累计概率 P**内采样 | 与温度**择一**小幅调优                |
| 最大输出  | 最大トークン数 | maxTokens      | 限制输出长度与成本      | 保证能“写完”，又不浪费                 |
| 停止串   | 停止シーケンス | Stop sequences | 提前停止           | 保证**JSON/表格封闭**              |

> 以上参数在 Bedrock 的 `InferenceConfiguration`/`converse` 中统一设置；官方把“**随机性与多样性**/ **长度**”作为两大参数族。([AWS ドキュメント][12])

---

## 4) AWS 官方“好提示”的结构化做法（可直接套）

1. **角色与目标**（谁、做什么、成功标准）。
2. **上下文**（RAG 命中片段、术语、约束、例外）。
3. **输入格式**（示例或字段说明）。
4. **输出格式**（JSON/Markdown 表格，给**明确 schema**）。
5. **示例（Few-shot）**：**正例/反例**各一；用分隔符或 XML 标签清晰包裹。
6. **禁止与边界**（可在文本指示里写清，也可由 **Guardrails** 平台化约束）。

> Bedrock 文档提供了**常见任务的模板与示例**（分类、要约、问答、带上下文/不带上下文），实践中用**清晰分隔**能显著提升一致性。([AWS ドキュメント][5], [Amazon Web Services, Inc.][6])

---

## 5) 与 RAG/Agent/Guardrails 的协同（平台级“提示工程”）

* **RAG（Knowledge Bases）**：把**对的证据**拼进提示（而不是“把整篇贴进去”），能显著降低**幻觉/成本**，并提供**引用**便于审计。([AWS ドキュメント][4])
* **Agent（多步骤编排）**：在**前处理/编排/KB 生成/后处理**四处用**高级提示模板**精细控制每一步的“话术与边界”，并可**引导参数**与**调用 API**。([AWS ドキュメント][7])
* **Guardrails**：对**输入与输出**执行业务/安全策略（话题限制、PII 遮罩、提示注入防护），作为**跨模型复用**的“负向约束”。（负向约束≠负向提示字段，更适合统一治理。）([AWS ドキュメント][1])

---

## 6) 常见模式与陷阱（考试 & 实战高频）

* **模式 A：Few-shot + 结构化输出**

  * 明确**格式（JSON/schema）** + 一两个**好示例** → 提升稳定性与可解析性。([AWS ドキュメント][5])
* **模式 B：RAG + 小模型**

  * 以**精确命中片段**替代“巨长上下文”，既省钱又更可信（带引用）。([AWS ドキュメント][4])
* **模式 C：图像负向提示**

  * **Titan Image** 等模型支持 `negativeText` 字段；注意**列概念词**，不要加“no …”。([AWS ドキュメント][8])

> **陷阱**：
>
> 1. 一次把所有约束都堆进一段话，**缺少分隔** → 模型难以解析。→ 用**分段/标签**清晰区块。([AWS ドキュメント][5])
> 2. 同时大幅调**温度**与 **Top-P** → 交互作用不可控。→ 任选其一微调。
> 3. 把**整篇文档**塞进上下文 → 费用高且“淹没要点”。→ 用 KB 的**分块+Top-K**增强。([AWS ドキュメント][3])

---

## 7) 术语小词典（中・日・英）

* 提示工程：**プロンプトエンジニアリング** / Prompt engineering ([AWS ドキュメント][1])
* 语境/上下文：**コンテキスト** / Context（含对话历史、RAG 命中块） ([AWS ドキュメント][4])
* 指示：**インストラクション** / Instructions（任务与格式要求） ([AWS ドキュメント][5])
* 负向提示：**ネガティブ・プロンプト** / Negative prompt（图像生成常见字段） ([AWS ドキュメント][8])
* 潜在空间：**潜在空間【せんざい くうかん】** / Latent space（高维概念表示） ([Amazon Web Services, Inc.][10])
* 分块：**チャンク化** / Chunking（KB 预处理） ([AWS ドキュメント][3])
* 嵌入：**埋め込み表現** / Embeddings（向量化） ([AWS ドキュメント][3])
* 统一对话接口：**Converse API** / `inferenceConfig`（温度/Top-P/maxTokens/stop） ([AWS ドキュメント][2])

---

## 8) 一句话总括

> **好提示 = 清晰指示 + 精准上下文 + 合理负向约束 + 合适参数**。
> 在 AWS 上，配合 **Bedrock 的 Converse API（调参）**、**Knowledge Bases（上下文/RAG）**、**Agent（多步模板）** 与 **Guardrails（安全治理）**，就能把“提示工程”从**写好一句话**升级为**工程化的、可复用的提示系统**。([AWS ドキュメント][2])

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html?utm_source=chatgpt.com "Prompt engineering concepts - Amazon Bedrock"
[2]: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html?utm_source=chatgpt.com "Converse - Amazon Bedrock"
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-chunking.html?utm_source=chatgpt.com "How content chunking works for knowledge bases"
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-it-works.html?utm_source=chatgpt.com "How Amazon Bedrock knowledge bases work"
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-templates-and-examples.html?utm_source=chatgpt.com "Prompt templates and examples for Amazon Bedrock text ..."
[6]: https://aws.amazon.com/blogs/machine-learning/prompt-engineering-techniques-and-best-practices-learn-by-doing-with-anthropics-claude-3-on-amazon-bedrock/?utm_source=chatgpt.com "Prompt engineering techniques and best practices"
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/advanced-prompts-templates.html?utm_source=chatgpt.com "Advanced prompt templates - Amazon Bedrock"
[8]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-titan-image.html?utm_source=chatgpt.com "Amazon Titan Image Generator G1 models - Amazon Bedrock"
[9]: https://aws.amazon.com/blogs/machine-learning/use-amazon-titan-models-for-image-generation-editing-and-searching/?utm_source=chatgpt.com "Use Amazon Titan models for image generation, editing, ..."
[10]: https://aws.amazon.com/blogs/machine-learning/how-latent-space-used-the-amazon-sagemaker-model-parallelism-library-to-push-the-frontiers-of-large-scale-transformers/?utm_source=chatgpt.com "How Latent Space used the Amazon SageMaker model ..."
[11]: https://www.amazon.science/publications/tie-your-embeddings-down-cross-modal-latent-spaces-for-end-to-end-spoken-language-understanding?utm_source=chatgpt.com "Tie your embeddings down: Cross-modal latent spaces for ..."
[12]: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_InferenceConfiguration.html?utm_source=chatgpt.com "InferenceConfiguration - Amazon Bedrock"
