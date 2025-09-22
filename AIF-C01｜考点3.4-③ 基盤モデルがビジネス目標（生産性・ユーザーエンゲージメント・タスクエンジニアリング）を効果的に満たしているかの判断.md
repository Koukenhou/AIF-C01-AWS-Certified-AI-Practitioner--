# AIF-C01｜3.4.3 **判定基盤模型是否有效达成业务目标（生产力・用户参与・任务工程）**／**基盤モデルがビジネス目標（生産性・ユーザーエンゲージメント・タスクエンジニアリング）を効果的に満たしているかの判断**

*（中文为主；术语三语：中・日〔カタカナ读法〕・英）*

---

## 0) 考点定位（你要会什么）

* 会把**业务目标**转成**可计算指标**：生产力（**生産性〔セイサンセイ〕／Productivity**）、用户参与（**ユーザー・エンゲージメント／User engagement**）、任务工程（**タスク・エンジニアリング／Task engineering**）。
* 知道在 **Amazon Bedrock** 上，如何用 **Evaluations（評価：自動＋人手）**、**CloudWatch 监控**、**Guardrails 指标**、**RAG 评估**、**（即将停服的）Evidently A/B 实验**等把**离线质量**与**在线业务 KPI**打通。([AWS Documentation][1])

---

## 1) 关键术语（中・日〔读法〕・英）

| 术语     | 日文（读法）                 | English                           | 解释                  |
| ------ | ---------------------- | --------------------------------- | ------------------- |
| 业务目标   | ビジネス目標（モクヒョウ）          | Business objectives               | 如**处理量↑/成本↓/满意度↑**  |
| 总体评价准则 | 総合評価基準（ソウゴウ・ヒョウカ・キジュン） | OEC: Overall Evaluation Criterion | 把多目标合成一个“北极星指标”     |
| 生产力    | 生産性（セイサンセイ）            | Productivity                      | AHT↓、任务/人时↑、交付周期↓   |
| 用户参与   | ユーザー・エンゲージメント          | User engagement                   | DAU、留存、CSAT、采纳率等    |
| 任务工程   | タスク・エンジニアリング           | Task engineering                  | **任务拆解/工具调用/完成率**   |
| 程序化评估  | 自動評価（ジドウ・ヒョウカ）         | Programmatic evaluation           | 传统/内置指标自动打分         |
| 人工评估   | 人手評価（ニンテ・ヒョウカ）         | Human evaluation                  | 评审员按**有用性/合规**打分    |
| RAG 评估 | RAG評価（アールエージー）         | RAG evaluation                    | **检索**与**生成**分开测    |
| 运营监控   | 運用監視（ウンヨウ・カンシ）         | Observability/Monitoring          | CloudWatch 指标/日志/告警 |

> AWS 官方强调：**模型/KB 的效果评估**用 **Bedrock Evaluations**（自动+人评），**运行期**用 **CloudWatch + 日志**，**RAG**有专用指标与作业流。([Amazon Web Services, Inc.][2])

---

## 2) “从业务到指标”的三层映射

### 2.1 业务→评测→运营 的三层

1. **业务层（Business KPIs）**：AHT、工单自助化率、获客转化率、CSAT/NPS、合规风险事件数等。AWS 指南建议把**业务价值/用户满意/技术性能**同时纳入度量框架。([AWS Documentation][3])
2. **评测层（Quality metrics）**：自动/人评分数，如**准确性/鲁棒性/有害性**、RAG 的**检索正确性/有据可依**等。([AWS Documentation][4])
3. **运营层（Ops metrics）**：**吞吐/延迟/P95**、**调用/代币量/成本**、**Guardrails 命中率**、**错误/限流**，均入 CloudWatch/日志。([AWS Documentation][5])

> 实务建议：为应用定义一个 **OEC**，如「**每成功任务成本（Cost per Successful Task）**」，由**离线质量×在线转化×成本**合成。AWS 方案文档也建议**同时跟踪业务价值、用户满意与技术性能**。([AWS Documentation][3])

---

## 3) 如何度量“生产力・参与度・任务工程”

### 3.1 生产力（生産性｜Productivity）

* **代表指标**：

  * **平均处理时长 AHT**↓、**任务/人时**↑；
  * **交付周期/上市时间**（Time to market）↓；
  * **缺陷密度**↓、**代码评审周期**↓（开发场景）。([AWS Documentation][6])
* **如何测**（AWS）：

  * **离线**：用 Bedrock Evaluations 保证**可用性/格式/安全**达标；
  * **在线**：启用 **Model Invocation Logging**，在 **Athena + QuickSight** 做**代币→成本**与**时延/吞吐**看板；结合业务系统埋点输出 AHT/完成量。([AWS Documentation][7])

### 3.2 用户参与（ユーザー・エンゲージメント｜Engagement）

* **代表指标**：**DAU/MAU**、**会话时长**、**留存率（D1/D7）**、**采纳率（答案被采纳/复制）**、**CSAT/NPS**。
* **如何测**：

  * **A/B 对比**不同提示/模型/Guardrails 策略，观察**会话深度/CSAT**变化。历史上可用 **CloudWatch Evidently** 做实验；**注意：官方宣布 Evidently 将于 2025-10-16 停止支持**，规划时请预留替代策略（如 AppConfig+自研实验或第三方）。([AWS Documentation][8])

### 3.3 任务工程（タスク・エンジニアリング｜Task engineering）

* **代表指标**：

  * **任务完成率**、**无关/拒答率**、**人工接管率**、**工具调用成功率**、**多步失败节点分布**。
* **如何测**：

  * **Agent/Flow Trace**：开启 **Trace**，逐步记录**观察→决策→动作**，定位失败/低效步骤；AgentCore/Flows 支持在控制台或 API 查看 trace。([AWS Documentation][9])
  * **生成式可观测性**：CloudWatch 的 **GenAI Observability** 与 AgentCore Observability（预览/新功能）集中展示 agent 指标与追踪。([AWS Documentation][10])

---

## 4) 以 AWS 官方能力构建“业务判定流水线”

### 4.1 离线评测（质量门槛）

* **模型/KB 评估**：用 **Bedrock Evaluations** 跑**程序化**（Accuracy/Robustness/Toxicity 等）与**人评**（有用性/风格/品牌一致性），亦可**自定义指标**（LLM-as-a-judge 提示+评分量表）。([AWS Documentation][4])
* **RAG 专项**：区分 **Retrieve-only** 与 **Retrieve-and-Generate**，看**检索正确性/有据可依**，用于比较不同 KB/嵌入/Top-k。([AWS Documentation][11])

### 4.2 上线观察（运营与成本）

* **启用调用日志**（Model Invocation Logging）→ **CloudWatch Logs**/**S3**；可计算**请求量、代币、延迟、错误**，并据此推估**成本/成功任务**。([AWS Documentation][7])
* **CloudWatch 指标**：Bedrock/AgentCore 提供**调用、429/4xx/5xx、节流**等运行指标；**Guardrails** 另提供**拦截/遮罩/通过**等比率。([AWS Documentation][12])
* **可视化**：用 **Athena + QuickSight** 做用量/成本/质量一体化看板。([Amazon Web Services, Inc.][13])

### 4.3 在线对照（实验 & 决策）

* **A/B/多变量实验**：对**提示模板/模型家族/系统指令/Guardrails 阈值**做对比，观测**业务 KPI**（如 CSAT、转化、AHT）。

  * **Evidently** 能做实验并给出**统计建议**，但**官方已公告 2025-10-16 停止支持**；短期可用，长期需替代（AppConfig+自研或第三方）。([AWS Documentation][14])

---

## 5) 场景化范式（三个“能落地”的判断模板）

### 5.1 客服自助（生产力为王）

**目标**：把 30% 人工工单转为自助；**OEC**=「**每成功解答成本**」。
**做法**：

1. **离线**：Evaluations 检查**正确性/有害性**；RAG 评估**检索命中**。([AWS Documentation][4])
2. **在线**：启用调用日志，统计**成功解答数**、**代币→成本**、**AHT**；Guardrails 监看**拦截率**。([AWS Documentation][7])
3. **判断**：若 **成本/成功任务**↓20%，**人工接管率**<10%，**CSAT**不降 → **达成**。

### 5.2 文档问答（参与度优先）

**目标**：**会话深度/留存**↑；**OEC**=「**每会话有效轮次**」。
**做法**：

1. **离线**：RAG 的 Retrieve-and-Generate，调嵌入/Top-k。([AWS Documentation][11])
2. **在线**：（短期）Evidently/（长期）替代实验，比较**提示模板**对**会话长度/采纳率/CSAT**影响；CloudWatch 跟**延迟/P95**防劣化。([AWS Documentation][8])
3. **判断**：变体 B **有效轮次**↑15% 且**P95 延迟**无显著上升 → **上线 B**。

### 5.3 多步 Agent（任务工程）

**目标**：提高**自动完成率**，降低**失败/回退**。
**做法**：

* 打开 **Trace**，统计**每步失败点**、**工具调用成功率**、**知识库命中率**；用 AgentCore/CloudWatch 的**GenAI Observability**集中查看。([AWS Documentation][9])
* 迭代**动作顺序/参数采集**；Guardrails 观察**策略阻断**是否过严影响完成率。([AWS Documentation][15])
  **判断**：**任务完成率**从 62%→81%，**人工接管率**<8%，**阻断率**稳定 → **目标达成**。

---

## 6) 指标清单（可直接抄到 OKR）

### 6.1 业务 KPI（Business）

* **成本/成功任务（CPS）**、**AHT**、**工单自助率**、**转化率/GMV**、**CSAT/NPS**。
* 参考 AWS 指南：在**业务价值/用户满意/技术性能**三维建立度量。([AWS Documentation][3])

### 6.2 质量 KPI（Evaluation）

* **准确性/鲁棒性/有害性**（内置）、**人评有用性/风格/品牌一致性**、**RAG 检索正确性/有据可依**、**自定义指标**（如“是否引用政策条款”）。([AWS Documentation][4])

### 6.3 运营 KPI（Ops）

* **吞吐/延迟 P50/P95**、**代币/请求/成本**、**错误/限流（4xx/5xx/429）**、**Guardrails 拦截/通过/遮罩率**、**Agent 步骤失败率**。([AWS Documentation][5])

---

## 7) 数据与看板落地（一步一脚印）

1. **启用调用日志**（Bedrock Console → Settings → Model invocation logging）→ 送 **CloudWatch Logs/S3**。([AWS Documentation][16])
2. **用 Athena/QuickSight** 汇总**代币/成本/延迟**，做**成本/成功任务**与**预算告警**。([Amazon Web Services, Inc.][13])
3. **Guardrails 指标**上 CloudWatch，设**阻断率异常**告警。([AWS Documentation][15])
4. **Evaluations 报告**存 S3，周期性复评（模型/提示/KB 变更）。([AWS Documentation][17])
5. **Agent/Flow Trace** 开启，导出关键**失败路径**供优化。([AWS Documentation][9])

---

## 8) 注意事项（踩坑提示）

* **实验平台**：CloudWatch Evidently **将于 2025-10-16 停止支持**；请评估替代（AppConfig+自研实验或第三方），并迁移实验指标与告警。([AWS Documentation][18])
* **“高分≠达标”**：离线 ROUGE/BERTScore/BLEU 很高，但业务 KPI（转化/自助率）可能无提升，务必**以 OEC 收敛**。([AWS Documentation][3])
* **成本黑洞**：只看**速度/质量**易忽略**代币与成本**；务必用 **Invocation Logging**+可视化把**成本/任务**纳入 OEC。([AWS Documentation][7])

---

## 9) “拿分速记”（问→答）

* **问：怎么判断 FM 达成业务目标？**
  **答**：先定义 **OEC**（业务价值×用户满意×技术性能），离线用 **Evaluations** 过质量门槛，在线以 **CloudWatch/日志/Guardrails 指标**与\*\*（替代）A/B 实验\*\*看业务 KPI 是否显著改善。([Amazon Web Services, Inc.][2])
* **问：RAG 应用如何评？**
  **答**：用 Bedrock 的 **Retrieve-only / Retrieve-and-Generate** 作业与内置指标，比较不同 KB/Embedding/Top-k。([AWS Documentation][11])
* **问：Agent 的“任务工程”如何量化？**
  **答**：启用 **Trace** 与 AgentCore Observability，量**步骤失败率/工具成功率/回退率**，并联动 Guardrails 阈值。([AWS Documentation][9])

---

## 参考（AWS 官方优先）

* **Bedrock Evaluations 总览**与**模型/KB 指标**、**人评/自定义指标**。([AWS Documentation][1])
* **RAG 评估**（Retrieve-only / Retrieve-and-Generate、指标与报告）。([AWS Documentation][11])
* **CloudWatch 监控**（Bedrock 指标、调用日志、知识库日志）。([AWS Documentation][5])
* **Guardrails 运行期指标与监控**。([AWS Documentation][15])
* **Athena + QuickSight** 用日志做用量/成本可视化。([Amazon Web Services, Inc.][13])
* **A/B 实验（Evidently）**：能力与**停服公告**。([AWS Documentation][8])
* **开发生产力度量（Prescriptive Guidance）**。([AWS Documentation][6])
* **企业级平台度量三维（业务价值/用户满意/技术性能）**。([AWS Documentation][3])
* **Agent/Flow Trace 与可观测性**（AgentCore/Flows）。([AWS Documentation][9])

---

### 一句话总结

> **“离线质量合格 + 在线 KPI 提升 + 成本可控”才叫达标。**在 AWS 上，用 **Bedrock Evaluations**（模型/KB 质量）→ **CloudWatch/日志/Guardrails**（运行期性能与风险）→ **实验对照**（在 Evidently 停服前或用替代方案），并以 **OEC** 收敛多目标，你就能**客观判定**基盤模型是否真正满足**生产力、用户参与、任务工程**等业务目标。([Amazon Web Services, Inc.][2])

需要的话，我可以把上面的 **OEC 计算模板**（含“成本/成功任务、采纳率、P95 时延、Guardrails 阻断率”权重示例）和 **Athena/QuickSight** 示例查询一并整理给你。

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation.html?utm_source=chatgpt.com "Evaluate the performance of Amazon Bedrock resources"
[2]: https://aws.amazon.com/bedrock/evaluations/?utm_source=chatgpt.com "Evaluate Foundation Models - Amazon Bedrock Evaluations"
[3]: https://docs.aws.amazon.com/pdfs/prescriptive-guidance/latest/strategy-enterprise-ready-gen-ai-platform/strategy-enterprise-ready-gen-ai-platform.pdf?utm_source=chatgpt.com "AWS Prescriptive Guidance - Building an enterprise-ready ..."
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-evaluation-metrics.html?utm_source=chatgpt.com "Use metrics to understand model performance"
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/monitoring.html?utm_source=chatgpt.com "Monitoring the performance of Amazon Bedrock"
[6]: https://docs.aws.amazon.com/prescriptive-guidance/latest/strategy-accelerate-software-dev-lifecycle-gen-ai/measuring-success.html?utm_source=chatgpt.com "Measuring the success of generative AI in software ..."
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-invocation-logging.html?utm_source=chatgpt.com "Monitor model invocation using CloudWatch Logs and ..."
[8]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Evidently.html?utm_source=chatgpt.com "Perform launches and A/B experiments with CloudWatch ..."
[9]: https://docs.aws.amazon.com/bedrock/latest/userguide/trace-events.html?utm_source=chatgpt.com "Track agent's step-by-step reasoning process using trace"
[10]: https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/observability-view.html?utm_source=chatgpt.com "View observability data for your Amazon Bedrock ..."
[11]: https://docs.aws.amazon.com/bedrock/latest/userguide/evaluation-kb.html?utm_source=chatgpt.com "Evaluate the performance of RAG sources using ..."
[12]: https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/gateway-advanced-observability-metrics.html?utm_source=chatgpt.com "Setting up CloudWatch metrics and alarms - Amazon Bedrock ..."
[13]: https://aws.amazon.com/blogs/business-intelligence/monitor-and-optimize-your-amazon-bedrock-usage-with-amazon-athena-and-amazon-quicksight/?utm_source=chatgpt.com "Monitor and optimize your Amazon Bedrock usage with ..."
[14]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Evidently-calculate-results.html?utm_source=chatgpt.com "How Evidently calculates results - Amazon CloudWatch"
[15]: https://docs.aws.amazon.com/bedrock/latest/userguide/monitoring-guardrails-cw-metrics.html?utm_source=chatgpt.com "Monitor Amazon Bedrock Guardrails using CloudWatch metrics"
[16]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/model-invocations.html?utm_source=chatgpt.com "Model Invocations - Amazon CloudWatch"
[17]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-evaluation-report-programmatic.html?utm_source=chatgpt.com "Review metrics for an automated model evaluation job in ..."
[18]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Evidently-featuretraffic.html?utm_source=chatgpt.com "See the current evaluation rules and audience traffic for a ..."
