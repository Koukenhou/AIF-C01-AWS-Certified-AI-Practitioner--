# AIF-C01｜3.1.6 **マルチステップのタスクにおけるエージェントの役割**

*AIF-C01 Domain 3, Task 3.1.6 — Role of agents (e.g., Amazon Bedrock Agents) in multi-step tasks*

> 目标：理解**什么是“智能体/Agent”**，它在**多步骤任务**中的职责与优势；清楚 **Amazon Bedrock Agents** 的**组成、运行流程、与 RAG/工具调用的关系、安全治理、与 Converse API 的区别**，并会在题目里“对号入座”。术语提供**中・日（含读法）・英**三语，讲解以中文为主。

---

## 0) 一页速览（看到题就能答）

| 你要记住的点                                                                              | 日文（读法）                        | English                                            |
| ----------------------------------------------------------------------------------- | ----------------------------- | -------------------------------------------------- |
| Agent 负责**规划 + 编排 + 执行动作**，把“多轮对话 + 调工具/API + 查知识库”的流程跑完。                           | エージェント【えーじぇんと】                | Agent                                              |
| Bedrock Agent 由**FM（基础模型）+ 指令 + 动作组（Action Groups）+ 知识库（可选）+ 提示模板**构成。              | アクショングループ／ナレッジベース／プロンプトテンプレート | Action groups / Knowledge bases / Prompt templates |
| 运行分 **前处理 → 编排循环（选动作或查 KB、引参数、得观测、再决策）→ 后处理**，由 `InvokeAgent` 发起，可开启**轨迹追踪**。       | 前処理／オーケストレーション／後処理            | Pre-processing / Orchestration / Post-processing   |
| 动作组可用 **OpenAPI 架构** + **可选 Lambda** 实现业务调用；KB 为回答**补上下文与引用**。                      | オープンAPI／ラムダ                   | OpenAPI / Lambda                                   |
| 也可不用 Agent，而用 \*\*Converse API 的 Tool Use（Function calling）\*\*自编排；**Agent 是托管编排**。 | ツールユース                        | Tool use / Function calling                        |

（以上要点出自官方用户指南与 API 文档。）([AWS ドキュメント][1])

---

## 1) 概念与职责｜What an “Agent” does

* **定义（中文）**：在生成式应用里，**Agent**是一个可**自主规划步骤并调用外部动作/检索知识**的“执行体”。给它一个目标，它会：**理解输入 → 选择动作或查 KB → 收集缺失参数 → 执行/汇总结果 → 再决策**，直到产出最终答复。
* **Bedrock Agents 的职责**：

  1. **理解与规划**（FM 生成“下一步打算/理由”）；
  2. **动作调用**（按**动作组**定义调用 API/Lambda，或**询问缺失参数**）；
  3. **知识增强**（查询**Knowledge Bases**拼接“依据”）；
  4. **多轮编排循环**（观察→再决策→直至完成）；
  5. **可选后处理**（格式化输出）。这些步骤由 `InvokeAgent` 统一驱动，并支持**Trace**跟踪每一步的理由、动作和观测。([AWS ドキュメント][1])

---

## 2) 组成组件（Build-time）｜What an Agent is made of

* **基础模型（FM）**：Agent 选择的 FM 用于理解输入、做编排决策并生成答复。
* **指令（Instructions）**：定义 Agent 的**角色与边界**；可用**高级提示模板**在“前处理/编排/KB 生成/后处理”各步细化指令。([AWS ドキュメント][1])
* **动作组（Action Groups）**：告诉 Agent **能做哪些事**。

  * **OpenAPI 模式**：在控制台或 S3 上传 **OpenAPI（JSON/YAML）**，定义可调用的 API 操作与参数；
  * **函数细节模式**：只定义需向用户**引出的参数**；
  * **可选 Lambda**：编写业务逻辑，Agent 在编排中把参数传给 Lambda，再用返回的结果继续推理。([AWS ドキュメント][2])
* **知识库（Knowledge Bases，选配）**：把企业数据接入 RAG，Agent 在需要时查询 KB，把命中的内容**总结/引用**后继续决策。可在创建/更新 Agent 时关联 KB。([AWS ドキュメント][3])
* **提示模板（Prompt Templates）**：Bedrock 提供**四个基础模板**（前处理/编排/KB 生成/后处理），可按需覆盖或关闭某一步以排错或定制策略。([AWS ドキュメント][1])
* **编排策略（Orchestration Strategy）**：可选“**高级提示**”或“**自定义编排**”以应对复杂多步流程与验证。([AWS ドキュメント][4])

---

## 3) 运行流程（Run-time）｜How multi-step orchestration works

1. **前处理（Pre-processing）**：规范化和校验输入（可选）。
2. **编排循环（Orchestration loop）**：

   * **FM 生成“理由/打算”** → 预测**调用哪个动作**或**查询哪个 KB**；
   * **参数不全**：Agent 会**向用户追问**或先**查 KB**补上下文；
   * 执行动作（调用 **Lambda** 或把参数回传给你由应用执行），得到**观测（Observation）**；
   * Agent 将观测**注入下一轮提示**再推理，**循环**直至完成或再次追问。
3. **后处理（Post-processing，默认关闭）**：统一格式输出。
4. **会话与追踪**：可保留**对话历史**增强上下文，且开启**Trace**观察每一步的提示、API 调用与 KB 查询记录。([AWS ドキュメント][1])

---

## 4) 与 RAG/知识库的协同（让回答“有依据”）

* 将 **Knowledge Bases** 与 Agent 关联后，Agent 可在编排中**自动查询 KB**，把命中的内容**总结/引用**后再决定下一步。
* KB 支持对**非结构化与结构化**数据检索（含 **NL→SQL**），方便在多步骤链路里先“查准数”，再“执行动作”。([AWS ドキュメント][3], [Amazon Web Services, Inc.][5])

---

## 5) 与 **Converse API 的 Tool Use**（Function calling）的区别

| 诉求                          | 适合 **Agent**                   | 适合 **Converse Tool Use**              |
| --------------------------- | ------------------------------ | ------------------------------------- |
| **多步骤**、需要**参数引导**与**循环编排** | ✅（托管**编排循环**、参数引导、Trace）       | ❌（需你自己在应用层循环与状态管理）                    |
| **动作目录**可复用、Schema 管理       | ✅（**Action Groups + OpenAPI**） | ⚠️（每次调用需传入工具定义）                       |
| **快速 API 级函数调用/简洁对接**       | ◯                              | ✅（在 `converse` 里直接定义工具；模型请求你去调用并回填结果） |
| **与 KB/Guardrails 一体化**     | ✅（Agent 可直接关联）                 | ◯（可分别调用）                              |

> 官方说明：**Tool use/Function calling**在 Bedrock 中通过 **Converse API** 提供；模型不会直接调用工具，而是**请求你去调用**并把结果回传。**Agent**则把这套循环由平台托管，并可挂接 **KB/Guardrails**。([AWS ドキュメント][6])

---

## 6) 安全与合规（Safety & Governance）

* **Guardrails for Amazon Bedrock**：可在**创建/更新 Agent**时**直接关联**；对**输入与输出**执行话题限制、PII 识别/遮罩等安全控制，且**跨模型通用**。([AWS ドキュメント][7], [Amazon Web Services, Inc.][8])

---

## 7) 典型业务用例（看到关键词就能归类）

* **客服/办事型 Copilot**：查政策（KB）→ 生成说明 → **建/改工单（动作组 API）** → 回传办理结果；必要时**追问缺参**。([AWS ドキュメント][9])
* **内部运营自动化**：读取报表/数据库（KB 的 NL→SQL）→ 汇总 → **调用审批/下发指令（Lambda）**。([Amazon Web Services, Inc.][5])
* **IT/DevOps 助手**：知识问答 + **执行变更**（OpenAPI/Lambda），并以 Trace 留痕审计。([AWS ドキュメント][1])

---

## 8) 成本与延迟的小抄（工程视角）

* **一个循环 = 一次或多次 FM 推理 +（可选）KB 检索 +（可选）动作调用** → **token 成本与延迟**随循环次数上升（控制**追问/步数上限**）。([AWS ドキュメント][1])
* **OpenAPI 粒度**要恰当：过细会导致**多次动作调用**；过粗会让参数引导困难。
* \*\*能在 KB 里“查准”\*\*再执行动作，减少反复尝试。
* **需要强 SLA** 时，可评估（在非 Agent 路径）**预留吞吐/实时端点**等形态做关键步骤的推理保底（此为通用推理形态取舍知识点）。

---

## 9) 术语小词典（中・日・英）

* 智能体/代理：**エージェント** / Agent ([AWS ドキュメント][1])
* 编排（工作流）：**オーケストレーション** / Orchestration ([AWS ドキュメント][1])
* 动作组：**アクショングループ** / Action group（OpenAPI/函数细节 + 可选 **Lambda**） ([AWS ドキュメント][2])
* 知识库：**ナレッジベース** / Knowledge Bases（RAG） ([AWS ドキュメント][3])
* 前/后处理：**前処理／後処理** / Pre-processing / Post-processing ([AWS ドキュメント][1])
* 观察/观测：**オブザベーション** / Observation（动作或 KB 的结果摘要） ([AWS ドキュメント][1])
* 调用入口：**InvokeAgent**（API） / エージェント呼び出し ([AWS ドキュメント][1])
* 工具调用：**ツールユース** / Tool use（Function calling, **Converse API**） ([AWS ドキュメント][6])
* 提示模板：**プロンプトテンプレート** / Prompt templates（四个基础模板，可覆盖） ([AWS ドキュメント][1])
* 安全护栏：**ガードレール** / Guardrails（可与 Agent 直接关联） ([AWS ドキュメント][7])

---

## 10) 一句话总括

> **多步骤任务 = 让 Agent 托管“计划-执行-再计划”循环**：
> 在 **Amazon Bedrock** 中，Agent 以 **FM+指令+动作组+KB+模板**为骨架，**`InvokeAgent`** 发起**前处理→编排循环→后处理**的全链路；在循环中它会**选动作/查 KB/引参数/汇总观测**，直至完成任务；全程可加 **Guardrails** 做安全治理。如果你的流程很简单、只需要“单次工具调用”，用 **Converse API 的 Tool Use** 自己编排就好；**复杂多步**就让 **Agent** 上场。([AWS ドキュメント][1])

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/agents-how.html "How Amazon Bedrock Agents works - Amazon Bedrock"
[2]: https://docs.aws.amazon.com/bedrock/latest/userguide/agents-api-schema.html?utm_source=chatgpt.com "Define OpenAPI schemas for your agent's action groups in ..."
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/agents-kb-add.html?utm_source=chatgpt.com "Augment response generation for your agent with knowledge ..."
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/orch-strategy.html?utm_source=chatgpt.com "Customize agent orchestration strategy - Amazon Bedrock"
[5]: https://aws.amazon.com/bedrock/knowledge-bases/?utm_source=chatgpt.com "Amazon Bedrock Knowledge Bases"
[6]: https://docs.aws.amazon.com/bedrock/latest/userguide/tool-use-inference-call.html?utm_source=chatgpt.com "Call a tool with the Converse API"
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/agents-guardrail.html?utm_source=chatgpt.com "Implement safeguards for your application by associating a ..."
[8]: https://aws.amazon.com/bedrock/guardrails/?utm_source=chatgpt.com "Amazon Bedrock Guardrails"
[9]: https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/develop-a-fully-automated-chat-based-assistant-by-using-amazon-bedrock-agents-and-knowledge-bases.html?utm_source=chatgpt.com "Develop a fully automated chat-based assistant by using ..."
