# AIF-C01｜4.1.7 **检测与监控偏差・信赖性・真实性的工具（标签质量・人类审计・子群分析・Clarify・Model Monitor・A2I 等）**

**バイアス・信頼性・真実性を検出およびモニタリングするためのツール（ラベル品質の分析・人間による監査・サブグループ分析・Amazon SageMaker Clarify・SageMaker Model Monitor・Amazon A2I など）を説明する**

*中文为主；三语术语：中・日〔カタカナ〕・英；结合 AWS 官方文档与产品能力。*

---

## 0) 本题要点（你需要会什么）

* 能把**偏差（バイアス／Bias）**、**信赖性（信憑性／トラストワージネス／Trustworthiness）**、**真实性（真実性／シンジツセイ／Veracity/Truthfulness）**分解成**可量化的检查与监控**。
* 熟练说明并操作下列工具/方法：
  **标签质量分析**（ラベル品質／*Label quality*：Ground Truth/GT Plus）、**人类审计**（人間監査／*Human review*：A2I）、**子群分析**（サブグループ分析／*Subgroup analysis*：Clarify）、**线上监控**（Model Monitor + Bias Drift）、**事实性/有据评估**（Bedrock Evaluations + Guardrails 的**情境根拠チェック**）。([AWS ドキュメント][1])

---

## 1) 术语对照（中・日〔读法〕・英）

| 中文   | 日文（读法）         | English               | 一句话释义                |
| ---- | -------------- | --------------------- | -------------------- |
| 偏差   | バイアス（バイアス）     | Bias                  | 针对某群体/属性的**系统性误差**   |
| 信赖性  | 信憑性（シンピョウセイ）   | Trustworthiness       | **可解释/可追溯/一致**，输出可信  |
| 真实性  | 真実性（シンジツセイ）    | Veracity/Truthfulness | **与事实/证据一致**，少“幻觉”   |
| 子群分析 | サブグループ分析       | Subgroup analysis     | 按语言/性别/地区等对**指标分组**  |
| 人类审计 | 人間監査（ニンゲン・カンサ） | Human review/audit    | **A2I** 对高风险样本进行人工复核 |

---

## 2) 工具全景图（工具 → 侧重 → 典型输出）

| 工具/方法                                | 侧重维度        | 关键能力                                                             | 典型输出/证据                                                     |
| ------------------------------------ | ----------- | ---------------------------------------------------------------- | ----------------------------------------------------------- |
| **SageMaker Ground Truth / GT Plus** | 标签质量（ラベル品質） | 标注工作流、**验证/调整（verify & adjust）**、置信度与复核 UI                       | 输出清单（manifest）、**置信度**、复核记录；GT Plus 审核 UI。([AWS ドキュメント][2]) |
| **Amazon A2I（Augmented AI）**         | 人类审计（HITL）  | 自定义**Human review loop**、阈值触发、与多服务集成                             | 审核工单、评审结果、可审计日志。([AWS ドキュメント][3])                           |
| **SageMaker Clarify**                | 偏差・可解释      | **训练前/后偏差**、SHAP 可解释；工作台可视化                                      | **Bias/Explainability 报告**（群体指标、特征贡献）。([AWS ドキュメント][4])     |
| **SageMaker Model Monitor**          | 线上监控        | **Bias drift 基线 + 调度**，CloudWatch 告警，特征归因漂移                      | 偏差漂移报告、违规文件（violations）。([AWS ドキュメント][5])                   |
| **Bedrock Evaluations**              | 真实性/稳健      | **RAG 评估**（Retrieve-only / Retrieve-and-Generate）、**鲁棒性/正确性**等指标 | 评估报告：**检索正确性/有据可依/鲁棒性**；作业与指标页面。([AWS ドキュメント][6])           |
| **Bedrock Guardrails**               | 事实性与安全      | \*\*情境根拠检查（Contextual grounding check）\*\*过滤幻觉；PII/有害性/提示攻击      | 每次调用的**拦截/通过**结果与策略命中详情。([AWS ドキュメント][7])                   |

---

## 3) 标签质量分析（Ground Truth / Ground Truth Plus）

### 3.1 为什么标签质量会影响偏差/真实性

* 错误或不一致的标签会**诱发系统性偏差**（如对少数语言标错），并拉低“真值基线”，进一步影响 Clarify/评估的判断。

### 3.2 你可以做什么（操作到证据）

1. **验证/调整（Verify & Adjust）**：对既有标注开二次任务进行**核验与修正**；API 要求另起 `LabelAttributeName` 标识**复核标签**。([AWS ドキュメント][2])
2. **置信度与抽检**：输出清单包含**置信度分**；按置信度/类目抽检，发现系统性问题。([AWS ドキュメント][8])
3. **GT Plus 审核 UI**：托管团队+可视化审核，**逐条回看并反馈**，直到标签符合“地面真值”。([Amazon Web Services, Inc.][9])
4. **自动化与流式**：\*\*自动化标注（Active learning）\*\*降低成本；**流式标注**支持连续数据接入，便于滚动迭代。([AWS ドキュメント][10])

> 实战例：客服意图分类“发票/投诉/退货”严重失衡 → 用 GT/GT Plus 为**长尾类**补样并复核，生成**高置信度**标签后再训练，偏差显著下降。

---

## 4) 人类审计（A2I）把“灰区”控住

### 4.1 什么时候触发人审

* **低置信度**、**高风险领域**（医疗/法律/金融）、**无出处或与证据冲突**、**系统性偏差可疑**时。

### 4.2 如何落地

* 使用 **A2I** 创建 **Human review loop**：定义**触发条件/表单/工人组**；内置与 **Rekognition** 等用例的工作流，也可自定义文本/对话审核。([AWS ドキュメント][3])
* **API & 审计**：A2I 提供**管理/监控人审循环**的 API，产出可追溯记录。([AWS ドキュメント][11])

> 实战例：RAG 回答“自信但没引用” → 触发 A2I 工单让法务审核，审核结论回写数据仓，供后续评估与微调。

---

## 5) 子群分析与偏差检测（Clarify）

### 5.1 训练前/后 + 解释

* **Pre-training**：检查数据层面的**群体分布偏差**；
* **Post-training**：检查**预测偏差/接受率差异**；
* **Explainability**：**SHAP** 看模型是否**过度依赖敏感属性**。([AWS ドキュメント][4])

### 5.2 如何运行

* 通过 **Clarify Processing Job** 与**分析配置（Analysis config）**定义**facet（敏感属性）/label**与度量集合；Studio 可视化 **Bias/Explainability 报告**。([AWS ドキュメント][12])

> 实战例：同一模型对「ja」用户 F1 明显低于「en」→ Clarify 报告显示**语言=ja**样本稀少且“渠道”特征 SHAP 值异常高 → 回到 **标签质量 + 数据再平衡**处理。

---

## 6) 上线后的连续监控（Model Monitor + Bias Drift）

* **流程**：“**先基线（Baseline）→ 再调度（Schedule）**”，定期导出偏差报告，**超阈值**通过 **CloudWatch** 告警。([AWS ドキュメント][5])
* **偏差漂移**：Clarify 与 Model Monitor 集成，支持**Bias Drift** 报告与**违规文件 schema**（含 facet、阈值等）。([AWS ドキュメント][13])
* **归因漂移**：监控**特征归因（SHAP）漂移**，识别“模型关注点”是否偏离。([AWS ドキュメント][14])
* **模型/数据质量**：Model Monitor 也覆盖常规**数据/模型质量**漂移与告警。([AWS ドキュメント][15])

> 实战例：数月后「年轻群体」通过率持续下降 → **Bias Drift** 报警 → 启动再训练与阈值校准流程。

---

## 7) 真实性/可信度评估（Bedrock Evaluations + Guardrails）

### 7.1 RAG/生成的“有据可依”

* **Bedrock Evaluations**：对 **RAG** 提供 **Retrieve-only** 与 **Retrieve-and-Generate** 两类评估，内置**检索正确性/有据可依**等指标；也可自定义指标与评审模型。([AWS ドキュメント][6])
* **总体评估**：除 RAG 外，还可评**语义鲁棒性/正确性**等，形成**自动+人评**的报告。([AWS ドキュメント][16])

### 7.2 阻断“幻觉”与不当内容

* **Guardrails 情境根拠チェック（Contextual grounding check）**：给定**参考来源**（如 RAG 命中文本/对话历史），Guardrails 对输出做**事实一致性检查**并**过滤幻觉**；同时可启用**有害性/PII/提示攻击**策略。([AWS ドキュメント][7])

> 实战例：总结会议纪要前，先把检索到的纪要片段作为“参考”，启用 Guardrails **contextual grounding**；Bedrock Evaluations 以“有据可依”指标对多个提示模板 A/B 比较，选更可靠者上线。([AWS ドキュメント][7])

---

## 8) “端到端”操作蓝图（可直接照抄）

1. **标签质量 →** Ground Truth/GT Plus 建**验证/调整**任务，查看**置信度**与审核 UI 反馈；把“低置信/争议”样本加入**人审白名单**。([AWS ドキュメント][2])
2. **分组评估 →** Clarify Processing Job（定义 facet/label）输出**Bias/Explainability** 报告；针对问题群体补样/再标。([AWS ドキュメント][12])
3. **事实性评估 →** Bedrock Evaluations（RAG/鲁棒性）+ 必要的人评；固化“阈值线”。([AWS ドキュメント][6])
4. **上线监控 →** Model Monitor 建 **Bias Drift 基线+调度**；偏差/归因漂移→ CloudWatch 告警。([AWS ドキュメント][5])
5. **运行保护 →** Guardrails 开启**情境根拠检查 + 有害/PII/注入**；对**低置信/冲突**样本触发 **A2I**。([AWS ドキュメント][7])

---

## 9) 迷你案例三连发（考试常见题型）

* **Q1｜不同语言群体指标差异大**（子群不公）
  **A**：Clarify 子群分析 → 找出“语言=ja”样本稀缺与模型依赖“渠道”过度（SHAP）→ 用 GT Plus 补标&复核 → 复跑 Clarify，差异收敛；上线用 **Bias Drift** 盯群体指标。([AWS ドキュメント][17])

* **Q2｜RAG 回答偶发“自信但没依据”**（真実性不足）
  **A**：Bedrock Evaluations（Retrieve-and-Generate）评“有据/正确”→ 调 chunk/Top-k/重排；Guardrails 启用**情境根拠检查**过滤幻觉；必要时 A2I 人审。([AWS ドキュメント][18])

* **Q3｜上线数月后某群体满意度下滑**（漂移）
  **A**：Model Monitor **Bias Drift** 报警→ 检查数据分布/阈值→ 触发再训练；结合**特征归因漂移**判断模型关注点是否跑偏。([AWS ドキュメント][5])

---

## 10) 口袋清单（秒选记忆）

* **标签质量先行**：GT/GT Plus **验证/调整** + **置信度抽检**，为偏差/真实性评估打好“地面真值”。([AWS ドキュメント][2])
* **偏差看 Clarify**：训练前/后 + SHAP；**上线看 Model Monitor 的 Bias Drift**（**先基线，后调度**）。([AWS ドキュメント][4])
* **真实性看 Bedrock Evaluations + Guardrails**：RAG “有据可依”与**情境根拠检查**联用，压制“幻觉”。([AWS ドキュメント][19])
* **高风险交给人审**：A2I **Human review loop** 是最后兜底与合规证据。([AWS ドキュメント][3])

---

## 参考（AWS 官方优先）

* **SageMaker Clarify**：偏差/解释、处理作业、分析配置与结果。([AWS ドキュメント][4])
* **Model Monitor**：模型与数据质量监控、**Bias Drift**、基线/调度/违规文件、特征归因漂移。([AWS ドキュメント][15])
* **Amazon A2I**：人审工作流、API、与服务集成示例（内容审核）。([AWS ドキュメント][3])
* **Ground Truth/GT Plus**：标注、**验证/调整**、置信度、审核 UI、自动化与流式。([AWS ドキュメント][2])
* **Bedrock Evaluations**：RAG 评估类型与指标、总体评估；创建评估作业。([AWS ドキュメント][6])
* **Bedrock Guardrails**：**情境根拠检查**过滤幻觉（contextual grounding check）。([AWS ドキュメント][7])
* **RAG 指南（PG）**：用 KBs 让回答“有据”，减少幻觉。([AWS ドキュメント][20])

---

### 一句话总结

> **先把“真值”打牢（GT/GT Plus），再用 Clarify 找“偏差”，用 Model Monitor 守“漂移”，用 Evaluations + Guardrails 证“有据”，高风险交给 A2I。** 这套闭环让“偏差・信赖性・真实性”变成**可检出、可监控、可审计**的工程能力，在 AWS 上可一步步落地。

[1]: https://docs.aws.amazon.com/sagemaker/latest/dg/gtp.html?utm_source=chatgpt.com "Use Amazon SageMaker Ground Truth Plus to Label Data"
[2]: https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateLabelingJob.html?utm_source=chatgpt.com "CreateLabelingJob - Amazon SageMaker - AWS Documentation"
[3]: https://docs.aws.amazon.com/sagemaker/latest/dg/a2i-use-augmented-ai-a2i-human-review-loops.html?utm_source=chatgpt.com "Using Amazon Augmented AI for Human Review"
[4]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-explainability.html?utm_source=chatgpt.com "Evaluate, explain, and detect bias in models"
[5]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-model-monitor-bias-drift-baseline.html?utm_source=chatgpt.com "Create a Bias Drift Baseline - Amazon SageMaker AI"
[6]: https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation-kb.html?utm_source=chatgpt.com "Evaluate the performance of RAG sources using ..."
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-contextual-grounding-check.html?utm_source=chatgpt.com "Use contextual grounding check to filter hallucinations in ..."
[8]: https://docs.aws.amazon.com/sagemaker/latest/dg/sms-data-output.html?utm_source=chatgpt.com "Labeling job output data - Amazon SageMaker AI"
[9]: https://aws.amazon.com/blogs/machine-learning/inspect-your-data-labels-with-a-visual-no-code-tool-to-create-high-quality-training-datasets-with-amazon-sagemaker-ground-truth-plus/?utm_source=chatgpt.com "Inspect your data labels with a visual, no code tool to ..."
[10]: https://docs.aws.amazon.com/sagemaker/latest/dg/sms-automated-labeling.html?utm_source=chatgpt.com "Automate data labeling - Amazon SageMaker AI"
[11]: https://docs.aws.amazon.com/augmented-ai/2019-11-07/APIReference/Welcome.html?utm_source=chatgpt.com "Welcome - Amazon Augmented AI"
[12]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-processing-job-run.html?utm_source=chatgpt.com "Run SageMaker Clarify Processing Jobs for Bias Analysis ..."
[13]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-model-monitor-bias-drift.html?utm_source=chatgpt.com "Bias drift for models in production - Amazon SageMaker AI"
[14]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-model-monitor-feature-attribution-drift.html?utm_source=chatgpt.com "Feature attribution drift for models in production"
[15]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-monitor.html?utm_source=chatgpt.com "Data and model quality monitoring with ..."
[16]: https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation.html?utm_source=chatgpt.com "Evaluate the performance of Amazon Bedrock resources"
[17]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-processing-job-analysis-results.html?utm_source=chatgpt.com "Analysis Results - Amazon SageMaker AI"
[18]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-evaluation-create-randg.html?utm_source=chatgpt.com "Creating a retrieve-and-generate RAG evaluation job"
[19]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-evaluation-metrics.html?utm_source=chatgpt.com "Use metrics to understand RAG system performance"
[20]: https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-serverless/grounding-and-rag.html?utm_source=chatgpt.com "Grounding and Retrieval Augmented Generation"
