# AIF-C01｜3.2.3 **プロンプトエンジニアリングの利点とベストプラクティス**

*AIF-C01 Domain 3, Task 3.2.3 — Benefits & best practices of prompt engineering (quality, experimentation, guardrails, discovery, specificity & conciseness, multi-message usage)*

> 目标：掌握如何用**提示工程**系统提升生成质量，并能在 **Amazon Bedrock** 上用**Playground/Converse、Guardrails、Knowledge Bases、Evaluations、模板**把最佳实践工程化落地。术语给出**中・日（读法）・英**三语；讲解以中文为主。

---

## 1) 为什么要做提示工程（利点）

* **提升输出质量与稳定性**：清晰的**指示（Instructions）**、**上下文（Context/RAG）**与**结构化输出**能显著提升相关性、可控性与可复现性。Bedrock 官方把这类方法收录为入门指南与模板。([AWS ドキュメント][1])
* **缩短迭代周期**：借助 **Playground/Converse API** 快速调参（temperature、top-p、maxTokens、stopSequences），一处改动可跨模型复用。([AWS ドキュメント][2])
* **可审计与降幻觉**：结合 **Knowledge Bases（RAG）** 把“命中片段”拼进提示并返回**引用**，让回答可追溯、可验证。([AWS ドキュメント][3])
* **责任与安全**：用 **Guardrails** 给**输入与输出**加统一的安全/合规策略（敏感话题、PII 遮罩、注入防护），跨多家 FM 复用。([AWS ドキュメント][4], [Amazon Web Services, Inc.][5])
* **可度量可比较**：用 **Bedrock Evaluations**（含自定义指标）对提示/模型/RAG 方案做离线评估与回归对比。([Amazon Web Services, Inc.][6])

---

## 2) 实验（実験【じっけん】/Experimentation）与“发现”（発見【はっけん】/Discovery）

> 思路：**小步快跑** → **A/B 比较** → **保留最小可行提示（MVP Prompt）**。

* **快速试验**：在 **Playground** 或样例代码里更换模板段落、示例数量、推理参数；Playground 适合探索“范围”，代码适合版本化与集成。([AWS ドキュメント][2])
* **度量对比**：用 **Bedrock Evaluations** 做**模型对比**（Model Eval）与 **RAG 评估**（Retrieve / Retrieve+Generate），可接**自定义指标**匹配你的业务标准。([Amazon Web Services, Inc.][6])
* **发现有效模式**：AWS/Anthropic 的最佳实践建议**用标签或分隔符清晰分区**（如 `<instructions>…</instructions>`、`<context>…</context>`），并**提供高质量示例**与**明确输出格式**。([Amazon Web Services, Inc.][7])

---

## 3) Guardrails（ガードレール／安全护栏）的集成

* **跨模型一致生效**：为不同用例创建多套 Guardrails，并在**提示前**与**生成后**统一过滤。适合把“禁止项”与“安全边界”**平台化**，而非仅在提示里写负向语句。([AWS ドキュメント][4], [Amazon Web Services, Inc.][5])
* **常见策略**：PII 识别/遮罩、话题限制、提示注入防护、**JSON 格式约束**配合停止序列（Stop Sequences）。([AWS ドキュメント][4])

---

## 4) 具体性与简洁性（具体性【ぐたいてきせい】& 簡潔さ【かんけつさ】）

* **具体而清晰**：AWS 与合作伙伴在会议资料中强调提示要**具体、清晰且可执行**（给目标读者、长度、风格、格式等）。([D1 AWS Static][8])
* **简洁聚焦**：避免把整篇背景塞进上下文；RAG 只拼**命中的片段**（控制 **Top-K** 与 chunk 大小），并指定**固定输出模式（JSON/表格）**。([AWS ドキュメント][3])

---

## 5) “多条消息/分段书写”的使用（複数メッセージの使用）

> 考点里的“複数のコメント”可理解为：将**多段信息**拆成**多条消息**或**多段模板区块**，让模型更好解析。

* **Converse API 多消息**：在 `messages` 中**按角色**（system/user/assistant）与**多条消息**组织内容，把**角色/规则**、**上下文**、**任务输入**分开发送，更可控、可复用。([AWS ドキュメント][2])
* **模板分区**：使用 **Prompt Templates** 将**指示/上下文/示例/输出规范**分成独立区块；在 **Agents** 中还能对**前处理/编排/KB 生成/后处理**设置**分步模板**。([AWS ドキュメント][9])

---

## 6) 可直接套用的 10 条最佳实践（AWS 官方要点合辑）

1. **分区清晰**：用分隔符/XML 标签把**任务、上下文、示例、输出格式**隔开。([Amazon Web Services, Inc.][7])
2. **先结构化，再创意**：优先指定**输出 schema（JSON/表格）**与**停止序列**；再调温度/Top-P。([AWS ドキュメント][9])
3. **示例为王**：**One-shot/Few-shot** 给**高质量正反例**；短小、贴近实际、覆盖边界。([Amazon Web Services, Inc.][7])
4. **用 RAG 喂证据**：把**相关片段**放进上下文并**附带引用**，别整篇贴。([AWS ドキュメント][3])
5. **守护栏兜底**：把**禁止项/PII/话题限制**平台化为 **Guardrails**，而非只写在提示里。([AWS ドキュメント][4])
6. **小步实验**：用 Playground/样例代码反复试，记录参数与模板版本。([AWS ドキュメント][2])
7. **量化评估**：用 **Bedrock Evaluations** 或**自定义指标**做回归，选择更稳的方案。([Amazon Web Services, Inc.][6])
8. **控制成本**：限制上下文长度与 `maxTokens`，减少不必要的 Top-K 命中与冗长输出。([AWS ドキュメント][10])
9. **避免参数“叠加发散”**：**温度**与 **Top-P**尽量**择一**小幅调优。参考官方与厂商建议。([Amazon Web Services, Inc.][7])
10. **模板化与复用**：把“好提示”沉淀为**模板**与**分步模板**，并进行版本化管理。([AWS ドキュメント][11])

---

## 7) 小型清单（落地到 Bedrock）

* **Converse API（统一对话/参数）**：以**多条消息**组织 system/user 内容，在 `inferenceConfig` 里设温度/Top-P/maxTokens/stop。([AWS ドキュメント][2])
* **Prompt Templates**：从官方模板起步（分类/要约/QA），根据任务增减区块与范例。([AWS ドキュメント][9])
* **Knowledge Bases（RAG）**：配置数据源→自动分块与向量化→检索拼接→带引用输出。([AWS ドキュメント][3])
* **Guardrails**：按用例创建策略，跨 FM 一致套用在**提示与回答**两端。([AWS ドキュメント][4])
* **Evaluations**：离线评估**不同模板/参数/模型**，选择最稳且性价比高的组合。([Amazon Web Services, Inc.][6])

---

## 8) 术语小词典（中・日・英）

* 提示工程：**プロンプトエンジニアリング**／Prompt engineering ([AWS ドキュメント][1])
* 上下文/语境：**コンテキスト**／Context（RAG 命中片段） ([AWS ドキュメント][3])
* 指示：**インストラクション**／Instructions（任务/格式/边界） ([AWS ドキュメント][9])
* 守护栏：**ガードレール**／Guardrails（输入/输出统一安全策略） ([AWS ドキュメント][4])
* 评估：**エバリュエーション**／Evaluations（模型/RAG/自定义指标） ([Amazon Web Services, Inc.][6])
* 模板：**プロンプトテンプレート**／Prompt templates（文本与分步模板） ([AWS ドキュメント][9])
* 对话接口：**Converse API**（多消息 + `inferenceConfig`） ([AWS ドキュメント][2])

---

### 一句话总括

> **好提示 = 清晰分区 + 证据就位 + 安全兜底 + 可度量迭代**。在 **Amazon Bedrock**：用 **Prompt Templates/Playground**起步、**Converse**控参数、**Knowledge Bases**供证据、**Guardrails**做治理、**Evaluations**做对比；保持**具体而简洁**，并用**多消息/分段**提升可控与复用性。([AWS ドキュメント][9], [Amazon Web Services, Inc.][6])

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html?utm_source=chatgpt.com "Prompt engineering concepts - Amazon Bedrock"
[2]: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html?utm_source=chatgpt.com "Converse - Amazon Bedrock"
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html?utm_source=chatgpt.com "Retrieve data and generate AI responses with ..."
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[5]: https://aws.amazon.com/bedrock/guardrails/?utm_source=chatgpt.com "Amazon Bedrock Guardrails"
[6]: https://aws.amazon.com/bedrock/evaluations/?utm_source=chatgpt.com "Evaluate Foundation Models - Amazon Bedrock Evaluations"
[7]: https://aws.amazon.com/blogs/machine-learning/prompt-engineering-techniques-and-best-practices-learn-by-doing-with-anthropics-claude-3-on-amazon-bedrock/?utm_source=chatgpt.com "Prompt engineering techniques and best practices"
[8]: https://d1.awsstatic.com/events/Summits/reinvent2023/AIM377_Prompt-engineering-best-practices-for-LLMs-on-Amazon-Bedrock.pdf?utm_source=chatgpt.com "Prompt engineering best practices for LLMs on Amazon Bedrock"
[9]: https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-templates-and-examples.html?utm_source=chatgpt.com "Prompt templates and examples for Amazon Bedrock text ..."
[10]: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_InferenceConfiguration.html?utm_source=chatgpt.com "InferenceConfiguration - Amazon Bedrock"
[11]: https://docs.aws.amazon.com/bedrock/latest/userguide/advanced-prompts-templates.html?utm_source=chatgpt.com "Advanced prompt templates - Amazon Bedrock"
