# AIF-C01｜3.2.2 **プロンプトエンジニアリングの手法**

*AIF-C01 Domain 3, Task 3.2.2 — Prompt-engineering techniques (zero/one/few-shot, chain-of-thought concept, templates)*

> 目标：把**零样本/单样本/少样本/思维链概念/模板化**几种常用提示手法讲清楚，并会在 **Amazon Bedrock** 里落地（Playground/Converse API/Advanced Prompt Templates）。术语给**中・日（读法）・英**三语；讲解以中文为主，并标注与考试相关的“秒选要点”。

---

## 1) 总览（你要知道的 5 件事）

* **提示工程（プロンプトエンジニアリング／Prompt engineering）**：通过**精心设计提示**（结构、示例、格式约束等）来获得更稳定、更相关的输出。AWS 官方将其视作构建生成式应用的基本能力。([Amazon Web Services, Inc.][1])
* **上下文是“燃料”**：RAG/知识库把命中片段**拼进提示**，效果往往强于盲目加长提示。([AWS ドキュメント][2])
* **Converse API** 提供统一接口与参数（`temperature/topP/maxTokens/stopSequences`），跨模型一致。([AWS ドキュメント][3])
* **模板化**能把“好提示”沉淀为**可复用模板**；在 **Bedrock Agents** 里还有**分步模板**（前处理/编排/KB 生成/后处理）。([AWS ドキュメント][4])
* **思维链（思考の連鎖／Chain-of-Thought）概念**：强调“**先推理再作答**”的思路；在工程与合规中更推荐**结构化答案/简要理由**等**可控替代**，而不是要求模型暴露详细内在推理痕迹。*（考试理解概念即可）* ([AWS ドキュメント][5])

---

## 2) ゼロショット｜**Zero-shot**（零样本）

* **定义**：**不给示例**，仅靠任务说明与上下文（可含 RAG 证据）让模型完成任务。
* **适用**：**任务清晰、规则明确**（分类、抽取、简单问答/摘要）。
* **提示骨架**：

  1. 角色与目标；2) 任务定义与判定标准；3) 输入/输出格式（最好是 JSON/表格）；4) 约束与边界（禁止内容、引用要求）；5)（可选）RAG 命中片段。
* **落地**：Bedrock 文档建议在零样本场景**明确任务与输出格式**；Playground/Converse 直接试。([AWS ドキュメント][6])

---

## 3) シングルショット（ワンショット）｜**One-shot**（单样本）

* **定义**：提供**1 个高质量示例**（Input→Output）来“定调”，降低歧义。
* **适用**：**格式要求较严格**（比如固定 JSON 键名）、或存在**风格**/口吻偏好。
* **要点**：示例要**短而完整**，与真实输入**分隔清晰**（如 XML/分隔符），防止“串场”。
* **落地**：在 Bedrock 的“**Prompt templates and examples**”页面里，文本分类/抽取等任务给出了模板可直接套用。([AWS ドキュメント][4])

---

## 4) フューショット（少数ショット）｜**Few-shot**（少样本）

* **定义**：提供**少量示例**（通常 2–10 个）概括任务变体与边界（**正例/反例**各一更佳）。
* **适用**：**边界复杂**、**多类别**、**需要风格一致**的任务（如客服话术、合规措辞）。
* **要点**：

  * 示例**覆盖典型边界**（易混淆/例外）；
  * 明确**格式/评分标准**；
  * 与推理参数配合（低温度、更稳定）。
* **落地**：AWS 文档与博客均建议 few-shot 提升稳定性，也可作为**Agent 高级提示模板**的一部分。([AWS ドキュメント][7], [Amazon Web Services, Inc.][8])

---

## 5) 思考の連鎖｜**Chain-of-Thought（CoT）概念**

* **概念理解（考试用）**：强调“**显式进行中间推理，再得结论**”的范式，可帮助复杂推理与多步任务。
* **工程实践（安全合规建议）**：在真实系统中，通常**不要求模型输出详细推理痕迹**；而是用**结构化步骤/关键依据/中间计算结果**等**可控替代**（例如“列出3个要点+最终结论”“给出引用的证据编号”），并借助 **Agent 的编排或 ReAct 策略**把“思考与行动”拆分为**受控步骤**。([AWS ドキュメント][5])
* **与 Bedrock**：在 **Agents** 中，默认**编排策略**基于 **ReAct（Reason+Act）** 模式，你可以自定义或用 Lambda 托管更严格的**多步决策与工具调用**。([AWS ドキュメント][5])

---

## 6) プロンプトテンプレート｜**Prompt Templates**（模板化）

* **定义**：把“好提示”做成**可复用模板**，包含**变量占位**、**分区结构**（任务/上下文/示例/输出格式/禁止项）。
* **Bedrock 场景**：

  * **Text prompts**：官方提供**分类/问答/摘要**模板；Playground 可直接复用并微调。([AWS ドキュメント][4])
  * **Agents**：暴露**四个基础模板**（前处理/编排/KB 生成/后处理），可**逐段覆盖**、**开/关步骤**、并配置**推理参数**与**占位变量**。([AWS ドキュメント][9])
* **优势**：**团队协作/版本化/回归评估**更容易；支持**A/B**对比与“模板库”治理。

---

## 7) 与 Bedrock 的“工程化落地”组合拳

* **Converse API（统一调参/多模型）**：一套消息与 `inferenceConfig` 跨模型通用；如需模型专属参数也能下传。([AWS ドキュメント][3])
* **Knowledge Bases（RAG）**：把**相关片段**放入提示，而不是整篇文档；Bedrock KB 负责**分块→嵌入→向量检索**。([AWS ドキュメント][2])
* **Advanced Prompt Templates（Agents）**：在**多步骤**任务中，对**每一步**（前处理/编排/KB/后处理）给**不同模板**与**参数**，并可自定义编排策略。([AWS ドキュメント][10])
* **Playground**：无代码试提示与参数，找“最小可行提示”。（面试/考试高频做法）([AWS ドキュメント][11])

---

## 8) 常见陷阱 → 修正

* **只写长段话，不分区** → **改用模板/分隔符**（任务/上下文/示例/输出）。([AWS ドキュメント][4])
* **上下文堆太多** → **RAG Top-K 限制** + 合理分块（KB 自动处理）。([AWS ドキュメント][2])
* **温度与 Top-P 同时大幅调** → 输出发散；**择一微调**，并配`maxTokens/Stop`控长度。([AWS ドキュメント][12])
* **复杂业务“只靠提示”** → 上 **Agent + 模板**，用 ReAct/自定义编排拆分**思考与行动**。([AWS ドキュメント][5])

---

## 9) 术语小词典（中・日・英）

* 提示工程：**プロンプトエンジニアリング**／Prompt engineering ([Amazon Web Services, Inc.][1])
* 零样本：**ゼロショット**／Zero-shot ([AWS ドキュメント][6])
* 单样本：**シングルショット（ワンショット）**／One-shot
* 少样本：**フューショット（少数ショット）**／Few-shot ([AWS ドキュメント][7])
* 思维链（概念）：**思考の連鎖**／Chain-of-Thought（CoT，概念理解，工程上用受控替代）([AWS ドキュメント][5])
* 模板：**プロンプトテンプレート**／Prompt template（文本与 Agent 分步模板）([AWS ドキュメント][4])
* 统一对话：**Converse API**（`inferenceConfig`）([AWS ドキュメント][3])
* 知识库/RAG：**ナレッジベース**／Knowledge Bases（分块→嵌入→检索）([AWS ドキュメント][2])

---

## 10) 一句话总括

> **零→单→少→模板化**逐步增强表达力；**RAG**供证据，**Converse**控参数，**Agents**做多步编排；“思维链”概念上理解、工程上用**结构化答案/简要理由/受控编排**替代。上述做法与模板均可在 **Amazon Bedrock 文档与示例**中直接套用。([AWS ドキュメント][4])

[1]: https://aws.amazon.com/what-is/prompt-engineering/?utm_source=chatgpt.com "What is Prompt Engineering? - AI ..."
[2]: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-it-works.html?utm_source=chatgpt.com "How Amazon Bedrock knowledge bases work"
[3]: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html?utm_source=chatgpt.com "Converse - Amazon Bedrock"
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-templates-and-examples.html?utm_source=chatgpt.com "Prompt templates and examples for Amazon Bedrock text ..."
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/orch-strategy.html?utm_source=chatgpt.com "Customize agent orchestration strategy - Amazon Bedrock"
[6]: https://docs.aws.amazon.com/sagemaker/latest/dg/jumpstart-foundation-models-customize-prompt-engineering.html?utm_source=chatgpt.com "Prompt engineering for foundation models"
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/advanced-prompts.html?utm_source=chatgpt.com "Enhance agent's accuracy using advanced prompt ..."
[8]: https://aws.amazon.com/blogs/machine-learning/prompt-engineering-techniques-and-best-practices-learn-by-doing-with-anthropics-claude-3-on-amazon-bedrock/?utm_source=chatgpt.com "Prompt engineering techniques and best practices"
[9]: https://docs.aws.amazon.com/bedrock/latest/userguide/agents-how.html?utm_source=chatgpt.com "How Amazon Bedrock Agents works"
[10]: https://docs.aws.amazon.com/bedrock/latest/userguide/advanced-prompts-templates.html?utm_source=chatgpt.com "Advanced prompt templates - Amazon Bedrock"
[11]: https://docs.aws.amazon.com/bedrock/latest/userguide/design-a-prompt.html?utm_source=chatgpt.com "Design a prompt - Amazon Bedrock"
[12]: https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference-call.html?utm_source=chatgpt.com "Using the Converse API - Amazon Bedrock"
