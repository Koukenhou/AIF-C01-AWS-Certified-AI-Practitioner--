# AIF-C01｜3.1.1 事前トレーニング済みモデルの**選択基準**

*AIF-C01 Domain 3, Task 3.1.1 — Criteria for selecting pre-trained (foundation) models*

> 目标：面向“**先选对模型，再谈定制**”，明确 **成本、模态、延迟、多语言、模型大小与复杂度、可定制性、输入/输出长度** 等关键维度；给出**三语对照**与**AWS 对应做法**（以 Bedrock 为主）。

---

## 0) 总览：一页速记（看到题干即可套用）

| 维度     | 日文（读法）     | English                 | 关键判断                                      | AWS 对应/依据                                                                                           |
| ------ | ---------- | ----------------------- | ----------------------------------------- | --------------------------------------------------------------------------------------------------- |
| 成本     | コスト        | Cost                    | 计费按**输入/输出 token**；高并发/稳定 SLA 可能需**预留吞吐** | Bedrock 价格=token 计费；可选 **Provisioned Throughput**。([AWS ドキュメント][1], [Amazon Web Services, Inc.][2]) |
| 模态     | モダリティ      | Modality                | 文本/图像/音频/视频/多模态                           | 选支持相应能力的 FM（见 **Model Catalog**；Nova 为多模态示例）。([AWS ドキュメント][3])                                      |
| 延迟     | レイテンシー     | Latency                 | 对话/交互要低延迟；批处理可走异步/批量                      | Bedrock/SageMaker 支持 **Realtime/Async/Batch/Serverless**；强 SLA 可用 **Provisioned**。([AWS ドキュメント][4]) |
| 多语言    | 多言語【たげんご】  | Multilingual            | 是否覆盖目标语言/脚本                               | 参考 **Model Catalog** 与各模型卡片说明。([AWS ドキュメント][3])                                                     |
| 模型大小   | モデルサイズ     | Model size              | 越大越强但成本/延迟高；小模型+RAG 常更划算                  | 通过 **KB/RAG** 用私有知识补能，降低对大模型依赖。([AWS ドキュメント][5])                                                    |
| 复杂度    | 複雑さ【ふくざつさ】 | Complexity              | 是否需要多步推理/工具调用/多模态对齐                       | 需要“会办事”→ **Agents**；需要安全→ **Guardrails**。([Cloudforecast][6], [AWS ドキュメント][7])                      |
| 可定制性   | カスタマイズ     | Customization           | 先提示工程/RAG；不够再 **微调/继续预训练**                | Bedrock 支持 **Fine-tuning/Continued pre-training**。([AWS ドキュメント][8])                                 |
| 上下文&输出 | 入力/出力の長さ   | Context & Output length | 关注**上下文窗口**、**max\_tokens** 与采样参数         | 推理参数（temperature/top\_p/max\_tokens）官方指南。([AWS ドキュメント][9])                                          |
| 区域供给   | リージョン      | Region availability     | 模型并非所有 Region 都可用                         | 以 **Supported models** 页面与控制台为准。([AWS ドキュメント][3])                                                   |

---

## 1) 成本（Cost｜コスト）

* **中文**：Bedrock 按**输入+输出 token**计费；若购买 **预留吞吐（Provisioned Throughput）**，则按小时计费，换来**更稳定的容量/延迟**。
* **日/读**：コスト｜**トークン課金**【とか】／**プロビジョンドスループット**
* **EN**：Token-based pricing; Provisioned Throughput
* **要点**：

  1. 文本越长/Top-K 越多 → 上下文变长 → **token 成本**上升。
  2. 低频/突发流量优先 **On-demand/Serverless/Async**；**持续高并发**或强 SLA 才考虑 **Provisioned**。
  3. 先用**小模型 + RAG**，往往比“大模型硬吃上下文”更省钱。
* **依据**：Bedrock 定价（token 计费）；Provisioned Throughput 说明（自定义模型**必须**配预留吞吐才能调用）。([AWS ドキュメント][1], [Amazon Web Services, Inc.][2])

---

## 2) 模态（Modality｜モダリティ）

* **中文**：按任务选模型能力：**文本**（QA/摘要/代码）、**图像**（生成/理解）、**音频**（ASR/TTS/对话）、**视频**（生成/理解）、**多模态**（图文音视频一体）。
* **日/读**：モダリティ｜**テキスト／画像／音声／動画／マルチモーダル**
* **EN**：Text / Image / Audio / Video / Multimodal
* **要点**：模型卡会标注能力；多模态如 **Amazon Nova** 覆盖文本、图像、视频、语音、API 调用与智能体。
* **依据**：**Supported models**（列出能力/Region/文档）；**Amazon Nova** 概览。([AWS ドキュメント][3])

---

## 3) 延迟（Latency｜レイテンシー）

* **中文**：对话式应用要求**低延迟**；长文本摘要/视频生成可接受**异步/批处理**。强 SLA 场景可用 **Provisioned Throughput** 把抖动降到最小。
* **日/读**：レイテンシー／**リアルタイム／非同期／バッチ**
* **EN**：Realtime / Asynchronous / Batch; Provisioned Throughput
* **依据**：Bedrock 预留吞吐（提升吞吐/稳定性/容量）。([AWS ドキュメント][4])

---

## 4) 多语言（Multilingual｜多言語【たげんご】）

* **中文**：确认模型的**语言覆盖**（字符集、领域术语、输入/输出方向）；用**术语表**或 **RAG** 提供企业术语。
* **EN**：Multilingual support
* **依据**：各模型在 **Supported models** 页面及其文档卡中声明语言能力。([AWS ドキュメント][3])

---

## 5) 模型大小与复杂度（Model size & Complexity｜モデルサイズ／複雑さ）

* **中文**：**更大**往往更强但**更贵更慢**；**更小**配合 **RAG/Prompt** 常能达到“**够用**且**可控**”的平衡。若需要**工具调用/工作流/多步推理**，优先选择\*\*支持智能体（Agents）\*\*的组合。
* **日/读**：モデルサイズ／**エージェント**
* **依据**：**Agents** 与 **Guardrails** 组件用于编排与安全治理，降低“复杂度外溢”。([Cloudforecast][6], [AWS ドキュメント][7])

---

## 6) 可定制性（Customization｜カスタマイズ）

* **中文**：优先级 **Prompt → RAG（KB）→ Fine-tuning → Continued Pre-training**。能用提示与 RAG 解决的尽量不微调，成本/风险最低；确需定制时，优先 **LoRA/参数高效微调**。
* **日/读**：**ファインチューニング／継続事前学習**
* **EN**：Fine-tuning; Continued Pre-training
* **依据**：Bedrock 支持**微调**与**继续预训练**流程与作业提交。([AWS ドキュメント][8])

---

## 7) 输入/输出长度（Context & Output｜入力/出力の長さ）

* **中文**：关注模型的**上下文窗口**与**max\_tokens**；通过**分块（Chunking）**与**Top-K**控制拼接上下文长度；用**推理参数**（temperature/top\_p/max\_tokens）调节稳定性与成本。
* **日/读**：**コンテキストウィンドウ／マックス トークン**
* **EN**：Context window; max\_tokens; inference params
* **依据**：推理参数官方指南与各模型参数页。([AWS ドキュメント][9])

---

## 8) 区域可用性（Region availability｜リージョン）

* **中文**：并非所有模型在所有 Region 提供；若受**数据主权/合规**约束，需要在指定 Region 选择**等价能力**模型或调整架构。
* **依据**：**Supported models** 列出每个模型的**可用区域**与**能力**。([AWS ドキュメント][3])

---

## 9) 结合 AWS：选型→落地的“4 步流”

1. **对齐任务与非功能**（模态/延迟/语言/合规/成本上限）。
2. **候选模型**（Model Catalog）对比：能力、Region、价格、上下文、是否支持定制。([AWS ドキュメント][3])
3. **先做原型**：用 **Bedrock Playground** 调提示与参数；若需知识 → 上 **Knowledge Bases（RAG）**；若需办事 → 加 **Agents**；全程 **Guardrails**。([AWS ドキュメント][5], [Cloudforecast][6])
4. **算账与收尾**：按 token 成本与并发峰值评估是否需要 **Provisioned Throughput**；若要定制 → 评估 **Fine-tuning/Continued pre-training** 的训练与托管成本。([Amazon Web Services, Inc.][2], [AWS ドキュメント][4])

---

## 10) 选择题“秒选”提示（关键词 → 答案）

* **“多模态（图/文/音/视）”** → 选**多模态 FM**（如 Nova 家族）或具备相应能力的模型。([AWS ドキュメント][10])
* **“强 SLA/高并发/稳定延迟”** → **Provisioned Throughput**；但成本↑。([AWS ドキュメント][4])
* **“成本爆表/上下文太长”** → 缩 Prompt；用 **RAG（KB）** 精准拼接；控制 **max\_tokens/Top-K**。([AWS ドキュメント][5])
* **“需要企业术语/风格”** → 先 **RAG**；不够再 **Fine-tuning**。([AWS ドキュメント][5])
* **“模型不在该 Region”** → 查 **Supported models**，替换等价模型或调整 Region。([AWS ドキュメント][3])

---

## 11) 术语小词典（中・日・英）

* 基盘模型：**基盤モデル【きばんモデル】** / Foundation Model (FM)
* 预留吞吐：**プロビジョンドスループット** / Provisioned Throughput
* 知识库/检索增强：**ナレッジベース（RAG）** / Knowledge Bases (RAG)
* 智能体：**エージェント** / Agents
* 守护栏：**ガードレール** / Guardrails
* 上下文窗口：**コンテキストウィンドウ** / Context window
* 采样参数：**temperature / top\_p / max\_tokens**（推論パラメータ）

---

### 一句话总括

> **选模型 = 任务与非功能匹配**：先看**模态/延迟/多语言/Region**，再算**token 成本**与是否要**预留吞吐**；能用**Prompt+RAG**解决的就别急着微调；若必须定制则评估 **Fine-tuning/Continued pre-training** 的训练与托管成本。上述信息以 **Model Catalog / Pricing / Provisioned Throughput / KB / Guardrails / Inference Params** 官方文档为准。([AWS ドキュメント][3])

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/bedrock-pricing.html?utm_source=chatgpt.com "Amazon Bedrock pricing"
[2]: https://aws.amazon.com/bedrock/pricing/?utm_source=chatgpt.com "Amazon Bedrock pricing"
[3]: https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported.html?utm_source=chatgpt.com "Supported foundation models in Amazon Bedrock"
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/prov-throughput.html?utm_source=chatgpt.com "Increase model invocation capacity with Provisioned ..."
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-it-works.html?utm_source=chatgpt.com "How Amazon Bedrock knowledge bases work"
[6]: https://www.cloudforecast.io/blog/aws-bedrock-pricing/?utm_source=chatgpt.com "A Comprehensive Guide to AWS Bedrock Pricing"
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[8]: https://docs.aws.amazon.com/bedrock/latest/userguide/custom-model-fine-tuning.html?utm_source=chatgpt.com "Customize a model with fine-tuning or continued pre- ..."
[9]: https://docs.aws.amazon.com/bedrock/latest/userguide/inference-parameters.html?utm_source=chatgpt.com "Influence response generation with inference parameters"
[10]: https://docs.aws.amazon.com/nova/latest/userguide/what-is-nova.html?utm_source=chatgpt.com "What is Amazon Nova?"
