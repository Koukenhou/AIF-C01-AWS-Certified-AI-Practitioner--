# AIF-C01｜2.3 生成 AI アプリを作るための AWS インフラとテクノロジー（中・日・英对照，日语含读法）

> 目标：看见一道题，能**说清用哪些 AWS 服务搭成生成式 AI 应用**、为什么选它们、有哪些**安全与合规优势**、以及**成本/性能的权衡点**。
> 记忆总纲：**“入口（模型/工具）→ 数据（检索/向量）→ 运行（推理/扩缩）→ 安全（治理/合规）→ 运营（监控/成本）”**。

---

## 0) 一页速览（背会就能拿分）

| 模块     | 日文（读法）                | English             | 你在 AWS 上用什么                                                                                                                                                                             |
| ------ | --------------------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 模型入口   | モデル入口【いりぐち】           | Model Access        | **Amazon Bedrock**（FM 统一 API / KB / Agents / Guardrails / Evaluations）; **SageMaker JumpStart**（开箱模型+模板）；**Bedrock Playground**（交互试验）; **PartyRock**（零代码原型）; **Amazon Q**（企业/开发者 AI 助手） |
| 检索/RAG | 検索拡張生成【けんさくかくちょうせいせい】 | RAG                 | **Bedrock Knowledge Bases**（向量化/检索托管）; **Amazon Kendra**（企业搜索）; **Amazon OpenSearch Serverless** / **Aurora PostgreSQL(pgvector)**（向量库）                                                 |
| 推理形态   | 推論形態【すいろん けいたい】       | Inference Patterns  | **SageMaker Endpoint**（リアルタイム/Async/Serverless/Batch）; **Bedrock API**（托管推理）                                                                                                            |
| 集成     | 連携【れんけい】              | Integration         | **Amazon Lex**（对话入口）、**Transcribe/Translate/Polly**（ASR/MT/TTS）、**Lambda / API Gateway / EventBridge**（编排）                                                                              |
| 安全/合规  | セキュリティ／コンプライアンス       | Security/Compliance | **IAM**、**VPC Endpoint**（私网访问 Bedrock/Kendra 等）、**KMS**（加密）、**CloudTrail**（审计）、**Guardrails**（话题/PII 限制）                                                                                |
| 监控/运维  | モニタリング／運用             | Monitoring/Ops      | **CloudWatch**（指标/日志）、**SageMaker Model Monitor**、**Cost Explorer/CloudWatch Metrics**（成本/延迟/吞吐）                                                                                        |

---

## 1) 构建生成式 AI 的关键 AWS 服务与功能

### 1.1 模型与原型（入口层）

* **Amazon Bedrock（アマゾン・ベッドロック / Bedrock）**：统一访问多家 **Foundation Models（基盤モデル【きばん】）**；内建

  * **Knowledge Bases（ナレッジベース）**：RAG 管道托管（分块/埋め込み/向量検索）。
  * **Agents（エージェント）**：工具/函数调用执行动作。
  * **Guardrails（ガードレール）**：安全/合规护栏（有害内容、PII、话题限制）。
  * **Evaluations（エバリュエーション）**：模型与提示的质量评估。
* **SageMaker JumpStart（セージメーカー・ジャンプスタート / JumpStart）**：开箱模型与解决方案模板，便于自定义训练/部署。
* **Bedrock Playground（プレイグラウンド）**：可视化试 Prompt、参数、比较模型。
* **PartyRock（パーティーロック）**：零/低代码快速做 Demo 与分享。
* **Amazon Q（アマゾン・キュー）**：

  * **Q Business**：企业知识助理（接数据源、权限内检索+回答）。
  * **Q Developer**：开发者助理（代码建议、评审、修复、文档问答）。

### 1.2 RAG / 向量与搜索（数据接入层）

* **Bedrock Knowledge Bases**：自动完成 **チャンク化(Chunking)** → **埋め込み(Embedding)** → **ベクトル検索(Vector Search)** → 将命中文档**连同引用**注入提示。
* **Amazon Kendra（ケンドラ）**：企业级语义搜索与 FAQ。
* **Amazon OpenSearch Service/Serverless（オープンサーチ）**：向量检索、BM25 混合检索。
* **Aurora PostgreSQL with pgvector / Amazon RDS PostgreSQL**：关系型 + 向量扩展。

### 1.3 推理与集成（应用层）

* **SageMaker Inference**：

  * **Real-time Endpoint（リアルタイム）**：低延迟在线服务。
  * **Asynchronous Inference（非同期）**：长耗时/大载荷任务。
  * **Batch Transform（バッチ）**：离线批量。
  * **Serverless Inference（サーバレス）**：低频突发、免管实例。
* **事件与编排**：**Lambda**（函数）、**API Gateway**（API 暴露）、**EventBridge**（事件总线）、**Step Functions**（工作流）。
* **会话入口**：**Amazon Lex**（意图/槽位/IVR）、**Amazon Connect**（联络中心）。
* **语音/翻译**：**Transcribe / Translate / Polly**。

### 1.4 安全与合规（治理层）

* **IAM（アクセス制御）**：最小权限控制；**条件策略**防越权。
* **VPC Endpoint（プライベート接続）**：在私网内调用 Bedrock/Kendra/OpenSearch 等。
* **KMS（暗号化）**：数据 at rest / in transit 加密。
* **CloudTrail（監査ログ）**：谁、何时、做了什么。
* **Bedrock Guardrails**：话题限制、有害内容/PII 过滤、提示注入防护。

### 1.5 监控与成本（运营层）

* **CloudWatch**：延迟（レイテンシ）、吞吐（スループット）、错误率、并发。
* **Model Monitor**：生产端监控数据/概念漂移。
* **Cost Explorer / Billing**：**トークン課金【かきん】(Token-based Pricing)**、**プロビジョンドスループット**（保性能配额）跟踪。

---

## 2) 使用 AWS 生成式 AI 服务构建应用的**优势**

| 中文    | 日文（读法）               | English              | 说明                                           |
| ----- | -------------------- | -------------------- | -------------------------------------------- |
| 易用性   | アクセシビリティ             | Accessibility        | Bedrock/Playground/PartyRock **低门槛**，不必自训模型。 |
| 低进入门槛 | 参入障壁が低い【さんにゅう しょうへき】 | Low barrier to entry | KB/RAG/Agents/Guardrails **托管**，少量工程即可上线。    |
| 效率    | 効率【こうりつ】             | Efficiency           | 统一 API、托管评估/安全；与现有 AWS 生态无缝集成。               |
| 成本效益  | 費用対効果【ひようたいこうか】      | Cost effectiveness   | Serverless / Async / 批处理，多形态降本；按 token 计费可控。 |
| 上市速度  | 市場投入までのスピード          | Time to market       | Playground → 原型 → KB/Agent → 托管上线，一条龙。       |
| 目标对齐  | 目標達成【もくひょう たっせい】     | Goal alignment       | 结合 **评估** 与 **业务指标** 验证价值闭环。                 |

---

## 3) 在 AWS 基础设施上部署生成式 AI 的**利点**

* **Security（セキュリティ）/Compliance（コンプライアンス）**：

  * IAM、KMS、CloudTrail、VPC Endpoint，满足企业合规与审计。
* **Shared Responsibility（責任共有モデル）**：AWS 管平台，你管数据/配置。
* **Safety（安全性）**：Guardrails、内容分类、PII 保护、模型评估。
* **Global Regions（リージョン展開）**：就近部署，降低延迟；注意模型**可用区域**差异。
* **Scalability（スケーラビリティ）**：Auto Scaling / Serverless / Provisioned Throughput（事前保留吞吐）。

---

## 4) 成本与性能的**权衡点**（Trade-off）

| 维度    | 日文（读法）    | 要点/建议                                                    |
| ----- | --------- | -------------------------------------------------------- |
| 响应性   | レイテンシ     | 对话/翻译 → **Real-time/Provisioned**；长任务 → **Async/Batch**。 |
| 吞吐    | スループット    | 高并发可用 **プロビジョンドスループット**（Bedrock/SageMaker 预留）。           |
| 区域覆盖  | リージョン展開   | 选离用户最近的 Region；确认模型在该区是否可用。                              |
| 价格模型  | トークン課金    | 控制 **上下文长度**、**输出 max tokens**；复用系统指令；Prompt 压缩。         |
| 生成成本  | 推論コスト     | 用 **小模型** + **RAG** 优化；缓存热回答；批量任务走 **Batch**。            |
| 自定义成本 | カスタムモデル費用 | 微调/继续预训前先做 **Prompt/RAG**；确需微调优先 **LoRA/QLoRA**。         |

---

## 5) 参考架构（你可以直接照此答题）

### 5.1 RAG 文档问答（企业知识）

```
S3/Docs → KB(Chunk+Embedding+Vector) → Bedrock(KB Retrieval)
        → Guardrails → Bedrock LLM → API Gateway/Lambda → App
监控：CloudWatch  | 审计：CloudTrail | 权限：IAM | 私网：VPC Endpoint
```

### 5.2 客服 Agent（能办事）

```
用户 → Lex/Connect → Bedrock Agents(工具调用/函数调用)
      → KB 检索(Knowledge Bases / Kendra) → 业务API(Lambda, Step Functions)
      → 结果/确认(HITL可选)
安全：Guardrails  | 记录：CloudTrail | 指标：CloudWatch
```

### 5.3 批量要约/翻译/代码修复

```
S3 批量文件 → EventBridge 触发 → Step Functions
  → (并行) Lambda/SM Batch → Bedrock 调用 → 写回 S3 → 报表
成本优先：Batch/Async | 重试与限流：SFN/Lambda  | 成本看板：Cost Explorer
```

---

## 6) 选择题“秒选”清单

1. **让模型会用公司文档** → **KB/RAG（Knowledge Bases / Kendra）**。
2. **怕越权/敏感** → **Guardrails + IAM + VPC Endpoint**。
3. **低频/突发** → **Serverless**；**长任务** → **Async**；**海量离线** → **Batch**；**交互** → **Real-time**。
4. **高并发稳定** → **Provisioned Throughput / 预留并发**。
5. **成本高** → 缩上下文、缓存、换小模型 + RAG、批处理。
6. **要能“执行动作”** → **Bedrock Agents + Step Functions/Lambda**。
7. **开发效率** → **Playground/PartyRock** 先原型；**Amazon Q** 提升编码与业务问答效率。

---

## 7) 小词典（中・日・英）

| 中文     | 日文（读法）         | English                |
| ------ | -------------- | ---------------------- |
| 基盘模型   | 基盤モデル【きばんモデル】  | Foundation Model (FM)  |
| 知识库    | ナレッジベース        | Knowledge Bases        |
| 代理/智能体 | エージェント         | Agent                  |
| 向量检索   | ベクトル検索         | Vector Search          |
| 守护栏    | ガードレール         | Guardrails             |
| 预留吞吐   | プロビジョンドスループット  | Provisioned Throughput |
| 令牌计费   | トークン課金         | Token-based Pricing    |
| 延迟/吞吐  | レイテンシ / スループット | Latency / Throughput   |
| 私网终端   | VPC エンドポイント    | VPC Endpoint           |

---

### 一句话总括

> **在 AWS 上做生成式 AI**：以 **Bedrock** 为**模型入口**，用 **KB/RAG** 接入你的知识，靠 **Agents** 执行动作，配 **Guardrails + VPC/IAM/KMS** 保安全，选 **Realtime/Async/Batch/Serverless** 平衡**延迟/成本**，再用 **CloudWatch/Trail** 做**运营与审计**。
> 按此框架答题，基本都能对上考点。
