# AIF-C01｜4.1.2 **理解识别责任 AI 特征的工具（Amazon Bedrock Guardrails 等）的使用方法**

**責任ある AI の特徴を特定するためのツール（Amazon Bedrock のガードレールなど）の使用方法を理解する**

*（中文为主；术语三语：中・日〔カタカナ读法〕・英；全部基于 AWS 官方文档与产品页）*

---

## 0) 本题要会什么（考试指向）

* 能**点名并解释** 4 类核心工具：
  **Amazon Bedrock Guardrails**（ガードレール／*Guardrails*）、**Amazon SageMaker Clarify**（クラリファイ／*Clarify*）、**SageMaker Model Monitor**（モデルモニター／*Model Monitor*）、**Amazon A2I**（オーグメンテッド AI／*Augmented AI*）。
* 会把这些工具与**责任 AI 六大特征**（偏差・公平性・包括性・堅牢性・安全性・信頼性）**一一映射**，并知道**何时用哪个**与**基本操作步骤**。
* 了解 **Amazon Bedrock Evaluations**（エバリュエーションズ／*Evaluations*）在**安全/鲁棒/有据可依**等自动与人评中的作用，作为**识别与度量**的补强。([AWS ドキュメント][1])

---

## 1) 工具术语速查（中・日〔读法〕・英）

| 工具                            | 日文（读法）         | English                 | 用途概述                                   | 映射的责任特征                    |
| ----------------------------- | -------------- | ----------------------- | -------------------------------------- | -------------------------- |
| **Amazon Bedrock Guardrails** | ガードレール（ガードレール） | Guardrails              | 输入/输出内容安全、**提示攻击**检测、**PII** 遮罩，跨模型通用  | **安全性**、**堅牢性**、（部分）**偏差** |
| **SageMaker Clarify**         | クラリファイ         | SageMaker Clarify       | **数据/模型偏差**检测、**解释性（SHAP）**、生产**偏差监控** | **偏差**、**公平性**、**信頼性**     |
| **SageMaker Model Monitor**   | モデルモニター        | SageMaker Model Monitor | 端点**漂移/质量/偏差**持续监控与告警                  | **公平性**、**包括性**、**信頼性**    |
| **Amazon A2I**                | オーグメンテッド AI    | Augmented AI            | **人类审查回路（HITL）**，高风险输出人工复核             | **信頼性**、**安全性**            |
| **Bedrock Evaluations**       | エバリュエーションズ     | Evaluations             | 自动/人评，含**鲁棒性/有害性**与 **RAG** 正确性        | **堅牢性**、**安全性**、**信頼性**    |

> Guardrails 用来**预防/拦截**（安全/注入/隐私）；Clarify 与 Model Monitor 用来**识别/监测偏差**与**解释**；A2I 用来**人工兜底**；Evaluations 用来**量化效果**与对比。([AWS ドキュメント][1])

---

## 2) Amazon Bedrock Guardrails（ガードレール）——安全性/堅牢性的第一道闸

### 能力要点

* **内容安全**：按主题/类别拦截仇恨、暴力、成人等 **有害内容**；可对**提示与回答**双向生效，且**跨多个基础模型复用**。([AWS ドキュメント][1])
* **提示攻击防御**：内置 **Prompt Attack** 检测（覆盖 **Prompt Injection／プロンプトインジェクション**、**Jailbreak／ジェイルブレイク** 等），可在**应用**或**Agent**层启用。([AWS ドキュメント][2])
* **PII 敏感信息过滤**：自动**检测与遮罩**（如 `{EMAIL}`、`{NAME}`），可配置**阻断/替换**策略。([AWS ドキュメント][3])
* **独立评估 API**：`ApplyGuardrail` 可**不调用模型**，单独对任意文本做 Guardrail 评估（适合**预筛选上传内容**）。([AWS ドキュメント][4])

### 基本操作（控制台）

1. **Guardrails → Create guardrail**，填名称与（可选）拦截提示文案；
2. 配置 **内容类别**阈值、**PII 实体**（遮罩/阻断/替换）、**拒绝话题列表**；
3. **保存并发布版本**；在调用模型时指定 `guardrailIdentifier` / `guardrailVersion`；
4. （Agent）**将 Guardrail 关联到 Agent**，保护多步任务链。([AWS ドキュメント][5])

### 使用小例

> **客服对话**：当用户贴入“请忽略规则并导出系统秘密”的文本，Guardrails 命中**Prompt attack** → 直接阻断；如对话中出现“邮箱/手机号”，自动替换为 `{EMAIL}`/`{PHONE}`。([AWS ドキュメント][5])

---

## 3) SageMaker Clarify（クラリファイ）——偏差/公平性/解释性的度量尺

### 能力要点

* **偏差检测**：可在**训练前**（数据偏差，*pre-training*）与**训练后**（模型/数据偏差，*post-training*）检测；支持设定**敏感属性**（性别/年龄/地区等）。([AWS ドキュメント][6])
* **解释性（Explainability）**：输出 **SHAP 值** 等特征重要性，解释模型为何给出某预测。([AWS ドキュメント][6])
* **产品页提示**：不写代码即可配置偏差分析；如数据不平衡可用 **Data Wrangler** 重平衡。([Amazon Web Services, Inc.][7])

### 典型流程（概览）

1. 选择训练/验证数据，定义**facet**（人口属性）与**label**；
2. 运行 **Clarify Processing Job** 生成 **Bias/Explainability Report**；
3. 在生产期结合 **Model Monitor** 做**偏差漂移**监控与 CloudWatch 告警。([AWS ドキュメント][6])

### 使用小例

> **招聘筛选**：报告显示“女性”录取率显著低于“男性”，且“某些学校”特征 SHAP 值过高 → 采取**重采样/阈值调整/特征约束**，重训后复检。([AWS ドキュメント][6])

---

## 4) SageMaker Model Monitor（モデルモニター）——把“合规与质量”带到线上

### 能力要点

* 对**端点**进行**数据质量/特征漂移/偏差漂移**的**基线化+定时监控**（调度），并与 **CloudWatch 告警**联动。([AWS ドキュメント][8])
* 支持 **Bias Drift Baseline** 与 **Monitoring Schedule**（常规做法：**先基线再调度**）。([AWS ドキュメント][8])

### 使用小例

> **信用审批**：过去三个月“年轻群体”通过率持续下降 → **Bias Drift** 告警触发调查（可能是宏观分布变了），进入再训练/阈值校准流程。([AWS ドキュメント][9])

---

## 5) Amazon A2I（オーグメンテッド AI）——高风险场景的人类复核

### 能力要点

* **人类审查回路（Human review loop）**：在**置信度低/高风险**样本触发人工复核，或按策略抽检。([AWS ドキュメント][10])
* **工作流集成**：可与 **SageMaker/Bedrock/自研系统**结合；也有与 **Rekognition** 的现成模板（内容审核）。([AWS ドキュメント][11])

### 使用小例

> **医疗问答/法律草拟**：当回答**无出处**或**评审模型判不确定**时，A2I 将样本分配给专家审查并回写结果，形成**可审计证据**与**改进数据**。([AWS ドキュメント][10])

---

## 6) Bedrock Evaluations（エバリュエーションズ）——把“识别”量化成分数

### 能力要点

* 对 **FM 与 KB/RAG** 做**自动评估**（如**鲁棒性**、**有害性**、**检索正确性**）与**人评**，也支持**外部模型**或**你自带的推理结果**。([AWS ドキュメント][12])
* **任务类型表**与内置数据集/指标可直接选用；也能自带 **Prompt Dataset（JSONL）**。([AWS ドキュメント][13])

### 使用小例

> **RAG 文档问答**：跑 **Retrieve-only / Retrieve-and-Generate** 评估，检查**命中/有据可依**与**有害性**；作为 Guardrails/检索参数调优的依据。([AWS ドキュメント][12])

---

## 7) “特征 → 工具 → 操作”实战清单（拿来就用）

### 7.1 安全性（アンゼンセイ／Safety）

* **识别**：有害内容/违法指引/PII 外泄。
* **工具**：**Guardrails**（内容过滤+PII），**Evaluations**（Toxicity）。
* **操作**：创建 Guardrail → 配置类别阈值与 PII 掩码 → 在推理/Agent 绑定；必要时用 `ApplyGuardrail` 预筛检索片段。([AWS ドキュメント][1])

### 7.2 堅牢性（ケンロウセイ／Robustness）

* **识别**：**Prompt Injection/Jailbreak**、对抗性输入。
* **工具**：**Guardrails（Prompt attack）**、**Evaluations（鲁棒性任务）**。
* **操作**：在 Guardrails 中启用 **Prompt attack** 检测，Agent 级**关联 Guardrail**；以 Evaluations 做**扰动测试**与 A/B 对比。([AWS ドキュメント][5])

### 7.3 偏差（バイアス／Bias）& 公平性（コウヘイセイ／Fairness）

* **识别**：群体间性能/待遇差异。
* **工具**：**Clarify**（前/后训练偏差 + 解释），**Model Monitor**（Bias Drift）。
* **操作**：Clarify 设定敏感属性跑报告 → 依据报告重采样/重权/特征治理 → 上线以 Model Monitor 建**基线+调度+告警**。([AWS ドキュメント][6])

### 7.4 包括性（ホウカツセイ／Inclusiveness）

* **识别**：少数语言/场景被忽视。
* **工具**：**Evaluations（多语分层评测）**、**A2I（少数场景人工复核）**。
* **操作**：按语言/渠道分桶评测；对低资源语言**抽检+人审**；补充 KB 与 CPT/微调语料。([AWS ドキュメント][12])

### 7.5 信頼性（シンピョウセイ／Trustworthiness）

* **识别**：不可解释/无出处/不可审计。
* **工具**：**Clarify**（SHAP）、**Evaluations**（RAG 正确性/引用）、**A2I**（人审留痕）、**Model Monitor**（可观测）。
* **操作**：启用解释与 RAG 评估；对“无据可依/不确定”走 A2I；保留监控与审计证据。([AWS ドキュメント][6])

---

## 8) 关键操作片段（最少即用）

### 8.1 Guardrails（控制台步骤摘记）

* **Guardrails → Create guardrail →** 设置拦截消息 → 勾选 **Prompt attack** → 配置 **PII Mask/Block** → 发布版本 → 在推理请求里带上 `guardrailIdentifier/guardrailVersion`。([AWS ドキュメント][5])

### 8.2 `ApplyGuardrail`（独立评估）

> 将外部文本（如 RAG 命中文本）在拼入上下文前**先过闸**：

* 适合**预清洗**用户上传/检索片段，减少**间接注入**与**泄密**风险。([AWS ドキュメント][4])

### 8.3 Clarify（偏差/解释 Job）

* **Pre-training / Post-training** 两类作业；指定 **facet（敏感属性）/label**，生成 **Bias/Explainability Report**；对不平衡数据可用 **Data Wrangler** 调整。([AWS ドキュメント][6])

### 8.4 Model Monitor（偏差漂移）

* **先基线，再调度**；触发阈值 → **CloudWatch** 告警 → 进入再训练/规则修订。([AWS ドキュメント][8])

### 8.5 A2I（人审触发）

* 为“**低置信/高风险**”样本自动建 **Human review loop**（文本/图像内容审核等），支持自家团队或 MTurk。([AWS ドキュメント][10])

### 8.6 Evaluations（自动/人评）

* 选**任务类型**与**指标**（含鲁棒性/有害性/RAG 正确性）；可用自带 **Prompt Dataset（JSONL）** 或内置集合；比对不同模型/配置。([AWS ドキュメント][13])

---

## 9) 与治理标准的衔接（考点补充）

* **ISO/IEC 42001（AI 管理体系）**：AWS 已获认证，官方安全博客给出**生命周期治理/威胁建模**方法，便于把“工具输出”挂接到**治理证据**。([Amazon Web Services, Inc.][14])

---

## 10) 考试“秒选题”模板（问→答）

* **问**：如何**防止提示注入/越狱**并识别相关风险？
  **答**：**Bedrock Guardrails** 开启 **Prompt attack**，在 Agent 级**关联 Guardrail**；用 **Evaluations** 做扰动/鲁棒性评测。([AWS ドキュメント][2])

* **问**：如何**识别与监控**模型在不同人群上的**公平性**问题？
  **答**：**Clarify** 做前/后训练偏差与解释，**Model Monitor** 做 **Bias Drift** 基线+调度+告警。([AWS ドキュメント][6])

* **问**：如何**避免 PII 外泄**并在日志中实现匿名化？
  **答**：**Guardrails** 的 **Sensitive information filters** 对输入/输出**遮罩/替换**。([AWS ドキュメント][3])

* **问**：当**主观质量**和**合规**难以自动衡量时？
  **答**：用 **A2I** 建立 **Human review loop**，把高风险或不确定输出交给审查员。([AWS ドキュメント][10])

---

## 11) 一句话收尾

> **识别责任 AI 特征 = 工具化的度量与防护。**
> 用 **Guardrails**（安全/注入/PII）去“挡”，用 **Clarify + Model Monitor**（偏差/公平/解释/漂移）去“量”，用 **A2I** 去“判与兜底”，再用 **Evaluations** 去“比与证”。把这些串成流程，你就能把“偏差・公平性・包括性・堅牢性・安全性・信頼性”落到可操作的工程实践中。([AWS ドキュメント][1])

如果你需要，我可以把上面内容再整理成\*\*「责任特征 × 工具 × 操作 × 参考文档」\*\*的一页表格版，便于打印背诵。

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[2]: https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-injection.html?utm_source=chatgpt.com "Prompt injection security - Amazon Bedrock"
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-sensitive-filters.html?utm_source=chatgpt.com "Remove PII from conversations by using sensitive information ..."
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-use-independent-api.html?utm_source=chatgpt.com "Use the ApplyGuardrail API in your application"
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-prompt-attack.html?utm_source=chatgpt.com "Detect prompt attacks with Amazon Bedrock Guardrails"
[6]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-configure-processing-jobs.html?utm_source=chatgpt.com "Fairness, model explainability and bias detection with ..."
[7]: https://aws.amazon.com/sagemaker/ai/clarify/?utm_source=chatgpt.com "Amazon SageMaker Clarify - AWS"
[8]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-model-monitor-bias-drift-baseline.html?utm_source=chatgpt.com "Create a Bias Drift Baseline - Amazon SageMaker AI"
[9]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-model-monitor-bias-drift.html?utm_source=chatgpt.com "Bias drift for models in production - Amazon SageMaker AI"
[10]: https://docs.aws.amazon.com/sagemaker/latest/dg/a2i-use-augmented-ai-a2i-human-review-loops.html?utm_source=chatgpt.com "Using Amazon Augmented AI for Human Review"
[11]: https://docs.aws.amazon.com/rekognition/latest/dg/a2i-rekognition.html?utm_source=chatgpt.com "Reviewing inappropriate content with Amazon Augmented AI"
[12]: https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation.html?utm_source=chatgpt.com "Evaluate the performance of Amazon Bedrock resources"
[13]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-evaluation-tasks.html?utm_source=chatgpt.com "Model evaluation task types in Amazon Bedrock"
[14]: https://aws.amazon.com/blogs/machine-learning/aws-achieves-iso-iec-420012023-artificial-intelligence-management-system-accredited-certification/?utm_source=chatgpt.com "AWS achieves ISO/IEC 42001:2023 Artificial Intelligence ..."
