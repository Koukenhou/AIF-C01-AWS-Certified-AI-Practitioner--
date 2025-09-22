# AIF-C01｜4.1.3 **理解在模型选择时的责任实践（环境考量・可持续性・道德主体性）**

**モデルを選択するうえでの責任ある慣行（環境への配慮・持続可能性・道徳的主体性など）を理解する**

*（中文为主；术语三语：中・日〔カタカナ读法〕・英；结合 AWS 官方文档与产品能力）*

---

## 0) 本题你要做到什么

* 把“**选模型**”从只看**准确率/成本**，升级为三维：**环境（Environmental）**、**可持续（Sustainability）**、**道德主体性（Ethical/Moral agency）**。
* 能把这些原则映射到 **AWS Well-Architected｜Sustainability Pillar** 的设计实践，以及 **Bedrock / SageMaker / A2I** 等工具上，形成**可执行清单**。

---

## 1) 关键术语（中・日〔读法〕・英）

| 概念    | 日文（读法）                      | English                               | 重点理解                                                             |
| ----- | --------------------------- | ------------------------------------- | ---------------------------------------------------------------- |
| 环境考量  | **環境への配慮（カンキョウ・ヘノ・ハイリョ）**   | Environmental considerations          | 关注**能耗/碳足迹**，选择更清洁的区域与更高效的技术。                                    |
| 可持续性  | **持続可能性（ジゾク・カノウセイ）**        | Sustainability                        | 以**更少资源**达成目标：对齐需求、选合适规模模型、优先托管与复用。([AWS ドキュメント][1])             |
| 道德主体性 | **道徳的主体性（ドウトクテキ・シュタイセイ）**   | Moral agency / Ethical responsibility | 明确**责任/治理/透明**与**人类监督**，防止滥用与误用。([Amazon Web Services, Inc.][2]) |
| 共享责任  | **共有責任モデル（キョウユウ・セキニン・モデル）** | Shared responsibility model           | **AWS 负责“云的可持续”**，**客户负责“云中可持续”**（优化工作负载）。                       |
| 碳核算范围 | **スコープ1/2/3**               | GHG Protocol scope 1/2/3              | 了解云工作负载排放多属**Scope 3**，需持续度量与减排。                                 |

---

## 2) 为什么这些在 AWS 里“必须做”

* **Well-Architected｜Sustainability Pillar** 把“**减少资源使用**与**提升效率**”设为明确设计原则，并给出了从**对齐需求（Align to demand）**、**软件/架构效率**、**数据生命周期**到**硬件与托管服务选择**的一整套最佳实践。([AWS ドキュメント][1])
* **可持续是共同责任**：AWS 优化“云的可持续”（高效基础设施、可再生能源、水资源治理），**你**优化“云中工作负载”（模型/数据/推理的能效）。
* **可再生能源与区域选择**：AWS 公布了可再生能源进展与可持续承诺（净零/水正等）；你可在**区域**与**服务选择**上做更清洁的决策。([Amazon Web Services, Inc.][3])
* **可量化**：用 \*\*Customer Carbon Footprint Tool（CCFT）\*\*查看账户层碳排估算并做月度对比。([AWS ドキュメント][4])

---

## 3) 选模型的“三维责任实践”与落地动作

### 3.1 环境への配慮（Environmental）

**要义**：在满足需求的前提下，**最少能耗/最低碳**地实现目标。

**实践清单（与 AWS 对应）**

1. **选低碳/高能效的区域与服务**：托管服务能把硬件高利用率带来的能效优势“分摊”到所有租户（**Use managed services**）。([AWS ドキュメント][1])
2. **“最小可用模型”原则**：能用**中小尺寸 FM + RAG/微调**就不要盲目追“超大模型”；避免重复大规模训练，把能耗交给 Bedrock 托管推理。([AWS ドキュメント][5])
3. **度量而非猜测**：用 **CCFT** 跟踪区域/服务带来的排放差异，辅以成本与延迟面板一起看。([AWS ドキュメント][4])
4. **数据也有碳**：按 **SUS 数据管理**最佳实践做**分级存储/生命周期**与**删除不再需要的数据**。([AWS ドキュメント][1])

**小例子**：FAQ 文档问答

* 不新训 70B 模型 → 采用 **RAG（Bedrock Knowledge Bases）+ 中等模型**；在较清洁电网区域部署。**结果**：同等准确度下**代币与能耗大幅下降**。([AWS ドキュメント][6])

---

### 3.2 持続可能性（Sustainability）

**要义**：**长期可维持**的性能/成本/合规/治理，避免“短期暴冲、长期难养”。

**实践清单（与 AWS 对应）**

1. **对齐需求（Align to demand）与按需伸缩**：把峰谷差交给**托管与无服务器**，减少空耗；把**可持续**当作\*\*非功能性需求（NFR）\*\*纳入目标。([AWS ドキュメント][1])
2. **选“足够好”的基础模型**：先用评估（**Bedrock Evaluations**）找到**更小但满足质量**的模型，再上线。([Amazon Web Services, Inc.][2])
3. **优先“托管复用”**：`SUS05-BP03 Use managed services` 能把硬件利用率、维护与能效优化交给 AWS。([AWS ドキュメント][1])
4. **数据生命周期与存储优化**：遵循 `SUS04` 类最佳实践（分类→冷热分级→过保留期即清理）。([AWS ドキュメント][1])
5. **持续监测与改进**：以 **Model Monitor** 建立**基线+调度**，发现漂移/质量下降及时回训；碳/成本/延迟做统一看板。([Amazon Web Services, Inc.][3])

**小例子**：营销生成文案

* 起步用小模型 + 轻量**指示微调**，通过 Evaluations 验证质量；当流量增长再升级规格或配**预置吞吐**。→ **成本/能耗与质量动态平衡**。([Amazon Web Services, Inc.][2])

---

### 3.3 道徳的主体性（Moral agency / Ethical responsibility）

**要义**：**谁负责**模型的**安全/公平/透明**与**人类监督**？

**实践清单（与 AWS 对应）**

1. **安全/稳健**：启用 **Bedrock Guardrails**（内容过滤、提示攻击检测、PII 遮罩）；必要时把 Guardrails 绑定到 Agent 流程。([AWS ドキュメント][7])
2. **公平/偏差**：用 **SageMaker Clarify** 做**前/后训练偏差检测**与**解释（SHAP）**；上线后由 **Model Monitor** 监控 **Bias Drift**。([AWS ドキュメント][8])
3. **人类监督（HITL）**：对**高风险或不确定**输出建立 **Amazon A2I** 审查回路；形成审计证据与改进数据。([AWS ドキュメント][9])
4. **治理与透明**：参考 **Responsible AI** 页面与 **ISO/IEC 42001**，将风险管理与威胁建模纳入全生命周期。([Amazon Web Services, Inc.][2])

**小例子**：法律问答

* Guardrails 拦截越狱与违法请求 → RAG 强制**附带出处** → 无出处或置信度低触发 **A2I** → Clarify 解释模型选择 → 全链路可审计、可追责。([AWS ドキュメント][5])

---

## 4) “选模型”的责任式决策树（可照抄到考试笔记）

1. **能否仅靠 RAG？**

   * 是 → **KB/RAG**；无需改参数，**最快/最低碳**。([AWS ドキュメント][6])
   * 否 → 进入 2。
2. **是否需要稳定结构化/风格一致？**

   * 是 → 选**小/中模型 + 指示微调**；评估合格再说放大。
3. **是否需要行业语体/知识储备但无标注？**

   * 是 → **CPT（持续事前训练）**；先评估**最小可用规模**。
4. **对比候选模型**：

   * 用 **Bedrock Evaluations** 对**准确/鲁棒/有害性**和**RAG 指标**打分；选择**达到业务阈值的最小模型**。([Amazon Web Services, Inc.][2])
5. **上线守护**：Guardrails、Clarify、Model Monitor、A2I 组合。([AWS ドキュメント][7])
6. **持续优化**：把**碳/成本/延迟/质量**与业务 KPI 同屏监控（含 **CCFT**）。([AWS ドキュメント][4])

---

## 5) 三个高频场景的“责任选择”示范

### A) 客服自助问答（多语言人群，日/中/英）

* **选择**：中等模型 + **KB/RAG**（覆盖多语文档），少量微调固化格式。
* **责任动作**：Guardrails（PII/有害内容/注入）、Clarify（语言群体分组评估）、A2I 抽检边缘案例。
* **环境/可持续**：托管推理+按需伸缩，定期用 CCFT 看碳。([AWS ドキュメント][6])

### B) 金融审批（高风险+解释要求）

* **选择**：较小可解释模型 + 结构化特征模型（需要**可解释/公平性**）。
* **责任动作**：Clarify（偏差/解释）、Model Monitor（Bias Drift）、A2I（人工复核低置信度），Guardrails 防越权回答。([AWS ドキュメント][8])
* **环境/可持续**：使用托管数据库/消息队列与批处理缩短峰值持续时间。([AWS ドキュメント][1])

### C) 生成式营销文案（海量调用）

* **选择**：以 **Evaluations** 先选**质量达标的小模型**；必要时**预置吞吐**保障延迟。
* **责任动作**：Guardrails（品牌/合规黑名单）、人评抽样（A2I）控制语气风险。
* **环境/可持续**：减少无效重试，复用模板与缓存。([Amazon Web Services, Inc.][2])

---

## 6) “一页速记”——责任模型选择对照

| 维度            | 问自己                             | 典型 AWS 做法                                                                         |
| ------------- | ------------------------------- | --------------------------------------------------------------------------------- |
| 环境（エンビロンメント）  | 这活能否**用 RAG**而不是再训？能否用**托管服务**？ | KB/RAG；**Use managed services**；选更清洁区域；CCFT 度量。([AWS ドキュメント][1])                  |
| 可持续（サステナビリティ） | 最小可用模型？可伸缩？数据是否分级与过期清理？         | Evaluations 选最小可用；SUS 数据/架构最佳实践；Model Monitor 持续。([AWS ドキュメント][1])                |
| 道德主体性（エシカル）   | 谁负责安全/公平/监督/透明？                 | Guardrails（安全/注入/PII）、Clarify（偏差/解释）、A2I（HITL）、ISO/IEC 42001 治理。([AWS ドキュメント][7]) |

---

## 7) 参考（官方优先）

* **Well-Architected｜Sustainability Pillar**：共享责任、Scope 1/2/3、SUS 最佳实践、用托管服务。
* **Sustainability at AWS / 可再生能源承诺与目标**。([Amazon Web Services, Inc.][3])
* **Customer Carbon Footprint Tool（度量碳）**。([AWS ドキュメント][4])
* **Responsible AI（维度、能力、AI Service Cards、ISO/IEC 42001 认证）**。([Amazon Web Services, Inc.][2])
* **Bedrock Guardrails（安全/注入/PII）**。([AWS ドキュメント][7])
* **SageMaker Clarify（偏差/解释）、Model Monitor（监控漂移）**。([AWS ドキュメント][8])
* **Amazon A2I（人类复核）**。([AWS ドキュメント][9])
* **RAG/Knowledge Bases（以检索代替再训）**。([AWS ドキュメント][6])

---

### 一句话收尾

> **责任导向的模型选择 = 最小可用模型 + 托管复用 + 可度量的减碳 + 可治理的安全/公平/监督。**
> 在 AWS 上，结合 **Well-Architected｜SUS** 原则（度量→优化→复用）与 **Guardrails / Clarify / Model Monitor / A2I / CCFT / Evaluations**，你可以把“环境・可持续・道德主体性”变成**可执行的选型准则与运行证据**。([AWS ドキュメント][1])

需要的话，我可以把上面的**责任式选型清单**做成「**检查表（Checklists）** + **权衡矩阵**」版，方便你项目立项/答辩时直接引用。

[1]: https://docs.aws.amazon.com/pdfs/wellarchitected/latest/sustainability-pillar/wellarchitected-sustainability-pillar.pdf "Sustainability Pillar - AWS Well-Architected Framework"
[2]: https://aws.amazon.com/ai/responsible-ai/ "Responsible AI – Building AI Responsibly – AWS"
[3]: https://aws.amazon.com/sustainability/?utm_source=chatgpt.com "Sustainable Cloud Computing | Amazon Web Services"
[4]: https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/ccft-overview.html?utm_source=chatgpt.com "Understanding the Customer Carbon Footprint Tool (CCFT)"
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html?utm_source=chatgpt.com "Retrieve data and generate AI responses with ..."
[6]: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-it-works.html?utm_source=chatgpt.com "How Amazon Bedrock knowledge bases work"
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[8]: https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-configure-processing-jobs.html?utm_source=chatgpt.com "Fairness, model explainability and bias detection with ..."
[9]: https://docs.aws.amazon.com/sagemaker/latest/dg/a2i-use-augmented-ai-a2i-human-review-loops.html?utm_source=chatgpt.com "Using Amazon Augmented AI for Human Review"
