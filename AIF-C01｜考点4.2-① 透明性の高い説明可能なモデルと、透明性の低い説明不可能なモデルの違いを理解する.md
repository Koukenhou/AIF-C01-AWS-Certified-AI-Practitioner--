# AIF-C01｜4.2.1 **理解“高透明・可解释”模型 与 “低透明・不可解释”模型 的差异**

**透明性の高い説明可能なモデルと、透明性の低い説明不可能なモデルの違いを理解する**

*中文为主；术语三语：中・日〔カタカナ读法〕・英；结合 AWS 官方文档与产品能力。*

---

## 0) 先讲考点（你要会什么）

* 能**下定义并对比**两类模型：

  * **可解释（Explainable／説明可能〔セツメイ・カノウ〕）**与**高透明（Transparent／トランスペアレント）**
  * **不可解释（Unexplainable／説明不可能〔セツメイ・フカノウ〕）**与**低透明（Opaque／オペーク）**
* 知道 **AWS 的透明度工件**：**SageMaker Clarify（可解释性与偏差）**、**SageMaker Model Cards（モデルカード）**、**AWS AI Service Cards（サービスカード）**、以及 **Responsible AI 维度**。([AWS ドキュメント][1])

---

## 1) 术语对照（中・日〔读法〕・英）

| 中文       | 日文（读法）            | English                     | 一句话理解                                                                              |
| -------- | ----------------- | --------------------------- | ---------------------------------------------------------------------------------- |
| 透明性（高/低） | 透明性（トウメイセイ）       | Transparency                | **外界能看到/理解**模型用途、限制、数据、风险与输出依据的程度。([D1 AWS Static][2])                             |
| 可解释性     | 説明可能性（セツメイ・カノウセイ） | Explainability              | **为何**得出某个预测/回答的**可理解证据**（局部/全局）。([AWS ドキュメント][1])                                 |
| 模型卡      | モデルカード            | Model card                  | 记录模型用途、数据、性能、局限的文档资产（SageMaker Model Cards）。([AWS ドキュメント][3])                      |
| 服务卡      | サービスカード           | AI Service Card             | AWS 对托管 AI 服务公开**用途/限制/设计选择**的透明度文档（如 Titan/Nova）。([Amazon Web Services, Inc.][4]) |
| 局部/全局解释  | ローカル/グローバル説明      | Local/Global explainability | 解释**单条预测**与**整体行为**（Clarify 提供 SHAP 等）。([AWS ドキュメント][1])                           |

---

## 2) “可解释 & 高透明” vs “不可解释 & 低透明” —— 核心差异

| 维度            | 高透明・可解释（例：线性/树/规则、带文档的生成式）                                         | 低透明・不可解释（例：深度神经网/大型FM黑箱）                                                      |
| ------------- | ------------------------------------------------------------------ | ----------------------------------------------------------------------------- |
| **推理可理解**     | **可溯因**：特征权重、路径、规则；或用 **SHAP** 给出贡献度。([AWS ドキュメント][1])             | **难溯因**：需事后解释；解释存在**保真度**与**稳定性**问题                                           |
| **文档透明**      | 具备 **Model Card / AI Service Card**；用途与限制清晰。([AWS ドキュメント][3])      | 若无公开卡片/数据来源说明 ⇒ 透明度低                                                          |
| **合规/审计**     | 易于与法规/内控对齐（信贷、医疗等高监管）。                                             | 需**附加治理**（强审计、HITL、人评、守护栏）才能达标                                                |
| **性能上限**      | 往往较稳健、可控，但在复杂任务上可能**上限较低**。                                        | 性能强、泛化好，但**可解释与治理成本更高**                                                       |
| **典型 AWS 支撑** | **Clarify**（解释+偏差）、**Model Cards**、**AI Service Cards**、**RAG 引用** | **Clarify（事后解释）**、**AI Service Cards**、**Guardrails**、**Evaluations**、**A2I** |

> AWS Responsible AI 强调：**Explainability／Transparency** 是核心维度之一，应贯穿全生命周期。([Amazon Web Services, Inc.][5])

---

## 3) AWS 上如何实现“解释与透明”

### 3.1 模型层解释（可解释性）

* **SageMaker Clarify**：

  * **模型可解释性（Model Explainability）**：对训练前后模型输出进行 **SHAP** 等归因（全局/局部）。
  * **常见用途**：调参、发现“过度依赖敏感特征”、向合规/业务说明“为何拒绝/通过”。([AWS ドキュメント][1])

### 3.2 文档层透明（透明工件）

* **SageMaker Model Cards**：为你的自建模型生成“**模型卡**”，统一记录**用途/数据/评测/风险/限制**，用于治理与审计。([AWS ドキュメント][3])
* **AWS AI Service Cards**：AWS 为托管服务（如 **Amazon Titan/Nova**）提供“**服务卡**”，公开**预期用途、限制、公平性与安全性考虑**，便于选型与风控。([Amazon Web Services, Inc.][4])

### 3.3 生成式特有的“真实性透明”

* **RAG + 引用**：回答附**出处**（可追溯）；
* **Bedrock Evaluations**：评估“**检索正确性/有据可依**、鲁棒性”等，形成可对比的报告。([D1 AWS Static][2])

---

## 4) 什么时候必须选“高透明・可解释”？

| 场景            | 透明/解释性要求              | 推荐做法（AWS）                                                                                |
| ------------- | --------------------- | ---------------------------------------------------------------------------------------- |
| **信贷审批/保险定价** | **强监管**；需给拒贷/定价**理由** | 首选**可解释模型**（树/线性/GBDT）+ **Clarify（SHAP）**；用 **Model Card** 归档。([AWS ドキュメント][1])          |
| **医疗分诊/药物推荐** | 高风险，需**可追责**          | 可解释模型或黑箱+强解释；**A2I** 人审；**Model Card** + 运行日志。                                           |
| **公共部门决策**    | 透明度与公平性要求高            | **Clarify 子群分析**；**Service Cards** 审核用途/限制；公开文档以符合法规与伦理。([Amazon Web Services, Inc.][4]) |
| **营销文案/创意生成** | 可解释性**相对次要**          | 选性能更强的 FM + **Service Cards** + **Guardrails/Evaluations** 管控风险。                         |

---

## 5) “黑箱也要负责”——不可解释模型的工程化补救

> 当你因性能或任务复杂度选择大型神经网络/基础模型（FM）时，务必加上**解释与透明度的外围控制**：

1. **事后解释**：用 \*\*Clarify（SHAP）\*\*做全局/局部归因，监控“关注点漂移”。([AWS ドキュメント][1])
2. **透明文档**：创建 **Model Card**；查阅所用服务的 **AI Service Card**（如 Titan/Nova）。([AWS ドキュメント][3])
3. **真实性**：采用 **RAG + Evaluations** 以“有据可依”度量与展示证据。([D1 AWS Static][2])
4. **安全/合规**：在 **Bedrock** 启用 **Guardrails**（有害/PII/提示攻击），并将策略与效果纳入透明度说明。
5. **人类监督**：对**高风险/低置信**样本触发 **A2I** 人审，并保存审计记录。

---

## 6) 例子：同一业务两种路线对比

### 6.1 信用卡额度提升（高透明方案）

* **模型**：GBDT（グラディエント・ブーステッド・ツリー／Gradient-Boosted Trees）
* **解释**：Clarify 产出 **SHAP**，说明“收入、偿付比、逾期次数”贡献度。
* **透明度**：SageMaker **Model Card** 记录数据时间窗、特征筛选原则与合规限制。([AWS ドキュメント][1])

### 6.2 企业知识库问答（性能优先方案）

* **模型**：FM（如 Amazon Titan 文本家族；参考 **Service Card** 了解用途/限制）([AWS ドキュメント][6])
* **真实性**：RAG + 引用；**Evaluations** 选“有据可依”更高的提示模板。([D1 AWS Static][2])
* **安全/透明**：启用 **Guardrails**；在系统文档中引用**Service Card**要点与本系统**使用边界**。

---

## 7) 操作清单（把“透明与解释”写进交付流程）

1. **立项**：明确本用例是否属于**高透明/解释**必需（法规/风险）。
2. **选型**：

   * 高透明需求 → 先试**可解释模型**；
   * 性能优先 → 选 FM，但承诺加上**Clarify + Model/Service Card + Evaluations + Guardrails**。
3. **文档**：

   * 生成并维护 **SageMaker Model Card**；
   * 研读并在说明书中引用相应 **AI Service Card**（Titan/Nova 等）。([AWS ドキュメント][3])
4. **评估**：**Clarify（SHAP/偏差）** + **Bedrock Evaluations（真实性/鲁棒性）**。([AWS ドキュメント][1])
5. **上线与监控**：记录调用与解释摘要到日志；对“漂移/不当输出”设告警与\*\*人审（A2I）\*\*回路。
6. **对外透明**：在隐私/合规页面公开**用途/限制/数据来源概要/人审机制**，对齐 **Responsible AI** 维度。([Amazon Web Services, Inc.][5])

---

## 8) 考试速记（问→答）

* **问：透明性 vs 可解释性有何不同？**
  **答**：**透明性**是**文档与过程公开**（用途/限制/数据/风险/证据）；**可解释性**是**对具体预测/回答的因果说明**（如 SHAP）。两者互补。([AWS ドキュメント][1])

* **问：AWS 如何提升模型透明度？**
  **答**：**SageMaker Model Cards**（你的模型）、**AWS AI Service Cards**（托管服务，如 Titan/Nova）。([AWS ドキュメント][3])

* **问：黑箱 FM 如何提高可解释/可信？**
  **答**：**Clarify（事后解释） + RAG 引用 + Evaluations（有据可依） + Guardrails + A2I**。([AWS ドキュメント][1])

---

## 参考（官方优先）

* **SageMaker Clarify：模型可解释性（SHAP）与偏差检测**。([AWS ドキュメント][1])
* **SageMaker Model Cards：模型卡治理与审计**。([AWS ドキュメント][3])
* **AWS AI Service Cards（透明度工件：用途/限制/公平性考虑）**；**Titan/Nova 服务卡**。([Amazon Web Services, Inc.][4])
* **Responsible AI（Explainability/Transparency 维度）**；**Well-Architected GenAI Lens**。([Amazon Web Services, Inc.][5])

---

### 一句话总结

> **“能解释 + 有文档 = 真透明”。**在 AWS 上，用 **Clarify** 给出**可解释性**，用 **Model Cards / AI Service Cards**提供**文档透明度**，再以 **RAG+Evaluations**证明**有据可依**、用 **Guardrails/A2I**控制**风险**，即可在“性能”和“责任”之间取得工程化平衡。

[1]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-model-explainability.html?utm_source=chatgpt.com "Model Explainability - Amazon SageMaker AI"
[2]: https://d1.awsstatic.com/products/generative-ai/responsbile-ai/AWS-Responsible-Use-of-AI-Guide-Final.pdf?utm_source=chatgpt.com "Responsible Use of AI Guide - awsstatic.com"
[3]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-cards.html?utm_source=chatgpt.com "Amazon SageMaker Model Cards"
[4]: https://aws.amazon.com/blogs/machine-learning/introducing-aws-ai-service-cards-a-new-resource-to-enhance-transparency-and-advance-responsible-ai/?utm_source=chatgpt.com "Introducing AWS AI Service Cards: A new resource to ..."
[5]: https://aws.amazon.com/ai/responsible-ai/?utm_source=chatgpt.com "Building AI Responsibly"
[6]: https://docs.aws.amazon.com/ai/responsible-ai/titan-text/overview.html?utm_source=chatgpt.com "Amazon Titan Text Lite and Titan Text Express - AWS AI ..."
