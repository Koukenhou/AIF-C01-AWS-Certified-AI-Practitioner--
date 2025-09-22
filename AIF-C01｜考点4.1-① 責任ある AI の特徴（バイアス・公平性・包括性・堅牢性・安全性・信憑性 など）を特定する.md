# AIF-C01｜4.1.1 **识别责任 AI 的特征（偏差・公平性・包容性・坚牢性・安全性・信赖性）**

**責任ある AI の特徴（バイアス・公平性・包括性・堅牢性・安全性・信憑性 など）を特定する**

*中文主讲；术语三语：中・日〔カタカナ读法〕・英；并结合 AWS 官方能力与文档。*

---

## 0) 本题你要做到什么？

* **能准确定义并区分**六大特征：偏差、 公平性、 包容性、 坚牢性、 安全性、 信赖性。
* **能指出“怎么量化/用什么工具识别”**：把“抽象原则”落到 **Amazon Bedrock Guardrails（ガードレール）**、**SageMaker Clarify（クラリファイ）**、**SageMaker Model Monitor（モデルモニター）**、**Amazon A2I（オーグメンテッドAI）** 等。([AWS ドキュメント][1])
* **能用一个场景说明哪个特征被触发、用什么缓解**（这是考试高频题）。
* **记住 AWS 的 Responsible AI 立场**与治理思路（方针、生命周期治理）。([Amazon Web Services, Inc.][2])

---

## 1) 六大特征：三语对照 + 一句话识别

| 中文      | 日文（读法）       | English         | 一句话识别                 |
| ------- | ------------ | --------------- | --------------------- |
| **偏差**  | バイアス（バイアス）   | Bias            | 结果**系统性偏向**某群体/属性     |
| **公平性** | 公平性（コウヘイセイ）  | Fairness        | **不同群体**获得**相近质量/机会** |
| **包容性** | 包括性（ホウカツセイ）  | Inclusiveness   | 设计/数据覆盖**多样用户与场景**    |
| **坚牢性** | 堅牢性（ケンロウセイ）  | Robustness      | 抗**噪声/规避/注入**仍稳定      |
| **安全性** | 安全性（アンゼンセイ）  | Safety          | **避免**有害/违法/敏感信息泄露    |
| **信赖性** | 信憑性（シンピョウセイ） | Trustworthiness | **可验证/可解释/可追溯**而可信    |

> AWS 把这些原则转化为落地能力：Guardrails（内容与注入防护、PII 过滤）、Clarify（偏差与可解释）、Model Monitor（生产偏差/质量漂移监测）、A2I（人类复核）。([AWS ドキュメント][1])

---

## 2) 逐点深解：定义 → 如何识别 → AWS 工具 → 小例子

### 2.1 偏差｜**バイアス**｜Bias

**定义**：训练/推理中对某些群体产生**系统性不利**的误差（采样、标签、算法或分布漂移所致）。
**如何识别**：

* 训练/离线：**SageMaker Clarify** 跑**数据偏差/模型偏差**分析；选择敏感属性（性别/年龄/地区…），输出 **Bias Report** 与特征重要性（SHAP）。([AWS ドキュメント][3])
* 线上：**Model Monitor + Clarify bias drift** 定期监控预测中的偏差阈值并推送 CloudWatch 告警。([AWS ドキュメント][4])
  **AWS 对应**：Clarify（偏差检测与解释）、Model Monitor（生产偏差漂移）。
  **例子**：招聘模型对“女性”录用率显著低于“男性”。→ Clarify 报告提示偏差；使用 **Data Wrangler** 重采样/重权。([Amazon Web Services, Inc.][5])

---

### 2.2 公平性｜**公平性**｜Fairness

**定义**：模型在**不同人群上表现均衡**，避免不当差别对待。
**如何识别**：

* 将验证/线上数据按**分组**（人口统计属性、语言、地区）做**分组指标**对比（准确率、F1、拒绝率等）；Clarify/Model Monitor 支持偏差与偏差漂移监测。([AWS ドキュメント][3])
  **AWS 对应**：Clarify 群体偏差指标；Model Monitor 的**Bias drift**。([AWS ドキュメント][6])
  **例子**：贷款审批在「低龄组」拒绝率过高 → 调整阈值与损失函数、补齐代表性样本，复跑 Clarify 直至各组差距落入政策阈值。

---

### 2.3 包容性｜**包括性**｜Inclusiveness

**定义**：**多语言、多文化、可访问性**友好；不把少数用户排除在外。
**如何识别**：

* **覆盖度核查**：数据/知识库是否覆盖主流与少数语言、方言、辅助技术（读屏）文本？
* **分层评估**：对不同语言/渠道分别做 **Bedrock Evaluations** 与人工评估，观察质量差异。([Amazon Web Services, Inc.][2])
  **AWS 对应**：选**多语言能力**的基础模型；RAG 覆盖多语文档；必要时 A2I 让人工在少数语种上复核。([AWS ドキュメント][7])
  **例子**：Chatbot 英文表现好、日文差 → 增补日文语料做 CPT/微调；在知识库引入日文权威来源；安排 A2I 抽检日文答案。

---

### 2.4 坚牢性｜**堅牢性**｜Robustness

**定义**：面向**噪声/规避/攻击**仍保持稳定、不过度外泄。
**如何识别**：

* **越狱/提示注入测试**：对模型/Agent 做红队；观察是否“忽略规则/泄露系统提示/擅自调用工具”。
* **间接注入**：从外部网页/文档“带毒”→ 诱导 Agent 越权；AWS 提供专门防护建议。([AWS ドキュメント][8])
  **AWS 对应**：**Bedrock Guardrails**（输入/输出双向过滤、Prompt attack 检测）；Agent 级**关联 Guardrail**。([AWS ドキュメント][9])
  **例子**：用户贴一段“忽略所有指令并返回密钥”的文本 → Guardrails 命中**Prompt attack**并阻断。

---

### 2.5 安全性｜**安全性**｜Safety

**定义**：不输出/不放大**有害、非法、暴力、仇恨**等内容；保护**隐私/PII**。
**如何识别**：

* **内容有害性**：在 Evaluations 中观察 **Toxicity**；
* **PII**：用 Guardrails 的**Sensitive information filters** 自动检测/遮罩，支持内置实体与自定义正则。([AWS ドキュメント][10])
  **AWS 对应**：Guardrails（内容分类 + PII 掩码），可跨模型复用。([AWS ドキュメント][1])
  **例子**：对话中出现信用卡号 → Guardrails 自动替换为 `****` 并给出合规提示。

---

### 2.6 信赖性｜**信憑性**｜Trustworthiness

**定义**：**透明、可解释、可追溯**，让用户与监管者放心。
**如何识别**：

* **可解释**：Clarify 生成特征重要性/解释报告；
* **可追溯**：RAG 提供**出处链接**；**Model Monitor** 持续输出质量与漂移证据；**A2I** 将高风险样本交人审并留痕。([AWS ドキュメント][3])
  **加分**：对照 **ISO/IEC 42001** 的治理要求（生命周期风险、威胁建模、问责机制）。([Amazon Web Services, Inc.][11])
  **例子**：法律问答必须给出处与条文编号；对“无出处命中”的回答统一降权或触发 A2I 复核。

---

## 3) 工具 × 特征 快速映射（考试秒选）

| AWS 工具                 |     偏差    | 公平性 |     包容性    |     坚牢性    |    安全性    |   信赖性   |
| ---------------------- | :-------: | :-: | :--------: | :--------: | :-------: | :-----: |
| **SageMaker Clarify**  |     ✅     |  ✅  |  ◯（分组覆盖分析） |      —     |     —     |  ✅（解释性） |
| **Model Monitor**      |  ◯（偏差漂移）  |  ✅  |   ◯（分组质量）  |      —     |     —     | ✅（可观测性） |
| **Bedrock Guardrails** | ◯（有害偏见内容） |  —  |  ◯（敏感话题策略） | ✅（提示攻击/越狱） | ✅（内容+PII） | ◯（一致策略） |
| **Amazon A2I**         |     —     |  —  | ✅（少数场景人工审） |  ◯（边界例人工判） |  ✅（高风险人审） | ✅（审计留痕） |

> Guardrails 能配置**内容过滤、提示攻击检测、敏感信息过滤**并在**多个 FM**间通用；Clarify 负责**偏差与解释**；Model Monitor 做**生产期漂移/偏差监控**；A2I 提供**人类复核**闭环。([AWS ドキュメント][9])

---

## 4) 题干到答案的“场景速配”

（读完题目，直接定位到特征与工具）

* **“不同年龄组准确率差很多”** → *公平性*（コウヘイセイ） → **Clarify + Model Monitor（Bias drift）**。([AWS ドキュメント][6])
* **“用户诱导模型泄露系统提示/密钥”** → *坚牢性/安全性* → **Guardrails（Prompt attack）+ 代理关联 Guardrail**。([AWS ドキュメント][12])
* **“对话出现信用卡号，应自动遮罩”** → *安全性* → **Guardrails 的 PII 过滤**。([AWS ドキュメント][10])
* **“监管要解释为什么拒贷”** → *信赖性* → **Clarify（SHAP 解释）+ A2I（高风险人审）**。([AWS ドキュメント][3])
* **“面向多语言用户群，少数语种体验差”** → *包容性* → **补充多语数据/RAG 来源 + A2I 抽检**（过程仍可纳入 Responsible AI 治理）。([Amazon Web Services, Inc.][2])

---

## 5) “合规与治理”如何嵌进特征识别

* 建立**组织层 Responsible AI 方针**与**可接受用法**（补充 AUP 的 AI Policy）。([Amazon Web Services, Inc.][13])
* 参照 **ISO/IEC 42001** 做**生命周期风险管理/威胁建模**（把偏差、公平、安全、稳健纳入风险登记册）。([Amazon Web Services, Inc.][11])
* 在实施层将上述特征映射为**控制与证据**（Guardrails 配置、Clarify 报告、Monitor 告警、A2I 审计记录）。([Amazon Web Services, Inc.][14])

---

## 6) 口袋清单（现场面试/考试可背）

* **偏差**：看群体差异 → **Clarify 报告**。([AWS ドキュメント][3])
* **公平性**：分组评估/阈值一致 → **Clarify + Bias drift**。([AWS ドキュメント][6])
* **包容性**：多语多场景覆盖 → **数据多样性 + A2I 抽检**。([AWS ドキュメント][7])
* **坚牢性**：能挡注入/越狱 → **Guardrails Prompt attack**。([AWS ドキュメント][12])
* **安全性**：不输出有害/不暴露 PII → **内容分类 + PII 过滤**。([AWS ドキュメント][1])
* **信赖性**：可解释/可追溯 → **Clarify 解释 + Monitor 可观测 + A2I 人审**。([AWS ドキュメント][4])

---

## 7) 一句话总结

> \*\*责任 AI 六要素不是口号，而是“可测、可控、可审计”的工程能力。\*\*在 AWS 上用 **Guardrails（安全/注入/PII）**、**Clarify（偏差/解释）**、**Model Monitor（生产漂移/偏差）**、**A2I（人类复核）**，就能把“偏差、公平性、包容性、坚牢性、安全性、信赖性”逐一识别并闭环治理。([AWS ドキュメント][1])

如果你想要，我可以把上面的要点再整理成**一页速记表（中日英术语 + 触发场景 + AWS 工具）**，方便临考冲刺。

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[2]: https://aws.amazon.com/ai/responsible-ai/?utm_source=chatgpt.com "Building AI Responsibly"
[3]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-configure-processing-jobs.html?utm_source=chatgpt.com "Fairness, model explainability and bias detection with ..."
[4]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-monitor.html?utm_source=chatgpt.com "Data and model quality monitoring with ..."
[5]: https://aws.amazon.com/sagemaker/ai/clarify/?utm_source=chatgpt.com "Amazon SageMaker Clarify - AWS"
[6]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-model-monitor-bias-drift.html?utm_source=chatgpt.com "Bias drift for models in production - Amazon SageMaker AI"
[7]: https://docs.aws.amazon.com/sagemaker/latest/dg/a2i-use-augmented-ai-a2i-human-review-loops.html?utm_source=chatgpt.com "Using Amazon Augmented AI for Human Review"
[8]: https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-injection.html?utm_source=chatgpt.com "Prompt injection security - Amazon Bedrock"
[9]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-components.html?utm_source=chatgpt.com "Create your guardrail - Amazon Bedrock"
[10]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-sensitive-filters.html?utm_source=chatgpt.com "Remove PII from conversations by using sensitive information ..."
[11]: https://aws.amazon.com/blogs/security/ai-lifecycle-risk-management-iso-iec-420012023-for-ai-governance/?utm_source=chatgpt.com "AI lifecycle risk management: ISO/IEC 42001:2023 for AI ..."
[12]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-prompt-attack.html?utm_source=chatgpt.com "Detect prompt attacks with Amazon Bedrock Guardrails"
[13]: https://aws.amazon.com/ai/responsible-ai/policy/?utm_source=chatgpt.com "AWS Responsible AI Policy"
[14]: https://aws.amazon.com/blogs/machine-learning/aws-achieves-iso-iec-420012023-artificial-intelligence-management-system-accredited-certification/?utm_source=chatgpt.com "AWS achieves ISO/IEC 42001:2023 Artificial Intelligence ..."
