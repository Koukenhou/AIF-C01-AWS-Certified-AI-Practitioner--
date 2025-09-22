# AIF-C01｜3.2.4 **プロンプトエンジニアリングの潜在的なリスクと限界**

*AIF-C01 Domain 3, Task 3.2.4 — Define risks & limitations in prompt engineering (exposure, poisoning, hijacking, jailbreak, etc.)*

> 目标：能**准确定义**提示工程中的主要风险（**露出/暴露、ポイズニング、ハイジャック、ジェイルブレイク**等），并**结合 AWS 官方**给出可执行的**缓解策略**（Guardrails、数据保护、评估与监控、RAG 数据治理、Agent 防护）。术语给出**中・日（读法）・英**三语，讲解以中文为主。

---

## 1) 风险总览（概念 & 三语对照）

| 风险           | 日文（读法）        | English                         | 一句话定义                                                                                                                |
| ------------ | ------------- | ------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **暴露 / 露出**  | 露出【ろしゅつ】      | Exposure (Data/Prompt exposure) | 通过提示或输出**泄露系统提示、密钥、PII、机密数据**等。Bedrock 默认**不**用你的输入/输出训练模型，减少外泄面。([AWS ドキュメント][1], [Amazon Web Services, Inc.][2])   |
| **中毒**       | ポイズニング        | (Data/Model) Poisoning          | 把**恶意或误导性数据**注入训练集或 **RAG 数据源**（含用户上传），导致错误或不当回答。([AWS ドキュメント][3])                                                   |
| **劫持**       | ハイジャック        | Hijack (Prompt/Session/Tool)    | 通过**提示操控**或**外部内容注入**改变系统意图，诱导 Agent 误调用工具/越权访问。([Amazon Web Services, Inc.][4])                                     |
| **越狱**       | ジェイルブレイク      | Jailbreak                       | 通过特制提示绕过安全对齐与内容限制（如要求“忽略所有规则”）。**Guardrails**可帮助识别与阻断。([Amazon Web Services, Inc.][5], [reinforce.awsevents.com][6]) |
| **提示注入（核心）** | プロンプトインジェクション | Prompt Injection                | 恶意输入**覆盖开发者指令**或诱导模型泄密/执行非预期动作，直接或\*\*间接（经外部数据/网页）\*\*发生。([AWS ドキュメント][7], [Amazon Web Services, Inc.][8])           |

---

## 2) 典型攻击面与“为什么会中招”

### 2.1 提示注入 / 越狱（Prompt Injection / Jailbreak）

* **现象**：攻击者通过精心构造的内容（对话、网页、文档），引导模型**忽略系统提示**或**执行违禁操作**（例如导出内嵌密钥、调用危险 API）。([AWS ドキュメント][7])
* **在多步任务/Agent 中**：恶意“外部观察”可**间接注入**（Indirect），诱导 Agent 选择错误动作或泄露信息。AWS 对 **间接注入**给出针对性策略。([Amazon Web Services, Inc.][8])
* **越狱区别**：越狱是注入的一类结果——**绕过安全对齐**，生成本应被拦截的内容。Bedrock **Guardrails**提供**输入/输出**双向过滤与话题/行为限制。([Amazon Web Services, Inc.][5], [reinforce.awsevents.com][6])

### 2.2 数据暴露（Exposure）

* **场景**：模型**复述机密**（训练或上下文中的敏感片段）、系统提示被套出、PII 未遮罩。
* **平台侧边界**：Bedrock 明确**不会**用你的输入/输出训练任何 Amazon 或第三方模型，也**不与第三方共享**，降低平台侧“再暴露”。但**应用侧**仍需做 PII 处理与最小化原则。([AWS ドキュメント][1], [Amazon Web Services, Inc.][2])

### 2.3 数据/知识库中毒（Poisoning）

* **训练集中毒**：恶意样本污染微调或继续预训练，使模型形成错误模式。
* **RAG 中毒**：向 **Knowledge Bases** 的数据源**投放提示注入文本或恶意内容**，在检索时被拼进上下文，间接操纵模型。AWS **安全参考架构**专门提示：需要在**摄取阶段**清洗/过滤注入与恶意内容。([AWS ドキュメント][3])

### 2.4 劫持（Hijacking）

* **Prompt/Session**：通过连续对话逐步改变系统意图。
* **Tool/Agent**：诱导 Agent 选择**越权**或**破坏性**动作（删库、越权查询）。AWS 安全博客给出**威胁建模**示例与缓解策略（最小权限、审计、WAF 等）。([Amazon Web Services, Inc.][4])

---

## 3) AWS 官方的缓解与工程化做法（按攻击面分层）

### 3.1 平台/模型层（Bedrock 侧）

* **Guardrails for Amazon Bedrock（ガードレール）**

  * **内容过滤**（仇恨/暴力/不当）、**提示攻击检测**（注入/越狱）、**PII 识别与遮罩**，对**输入与输出**同时生效，且**跨模型复用**。([Amazon Web Services, Inc.][5], [AWS ドキュメント][9])
* **数据保护与隐私**

  * Bedrock **不存储日志、不将输入/输出用于训练**，也**不共享**给第三方；官方 FAQ 重申这一点。([AWS ドキュメント][1], [Amazon Web Services, Inc.][2])

### 3.2 应用/提示层

* **安全提示工程**

  * 使用**结构化模板**（将“指示/上下文/示例/约束/输出格式”分区），对外部内容加\*\*“不盲从”指令\*\*；对 JSON/表格输出加**停止序列**与**模式约束**。AWS 博客示例展示了可落地模板。([Amazon Web Services, Inc.][10])
* **多消息与权限边界**

  * 使用 **Converse API** 的**多消息**与**system 指令**固定边界，避免被 user 覆盖；监控 `maxTokens`/温度，减少过度“扩写”。([AWS ドキュメント][11])

### 3.3 RAG/数据层

* **摄取前过滤与清洗**

  * 对进入 KB 的数据执行**注入检测/恶意内容过滤**；在**上传阶段**识别并**剔除/标记**可疑片段（含正则与 PII 工具）。AWS 安全指引建议在**数据管线**就处理。([AWS ドキュメント][3])
* **最小上下文**

  * 控制 **Chunk 大小与 Top-K**，避免把整篇“脏数据”拼入上下文。([AWS ドキュメント][12])

### 3.4 Agent/多步编排层

* **为 Agent 绑定 Guardrails**，并开启\*\*追踪（Trace）\*\*审计每次动作/观察；**参数引导**只收集必要字段。([AWS ドキュメント][7])
* **防“间接注入”**

  * 对**外部来源**（网页、用户上传）加**中和提示**与**可信来源过滤**；必要时**人工/规则**复核命中片段。([Amazon Web Services, Inc.][8])

### 3.5 运维/治理层

* **威胁建模 + 监控审计**

  * 结合 **AWS Security Blog** 的威胁建模方法，按 STRIDE/用例枚举风险；用 **CloudTrail/日志**审计调用；配合 WAF、API Gateway 做外围防护。([Amazon Web Services, Inc.][4])
* **评估与红队**

  * 用 **Bedrock Evaluations**做离线评估（召回/有引据/安全项）；开展**越狱/注入**红队测试，迭代 Guardrails 与模板。([AWS ドキュメント][13])
* **对齐治理标准**

  * 参考 **ISO/IEC 42001** 等治理实践，将注入/越狱视为**完整生命周期**风险，建立**度量与控制**。([Amazon Web Services, Inc.][14])

---

## 4) “考试可秒选”的缓解清单（问法 → 答法）

* **问：如何防 Prompt Injection / Jailbreak？**
  答：**Guardrails（输入/输出）+ 安全提示模板 + RAG 数据清洗 + 最小上下文 + 评估与监控**；Agent 需**关联 Guardrails**并防**间接注入**。([Amazon Web Services, Inc.][5])
* **问：如何防 KB/RAG 被“下毒”？**
  答：**摄取前过滤**（注入/恶意/PII）、**来源可信度管理**、**最小 Top-K**；必要时人工复核关键源。([AWS ドキュメント][3])
* **问：数据会被平台拿去训练吗？**
  答：**不会**。Bedrock **不**用你的输入/输出训练，也**不共享**给第三方。([AWS ドキュメント][1], [Amazon Web Services, Inc.][2])
* **问：Agent 为什么更易被劫持？**
  答：因为它能**调用动作/API**。需**最小权限、参数白名单、Trace 审计**与 Guardrails。([Amazon Web Services, Inc.][4])

---

## 5) 实操模板（可直接套改）

**安全提示模板骨架（文本任务）**

```
<System>
あなたは{役割}です。次のポリシーに従ってください：
- 禁止：{否定事項の要点}（詳細はGuardrailsで強制）
- 出力形式：JSON（schema 下）
- 不明なら「情報不足」と答え、推測しない
</System>

<Instructions>
{任务定义（判定标准/读者/长度）}
</Instructions>

<Context>
{RAG 命中片段（含出典/来源id）…}
</Context>

<Examples>
{One/Few-shot 正反例…}
</Examples>

<OutputSchema>
{固定键名/类型；必要时Stop Sequences 封口}
</OutputSchema>
```

要点：**分区清晰**、**引用就位**、**输出可验证**、**把禁止项平台化到 Guardrails**。([Amazon Web Services, Inc.][10])

---

## 6) 术语小词典（中・日・英）

* 暴露/露出：**露出【ろしゅつ】** / Exposure（Data/Prompt）
* 中毒：**ポイズニング** / Poisoning（Training / RAG） ([AWS ドキュメント][3])
* 劫持：**ハイジャック** / Hijacking（Prompt/Session/Tool） ([Amazon Web Services, Inc.][4])
* 越狱：**ジェイルブレイク** / Jailbreak（Bypass safety） ([Amazon Web Services, Inc.][5])
* 提示注入：**プロンプトインジェクション** / Prompt Injection（Direct/Indirect） ([AWS ドキュメント][7], [Amazon Web Services, Inc.][8])
* 守护栏：**ガードレール** / Guardrails（输入/输出过滤、PII、注入/越狱检测） ([Amazon Web Services, Inc.][5], [AWS ドキュメント][9])
* 知识库：**ナレッジベース** / Knowledge Bases（RAG，含预处理/检索/拼接） ([AWS ドキュメント][12])
* 评估：**エバリュエーション** / Evaluations（含 RAG 评估） ([AWS ドキュメント][13])

---

### 一句话总括

> **提示工程的风险≠只靠一句“别做坏事”能解决。**在 AWS 上，用 **Guardrails（输入/输出）**、**安全提示模板**、**RAG 数据摄取过滤**、**Agent 关联 Guardrails + Trace**、**评估与威胁建模**，把**注入/越狱/中毒/劫持**从**流程与平台**两侧同时收敛；同时，Bedrock 的**数据不用于训练/不对外共享**提供平台级隐私保障。([Amazon Web Services, Inc.][5], [AWS ドキュメント][1])

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html?utm_source=chatgpt.com "Data protection - Amazon Bedrock"
[2]: https://aws.amazon.com/bedrock/faqs/?utm_source=chatgpt.com "Amazon Bedrock FAQs - Generative AI - AWS"
[3]: https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/gen-ai-rag.html?utm_source=chatgpt.com "Capability 2. Providing secure access, usage, and ..."
[4]: https://aws.amazon.com/blogs/security/threat-modeling-your-generative-ai-workload-to-evaluate-security-risk/?utm_source=chatgpt.com "Threat modeling your generative AI workload to evaluate ..."
[5]: https://aws.amazon.com/bedrock/guardrails/?utm_source=chatgpt.com "Amazon Bedrock Guardrails"
[6]: https://reinforce.awsevents.com/content/dam/reinforce/2024/slides/GRC325_Build-responsible-AI-applications-with-Guardrails-for-Amazon-Bedrock.pdf?utm_source=chatgpt.com "Build responsible AI applications with Guardrails for ..."
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-injection.html?utm_source=chatgpt.com "Prompt injection security - Amazon Bedrock"
[8]: https://aws.amazon.com/blogs/machine-learning/securing-amazon-bedrock-agents-a-guide-to-safeguarding-against-indirect-prompt-injections/?utm_source=chatgpt.com "A guide to safeguarding against indirect prompt injections"
[9]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-sensitive-filters.html?utm_source=chatgpt.com "Remove PII from conversations by using sensitive information ..."
[10]: https://aws.amazon.com/blogs/machine-learning/secure-rag-applications-using-prompt-engineering-on-amazon-bedrock/?utm_source=chatgpt.com "Secure RAG applications using prompt engineering on ..."
[11]: https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference-call.html?utm_source=chatgpt.com "Using the Converse API - Amazon Bedrock"
[12]: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-it-works.html?utm_source=chatgpt.com "How Amazon Bedrock knowledge bases work"
[13]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-evaluation-prompt.html?utm_source=chatgpt.com "Create a prompt dataset for a RAG evaluation in ..."
[14]: https://aws.amazon.com/blogs/security/ai-lifecycle-risk-management-iso-iec-420012023-for-ai-governance/?utm_source=chatgpt.com "ISO/IEC 42001:2023 for AI governance | AWS Security Blog"
