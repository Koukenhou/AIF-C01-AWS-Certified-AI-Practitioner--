# AIF-C01｜2.3.2 生成 AI サービスを使ってアプリを作るメリット

*AIF-C01 Domain 2, Task 2.3.2 — Benefits of building apps with AWS generative-AI services*

> 目标：清晰说明用 **AWS 生成式 AI 服务**（Amazon Bedrock / Bedrock Playground / Knowledge Bases / Agents / Guardrails / SageMaker JumpStart / PartyRock / Amazon Q 等）构建应用时的**核心优势**，以及这些优势在**产品化**与**考试题干**中的体现。术语提供**中・日（含读法）・英**三语。

---

## 0) 先认清“积木”（哪些服务在起作用）

* **Amazon Bedrock（ベッドロック）/ Amazon Bedrock**：统一调用多家 **基盘模型【きばんモデル】/ Foundation Models**；内置 **Knowledge Bases（ナレッジベース）/ RAG**、**Agents（エージェント）**、**Guardrails（ガードレール）**、**Evaluations**。
* **Bedrock Playground（プレイグラウンド）**：无代码试 Prompt/参数/模型。
* **SageMaker JumpStart（ジャンプスタート）**：开箱模型/模板，便于**微调与部署**。
* **PartyRock（パーティーロック）**：零/低代码快速做 Demo。
* **Amazon Q（アマゾン・キュー）**：**Q Business**（企业知识助理）、**Q Developer**（开发者助理）。
* （配套）**IAM / KMS / VPC Endpoint / CloudTrail / CloudWatch / Step Functions / Lambda / API Gateway / Connect / Lex / Transcribe / Translate / Polly**。

---

## 1) アクセシビリティ【アクセスシビリティ】/ Accessibility

**中文**：**易用性**——几乎“开箱即用”，从控制台或 API 即可调用模型。
**为什么能做到**

* Bedrock 提供**统一 API** 与**控制台 Playground**；无需自管集群/驱动/权重。
* PartyRock & Playground **零/低代码**原型；Amazon Q **开箱**面向员工。
  **场景例**：产品经理 30 分钟搭一个“文档问答”原型给业务试用。
  **考试信号词**：*easy to start / console / no infra / unified API / no code* → 选 **Bedrock/Playground/PartyRock**。

---

## 2) 参入障壁の低さ【さんにゅう しょうへき の ひくさ】/ Low barrier to entry

**中文**：**入门门槛低**——不需要自己训练模型或维护推理集群。
**为什么能做到**

* **RAG 托管：Knowledge Bases** 自动完成 **分块/嵌入/向量检索**。
* **Agents** 将“调用业务 API 的逻辑”图形化配置；**Guardrails** 一处配置、全模型生效。
  **场景例**：把 S3/SharePoint 文档连进 KB → 秒变企业搜索/问答。
  **考试信号词**：*RAG managed / agent / guardrails / no training*。

---

## 3) 効率性【こうりつせい】/ Efficiency

**中文**：**效率高**——开发、集成、运营的效率全面提升。
**为什么能做到**

* **统一入口**：模型切换/AB 对比在同一平台完成（Playground/Evaluations）。
* **一站式**：KB（检索）、Agents（工具）、Guardrails（安全）让你少写样板。
* **与 AWS 生态**：Step Functions/Lambda/Connect/Lex/CloudWatch 无缝衔接。
  **场景例**：客服机器人从“能回答”到“能办事”（建工单/改预约）只需在 Agents 里接函数。
  **考试信号词**：*workflow integration / evaluations / pipeline / end-to-end*。

---

## 4) 費用対効果【ひよう たいこうか】/ Cost-effectiveness

**中文**：**性价比**——“按量付费 + 形态可选”让成本可控。
**为什么能做到**

* **Token-based 计费**，上下文与输出可控（缩短 Prompt/设置 max tokens）。
* 推理**多形态**：**Serverless**（低频/突发最省）、**Async**（长任务不占并发）、**Batch**（离线大吞吐）。
* **选择更小模型 + RAG** 通常更省钱且满足需求。
  **场景例**：批量要约夜间跑 Batch；客服高峰前开 **Provisioned Throughput（预留吞吐）** 保稳定。
  **考试信号词**：*serverless / async / batch / provisioned throughput / cost per inference / token cost*。

---

## 5) 市場投入までのスピード【しじょう とうにゅう まで の スピード】/ Speed to market

**中文**：**上市速度快**——从 idea 到 PoC/试点非常快。
**为什么能做到**

* Playground/PartyRock 即时试验；**KB/Agents/Guardrails** “搭积木”。
* **JumpStart** 一键部署/微调；Bedrock **Evaluations** 自动比较质量。
  **场景例**：一周内上线“内部知识 Copilot”试点，随后按反馈迭代。
  **考试信号词**：*prototype quickly / PoC / go-live fast*。

---

## 6) ビジネス目標の達成能力【もくひょう の たっせい のうりょく】/ Ability to achieve business goals

**中文**：**对齐业务目标**——不仅能“生成”，还能**执行动作**与**可控合规**。
**为什么能做到**

* **Agents** 调用业务 API → 变“答题”为“办事”；
* **Guardrails** 过滤不当内容/限制话题/PII 脱敏；
* **Evaluations**（离线/对比）帮助用**业务指标**评估（正确性/相关性/毒性/可引用）。
  **场景例**：面向客服的 Agent：先从 KB 检索政策 → 生成回答 → 触发退款流程（需二次确认）。
  **考试信号词**：*agent / tool use / guardrails / evaluation / KPI / groundedness*。

---

## 7) 汇总对照（考试速记）

| 优势    | 日文（读法）   | English            | 直接落地的 AWS 组件                                              |
| ----- | -------- | ------------------ | --------------------------------------------------------- |
| 易用性   | アクセシビリティ | Accessibility      | **Bedrock API / Playground / PartyRock**                  |
| 入门门槛低 | 参入障壁の低さ  | Low barrier        | **Knowledge Bases / Agents / Guardrails**                 |
| 效率    | 効率性      | Efficiency         | **Evaluations / Step Functions / Lambda / Connect / Lex** |
| 性价比   | 費用対効果    | Cost-effectiveness | **Serverless / Async / Batch / Provisioned Throughput**   |
| 上市速度  | スピード     | Speed to market    | **Playground / JumpStart / KB/Agents 即插即用**               |
| 目标对齐  | 目標達成能力   | Business goal fit  | **Guardrails / Evaluations / Q Business & Q Developer**   |

---

## 8) 典型问法 → 答案模板

* **Q**：我们想快速做内部知识问答，后面接流程办理，AWS 有什么优势？
  **A**：**Bedrock + KB（RAG）+ Agents + Guardrails**，**Playground** 快速试，**Serverless/Async** 控成本，**CloudWatch** 监控。
* **Q**：成本压力大、延迟敏感怎么平衡？
  **A**：**小模型 + RAG**；**缩短上下文/输出**；**Serverless/Async/Batch**；高峰用 **Provisioned Throughput**。
* **Q**：如何保证安全合规？
  **A**：**Guardrails**（话题/PII）、**VPC Endpoint**（私网）、**KMS**（加密）、**IAM/CloudTrail**（最小权限与审计）。

---

## 9) 术语小词典（中・日・英）

* 易用性：**アクセスシビリティ** / Accessibility
* 入门门槛：**参入障壁【さんにゅう しょうへき】** / Barrier to entry
* 效率：**効率性【こうりつせい】** / Efficiency
* 费用对效：**費用対効果【ひよう たいこうか】** / Cost-effectiveness
* 上市速度：**市場投入までのスピード** / Speed to market
* 目标达成：**目標達成能力** / Ability to achieve business goals
* 预留吞吐：**プロビジョンドスループット** / Provisioned Throughput
* 令牌计费：**トークン課金** / Token-based pricing

---

### 一句话总括

> **用 AWS 做生成式 AI** = 统一模型入口（Bedrock） + 托管拼装（KB/Agents/Guardrails/Evaluations） + 快速试验（Playground/PartyRock/JumpStart） + 成本与安全的工程化选项（Serverless/Async/Batch/Provisioned、IAM/KMS/VPC/Trail）。
> 这些正对应考试里“**易用性、低门槛、效率、性价比、上市速度、业务达成**”六大优势。
