# AIF-C01｜2.3.1 生成 AI アプリ開発のための AWS サービスと機能

*AIF-C01 Domain 2, Task 2.3.1 — Identify AWS services & features to build generative-AI apps*

> 目标：看题能**准确点名**哪些 AWS 服务适合做生成式 AI 应用，并说清它们**能做什么、怎么用、适用场景与取舍**。
> 术语均含 **中文 → 日文（片假名读法）→ English**，讲解以中文为主，并对齐官方文档。

---

## 1) Amazon Bedrock（贝德罗克）

* **日**：アマゾン・ベッドロック【べっどろっく】
* **EN**：Amazon Bedrock
* **一句话**：**托管的基础模型（FM）统一入口 + 安全治理 + RAG + Agent + 评估**，用 **一个 API** 调不同厂商/亚马逊的模型。([Amazon Web Services, Inc.][1])

### 能力（官方要点）

* **统一 API 访问多家 FM**（AI21/Anthropic/Cohere/Mistral/Meta、Amazon Titan/Nova 等）。([Amazon Web Services, Inc.][1])
* **Playgrounds**：控制台里无代码试 Prompt、模型与推理参数；等价于调用 `InvokeModel/Converse` 等 API。([AWS ドキュメント][2])
* **Knowledge Bases（RAG 托管）**：把你私有文档做 **分块→嵌入→向量检索**，查询时自动拼接最相关内容。([AWS ドキュメント][3], [Amazon Web Services, Inc.][4])
* **Agents**：可配置的 **工具/函数调用** 智能体，能用 API 执行动作并与 KB 协同。([AWS ドキュメント][5])
* **Guardrails**：跨模型复用的**安全/合规护栏**（话题限制、PII、毒性/攻击检测等）；可单独用 `ApplyGuardrail`。([AWS ドキュメント][6])
* **Evaluations**：自动/对比评估模型与 KB/RAG 的**正确性/鲁棒性**等指标；支持“LLM as a judge”。([AWS ドキュメント][7])

### 何时选 Bedrock

* 想**最快上线**生成式应用，又要**企业级治理**（VPC/IAM/CloudTrail/KMS）。
* 需要 **RAG、Agent、Guardrails、评估** 这些**一站式**组件。

### 上手路径（官方）

1. **Console** 打开 **Playground** 试模型与提示；再迁移到 SDK。([AWS ドキュメント][8])
2. 需要企业知识 → **Knowledge Bases**；要“能办事” → 配 **Agents**；全程套 **Guardrails**。([AWS ドキュメント][3])

---

## 2) Amazon Bedrock Playground（交互试验台）

* **日**：アマゾン・ベッドロック・プレイグラウンド
* **EN**：Amazon Bedrock Playground
* **作用**：**可视化调 Prompt/参数/模型** 的实验环境；跑一次 Prompt ≈ 一次 API 调用。([AWS ドキュメント][2])
* **适用**：产品/算法/业务**共同迭代**提示词与参数；保存范例再落地到代码或 IaC。
* 也可在 **SageMaker（Unified Studio）** 内使用 Bedrock playgrounds。([AWS ドキュメント][9])

---

## 3) Amazon SageMaker JumpStart（模型与方案“加速器”）

* **日**：アマゾン・セージメーカー・ジャンプスタート
* **EN**：Amazon SageMaker JumpStart
* **一句话**：SageMaker 里的 **模型/解决方案集市**——含**公开与专有**基础模型、示例笔记本与模板，便于**定制训练与托管**。([AWS ドキュメント][10])
* **典型用法**：在 **SageMaker** 里选模型 → 一键部署/微调 → 接入训练/特征/监控全家桶。([Amazon Web Services, Inc.][11], [AWS ドキュメント][12])
* **何时选**：你需要**更可控的训练/托管**（自带算法/容器/自定义代码），或想把开源/第三方 FM **纳入自家 MLOps**。

---

## 4) PartyRock（零/低代码原型）

* **日**：パーティーロック
* **EN**：PartyRock
* **作用**：**零/低代码**拼装生成式小应用；可改模型、提示与高级参数，快速分享 Demo。([partyrock.aws][13])
* **何时用**：团队内**头脑风暴/PoC**、教学演示、业务快速试想法。

---

## 5) Amazon Q（企业/开发者 AI 助手）

* **日**：アマゾン・キュー
* **EN**：Amazon Q

### 5.1 Q Business（企业知识助手）

* **日**：Q ビジネス
* **EN**：Amazon Q Business
* **作用**：把企业数据源（文档/工单/Slack/SharePoint 等）接入，做**检索问答/洞察/轻量自动化**；可 **插件** 联通常见 SaaS。([AWS ドキュメント][14], [Amazon Web Services, Inc.][15])
* **适用**：**内部问答、知识助理、面向员工的 Copilot**。

### 5.2 Q Developer（开发者助理）

* **日**：Q デベロッパー
* **EN**：Amazon Q Developer
* **作用**：IDE 插件/控制台里进行**代码理解、生成、重构、排错、AWS 架构与最佳实践答疑**。([Amazon Web Services, Inc.][16], [AWS ドキュメント][17])
* **适用**：**研发提效、运维/基础设施脚本、单元测试辅助**。

---

## 6) 服务能力对照速查（考试高频）

| 需求                | 首选服务（中/日/英）                                     | 核心功能                                                    | 官方要点                                                |
| ----------------- | ----------------------------------------------- | ------------------------------------------------------- | --------------------------------------------------- |
| 统一调用多家 FM、企业级治理   | **Bedrock** / ベッドロック / Amazon Bedrock           | 统一 API、RAG（KB）、Agents、Guardrails、Evaluations、Playground | ([Amazon Web Services, Inc.][1], [AWS ドキュメント][3])   |
| 无代码试 Prompt/参数    | **Playground** / プレイグラウンド / Bedrock Playground  | 交互试验，等价 API 调用                                          | ([AWS ドキュメント][2])                                   |
| 零/低代码做 Demo       | **PartyRock** / パーティーロック / PartyRock            | 拼装 AI 小应用，改模型与提示                                        | ([partyrock.aws][13])                               |
| 模型/方案模板 + 定制训练/托管 | **JumpStart** / ジャンプスタート / SageMaker JumpStart  | 公有+专有 FM、示例笔记本、快速部署与微调                                  | ([AWS ドキュメント][10], [Amazon Web Services, Inc.][11]) |
| 企业内问答/工作流         | **Q Business** / Q ビジネス / Amazon Q Business     | 连接企业数据源/插件，面向员工的 Copilot                                | ([AWS ドキュメント][14])                                  |
| 开发者编码助理           | **Q Developer** / Q デベロッパー / Amazon Q Developer | 代码生成/修复/最佳实践/架构答疑                                       | ([Amazon Web Services, Inc.][16], [AWS ドキュメント][17]) |

---

## 7) 选择题“秒选”原则（看到关键字就定位）

* **“多家模型 / 安全护栏 / RAG / Agent / 评估”** → **Bedrock**（KB/Agents/Guardrails/Evals/Playground）。([AWS ドキュメント][3])
* **“无代码试提示”** → **Bedrock Playground**。([AWS ドキュメント][2])
* **“零代码快速 Demo/教学”** → **PartyRock**。([partyrock.aws][13])
* **“要把模型纳入自家训练/托管/MLOps”** → **SageMaker JumpStart** 起步。([AWS ドキュメント][10])
* **“内部员工检索问答/洞察/自动化”** → **Amazon Q Business**。([AWS ドキュメント][14])
* **“开发提效/IDE 插件/代码生成”** → **Amazon Q Developer**。([Amazon Web Services, Inc.][16])

---

## 8) 实战组合参考

* **企业文档问答（RAG）**：Bedrock **Knowledge Bases** + **Guardrails** + **Evaluations**（对比 KB 检索质量）→ 前端 or Lex。([AWS ドキュメント][3])
* **能办事的客服 Agent**：Bedrock **Agents**（函数调用/业务 API） + KB + Guardrails。([AWS ドキュメント][5])
* **工程化落地**：需要重训练/托管 → 用 **JumpStart + SageMaker**；需要企业入口 → **Q Business** 嵌入 Slack/Teams/Office。([Amazon Web Services, Inc.][11], [AWS ドキュメント][18])

---

## 9) 术语小词典（中・日・英）

* 基础模型：**基盤モデル【きばんモデル】** / Foundation Model (FM)
* 知识库（RAG）：**ナレッジベース** / Knowledge Bases
* 代理/智能体：**エージェント** / Agents
* 安全护栏：**ガードレール** / Guardrails
* 评估：**エバリュエーション** / Evaluations
* 交互试验台：**プレイグラウンド** / Playground
* 零代码：**ノーコード** / No-code
* 模型加速器：**ジャンプスタート** / JumpStart

---

### 一句话总括

> **做生成式 AI 应用的“五件套”**：**Bedrock（模型/治理）＋ Playground（试验）＋ KB/Agents（知识/动作）＋ Guardrails（安全）＋ Evaluations（质量）**；
> 需要更深的训练/托管选 **JumpStart/SageMaker**，要企业场景现成入口用 **Amazon Q（Business/Developer）**。

[1]: https://aws.amazon.com/documentation-overview/bedrock/?utm_source=chatgpt.com "Amazon Bedrock Documentation - AWS"
[2]: https://docs.aws.amazon.com/bedrock/latest/userguide/playgrounds.html?utm_source=chatgpt.com "Generate responses in the console using playgrounds"
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html?utm_source=chatgpt.com "Retrieve data and generate AI responses with ..."
[4]: https://aws.amazon.com/bedrock/knowledge-bases/?utm_source=chatgpt.com "Amazon Bedrock Knowledge Bases"
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html?utm_source=chatgpt.com "Automate tasks in your application using AI agents"
[6]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation.html?utm_source=chatgpt.com "Evaluate the performance of Amazon Bedrock resources"
[8]: https://docs.aws.amazon.com/bedrock/latest/userguide/getting-started-console.html?utm_source=chatgpt.com "Getting started in the Amazon Bedrock console"
[9]: https://docs.aws.amazon.com/sagemaker-unified-studio/latest/userguide/bedrock-playgrounds.html?utm_source=chatgpt.com "Experiment with the Amazon Bedrock playgrounds"
[10]: https://docs.aws.amazon.com/sagemaker/latest/dg/jumpstart-foundation-models.html?utm_source=chatgpt.com "Amazon SageMaker JumpStart Foundation Models"
[11]: https://aws.amazon.com/sagemaker/ai/jumpstart/getting-started/?utm_source=chatgpt.com "Getting Started with Amazon SageMaker JumpStart"
[12]: https://docs.aws.amazon.com/sagemaker/latest/dg/studio-jumpstart.html?utm_source=chatgpt.com "SageMaker JumpStart pretrained models"
[13]: https://partyrock.aws/guide/building?utm_source=chatgpt.com "PartyRock | Building an app"
[14]: https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/what-is.html?utm_source=chatgpt.com "Amazon Q Business"
[15]: https://aws.amazon.com/q/business/?utm_source=chatgpt.com "Amazon Q Business - AI Assistant for Enterprise"
[16]: https://aws.amazon.com/q/developer/?utm_source=chatgpt.com "Amazon Q Developer - Generative AI"
[17]: https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/what-is.html?utm_source=chatgpt.com "Amazon Q Developer"
[18]: https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/how-it-works.html?utm_source=chatgpt.com "How Amazon Q Business works"
