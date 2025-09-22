# AIF-C01｜4.2.4 **理解“以人为中心”的可解释 AI 设计原则（Human-Centered XAI）**

**説明可能なAIのための人間中心設計の原則を理解する**

*中文为主；术语三语：中・日〔カタカナ读法〕・英；结合 AWS 官方工具与文档。*

---

## 0) 考点定位（你需要会什么）

* 解释**什么叫“以人为中心”的可解释 AI**（Human-Centered Explainable AI），并能把它拆成**可执行的设计原则**。
* 会把这些原则**落到 AWS 工具链**：**SageMaker Clarify**（解释/偏差）、**SageMaker Model Cards**（模型卡）、**AWS AI Service Cards**（服务卡）、**Amazon A2I**（人审）、**Amazon Bedrock Guardrails**（安全/情境根拠チェック）、**Bedrock Evaluations**（真实性/鲁棒性/RAG 评估）、**Well-Architected Generative AI Lens**（治理基线）。([AWS ドキュメント][1])

---

## 1) 关键术语（中・日〔读法〕・英）

| 概念     | 日文（读法）                | English                    | 一句话理解                                                  |
| ------ | --------------------- | -------------------------- | ------------------------------------------------------ |
| 以人为中心  | 人間中心（ニンゲン・チュウシン）      | Human-centered             | 先定义**谁在用/谁受影响**，再决定**解释呈现**与**控制**方式。([AWS ドキュメント][1]) |
| 可解释性   | 説明可能性（セツメイ・カノウセイ）     | Explainability             | 让人**理解“为何是这个结果”**（局部/全局）。([AWS ドキュメント][2])             |
| 透明性    | 透明性（トウメイセイ）           | Transparency               | **用途/限制/数据/评测/风险**等信息对相关方可见（模型卡/服务卡）。([AWS ドキュメント][3]) |
| 人在回路   | 人間の関与（ヒューマン・イン・ザ・ループ） | Human-in-the-loop (HITL)   | 高风险/低置信时**触发人工复核**（A2I）。([AWS ドキュメント][4])              |
| 情境根拠检查 | 文脈根拠チェック（ブンミャク・コンキョ）  | Contextual grounding check | 用**参考源**校验回答，过滤幻觉（Guardrails）。([AWS ドキュメント][5])        |

---

## 2) 人本可解释的十条核心原则（定义 → 如何做 → AWS 落地 → 小例）

> 这些原则与 **Well-Architected｜Responsible AI** 维度（公平、公信、透明、监督）一致，可作为设计清单使用。([AWS ドキュメント][1])

### P1. 受众适配（Audience-appropriate）

* **定义**：不同角色要不同解释（终端用户/运营/审计/开发）。
* **如何做**：为每类受众定义**解释深度**与**术语层级**。
* **AWS**：**Model Cards** 写清“目标受众 & 不适用人群”；**Clarify** 输出给**专家**，**简化摘要**给**业务**。([AWS ドキュメント][3])
* **例**：信贷拒绝短信提供**要点理由**与**复议流程**；审计包包含 **SHAP 排名**与分组指标。

### P2. 行动导向（Actionable）

* **定义**：解释应指向**可改进/可申诉**的动作，而非仅展示权重。
* **如何做**：给出**可操作建议**或**复议路径**。
* **AWS**：在 **Model Card**“风险与缓解”中写**可行对策**；**A2I** 为申诉样本走人审。([AWS ドキュメント][3])

### P3. 忠实与简洁的平衡（Faithfulness vs. Simplicity）

* **定义**：解释既**尽量忠实**于模型机制，又**易懂**。
* **如何做**：**局部 SHAP** + **全局重要性**；可折叠 UI。
* **AWS**：**Clarify** 产出局部/全局归因；**Model Cards** 汇总要点图表。([AWS ドキュメント][2])

### P4. 证据可核查（Evidence-backed）

* **定义**：生成式回答**有出处**，可被验证。
* **如何做**：**RAG** 强制引用；无据则降级/拒答。
* **AWS**：**Bedrock Evaluations** 跑 **RAG“有据可依/检索正确性”**；**Guardrails 情境根拠检查**过滤无据陈述。([AWS ドキュメント][6])
* **例**：企业文档助手回答尾部列**来源片段**与**链接**。

### P5. 不确定性沟通（Uncertainty disclosure）

* **定义**：如答案不确定，需**如实提示**并建议后续步骤。
* **如何做**：显示“置信范围/低置信提醒”；触发人审。
* **AWS**：结合**评估分**与业务阈值，用 **A2I** 处理低置信样本。([AWS ドキュメント][4])

### P6. 公平与子群可见（Fairness & Subgroup visibility）

* **定义**：解释应反映**不同群体**的表现与影响。
* **如何做**：所有评测/上线监控**必须带分组指标**。
* **AWS**：**Clarify**（训练前/后偏差 + SHAP）、**Model Monitor Bias Drift** 线上守护。([AWS ドキュメント][2])

### P7. 隐私与最小暴露（Privacy-first）

* **定义**：解释/日志**不暴露 PII** 与敏感数据。
* **如何做**：使用脱敏与分级权限；解释中用**泛化术语**。
* **AWS**：**Guardrails PII** 过滤；遵循 **Responsible AI** 的数据保护实践。([AWS ドキュメント][7])

### P8. 可访问性与本地化（Accessibility & Locale）

* **定义**：解释对**多语言/读屏/色弱**用户友好。
* **如何做**：多语界面、可读格式、避免仅靠颜色。
* **AWS**：**Service Cards** 明确模型的语言适配；在 **Model Card** 标注目标语言/可访问性注意项。([Amazon Web Services, Inc.][8])

### P9. 人在回路与监督（Human oversight）

* **定义**：高风险任务**必须有人审**，并有**审计证据**。
* **如何做**：定义触发条件（领域/置信度/争议），保留人审记录。
* **AWS**：**Amazon A2I** 人审工作流 + **CloudWatch 事件**，可追溯。([AWS ドキュメント][4])

### P10. 生命周期透明（Lifecycle transparency）

* **定义**：对内外部**持续披露**“用途/限制/评测/更新”。
* **如何做**：创造与维护**模型卡**；引用**AI 服务卡**。
* **AWS**：**SageMaker Model Cards**（可导出 PDF，带审批） + **AWS AI Service Cards**（例如 Nova/Titan）。([AWS ドキュメント][3])

---

## 3) 从“原则”到“流程”：人本 XAI 5D 流程（Discover→Define→Design→Deliver→Operate）

| 阶段               | 目标        | 关键活动                                 | AWS 工具                                                         |
| ---------------- | --------- | ------------------------------------ | -------------------------------------------------------------- |
| **Discover（调研）** | 谁在用/谁受影响  | 用户画像、风险/法规识别                         | **Responsible AI 指南**（维度对照表）([AWS ドキュメント][1])                  |
| **Define（定义）**   | 解释需求与阈值   | 确定**受众、阈值、申诉路径**                     | 写入 **Model Card**“用途/阈值/不适用”([AWS ドキュメント][3])                  |
| **Design（设计）**   | 解释与 UI 模式 | SHAP 图、引用、低置信提示、A2I 触发               | **Clarify**、**Guardrails grounding**、**A2I** ([AWS ドキュメント][2]) |
| **Deliver（交付）**  | 验证与发布     | **Bedrock Evaluations**（真实性/鲁棒）、分组评测 | **Evaluations（RAG/鲁棒）** ([AWS ドキュメント][6])                      |
| **Operate（运行）**  | 监控与再评估    | **Bias Drift**、归因漂移、卡片版本化            | **Model Monitor**、**Model Cards 管理** ([AWS ドキュメント][9])         |

---

## 4) 可解释 UI「模式库」（拿来即用）

* **Why-chips（为何标签）**：展示 Top-k 影响因子（来自 SHAP）。([AWS ドキュメント][2])
* **引用脚注**：RAG 回答末尾列出处+片段 → **一键展开**。([AWS ドキュメント][9])
* **低置信横幅**：提示“此答案可能不完整”，并**提供人审按钮**（A2I）。([AWS ドキュメント][4])
* **策略拦截提示**：由 **Guardrails** 给出“被拦截原因”（如：PII/越权/无据）。([AWS ドキュメント][7])
* **模型/服务卡链接**：在“关于本模型”中链接 **Model Card/AI Service Card**。([AWS ドキュメント][3])

---

## 5) 评估指标（用数据验证“以人为中心”）

* **解释理解度**：任务型问卷/可用性测试完成率、误解率、**解释阅读时长**。
* **行动性**：被解释驱动的**纠错/复议成功率**、用户自助完成率。
* **真实性**：**有据可依分**、检索正确性、**幻觉率**（Guardrails & Evaluations）。([AWS ドキュメント][7])
* **公平性**：按语言/群体的**分组指标**差异（Clarify/Monitor）。([AWS ドキュメント][2])
* **透明覆盖**：**模型卡字段完备度**、更新频率、外部文档点击率（服务卡/使用须知）。([AWS ドキュメント][10])

---

## 6) 三个场景化例子（如何落地人本 XAI）

### A. 信贷审批解释（监管高）

* **做法**：

  1. Clarify 出 **局部 SHAP**（单笔）与**全局 SHAP**（总体）；
  2. 模型卡记录**拒贷阈值/限制**；
  3. 低置信触发 **A2I** 复核；
  4. 公示**服务卡/模型卡**链接与**复议渠道**。
* **收益**：满足“**可解释 + 可追责**”，通过合规检查。([AWS ドキュメント][2])

### B. 企业文档助手（真实性优先）

* **做法**：

  1. RAG + **Bedrock Evaluations**（Retrieve-and-Generate）；
  2. **Guardrails 情境根拠**启用；
  3. UI 显示**引用**与**低置信提示**，无据时建议“联系支持”。
* **收益**：**幻觉率显著下降**，用户更信任。([AWS ドキュメント][6])

### C. 公共部门咨询（多语与可访问性）

* **做法**：

  1. 明确目标语言与可访问性要求写入**模型卡**；
  2. 分组评测（语言/设备）；
  3. 服务说明引用**AI Service Cards** 的适用/限制。
* **收益**：透明合规，弱势群体体验提升。([AWS ドキュメント][3])

---

## 7) 设计与实施模板（可直接套改）

### 7.1 「解释策略」片段

```
受众：{终端用户 | 审计 | 运维}
最少解释：{Top-3 因子 | 出处列表 | 低置信提示}
行动路径：{复议入口 | 人审触发阈值 x% | 联系渠道}
隐私约束：{PII 脱敏方式 | 日志留存策略}
透明资产：{Model Card 链接 | Service Card 链接 | 版本/日期}
```

### 7.2 AWS「落地清单」

* Clarify 生成 **SHAP + 偏差**报告 → 嵌入 **Model Card**。([AWS ドキュメント][2])
* Bedrock **Evaluations**（RAG/鲁棒性）定基线；变更后复评。([AWS ドキュメント][6])
* **Guardrails** 开启 **有害/PII/注入 + 情境根拠检查**。([AWS ドキュメント][7])
* **A2I** 配置人审流（低置信/高风险触发）。([AWS ドキュメント][4])
* **Model Cards** 版本化/导出 PDF；**Service Cards** 链接入产品文档。([AWS ドキュメント][10])

---

## 8) 与考试紧密相关的要点（问→答）

* **问：什么是“以人为中心的可解释 AI”？**
  **答**：以**用户与受影响者**为起点，按**受众适配/行动导向/证据可核查/不确定性沟通/人在回路/生命周期透明**等原则，提供**可信、可用、可申诉**的解释；在 AWS 上用 **Clarify、Model/Service Cards、Guardrails、A2I、Evaluations** 落地。([AWS ドキュメント][1])
* **问：如何证明“解释做得对”？**
  **答**：用 **Evaluations** 与**分组指标**做前后对比；上线后以 **Bias Drift/归因漂移**监控；模型卡留痕可审计。([AWS ドキュメント][6])

---

## 参考（官方为主）

* **AWS Responsible AI（维度与工具总览）**：Guardrails／Bias & Explainability／Human-in-the-loop。([Amazon Web Services, Inc.][11])
* **Well-Architected｜Generative AI Lens｜Responsible AI 章节**。([AWS ドキュメント][1])
* **SageMaker Clarify（偏差与可解释性）**。([AWS ドキュメント][2])
* **SageMaker Model Cards（创建/管理/导出 PDF）**。([AWS ドキュメント][3])
* **Amazon A2I（人审工作流与触发）**。([AWS ドキュメント][4])
* **Bedrock Guardrails（情境根拠检查/有害/PII/注入）**。([AWS ドキュメント][7])
* **Bedrock Evaluations（模型与 RAG 评估指标）**。([AWS ドキュメント][6])
* **AWS AI Service Cards（Nova/Titan 等，透明度工件）**。([Amazon Web Services, Inc.][8])

---

### 一句话收尾

> **“解释不是图，而是体验。”** 把**受众、证据、行动、监督**织进产品：**Clarify** 给出因果线索，**RAG+Evaluations**保证有据，**Guardrails**兜住安全，**A2I**看护高风险，**Model/Service Cards**对内外透明——这就是在 AWS 上实现**人本可解释 AI**的工程方法。

[1]: https://docs.aws.amazon.com/wellarchitected/latest/generative-ai-lens/responsible-ai.html?utm_source=chatgpt.com "Responsible AI - Generative AI Lens"
[2]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-configure-processing-jobs.html?utm_source=chatgpt.com "Fairness, model explainability and bias detection with ..."
[3]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-cards.html?utm_source=chatgpt.com "Amazon SageMaker Model Cards"
[4]: https://docs.aws.amazon.com/sagemaker/latest/dg/a2i-use-augmented-ai-a2i-human-review-loops.html?utm_source=chatgpt.com "Using Amazon Augmented AI for Human Review"
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-contextual-grounding-check.html?utm_source=chatgpt.com "Use contextual grounding check to filter hallucinations in ..."
[6]: https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation.html?utm_source=chatgpt.com "Evaluate the performance of Amazon Bedrock resources"
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[8]: https://aws.amazon.com/about-aws/whats-new/2024/12/aws-ai-service-cards-advance-responsible-generative-ai/?utm_source=chatgpt.com "Announcing new AWS AI Service Cards to advance ..."
[9]: https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation-kb.html?utm_source=chatgpt.com "Evaluate the performance of RAG sources using ..."
[10]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-cards-manage.html?utm_source=chatgpt.com "Model cards actions - Amazon SageMaker AI"
[11]: https://aws.amazon.com/ai/responsible-ai/?utm_source=chatgpt.com "Building AI Responsibly"
