# AIF-C01｜5.2.4 **ガバナンスプロトコルに従うためのプロセス**

*AIF-C01 Domain 5, Task 5.2.4 — Process to follow governance protocols (policies, review cycles/strategies, Generative-AI Security Scoping Matrix, governance frameworks, transparency standards, team training)*

> 目标：给出一套**可执行**、**可审计**、**可复制**的生成式/传统 AI 治理流程。重点结合 AWS 官方框架（**Well-Architected Generative AI Lens**, **Generative-AI Security Scoping Matrix**, **Responsible AI 指南**）、工具（**Bedrock Guardrails**, **SageMaker Model Cards & Model Registry**, **Audit Manager/Artifact**）与做法。([AWS ドキュメント][1])

---

## 1) 首先对齐“治理框架”与组织政策

**日**：ガバナンスフレームワーク【がばなんす】｜**EN**：Governance Framework

* **采用 AWS Well-Architected Generative AI Lens**：在原有五大支柱基础上扩展生成式 AI 特有考量（数据、模型、代理、RAG、Guardrails、监控等），作为**系统评审模板**与落地清单。建议把 WAFR（Well-Architected Review）正式纳入你的**项目门禁与复盘机制**。([AWS ドキュメント][1])
* **采用 Responsible AI 维度**（公平、可解释、隐私与安全、安全性、可控性、真实性/鲁棒性），将其转化为**可度量控制项**（如偏差评估、隐私评估、Guardrails 通过率、不良输出率）。([AWS ドキュメント][2])
* **与 ISO/IEC 42001（AI 管理体系）对齐**：把上述流程编制成组织级 **AIMS**（AI Management System）制度与角色分工；AWS 已通过 42001 认证，可作为供应商尽调证据与映射参考。([Amazon Web Services, Inc.][3])

---

## 2) 用“生成式 AI セキュリティ・スコーピング・マトリクス”界定范围

**日**：生成AIセキュリティ・スコーピング・マトリクス【まとりくす】｜**EN**：Generative-AI Security Scoping Matrix

* **是什么**：AWS 安全团队提出的**分层/分象限**思维模型，用于按**工作负载类型与风险**选择控制优先级（数据/模型/应用/代理/RAG/工具调用等不同面）。([Amazon Web Services, Inc.][4])
* **怎么用**：

  1. 标注你的用例（仅文本对话、带工具调用、接入企业数据、能下发动作等）；
  2. 逐格勾选**必须控制**（身份最小化、私网访问、Guardrails、PII 清理、引用/事实核验、模型选择与推理参数边界等）；
  3. 形成\*\*项目级“安全/合规范围说明”\*\*附在设计评审包。
* **官方入口**（图解/演示版本）：AWS 安全博客与专页。([Amazon Web Services, Inc.][4])

---

## 3) 政策（Policy）—从书面原则到“策略即代码”

**日**：ポリシー【ぽりしー】｜**EN**：Policies

* **组织政策**：把 Responsible AI 维度写成条款（例如“不得输出医疗建议”“必须附引用”）。([AWS ドキュメント][2])
* **技术政策落地**：

  * **Bedrock Guardrails** 作为**政策执行层**：内容过滤、提示攻击检测、PII 保护、拒答主题、上下文扎实性检查；支持跨模型/跨平台统一施加，并可用 **IAM 策略强制**。([AWS ドキュメント][5])
  * **SageMaker Model Registry + Unified Model Cards**：对每个模型**记录用途/限制/数据来源/指标/风险**，治理审批后才能推生产；统一治理工作流与共享。([Amazon Web Services, Inc.][6])
  * **审计与证据**：**Audit Manager** 选用框架（如 SOC2/ISO27001）自动采证；**Artifact** 拉取 AWS 的 ISO/SOC 证明以支撑“云之下的合规”。([AWS ドキュメント][7])

---

## 4) 评审周期（レビューサイクル）与策略（レビュー戦略）

**日**：レビューサイクル／レビュー戦略｜**EN**：Review Cycle / Strategy

* **阶段评审**：

  1. **设计评审**（立项/重大变更）→ Scoping Matrix 定级 + GenAI Lens WAFR；
  2. **上线前评审**→ Guardrails 覆盖率、Model Card 完整度、数据/日志/加密/私网检查；
  3. **运行评审**（月/季）→ 质量/偏差/幻觉/越权统计，触发再训练或规则更新。([AWS ドキュメント][1])
* **策略要点**：

  * **“事前门禁 + 事中监控 + 事后复盘”** 三段闭环；
  * 以 **GenAI Lens** 的问题列**检查表**化；重大变更与事故后做 **Post-mortem** 并更新政策与 Guardrails。([AWS ドキュメント][1])

---

## 5) 透明性基准（Transparency Standards）

**日**：透明性基準【とうめいせい きじゅん】｜**EN**：Transparency Standards

* **AWS AI Service Cards**（服务级透明）：官方提供\*\*“用途、限制、设计选择、优化建议”**的透明文档，便于理解托管 AI 服务的适用边界。你可把链接并入你的**服务目录**与**风险提示\*\*。([Amazon Web Services, Inc.][8])
* **SageMaker Model Cards**（模型级透明）：把**你自家模型**的用途/限制/数据出典/评估写清楚，导出 PDF 归档，配合审批与发布闸门。([Amazon Web Services, Inc.][6])
* **Bedrock 的数据/隐私承诺**：官方“**不使用你的数据训练模型**”，并提供端到端加密与审计集成；适合写入**对外告知**与 **DPIA** 里。([Amazon Web Services, Inc.][9])

---

## 6) 团队培训要件（チームトレーニング要件）

**日**：チームトレーニング要件【ようけん】｜**EN**：Team Training Requirements

* **治理意识**：Responsible AI 六维（公平/可解释/隐私与安全/安全性/可控性/真实性鲁棒）。([AWS ドキュメント][2])
* **工程手册**：

  * 如何配置与**验证 Guardrails**（阈值、拒答主题、PII 规则、评测集）。([Amazon Web Services, Inc.][10])
  * **GenAI Lens WAFR 流程**与常见反模式。([AWS ドキュメント][1])
  * **Model Cards/Registry** 的填写标准与发布流程。([Amazon Web Services, Inc.][6])
* **审计工具**：Audit Manager 证据查看与报告导出；Artifact 文档使用规范（下载、签收、共享范围）。([AWS ドキュメント][7])

---

## 7) 端到端“遵循治理”的**标准流程（SOP）**

> 适用于 Bedrock + RAG + SageMaker 的混合场景；可直接改写为你们组织的流程文档。

1. **立项**：用 **Scoping Matrix** 定级（是否接入企业数据、能否下发动作、是否代理/工具）。([Amazon Web Services, Inc.][4])
2. **架构设计**：按 **GenAI Lens** 出设计答辩材料（身份最小化、私网/区域、日志与观测、Guardrails、事实与引用、模型与参数）。([AWS ドキュメント][1])
3. **政策绑定**：

   * 建 **Guardrails**（输入/输出/多模态），并开启 **IAM Policy-based enforcement**；
   * 建 **Model Card** 并入 **Model Registry**，定义审批者与发布闸门。([Amazon Web Services, Inc.][11])
4. **透明与告知**：服务目录里链接**AI Service Cards**；对外披露数据与隐私承诺（引用 Bedrock 官方承诺）。([Amazon Web Services, Inc.][8])
5. **上线前评审**：跑 **WAFR（GenAI Lens）**；用 **Audit Manager** 生成“就绪报告”（加密/日志/IAM/私网等证据）。([AWS ドキュメント][1])
6. **运行与监控**：

   * 质量与偏差监测（Responsible-AI 指标）；
   * Guardrails 命中率与拒答率；
   * 模型输出可解释与引用点击率。([AWS ドキュメント][2])
7. **复盘与改进**：月度/季度 **评审**（更新 Guardrails、模型与参数、RAG 索引与过滤规则）；重大事件做 **Post-mortem** 并更新政策与 SOP。([AWS ドキュメント][1])

---

## 8) 实战示例（道路影像×生成式说明）

**目标**：输出“为什么该路段被评为 D 级？”的**有引用**答复。

* **范围界定**：RAG + 只读工具（查询 GIS/巡检库），**禁止执行变更动作**。→ Scoping Matrix 选“只读代理”。([Amazon Web Services, Inc.][4])
* **政策执行**：

  * **Guardrails** 禁止医疗/个隐、启用 PII 过滤与“危险操作”否定主题；
  * 回答必须附 **引用（citations）** 与“根据××标准第×条”用语模板。([AWS ドキュメント][5])
* **透明文档**：

  * **Model Card** 记录训练数据来源（Glue 表）、夜间/雨天限制、F1/Recall、再训练节奏；
  * 服务目录挂 **AI Service Cards** 链接与 Bedrock 数据承诺说明。([Amazon Web Services, Inc.][6])
* **评审与运行**：

  * 上线前过 **GenAI Lens WAFR**；
  * 运行监控“事实冲突率/拒答率/引用失效率”；季度复盘更新 Guardrails 与知识库。([AWS ドキュメント][1])

---

## 9) 术语小词典（中・日・英，带读法）

* 治理框架：**ガバナンスフレームワーク**／Governance framework（**GenAI Lens**, Responsible-AI 维度）。([AWS ドキュメント][1])
* 范围矩阵：**生成AIセキュリティ・スコーピング・マトリクス**／Generative-AI Security Scoping Matrix。([Amazon Web Services, Inc.][4])
* 护栏：**ガードレール**／Guardrails（Bedrock 统一安全控制，支持 **IAM 强制**）。([Amazon Web Services, Inc.][12])
* 模型卡/注册表：**モデルカード／モデルレジストリ**／Model Cards & Model Registry（治理审批与透明）。([Amazon Web Services, Inc.][6])
* 透明文档：**AI サービスカード**／AI Service Cards（服务级用途与限制说明）。([Amazon Web Services, Inc.][8])
* 评审：**ウェルアーキテクテッドレビュー（WAFR）**／Well-Architected Review。([AWS ドキュメント][1])
* 取证与审计：**オーディットマネージャー／アーティファクト**／Audit Manager & Artifact（证据自动化与平台证明）。([AWS ドキュメント][7])
* 管理体系：**ISO/IEC 42001**（AI 管理体系认证）。([Amazon Web Services, Inc.][3])

---

### ✅ 一句话总括

> **“定框架（GenAI Lens + Responsible-AI）→ 画范围（Scoping Matrix）→ 政策即代码（Guardrails/Model Cards）→ 周期评审（WAFR）→ 透明取证（Audit Manager/Artifact）→ 持续训练（团队与模型）”。** 这就是在 AWS 上遵循生成式 AI 治理协议的**可操作流程**。 ([AWS ドキュメント][1])

如果你愿意，我可以把上述 SOP 改写成\*\*“一页 A3 挂墙版流程图 + 勾选清单”\*\*，便于你的团队在每次评审时逐项核对。

[1]: https://docs.aws.amazon.com/wellarchitected/latest/generative-ai-lens/generative-ai-lens.html?utm_source=chatgpt.com "AWS Well-Architected Framework - Generative AI Lens"
[2]: https://docs.aws.amazon.com/wellarchitected/latest/generative-ai-lens/responsible-ai.html?utm_source=chatgpt.com "Responsible AI - Generative AI Lens - AWS Documentation"
[3]: https://aws.amazon.com/compliance/iso-42001-faqs/?utm_source=chatgpt.com "ISO 42001 Artificial Intelligence Management System"
[4]: https://aws.amazon.com/blogs/security/securing-generative-ai-an-introduction-to-the-generative-ai-security-scoping-matrix/?utm_source=chatgpt.com "An introduction to the Generative AI Security Scoping Matrix"
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock Guardrails"
[6]: https://aws.amazon.com/blogs/machine-learning/improve-governance-of-models-with-amazon-sagemaker-unified-model-cards-and-model-registry/?utm_source=chatgpt.com "Improve governance of models with Amazon SageMaker ..."
[7]: https://docs.aws.amazon.com/audit-manager/latest/userguide/framework-overviews.html?utm_source=chatgpt.com "Supported frameworks in AWS Audit Manager"
[8]: https://aws.amazon.com/blogs/machine-learning/introducing-aws-ai-service-cards-a-new-resource-to-enhance-transparency-and-advance-responsible-ai/?utm_source=chatgpt.com "Introducing AWS AI Service Cards: A new resource to enhance ..."
[9]: https://aws.amazon.com/bedrock/?utm_source=chatgpt.com "Amazon Bedrock - Generative AI"
[10]: https://aws.amazon.com/blogs/machine-learning/build-responsible-ai-applications-with-amazon-bedrock-guardrails/?utm_source=chatgpt.com "Build responsible AI applications with Amazon Bedrock Guardrails"
[11]: https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-guardrails-announces-iam-policy-based-enforcement-to-deliver-safe-ai-interactions/?utm_source=chatgpt.com "Amazon Bedrock Guardrails announces IAM Policy-based ..."
[12]: https://aws.amazon.com/bedrock/guardrails/?utm_source=chatgpt.com "Amazon Bedrock Guardrails"
