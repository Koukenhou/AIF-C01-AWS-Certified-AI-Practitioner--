# AIF-C01｜4.2.3 **识别模型“安全性×透明性”的取舍（解释可能性×性能的度量）**

**モデルの安全性と透明性の間のトレードオフを特定する（解釈可能性とパフォーマンスを測定する など）**

*中文为主；术语三语：中・日〔カタカナ读法〕・英；结合 AWS 官方文档与原生工具。*

---

## 0) 本题你要做到什么

* 能**定义并识别**三组常见取舍：

  1. **解释可能性（説明可能性｜セツメイ・カノウセイ｜Explainability）⇆ 性能（パフォーマンス｜Performance）**
  2. **安全性（安全性｜アンゼンセイ｜Safety/Robustness）⇆ 有用性/自由度（ユースフルネス｜Usefulness）**
  3. **透明性（透明性｜トウメイセイ｜Transparency）⇆ 迭代速度/成本（スピード/コスト）**
* 会用 **SageMaker Clarify**、**Model Cards**、**AI Service Cards**、**Bedrock Guardrails**、**Bedrock Evaluations（含 RAG 指标）**、**Model Monitor** 量化与管理这些取舍。([AWS ドキュメント][1])

---

## 1) 三语术语速查

| 中文      | 日文（读法）                 | English           | 你要记住的点                                                      |
| ------- | ---------------------- | ----------------- | ----------------------------------------------------------- |
| 解释可能性   | 説明可能性（セツメイ・カノウセイ）      | Explainability    | 单样本/全局**为什么**给出此结果（如 SHAP）([AWS ドキュメント][1])                 |
| 透明性     | 透明性（トウメイセイ）            | Transparency      | **用途/限制/数据/评测/风险**是否可见（Model/Service Card）([AWS ドキュメント][2]) |
| 安全性/坚牢性 | 安全性・堅牢性（アンゼンセイ・ケンロウセイ） | Safety/Robustness | 有害/PII/注入/幻觉的**预防与检测**（Guardrails）([AWS ドキュメント][3])         |
| 性能      | パフォーマンス                | Performance       | 质量（准确/有据）× 延迟 × 成本（评测/Evals）([AWS ドキュメント][4])               |

---

## 2) 典型取舍图谱（概念→AWS工具→可测指标）

### 2.1 解释可能性 ⇆ 性能

* **现象**：可解释模型（线性/树/规则）更易说明，但在多模态/复杂语义任务上**上限可能较低**；大型 FM **性能强**但需**事后解释**。
* **AWS 工具/证据**：

  * **Clarify（クラリファイ）**：SHAP 全局/局部解释 + 训练前/后偏差分析。([AWS ドキュメント][1])
  * **Model Cards（モデルカード）**：把解释、数据、评测与风险归档为**透明证据**。([AWS ドキュメント][2])
  * **AI Service Cards（サービスカード）**：托管模型用途/限制/设计选择公开，便于评估“透明成本”。([Amazon Web Services, Inc.][5])
* **可测**：

  * *解释性*：Top-k 特征的**SHAP 贡献度集中度/稳定性**（跨批次）、解释覆盖率；
  * *性能*：任务指标（准确率/F1/EM）、生成式的**有据可依/检索正确性**（RAG Eval）。([AWS ドキュメント][4])

### 2.2 安全性 ⇆ 有用性/自由度

* **现象**：更严格的**内容过滤/PII/注入防护**与**事实性校验**可降低风险，但也可能**增加拒答率/延迟**、降低**召回**。
* **AWS 工具/证据**：

  * **Bedrock Guardrails（ガードレール）**：内容过滤、PII、**Prompt-attack**、**情境根拠チェック（Contextual grounding check）** 与 **Automated Reasoning checks**。([AWS ドキュメント][3])
  * **Bedrock Evaluations**：有害性（toxicity）、鲁棒性、RAG 指标对比不同策略/阈值。([AWS ドキュメント][4])
* **可测**：

  * 拒答率、被拦截率、幻觉率（grounding/faithfulness）、有害性命中率、**端到端延迟**。([AWS ドキュメント][4])

### 2.3 透明性 ⇆ 迭代速度/成本

* **现象**：**写卡/填数/分组评估/审计**增加流程成本，但换来**可审计与合规**。
* **AWS 工具/证据**：**Model Cards**（版本化/导出 PDF）、**AI Service Cards**、**Well-Architected GenAI Lens 的 Responsible AI 章节**。([AWS ドキュメント][6])
* **可测**：

  * 透明工件**覆盖率**（卡片字段完整度）、更新频率、**审计通过率**与**变更周期**。

---

## 3) 度量方法（Metrics & Protocols）

### 3.1 “解释可能性”的度量（局部+全局）

* **局部解释稳定性**：同一样本微扰（小改字/顺序）后，**Top-k SHAP 特征集合的一致率**。
* **全局特征归因**：主特征的**SHAP 贡献累计占比**、跨时间窗漂移（搭配 **Model Monitor** 观察“特征归因漂移”）。([AWS ドキュメント][1])
* **文档透明度**：**Model Card** 是否包含**数据来源/分组指标/解释图/风险**等关键段。([AWS ドキュメント][2])

### 3.2 “安全性”的度量

* **有害性/政策违规率**：Guardrails 的分类命中；**拦截原因分布**（主题、PII、注入）。([AWS ドキュメント][3])
* **事实一致性**：RAG 的 **Retrieve-and-Generate** 评估中的**有据可依/检索正确性**，以及 **Contextual grounding check** 的通过率/拒答率。([AWS ドキュメント][7])

### 3.3 “性能/成本/延迟”的度量

* **任务质量**：准确/EM/F1/ROUGE（摘要）或业务 KPI（转化、一次解决率）
* **SLA**：端到端延迟、P95/P99
* **成本**：每调用成本、每正确答案成本（Cost-per-Good-Answer）

> 建议在 **Bedrock Evaluations** 中建立**同一数据集**下的 A/B 基线，改变**Guardrails 策略/提示风格/候选模型**后重测，直观看到取舍曲线。([AWS ドキュメント][4])

---

## 4) 三个高频“取舍”场景（含操作步骤）

### 4.1 信贷审批（解释优先）

**目标**：合规 & 可解释 & 群体公平

1. 选型：优先**可解释模型**（树/线性）；
2. **Clarify**：训练前/后偏差 + **SHAP**；在 **Model Card** 固化阈值与风险；([AWS ドキュメント][1])
3. 生成式组件（理由说明/通知文案）→ 查 **AI Service Cards**，启用 **Guardrails**（防有害/PII）。([Amazon Web Services, Inc.][5])
   **取舍**：较小的模型+强解释可能略降准确，但**审计通过率↑、法律风险↓**。

### 4.2 企业文档问答（真实性与安全优先）

**目标**：**有据可依**、少幻觉、合规

1. RAG：**KB 策展**；**Evaluations** 评“检索正确性/有据可依”；([AWS ドキュメント][4])
2. **Guardrails**：开 **Contextual grounding check** + **PII/注入**；([AWS ドキュメント][8])
3. 观测：记录**拒答率、幻觉率、延迟**，对比不同阈值/重排策略。
   **取舍**：**拦截更严格 ⇒ 拒答率↑/延迟↑**，但**幻觉率↓/法律风险↓**。

### 4.3 营销生成（性能与自由度优先）

**目标**：产能与多样性

1. 选更强 FM；用 **AI Service Cards** 明确限制；([Amazon Web Services, Inc.][5])
2. **Guardrails**：仅保留品牌黑名单/基本有害过滤；
3. **Evaluations**：以“文本质量/多样性”对比温度/解码参数；
   **取舍**：自由度↑带来创意，但需\*\*更强人审（A2I）\*\*抽检以控风险。

---

## 5) 工具到行动（把取舍工程化）

| 目标         | AWS 工具                        | 最小动作（落地即用）                                                                               |
| ---------- | ----------------------------- | ---------------------------------------------------------------------------------------- |
| 提升解释与透明    | **Clarify** + **Model Cards** | 跑 **SHAP + 偏差**并嵌入 **Model Card**；卡片含“用途/数据/阈值/风险/再评估周期”。([AWS ドキュメント][1])               |
| 控制有害/注入/幻觉 | **Guardrails**                | 启用**内容过滤/PII/Prompt-attack**与**情境根拠チェック**；比对策略强弱的 A/B。([AWS ドキュメント][3])                  |
| 量化真实性与质量   | **Bedrock Evaluations**       | 对 **Retrieve-only / Retrieve-and-Generate** 与“正确性/鲁棒性/有害性”出报告；选最优模板/模型。([AWS ドキュメント][4]) |
| 运行期守护      | **Model Monitor**             | 建**基线 + 调度**，看**偏差漂移/特征归因漂移**，阈值告警。([AWS ドキュメント][9])                                     |
| 对外透明       | **AI Service Cards**          | 在产品文档中引用所用模型的**用途/限制/设计选择**要点。([Amazon Web Services, Inc.][5])                           |

---

## 6) 决策矩阵（考试可速背）

| 优先级 | 若你最看重…   | 应优先选择                                              | 接受的代价       | 关键证据                                                             |
| --- | -------- | -------------------------------------------------- | ----------- | ---------------------------------------------------------------- |
| ①   | 合规/可解释   | 可解释模型 + 强 **Clarify** + 完整 **Model Card**          | 可能牺牲少量精度/泛化 | SHAP & 分组指标、卡片 PDF、审批记录 ([AWS ドキュメント][1])                        |
| ②   | 真实性/安全   | RAG + 严格 **Guardrails** + **Contextual grounding** | 拒答率/延迟上升    | RAG Eval 报告、Guardrails 命中日志 ([AWS ドキュメント][8])                    |
| ③   | 创意/速度/成本 | 大模型 + 宽松策略 + 人审抽检                                  | 幻觉/有害风险更高   | 文案 A/B、A2I 审核记录、Service Card 限制 ([Amazon Web Services, Inc.][5]) |

---

## 7) 实操清单（最小可行流程）

1. **设目标**：写清本用例优先级（解释/安全/性能）。
2. **建基线**：用 **Evaluations** 固化同一数据集的质量/有害/延迟基线。([AWS ドキュメント][4])
3. **启防护**：用 **Guardrails** 逐级收紧（记录拒答率/幻觉率变化曲线）。([AWS ドキュメント][3])
4. **做解释**：用 **Clarify** 产出 SHAP 与偏差，写进 **Model Card**。([AWS ドキュメント][1])
5. **上线监控**：**Model Monitor** 盯偏差/归因漂移；阈值触发再训或策略回调。([AWS ドキュメント][9])
6. **对外透明**：在帮助中心/隐私页引用**AI Service Cards**要点与限制。([Amazon Web Services, Inc.][5])

---

## 8) 误区与纠偏

* **只看总体指标，不看分组** → 忽视群体伤害（用 Clarify 子群分析）。([AWS ドキュメント][1])
* **把 Guardrails 全关**以追求“更有用” → 有害/注入/PII 风险陡增（逐级调参而非关闭）。([AWS ドキュメント][3])
* **没有证据链** → 审计与复用困难（Model Cards/Service Cards 要齐全）。([AWS ドキュメント][2])

---

## 参考（官方优先）

* **SageMaker Clarify**：偏差与解释、处理作业与可视化。([AWS ドキュメント][1])
* **SageMaker Model Cards**：概念、管理与 API。([AWS ドキュメント][2])
* **AWS AI Service Cards**（Nova/Titan 等）：透明度工件汇总页与上新公告。([Amazon Web Services, Inc.][5])
* **Bedrock Guardrails**：内容/PII/提示攻击、**情境根拠チェック**与**Automated Reasoning checks**。([AWS ドキュメント][3])
* **Bedrock Evaluations（RAG）**：两类作业与指标、控制台报告。([AWS ドキュメント][4])
* **Well-Architected GenAI Lens｜Responsible AI**：治理维度与设计权衡。([AWS ドキュメント][10])
* **Bedrock 数据保护**：账户/最小权限/传输与静态加密等边界。([AWS ドキュメント][11])

---

### 一句话总结

> **取舍不可避免，但可被量化与管理**：用 **Clarify+Model Cards/Service Cards**做“解释与透明”，用 **Guardrails+Evaluations**做“安全与真实性”的曲线调参，用 **Model Monitor**守住“线上漂移”。把曲线画清楚，你就能在**解释/安全/性能**之间做出可审计的工程选择。

[1]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-explainability.html?utm_source=chatgpt.com "Evaluate, explain, and detect bias in models"
[2]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-cards.html?utm_source=chatgpt.com "Amazon SageMaker Model Cards"
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-evaluation-metrics.html?utm_source=chatgpt.com "Use metrics to understand RAG system performance"
[5]: https://aws.amazon.com/ai/responsible-ai/resources/?utm_source=chatgpt.com "Responsible AI Tools and Resources"
[6]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-cards-manage.html?utm_source=chatgpt.com "Model cards actions - Amazon SageMaker AI"
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-eval-llm-results.html?utm_source=chatgpt.com "Review metrics for RAG evaluations that use LLMs (console)"
[8]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-contextual-grounding-check.html?utm_source=chatgpt.com "Use contextual grounding check to filter hallucinations in ..."
[9]: https://docs.aws.amazon.com/wellarchitected/latest/generative-ai-lens/generative-ai-lens.html?utm_source=chatgpt.com "AWS Well-Architected Framework - Generative AI Lens"
[10]: https://docs.aws.amazon.com/wellarchitected/latest/generative-ai-lens/responsible-ai.html?utm_source=chatgpt.com "Responsible AI - Generative AI Lens"
[11]: https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html?utm_source=chatgpt.com "Data protection - Amazon Bedrock"
