# AIF-C01｜3.1.2 推論パラメータが応答へ与える影響

*AIF-C01 Domain 3, Task 3.1.2 — How inference parameters affect model responses*

> 目标：掌握**温度（Temperature）**、**Top-K/Top-P**、**入出力长度（上下文窗口 & max\_tokens）**、**停止シーケンス（Stop Sequences）**、**ペナルティ（Penalties）**等对输出**随机性、确定性、长度、稳定性、成本**的影响；能给出**场景化参数建议**与**常见陷阱**。文内给出**中・日（含读法）・英**对照。

---

## 1) 全局观念（先抄底线）

* **推論パラメータ【すいろん…】/Inference parameters**用于**限制或调整**生成过程：要么**改变候选分布**（更随机/更保守），要么**限制输出**（长度/提前终止）。官方将其分为两类：**随机性与多样性**（Randomness & Diversity）与**长度控制**（Length）。([AWS ドキュメント][1])
* **不同模型**的默认值/取值范围**不完全相同**，以**各模型文档**与 API **InferenceConfiguration**/`converse` 为准。([AWS ドキュメント][2])

---

## 2) 随机性与多样性（Randomness & Diversity）

### 2.1 温度｜温度【おんど】｜**Temperature**

* **作用**：改变下一个 token 的**概率分布陡峭度**；**低温** → 更**确定**（高概率词），**高温** → 更**多样**（采样更分散）。([AWS ドキュメント][1])
* **典型用法**：

  * 事实问答/分类/代码：**0–0.2**（高确定性）。
  * 创意写作/头脑风暴：**0.7–1.0**（更多样）。

### 2.2 トップK｜**Top-K**

* **作用**：仅在\*\*“前 K 个最可能”\*\*候选里采样；**小 K**更保守，**大 K**更开放。([AWS ドキュメント][1])

### 2.3 トップP（核采样）｜**Top-P (Nucleus)**

* **作用**：在**累计概率达到 P** 的集合中采样；**小 P**更保守，**大 P**更开放。([AWS ドキュメント][1])

> **实务注意**：Anthropic（Claude）文档明确建议**调温度或 Top-P 二选一**，不要同时大幅调两者，以免交互作用难以预期。([AWS ドキュメント][3])

---

## 3) 长度与终止（Length & Stopping）

### 3.1 入力長・出力長｜**Context window & max\_tokens**

* **输入**受**上下文窗口（Context window）**约束（模型特性）；**输出**由 `max_tokens` 限制，模型也可能**提前停止**（自然结束或遇到 stop）。([AWS ドキュメント][4])
* **上下文窗口**随模型不同而不同（例如 Claude 家族提供**长上下文**与**更长上下文的 Beta**）。([Amazon Web Services, Inc.][5], [Anthropic][6])
* **账户/区域还受 Bedrock 配额**与 token 速率管控；需留意**配额与计数**。([AWS ドキュメント][7])

### 3.2 停止シーケンス｜**Stop Sequences**

* **作用**：出现指定子串即**停止生成**；常用于**JSON 封闭**、**多轮链式调用**安全中断、或**避免越界**。([AWS ドキュメント][4])

### 3.3 ペナルティ（长度/重复等）｜**Penalties**

* **作用**：对**重复 token、频率、类型、长度**等施加惩罚，减少啰嗦或循环。是否支持、取值范围视模型而定。([AWS ドキュメント][1])

---

## 4) API 视角（你在代码里看到什么）

* **Converse / ConverseStream（统一接口）**：在 `inferenceConfig` 里设 `temperature / topP / maxTokens / stopSequences`；如模型有**独有参数**也可传递。流式接口能**降低体感延迟**。([AWS ドキュメント][8], [Boto3][9])
* **InferenceConfiguration（运行时）**：`maxTokens`（最多输出 token），`stopSequences`（停止串），`temperature`，`topP` 等。([AWS ドキュメント][2])
* **模型专属页（例：Claude）**：同时列出 `temperature/top_p/top_k/max_tokens/stop_sequences`，并在响应中返回 `usage.input_tokens/output_tokens`，便于**成本监控**。([AWS ドキュメント][3])

---

## 5) 参数 → 输出效果「对照速查」

| 参数    | 日文（读法）        | English        | 降低取值的效果    | 提高取值的效果    |
| ----- | ------------- | -------------- | ---------- | ---------- |
| 温度    | 温度【おんど】       | Temperature    | 更保守、更一致    | 更多样、更有创意   |
| Top-K | トップK          | Top-K          | 候选更少，趋向高概率 | 候选更多，含低概率词 |
| Top-P | トップP          | Top-P          | 截断更严，趋向高概率 | 截断更松，更发散   |
| 最大出力長 | 最大トークン数【さいだい】 | max\_tokens    | 输出更短、成本更低  | 输出更长，成本更高  |
| 停止串   | 停止シーケンス       | Stop sequences | 更早停，避免越界   | 放宽停点，可能话多  |
| 惩罚    | ペナルティ         | Penalties      | 抑制重复/冗长    | （减弱惩罚）更自由  |

*依据：Bedrock 用户指南“随机性与多样性/长度”章节与 API 参数定义。* ([AWS ドキュメント][1])

---

## 6) 任务场景的「起步档位」建议（先跑通，再微调）

> 注：不同模型的默认与范围不同，请以**模型卡片/控制台**为准，再做微调。([AWS ドキュメント][10])

* **事实问答 / 企业检索（RAG）**

  * 温度 **0–0.2**；Top-P **0.8–0.95**（或不调，仅用温度）。
  * `max_tokens` 够用即可；使用 **Stop** 保持句子/JSON 封闭。
  * *理由*：稳定、可复现，减少幻觉；RAG 由检索保证多样性。([AWS ドキュメント][1])
* **要约/改写**

  * 温度 **0.1–0.4**；适当提高 `max_tokens`；必要时加**长度/重复惩罚**。([AWS ドキュメント][1])
* **代码生成/格式化 JSON**

  * 温度 **0–0.2**；配置 **Stop**（如 `\n\n` 或 `}`）；限制 `max_tokens` 避免“跑题”。([AWS ドキュメント][4])
* **创意文案/头脑风暴**

  * 温度 **0.7–1.0**；Top-P **0.9–1.0**；`max_tokens` 放宽。
  * *备注*：避免同时重度调整温度与 Top-P。([AWS ドキュメント][3])
* **长文/多模态输入**

  * 选择**长上下文**模型（如 Claude 长上下文/更长上下文 Beta）；注意**配额**与**计费**。([Amazon Web Services, Inc.][5], [AWS ドキュメント][7])
* **实时对话**

  * 结合 **ConverseStream** 流式返回，改善体感延迟；但**token 费用不变**。([Boto3][11])

---

## 7) 与“长度”的工程化联动（RAG & Chunk）

* **KB/RAG 的 Chunk/Top-K** 直接影响**拼接上下文**长度与**token 成本/延迟**；推荐**中等粒度 chunk（约 300–800 tokens）**与**小 Top-K**起步，再按召回率/命中质量调优。([AWS ドキュメント][12], [Amazon Web Services, Inc.][13])

---

## 8) 常见陷阱（考试 & 实战都高频）

1. **“温度高 + Top-P 大”** → 输出**发散**且**不稳定**；在 Claude 系列中**不建议**同时大幅调两者。([AWS ドキュメント][3])
2. **只加大 `max_tokens`** 企图“补齐答案”，但**上下文不足/错误**仍会导致幻觉；应先保证**检索命中**与**提示结构**。([AWS ドキュメント][1])
3. **上下文溢出**（Context Window Overflow）：输入超出模型窗口会**截断/异常**；务必**切分**与**估算 token**，并关注**配额/速率**。([Amazon Web Services, Inc.][14], [AWS ドキュメント][7])
4. **没有 Stop** 的 JSON/表格生成易“跑偏”；加**停止串**与**严格指示**可显著提升稳定性。([AWS ドキュメント][4])

---

## 9) 术语小词典（中・日・英）

* 推论参数：**推論パラメータ【すいろん…】** / Inference parameters ([AWS ドキュメント][1])
* 温度：**温度【おんど】（テンパレチャー）** / Temperature ([AWS ドキュメント][1])
* Top-K：**トップK** / Top-K ([AWS ドキュメント][1])
* Top-P（核采样）：**トップP（ニュークリアス）** / Top-P (Nucleus) ([AWS ドキュメント][1])
* 上下文窗口：**コンテキスト・ウィンドウ** / Context window ([Amazon Web Services, Inc.][5])
* 最大输出 token：**最大トークン数** / `max_tokens` / `maxTokens` ([AWS ドキュメント][4])
* 停止序列：**停止シーケンス** / Stop sequences ([AWS ドキュメント][4])
* 惩罚项：**ペナルティ**（长度/重复/频率 等）/ Penalties ([AWS ドキュメント][1])
* 流式响应：**ストリーミング** / ConverseStream ([Boto3][11])
* 统一调参接口：**Converse API**（`inferenceConfig`） ([AWS ドキュメント][8])

---

## 10) 一句话总括

> **调参顺序**：先**结构化提示**与**RAG 命中**，再用**温度/Top-P/Top-K**调“稳或活”；用 `max_tokens`、**Stop** 与**惩罚项**控长度与格式；**长上下文**选对模型并注意**配额**。所有取值以**Bedrock 用户指南/模型专属页**与 **Converse/InferenceConfiguration** 文档为准。([AWS ドキュメント][1])

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/inference-parameters.html "Influence response generation with inference parameters - Amazon Bedrock"
[2]: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_InferenceConfiguration.html?utm_source=chatgpt.com "InferenceConfiguration - Amazon Bedrock"
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-anthropic-claude-messages-request-response.html "Request and Response - Amazon Bedrock"
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference-call.html?utm_source=chatgpt.com "Using the Converse API - Amazon Bedrock"
[5]: https://aws.amazon.com/bedrock/anthropic/?utm_source=chatgpt.com "Anthropic's Claude in Amazon Bedrock"
[6]: https://docs.anthropic.com/en/docs/build-with-claude/context-windows?utm_source=chatgpt.com "Context windows"
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/quotas.html?utm_source=chatgpt.com "Quotas for Amazon Bedrock"
[8]: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html?utm_source=chatgpt.com "Converse - Amazon Bedrock"
[9]: https://boto3.amazonaws.com/v1/documentation/api/1.34.141/reference/services/bedrock-runtime/client/converse.html?utm_source=chatgpt.com "converse - Boto3 1.34.141 documentation"
[10]: https://docs.aws.amazon.com/bedrock/latest/userguide/foundation-models-reference.html?utm_source=chatgpt.com "Amazon Bedrock foundation model information"
[11]: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/bedrock-runtime/client/converse_stream.html?utm_source=chatgpt.com "converse_stream - Boto3 1.40.23 documentation"
[12]: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-chunking.html?utm_source=chatgpt.com "How content chunking works for knowledge bases"
[13]: https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-knowledge-bases-now-supports-advanced-parsing-chunking-and-query-reformulation-giving-greater-control-of-accuracy-in-rag-based-applications/?utm_source=chatgpt.com "Amazon Bedrock Knowledge Bases now supports ..."
[14]: https://aws.amazon.com/blogs/security/context-window-overflow-breaking-the-barrier/?utm_source=chatgpt.com "Context window overflow: Breaking the barrier"
