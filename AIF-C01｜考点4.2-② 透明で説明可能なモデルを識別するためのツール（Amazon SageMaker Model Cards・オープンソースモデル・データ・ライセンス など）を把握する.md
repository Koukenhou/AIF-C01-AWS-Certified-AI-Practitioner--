# AIF-C01｜4.2.2 **掌握识别“透明・可解释”模型的工具（SageMaker Model Cards・开源模型卡・数据卡・许可等）**

**透明で説明可能なモデルを識別するためのツール（Amazon SageMaker Model Cards・オープンソースモデル・データ・ライセンス など）を把握する**

*中文为主；术语三语：中・日〔カタカナ读法〕・英；大量结合 AWS 官方资料与可操作步骤。*

---

## 0) 本题你要做到什么（考试导向）

* 认得**四类“透明度工件”**：

  1. **模型卡**（Model Card｜モデルカード）——你的模型；
  2. **服务卡**（AI Service Card｜サービスカード）——AWS 托管 AI 服务/基础模型；
  3. **开源模型卡 & 数据卡**（Hugging Face **Model Card** / **Datasheet**）——第三方/自建；
  4. **许可信息**（License｜ライセンス）——模型/数据/代码的使用边界。
* 会用 **SageMaker Model Cards** 生成/维护模型卡；会在 **Responsible AI / AI Service Cards** 页面查阅 AWS 模型/服务的透明信息；会检查**开源模型卡 + 许可**是否满足业务与合规；能把**解释性证据**（如 Clarify 的 SHAP）写入卡片以完成闭环。([AWS ドキュメント][1])

---

## 1) 关键术语（中・日〔读法〕・英）

| 概念        | 日文（读法）         | English                | 要点                                                                                        |
| --------- | -------------- | ---------------------- | ----------------------------------------------------------------------------------------- |
| 模型卡       | モデルカード（モデルカード） | Model Card             | 记录**用途、数据、评测、风险**等；SageMaker 提供**原生**模型卡与 API。([AWS ドキュメント][1])                           |
| 服务卡       | サービスカード        | AI Service Card        | AWS 对托管 AI 服务/基础模型的**用途、限制、设计选择**等透明文档（如 **Nova/Titan**）。([Amazon Web Services, Inc.][2]) |
| 开源模型卡     | オープンソース・モデルカード | Open-source model card | Hugging Face 等平台的**README/元数据**，描述训练数据、限制、评测、许可。([Hugging Face][3])                       |
| 数据卡/数据说明书 | データカード／データシート  | Datasheet for Datasets | 记录**数据来源、组成、采集过程、许可**的透明文档。([arXiv][4])                                                   |
| 许可        | ライセンス          | License                | 约束**可用范围/再分发/商用/衍生**等（Apache-2.0、MIT、CC、专有条款等）。                                           |

---

## 2) 工具地图：如何“识别透明・可解释的模型”

| 工具/来源                                 | 你能从中看到什么                                           | 用于判断什么                                                                                   |
| ------------------------------------- | -------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **SageMaker Model Cards**（你的模型）       | 训练数据范围、评测集与指标、偏差与解释（可嵌 SHAP 图/数值）、风险与使用限制、负责人与审批状态 | 这是否是**可解释/可审计**的交付物；是否有**完整证据链**支撑上线与复审。([AWS ドキュメント][1])                                |
| **AWS AI Service Cards**（AWS 托管模型/服务） | 适用/不适用场景、负责任 AI 设计选择、性能/调优建议、局限与风险                 | 所选**Bedrock 模型/服务**是否满足用例的**透明与限制**要求（例如 Nova/Titan 系列）。([Amazon Web Services, Inc.][2]) |
| **开源 Model Card（HF 等）**               | 训练语料概览、评测指标、伦理考量、已知偏差、**许可**                       | 第三方模型是否**信息充分**、是否**允许商用/微调/分发**。([Hugging Face][3])                                     |
| **Datasheets for Datasets**（数据卡）      | 数据动机/组成/采集/清洗/许可/风险                                | 训练/评测/检索语料是否**可追溯**、**合法可用**与**代表性充分**。([arXiv][4])                                      |

> AWS 的 **Responsible AI** 官方页明确把 **AI Service Cards** 作为提升透明度的核心资源；近年新增了 **Nova/Titan** 多张服务卡。([Amazon Web Services, Inc.][2])

---

## 3) Amazon SageMaker Model Cards（模型卡）——你自己的“透明度单据”

### 3.1 能力与价值

* 在**一个位置**汇总模型**全生命周期**关键信息，用于**治理与合规**（审批、再认证、审计）。([AWS ドキュメント][1])
* 可经**控制台向导**创建与维护，也提供 **API/JSON 模式**（`ModelCard` 资源）。([AWS ドキュメント][5])

### 3.2 典型结构（建议最少字段）

1. **用途 & 业务范围**（Intended use）：目标任务、用户、禁止用途；
2. **数据**：训练/验证/线上数据来源与时间窗、数据卡链接；
3. **评测**：数据集、指标（含**分组/子群**）、阈值、A/B 结论；
4. **解释 & 公平**：**Clarify** 的 SHAP 总结与偏差度量；([AWS ドキュメント][6])
5. **风险 & 限制**：幻觉/偏差/已知失败模式、Guardrails 策略；
6. **运营**：监控与告警（Model Monitor / CloudWatch）、再训练触发；
7. **审批流**：负责人、评审日期、版本/变更记录。

### 3.3 三步创建（控制台）

1. **SageMaker Console → Governance → Model cards → Create**；
2. 按向导填写**用途/数据/评测/风险**；
3. 生成卡片，提交**审批/发布**（可后续版本化）。([AWS ドキュメント][5])

> 实战建议：把 **Clarify** 的**偏差/解释**报告与 **Bedrock Evaluations** 的**真实性/鲁棒**结果嵌入卡片，形成**可验证证据**。([AWS ドキュメント][6])

---

## 4) AWS AI Service Cards（服务卡）——托管模型/服务的“透明说明书”

* 官方资源页集中列出卡片；**每张卡**说明**预期用途、限制、负责任设计选择、优化建议**；卡片会随服务演进而更新（例如 **Amazon Nova Reel / Nova Micro / Titan** 等均有卡）。([Amazon Web Services, Inc.][2])
* **用法**：选型/合规评审时把**候选模型的服务卡**对比；将**不适用场景**写进你产品的使用条款或提示中。

---

## 5) 开源模型 & 数据：如何读“模型卡 / 数据卡 / 许可”

### 5.1 读 Model Card（HF）

* 关注**训练数据来源**（是否含抓取的网页、版权状态）、**评测集与指标**、**已知偏差/限制**、**适用/不适用**与**安全考量**、**License**。([Hugging Face][3])

### 5.2 读 Datasheet（数据卡）

* 看**动机/组成/采集**、**清洗与标注**、**许可/合规**与**已知问题**；若无数据卡，至少用**仓库 README + issue**查证来源与限制。([arXiv][4])

### 5.3 许可（License）三问

1. **是否允许商用/微调/再分发**？
2. **是否要求署名/开源衍生**（Copyleft/ShareAlike）？
3. **是否有额外行为限制**（如“不得用于生物/军事”等自定义条款）？

> HF 提供**模型/数据卡规范与工具**（可生成/更新卡片），便于在企业仓库中标准化记录。([Hugging Face][7])

---

## 6) 把“解释性证据”写进透明度工件（操作串联）

1. **离线解释/偏差**：用 **SageMaker Clarify** 生成 **SHAP + 偏差**报告。([AWS ドキュメント][6])
2. **真实性与鲁棒**（生成式/RAG）：用 **Bedrock Evaluations** 跑 **Retrieve-only / Retrieve-and-Generate** 与**鲁棒/正确性**作业。([AWS ドキュメント][8])
3. **模型卡归档**：在 **Model Card** 的评测/风险章节嵌入以上证据，并给出**最小上线阈值**与再评估周期。([AWS ドキュメント][1])
4. **服务卡引用**：在你产品文档中**引用所选 Bedrock 模型的 Service Card**要点（用途/限制/设计选择），对齐**对外透明**。([Amazon Web Services, Inc.][2])
5. **运行期**：把**监控与审计路径**（CloudWatch、Model Monitor）也写进卡片作为“运维透明”。([AWS ドキュメント][6])

---

## 7) 场景化范式（如何“识别为透明模型”）

### 7.1 信贷审批（高监管 → 可解释优先）

* **识别流程**：

  * 首选**可解释模型**（树/线性）→ 用 **Clarify** 出 SHAP + 群体偏差；
  * 用 **SageMaker Model Card** 记录**数据/评测/阈值**与审批日志；
  * 若引入 Bedrock 生成式组件（说明文案），查阅对应**Service Card**确保**用途/限制**清晰。([AWS ドキュメント][6])
* **判断**：卡片证据完整、分组指标达标、服务限制已披露 ⇒ **可视为“透明・可解释”**。

### 7.2 企业文档助手（生成式 → 真实性优先）

* **识别流程**：

  * 候选 FM 的 **Service Card** 对比用途/限制；
  * RAG + **Bedrock Evaluations** 看“有据可依/检索正确性”；
  * 在 **Model Card** 里写明“引用策略/无据时退让”，附**Guardrails**配置与上线阈值。([Amazon Web Services, Inc.][2])

### 7.3 采用开源中文模型（许可与来源优先）

* **识别流程**：

  * 读取 **HF 模型卡**与**License**，核对**商用/微调/再分发**条款；
  * 若模型卡缺少训练数据来源/限制 ⇒ **透明度不足**，需补充数据/重新评估或更换；
  * 内部生成一份 **SageMaker Model Card** 记录**二次训练数据卡/评测**与法务审查结论。([Hugging Face][3])

---

## 8) 速用清单（Checklists）

### 8.1 “模型是否透明可解释？”十问

1. 是否有**模型卡/服务卡**（且更新至最新版本）？([AWS ドキュメント][5])
2. 训练/验证数据**来源可追溯**（有数据卡/链接）？([arXiv][4])
3. 指标是否包含**分组/子群**结果与阈值？
4. 是否有**解释性证据**（SHAP 等）与**主要特征列表**？([AWS ドキュメント][6])
5. 生成式是否评过**真实性/有据可依**（RAG 作业）？([AWS ドキュメント][8])
6. 是否明确**不适用场景/已知风险**（来自服务卡/模型卡）？([Amazon Web Services, Inc.][2])
7. 许可是否**允许商用/微调/托管**？需不需要**署名/共享相同**？
8. 是否有**运行期监控**与**再评估**计划（Model Monitor/时间阈值）？([AWS ドキュメント][6])
9. 是否写明**数据保护边界**与合规措施（Bedrock/AgentCore 数据保护条款）？([AWS ドキュメント][9])
10. 审批人与责任人是否在**卡片中明确**（可审计）？([AWS ドキュメント][1])

### 8.2 “开源模型合规”四步

* **看卡**（HF Model Card）→ **看证据**（评测/限制/偏差）→ **看 License**（可用性）→ **自建 Model Card** 记录你的适配与结论。([Hugging Face][3])

---

## 9) 与考试相关的要点摘记（问→答）

* **问：如何用 AWS 工具识别模型是否透明可解释？**
  **答**：用 **SageMaker Model Cards** 统一记录**用途/数据/评测/解释/风险**；查阅 **AI Service Cards** 了解托管模型的**用途与限制**；开源模型核对 **Model Card + License**；把 **Clarify/Bedrock Evaluations** 的证据写入卡片并设**再评估**机制。([AWS ドキュメント][1])
* **问：为什么还要看“服务卡”？**
  **答**：服务卡是 AWS 的**官方透明工件**，覆盖**预期用途、限制与设计选择**，并持续更新（Nova/Titan 等）。([Amazon Web Services, Inc.][2])
* **问：数据/许可如何体现在“透明”里？**
  **答**：用**数据卡**记录来源与许可；**在模型卡引用并约束用途**；无数据卡或许可不清＝**透明度不足**与合规风险。([arXiv][4])

---

## 参考（官方优先）

* **Amazon SageMaker Model Cards（概念与指南 / 控制台创建 / API）**。([AWS ドキュメント][1])
* **AWS Responsible AI & AI Service Cards（总览与 Nova/Titan 示例）**。([Amazon Web Services, Inc.][2])
* **SageMaker Clarify（解释性与偏差、上线监控）**。([AWS ドキュメント][6])
* **Hugging Face Model Cards（指南 & 规范）**。([Hugging Face][3])
* **Datasheets for Datasets（数据透明框架）**。([arXiv][4])
* **Bedrock / AgentCore 数据保护（边界与最佳实践）**。([AWS ドキュメント][9])

---

### 一句话总结

> **“可解释 + 可证据 + 可审计 + 有许可” = 透明模型。**
> 在 AWS 上，用 **SageMaker Model Cards** 管“你的模型”，查 **AI Service Cards** 了解“托管模型”，核对 **开源模型卡/数据卡/许可**定位风险，把 **Clarify** 与 **Bedrock Evaluations** 的**解释与真实性证据**写进卡片，就能**系统性识别与证明**一个模型的**透明与可解释**。

[1]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-cards.html?utm_source=chatgpt.com "Amazon SageMaker Model Cards"
[2]: https://aws.amazon.com/ai/responsible-ai/?utm_source=chatgpt.com "Building AI Responsibly"
[3]: https://huggingface.co/docs/hub/en/model-cards?utm_source=chatgpt.com "Model Cards"
[4]: https://arxiv.org/abs/1803.09010?utm_source=chatgpt.com "[1803.09010] Datasheets for Datasets"
[5]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-cards-create.html?utm_source=chatgpt.com "Create a model card - Amazon SageMaker AI"
[6]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-explainability.html?utm_source=chatgpt.com "Evaluate, explain, and detect bias in models"
[7]: https://huggingface.co/docs/hub/en/model-card-guidebook?utm_source=chatgpt.com "Model Card Guidebook"
[8]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-configure-processing-jobs.html?utm_source=chatgpt.com "Fairness, model explainability and bias detection with ..."
[9]: https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html?utm_source=chatgpt.com "Data protection - Amazon Bedrock"
