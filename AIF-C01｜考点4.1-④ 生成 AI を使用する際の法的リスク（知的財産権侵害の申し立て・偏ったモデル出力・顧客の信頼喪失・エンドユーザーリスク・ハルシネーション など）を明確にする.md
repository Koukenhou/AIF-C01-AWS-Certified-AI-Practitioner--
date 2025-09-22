# AIF-C01｜4.1.4 **明确生成式 AI 的法律风险（著作权/偏见/信任/终端用户风险/幻觉 等）**

**生成 AI を使用する際の法的リスク（知的財産権侵害の申し立て・偏ったモデル出力・顧客の信頼喪失・エンドユーザーリスク・ハルシネーション など）を明確にする**

*中文为主；术语三语：中・日〔カタカナ读法〕・英；结合 AWS 官方文档与产品能力。以下内容仅供合规/工程参考，**不构成法律意见**。*

---

## 0) 考点定位（你要会什么）

* 能**点名并准确定义**与生成式 AI 相关的**法律与合规风险**类目，并给出**AWS 官方能力**对应的**工程化缓解**。
* 清楚 **Amazon Bedrock** 的**数据保护边界**与\*\*（特定服务的）版权赔偿条款**适用范围与**除外项\*\*（考试易考“适用/不适用”）。([AWS ドキュメント][1])

---

## 1) 术语速查（中・日〔读法〕・英）

| 风险/概念      | 日文（读法）                            | English                | 一句话理解                    |
| ---------- | --------------------------------- | ---------------------- | ------------------------ |
| 知的财产权侵害的主张 | **知的財産権侵害の申し立て（チテキ・ザイサンケン・シンガイ）** | IP infringement claim  | 生成内容被主张**侵犯版权/专利/商标**等权利 |
| 偏见输出       | **偏ったモデル出力（カタヨッタ・モデル・シュツリョク）**    | Biased model output    | 对群体/属性**不公平**的输出         |
| 客户信任流失     | **顧客の信頼喪失（コキャク・ノ・シンライ・ソウシツ）**     | Loss of customer trust | 因安全/隐私/错误而失去用户信任         |
| 终端用户风险     | **エンドユーザーリスク**                    | End-user risk          | 误导/伤害终端用户（医疗、金融等高风险）     |
| 幻觉         | **ハルシネーション**                      | Hallucination          | 模型“**编造**”事实/出处/数据       |
| 责任 AI 政策   | **責任あるAIポリシー**                    | Responsible AI Policy  | 平台/客户的**负责任使用**准则与约束     |

---

## 2) 风险总览（定义 → 触发场景 → 后果）

### 2.1 知的财产权侵害的主张（IP infringement claim）

* **定义**：第三方主张你的**生成输出**侵犯了其版权/其他 IP 权利。
* **触发**：生成图/文与某作品**高度相似**；在**忽略安全指引**或**未启用过滤**情况下大规模商用。
* **后果**：法律诉讼、下架、赔偿、声誉损害。
* **AWS 关键点（背诵）**：

  * AWS 对\*\*“特定已普遍可用（GA）的 Amazon 生成式 AI 服务/模型”**提供**无上限的版权（IP）赔偿（indemnity）\*\*，条款见 **Service Terms §50.10** 与 **Bedrock FAQ**；例如 **Amazon Nova/Titan 等**属于“被赔偿的生成式 AI 服务”（Indemnified Generative AI Services）。([Amazon Web Services, Inc.][2])
  * **除外情形**（易考）：**未启用可用的过滤/工具**、**违反服务条款**、**你提供的输入本身侵权**、**你对模型做了导致侵权的微调/定制**等，则 **不在赔偿范围**。([Amazon Web Services, Inc.][2])

### 2.2 偏见输出（Biased outputs）

* **定义**：输出对特定群体**系统性不利**（录用率、拒绝率、语气歧视）。
* **后果**：涉嫌反歧视法规、监管处罚、品牌受损。
* **AWS 能力**：**SageMaker Clarify**（训练前/后偏差检测 + SHAP 解释）、**Model Monitor**（生产期 Bias Drift），并可在 **Bedrock Evaluations** 中把“公平性/群体差异”纳入评估。([Amazon Web Services, Inc.][3])

### 2.3 客户信任流失（Loss of trust）

* **定义**：用户感知系统**不安全、不尊重隐私/版权、经常出错**。
* **后果**：退订/投诉/监管关注。
* **AWS 能力**：

  * **Bedrock 数据保护**：**不**将你的输入/输出用于训练，**不与第三方共享**；支持**传输/静态加密**与**最小权限**。([AWS ドキュメント][1])
  * **Responsible AI Policy**：平台明确**负责任使用**要求（补充 AUP 与 Service Terms）。([Amazon Web Services, Inc.][3])
  * **调用审计**：**Model Invocation Logging** 输出输入/输出/元数据到 CloudWatch/S3，便于**取证**与**追责**。([AWS ドキュメント][4])

### 2.4 终端用户风险（End-user risk）

* **定义**：输出**影响用户安全/财务/健康**的场景（医疗、金融、法律），造成误导或损害。
* **AWS 能力**：

  * **A2I**（Human-in-the-loop）在人类复核机制下**兜底高风险**；
  * **威胁建模**：AWS 安全博客/大会课程提供**生成式 AI 威胁建模**方法，覆盖**注入/越权/凭据泄露**等威胁。([Amazon Web Services, Inc.][5])

### 2.5 幻觉（Hallucination）

* **定义**：模型生成**貌似真实**但**无依据/不正确**的信息。
* **后果**：合规与信誉风险（“编造出处/结论”）。
* **AWS 能力**：

  * **Guardrails + 真值检查**：官方“Responsible AI”页与新品能力指出**可显著拦截有害内容并过滤大量幻觉**（RAG/摘要场景）；**自动化推理检查（Automated Reasoning checks）**有助于提高**事实验证**与**最小化幻觉**。([Amazon Web Services, Inc.][6])
  * **RAG 评估**：Bedrock 提供 **RAG 的 Retrieve-only / Retrieve-and-Generate** 评估流程，以**检索正确性/有据可依**度量幻觉。([Amazon Web Services, Inc.][7])
  * **Bedrock Agents 介入**：官方博客演示**检测到幻觉→切换人工客服/人审**的工作流。([Amazon Web Services, Inc.][8])

---

## 3) 逐项“法律风险 × AWS 缓解”对照

| 风险          | 触发点（例）                 | 主要后果       | **AWS 缓解（工程化）**                                                                                                                                                                            |
| ----------- | ---------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **IP 侵权主张** | 生成图文与某作品高度相似；未启用过滤直接商用 | 诉讼/赔偿/下架   | ① 选用**受赔偿的 GA Amazon 模型/服务**（Nova/Titan…）；② **开启 Guardrails**（内容与敏感过滤）；③ **保留调用日志**（取证与举证）；④ 严格遵守 **Service Terms §50.10** 的**启用过滤/合规使用**等**前提**与**除外**条款。([Amazon Web Services, Inc.][2]) |
| **偏见/不公平**  | 招聘/信贷对某群体拒绝率更高         | 反歧视风险/品牌受损 | ① **Clarify** 训练前/后偏差检测+解释；② **Model Monitor** 建基线与 Bias Drift 告警；③ 在 **Evaluations** 设群体分桶的人评维度。([Amazon Web Services, Inc.][3])                                                          |
| **信任流失**    | “平台会拿我的数据训练？”“日志不可审计？” | 留存下降/投诉    | ① **Bedrock 不用你的数据训练**、不与第三方共享；② **启用 Invocation Logging**（CloudWatch/S3）；③ 公布 **负责任 AI 政策**与适用范围给终端用户。([AWS ドキュメント][1])                                                                   |
| **终端用户风险**  | 医疗/法律回答不确定却“拍板”        | 误导/人身财产损失  | ① **A2I** 人类复核高风险样本；② **Guardrails** 阻断危险指引；③ 依 **威胁建模指南**做越权/注入防护。([Amazon Web Services, Inc.][8])                                                                                        |
| **幻觉**      | 无出处/编造数据/“一本正经地错”      | 法务/信任风险    | ① **RAG 评估**“检索正确性/有据可依”；② **Guardrails** + **自动化推理检查**提高事实性；③ 幻觉检测到即**切人审/降级**（Agents）。([Amazon Web Services, Inc.][7])                                                                   |
| **隐私/PII**  | 输出中泄露手机号/邮箱/卡号         | 合规处罚/罚款    | ① **Guardrails PII 过滤**（检测/遮罩/阻断，支持自定义正则）；② **数据保护**与**最小权限**。([AWS ドキュメント][9])                                                                                                            |

---

## 4) “合规模板”——把风险写进你的产品设计（可直接套改）

### 4.1 安全与合规配置（Bedrock）

* **Guardrails**：开启**内容类别**与**Prompt Attack** 检测；配置 **PII** 的**遮罩/阻断**；在 **Agent**/应用入口**绑定同一 Guardrail**。([AWS ドキュメント][10])
* **调用日志**：在 **Settings → Model invocation logging** 启用到 **CloudWatch/S3**，并为 **Knowledge Bases** 开**摄取日志**（取证链）。([AWS ドキュメント][4])

### 4.2 法务条款与使用边界（与法务沟通要点）

* **服务条款 §50.10（赔偿）**：确认**所用模型/服务在清单内**，并满足**启用过滤/合规使用**等前提；对**自定义微调**可能触发**除外情形**的项目，需额外评估。([Amazon Web Services, Inc.][2])
* **Responsible AI Policy** 与 **AI Service Cards**：在用户文档中声明**适用场景/限制**与**负责任使用**要求。([Amazon Web Services, Inc.][3])

### 4.3 幻觉最小化与“可验证”

* **RAG 评估（Retrieve-only / Retrieve-and-Generate）**：上线前用 **Bedrock Evaluations** 评“**命中/有据可依**”；回答中**强制带出处**。([Amazon Web Services, Inc.][7])
* **自动化推理检查**：适配可用模型的**事实验证**能力，提高**正确性**并降低幻觉。([Amazon Web Services, Inc.][11])
* **人审触发**：无出处/置信度低 → 走 **A2I**。([Amazon Web Services, Inc.][8])

---

## 5) 三个高频情境题（读题→秒配策略）

1. **“生成图片被艺术家投诉侵权”**

   * 选用 **GA 的 Amazon Titan/Nova 图像模型**并满足 §50.10 前提（启用过滤/合法使用）→ **可享 IP 赔偿**；同时保留 **调用日志**与**提示/输出**证据。([AWS ドキュメント][12])

2. **“客服机器人在某语言群体上频繁失礼/出错”**

   * 定位为**偏见/包容性**问题：**Clarify** 跑分组偏差；**Model Monitor** 建 Bias Drift；在 **Evaluations** 增加该语种人评；必要时**补语料/CPT/微调**。([Amazon Web Services, Inc.][3])

3. **“RAG 回答自信但无出处”**

   * 用 **RAG 评估**核验命中；**Guardrails**+**自动化推理检查**限制无据陈述；触发 **A2I** 人审再答。([Amazon Web Services, Inc.][7])

---

## 6) “项目落地检查表”（法律风险最小化）

* **模型与条款**

  * ✅ 所选模型/服务**在 §50.10 赔偿清单**内（如 Nova/Titan GA 型号），并**启用官方过滤**；记录**版本/配置**。([Amazon Web Services, Inc.][2])
* **数据与隐私**

  * ✅ 明示 **Bedrock 不用于训练/不共享**；**Guardrails PII** 开启；**最小权限**与加密。([AWS ドキュメント][1])
* **评估与监控**

  * ✅ 上线前：**Evaluations**（鲁棒/有害/RAG 有据）；上线后：**Invocation Logging/KB Logs** + **Model Monitor**（Bias Drift）。([AWS ドキュメント][4])
* **人类监督与应急**

  * ✅ **A2I** 审查高风险；**SOP**：用户投诉/权利主张 → 回溯日志、冻结相关回答、切换安全模式（只返回出处/不推断）。([Amazon Web Services, Inc.][8])
* **治理与责任**

  * ✅ 采纳 **Responsible AI Policy** 与 **AI Service Cards** 信息到用户条款/帮助中心；完成**威胁建模**记录。([Amazon Web Services, Inc.][3])

---

## 7) 参考（官方优先）

* **Bedrock 数据保护**（不用于训练/不共享；加密/权限）。([AWS ドキュメント][1])
* **Guardrails 总览/PII/Prompt-attack**。([AWS ドキュメント][10])
* **Responsible AI（产品页/政策）**。([Amazon Web Services, Inc.][6])
* **IP 赔偿**（Bedrock FAQ、Service Terms §50.10、Nova/Titan Service Cards）。([Amazon Web Services, Inc.][13])
* **Invocation Logging/KB Logging**（CloudWatch/S3）。([AWS ドキュメント][4])
* **威胁建模**（安全博客/re\:Inforce 资料）。([Amazon Web Services, Inc.][5])
* **减少幻觉**（Responsible AI 页、Agents 博客、自动化推理检查、新功能）。([Amazon Web Services, Inc.][6])
* **RAG 可靠性评估**（KB/RAG 评估）。([Amazon Web Services, Inc.][7])

---

### 一句话总结

> **法律与合规风险 = “选对模型/条款” + “前置过滤/评估” + “运行期可审计/可兜底”。** 在 AWS 上，借助 **§50.10 赔偿条款适用的 GA 模型**、**Guardrails（安全/PII/注入）**、**Clarify/Model Monitor（偏差）**、**Evaluations（RAG/有害/鲁棒）**、**A2I（人类复核）** 与 **Invocation Logging（证据链）**，可以把 **IP 侵权、偏见、信任、终端用户风险、幻觉** 从**法律条款 + 工程控制**两侧同时收敛。([Amazon Web Services, Inc.][2])

如果你愿意，我可以把 **“风险→控制→AWS证据（日志/报告/条款）”** 做成一页 Checklist，方便你在项目立项/审查会直接打勾落档。

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html?utm_source=chatgpt.com "Data protection - Amazon Bedrock"
[2]: https://aws.amazon.com/service-terms/?utm_source=chatgpt.com "AWS Service Terms - Amazon.com"
[3]: https://aws.amazon.com/ai/responsible-ai/policy/?utm_source=chatgpt.com "AWS Responsible AI Policy"
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-invocation-logging.html?utm_source=chatgpt.com "Monitor model invocation using CloudWatch Logs and ..."
[5]: https://aws.amazon.com/blogs/security/threat-modeling-your-generative-ai-workload-to-evaluate-security-risk/?utm_source=chatgpt.com "Threat modeling your generative AI workload to evaluate ..."
[6]: https://aws.amazon.com/ai/responsible-ai/?utm_source=chatgpt.com "Building AI Responsibly"
[7]: https://aws.amazon.com/blogs/machine-learning/evaluate-the-reliability-of-retrieval-augmented-generation-applications-using-amazon-bedrock/?utm_source=chatgpt.com "Evaluate the reliability of Retrieval Augmented Generation ..."
[8]: https://aws.amazon.com/blogs/machine-learning/reducing-hallucinations-in-large-language-models-with-custom-intervention-using-amazon-bedrock-agents/?utm_source=chatgpt.com "Reducing hallucinations in large language models with ..."
[9]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-sensitive-filters.html?utm_source=chatgpt.com "Remove PII from conversations by using sensitive information ..."
[10]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[11]: https://aws.amazon.com/blogs/aws/minimize-ai-hallucinations-and-deliver-up-to-99-verification-accuracy-with-automated-reasoning-checks-now-available/?utm_source=chatgpt.com "Minimize AI hallucinations and deliver up to 99% ..."
[12]: https://docs.aws.amazon.com/ai/responsible-ai/titan-image-generator/overview.html?utm_source=chatgpt.com "Amazon Titan Image Generator - AWS AI Service Cards"
[13]: https://aws.amazon.com/bedrock/faqs/?utm_source=chatgpt.com "Amazon Bedrock FAQs - Generative AI - AWS"
