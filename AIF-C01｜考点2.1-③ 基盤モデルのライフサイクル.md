# AIF-C01｜2.1-③ 基盤モデルのライフサイクル（中・日・英对照，日语含读法）

> 目标：按 **Domain 2, Task 2.1** 的第三小点，系统说明 **基础模型（FM, Foundation Model）** 从“选择 → 定制 → 评估 → 部署 → 反馈改进”的全流程。
> 形式：每一阶段给出 **中文解释 → 日文（读法）→ English → 决策要点/常见坑 → AWS 上的典型做法**。

---

## 0) 总览｜What & Why

* **基盘模型**：**基盤モデル【きばんモデル】／ファウンデーションモデル**（Foundation Model）
* **定义**：在海量数据上**事前トレーニング【じぜん…】/Pre-training** 的通用模型，可直接使用或通过 **ファインチューニング/Fine-tuning** 等方式做定制。
* **生命周期口诀**：**“选数据→选模型→事前训练/利用→微调→评估→部署→反馈改进”**。
* **AWS 视角**：常以 **Amazon Bedrock** 为入口（选模型/推理/KB/RAG/Guardrails/Agents），也可在 **SageMaker** 做更底层的训练与托管。

---

## 1) データ選択【でーた せんたく】｜Data Selection

**做什么**：确定用于**预训练/继续预训练/微调**的数据域、类型与质量策略。
**决策要点**

* **覆盖面**（通用 vs 行业领域）：是否包含目标任务语言、专业术语、风格。
* **质量**：去重、脏词过滤、PII 脱敏、标注一致性。
* **合法性**：版权、合规（个人信息/商业秘密）。
  **常见坑**：只追“量”，忽视**分布匹配**与**数据治理**。
  **AWS 做法**：
* 存储与治理：**Amazon S3** + **Lake Formation/Glue**；
* 脱敏与合规：**Bedrock Guardrails（ガードレール）/PII 过滤**；
* 检索型场景：优先 **Knowledge Bases（KB）+ RAG**，减少对模型“硬记忆”的需求。

---

## 2) モデル選択【もでる せんたく】｜Model Selection

**做什么**：在可选 FM 中挑一个**性能/成本/延迟/上下文**合适的模型（文本/多模态/图像/语音）。
**决策要点**

* **能力**：推理/代码/长上下文/多语言/多模态；
* **SLA**：延迟、吞吐、可用区；
* **成本**：每 1K tokens 价格、上下文窗口影响；
* **治理**：合规、可用的 Guardrails。
  **常见坑**：只看榜单不看**场景与成本**；忽略**上下文窗口**。
  **AWS 做法**：在 **Bedrock** 里对比不同厂商/家族（文本、图像、多模态），用 **Playground/Evaluations** 快速试跑与对比。

---

## 3) 事前トレーニング【じぜん…】｜Pre-training

> 对考生：了解“什么是预训练”即可；AIF-C01 不要求你亲自训练大模型。

**做什么**：在大规模通用语料上学习语言/世界知识与推理能力。
**补充**：企业通常**不自行从零预训**，而是使用**现成 FM**，必要时做\*\*継続事前学習（Continued Pre-training）\*\*以适配特定语域。
**AWS 做法**：**选择现成 FM**；若需再学习特定语料，可在 **SageMaker** 做继续预训练，或用 Bedrock 支持的定制路径（视模型而定）。

---

## 4) ファインチューニング｜Fine-tuning（模型定制）

**做什么**：用小规模**领域数据**让 FM 适配企业任务/风格。
**常见方式**

* **SFT（监督微调）**：问→答/指令→回应成对样本；
* **LoRA/QLoRA**：参数高效微调（**省显存/成本**）；
* **Continued Pre-training**：继续在领域语料上训练；
* **指令微调/偏好对齐**：让输出更“听话”（如 RLAIF/DPO，考试只需认知名词）。
  **决策要点**：是否真的需要微调？**能用 RAG/提示工程解决的先不用微调**，可减成本与风险。
  **AWS 做法**：
* **Bedrock Model Customization**（不同模型提供 SFT/参数高效微调等能力）；
* **SageMaker JumpStart/Training** 做自定义训练；
* 数据放 **S3**，过程由 **CloudWatch** 监控。

---

## 5) 評価【ひょうか】｜Evaluation

**做什么**：用**自动指标 + 人类评审**检查模型质量与安全。
**自动指标（示例）**

* **任务向**：摘要（ROUGE）、翻译（BLEU/COMET）、RAG（Groundedness/检索召回）、代码（单测通过率）。
* **分类向**：Accuracy/Precision/Recall/F1、AUC。
  **人工评估**：事实性、相关性、风格、毒性/合规、可用性。
  **常见坑**：只看自动分数，不看**业务/安全**维度。
  **AWS 做法**：**Bedrock Evaluations**（基于基准集或自定义集评测）；离线脚本用 **SageMaker Processing** 跑指标。

---

## 6) デプロイ【でぷろい】｜Deployment

**做什么**：把定制或选定的 FM **上线**。
**形态选择**

* **Managed API（托管）**：直接用 **Bedrock Invoke API**；
* **Self-host（自托管）**：在 **SageMaker Endpoint / EC2 / EKS** 部署（只有当模型或依赖无法在托管服务中满足时）。
  **推理策略**
* **リアルタイム/低延迟**：对话与交互；
* **バッチ/非同期**：长文摘要、批量生成；
* **サーバレス**：低频突发更省钱（注意冷启动）。
  **治理**：统一 **Guardrails**（话题限制/PII 遮罩/安全分类），与 **KB/RAG** 绑定提供引用。
  **成本位点**：上下文长度、生成长度（token 费用）、并发与缓存。

---

## 7) フィードバック｜Feedback（持续改进）

**做什么**：收集**用户评分/点击/人工审核**与**日志**，改进提示、KB、检索、微调数据。
**闭环**

1. 线上日志与反馈 →
2. 生成训练/评测集（可含**偏好数据**）→
3. **Prompt/RAG/微调** 迭代 →
4. **A/B/金丝雀发布** →
5. **Model Monitor** + 指标看护（延迟、成本、毒性/拒答率、幻觉率）。
   **AWS 做法**：**CloudWatch + Bedrock Evaluations + SageMaker Model Monitor**；数据回流 S3，定期触发再训练/再评估 Pipeline。

---

## 8) 一页速查｜阶段-要做什么-AWS 对应

| 阶段       | 日文（读法）      | English         | 关键决策                 | AWS 常用组件                             |
| -------- | ----------- | --------------- | -------------------- | ------------------------------------ |
| 数据选择     | データ選択【せんたく】 | Data Selection  | 领域覆盖/质量/合规           | S3, Glue, Guardrails(PII), KB/RAG    |
| 模型选择     | モデル選択       | Model Selection | 能力/延迟/成本/窗口          | Bedrock（模型库、Playground/Evals）        |
| 预训练/继续预训 | 事前トレーニング    | Pre-training    | 是否需要从零或继续学           | Bedrock 现成FM；SageMaker 继续预训          |
| 微调       | ファインチューニング  | Fine-tuning     | SFT/LoRA/指令微调；先 RAG？ | Bedrock Customization / SageMaker    |
| 评估       | 評価【ひょうか】    | Evaluation      | 自动+人工；任务/安全/RAG      | Bedrock Evaluations, Processing      |
| 部署       | デプロイ        | Deployment      | 托管/自托管；实时/批/异步/无服    | Bedrock API, SM Endpoint/Serverless  |
| 反馈       | フィードバック     | Feedback        | 日志/偏好/再训练触发          | CloudWatch, Model Monitor, Pipelines |

---

## 9) 考试“秒选”提示（题干关键词 → 阶段）

* **“术语/风格不对/私域知识缺失”** → 先 **RAG/KB**；还不够再 **Fine-tune**。
* **“成本过高/延迟不稳”** → **模型选择/部署形态** 问题（换更小模型、批/异步、Serverless/缓存）。
* **“回答不可靠/幻觉”** → **评估&治理**（RAG 引用 + Guardrails + Prompt 结构化）。
* **“版本不可复现/上线混乱”** → 用 **Pipelines/Experiments** 做可复现与可回滚。

---

## 10) 术语小词典（中・日・英）

* 基盘模型：**基盤モデル**（きばんモデル）/ Foundation Model
* 预训练：**事前トレーニング** / Pre-training
* 微调：**ファインチューニング** / Fine-tuning（SFT/LoRA/QLoRA）
* 继续预训练：**継続事前学習** / Continued Pre-training
* 检索增强：**検索拡張生成** / RAG（KB/向量检索）
* 守护栏：**ガードレール** / Guardrails
* 评估：**評価** / Evaluation（自动+人工，任务+安全）
* 部署：**デプロイ** / Deployment（Managed API / Self-host）
* 反馈循环：**フィードバック ループ** / Feedback Loop
* 金丝雀发布：**カナリアリリース** / Canary Release

---

### 一句话总括

> **FM 生命周期 = 选好数据与模型 → 先 RAG/提示，再微调 → 用自动+人工评估把关 → 选对部署形态并加 Guardrails → 收集反馈持续改进。**
