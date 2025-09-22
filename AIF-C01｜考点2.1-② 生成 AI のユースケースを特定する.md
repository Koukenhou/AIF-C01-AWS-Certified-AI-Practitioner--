# AIF-C01｜2.1-② 生成 AI のユースケースを特定する（中・日・英对照，日语含读法）

> 目的：看见一个业务场景，能**快速判断**是否属于“生成式 AI”的常见用例，并说出**输入/输出、关键难点、在 AWS 上的常见实现路径**。
> 术语均含 **日文写法 +【读音】+ English**，讲解以中文为主。

---

## 0) 一页速览（背会就能拿分）

* **文本类**：要约【ようやく】/Summarization、チャットボット/Chatbot、翻訳【ほんやく】/Translation、コード生成【せいせい】/Code Generation。
* **媒体类**：画像生成【がぞう せいせい】/Image Gen、動画生成【どうが せいせい】/Video Gen、音声生成【おんせい せいせい】/Audio Gen（TTS/Voice）。
* **企业场景**：カスタマーサービスエージェント/Customer Service Agent、検索【けんさく】/Search（RAG）、レコメンデーションエンジン/Recommendation Engine。
* **答题模板**：**输入 → 输出 → 价值 → 风险/治理（Guardrails）→ AWS 组合**（Bedrock/KB/Kendra/Lex/Polly/Transcribe/Translate/Personalize…）。

---

## 1) 画像生成【がぞう せいせい】｜Image Generation

**EN**: Image Generation
**输入/输出**：文本提示（Prompt）/参考图 → 生成图片（产品图、广告图、示意图）。
**价值**：快速出图、A/B 版本探索、成本低于纯人力。
**风险**：版权/风格合规、人物/商标、Prompt 注入。需 **Guardrails** 与使用政策。
**AWS 常见实现**：

* **Amazon Bedrock**（选择图像生成 FM，如 Titan Image 等）
* **Guardrails**（敏感内容过滤/PII 遮罩）
* **Serverless 推理**（低频按需）
  **考试关键词**：*text-to-image / diffusion / style / safety*。

---

## 2) 動画生成【どうが せいせい】｜Video Generation

**EN**: Video Generation
**输入/输出**：脚本/分镜/参考图 → 短视频/过场动画。
**价值**：营销短片、教程 DEMO、低成本反复迭代。
**难点**：时序一致性、长时长成本高、审核与版权。
**AWS 常见实现**：以 **Bedrock**（若支持相关多模态/视频 FM）为核心，或结合第三方；存储于 **S3**，转码 **MediaConvert**。
**考试关键词**：*text-to-video / storyboard / temporal consistency*。

---

## 3) 音声の生成【おんせい の せいせい】｜Audio Generation（含 TTS）

**EN**: Audio/Voice Generation (TTS)
**输入/输出**：文本/角色设定 → 自然语音；或语音风格化。
**价值**：旁白、读屏无障碍、IVR 语音。
**AWS 常见实现**：

* **Amazon Polly**（Text-to-Speech）
* 搭配 **Transcribe**（ASR，语音转文字）形成**听说双向**
* 用 **Lex** 做语音对话入口
  **考试关键词**：*TTS / voice / speech synthesis*。

---

## 4) 要約【ようやく】｜Summarization

**EN**: Summarization
**输入/输出**：长文/会议纪要/技术报告 → 摘要（要点/行动项/风格约束）。
**价值**：缓解信息过载、提升阅读效率。
**难点**：事实性与引用（避免“幻觉”）。
**AWS 常见实现**：

* **Bedrock LLM** + **Knowledge Bases（RAG）**（把来源段落并入提示，附可点击引用）
* 辅助 **Comprehend**（关键词/实体抽取）
  **考试关键词**：*document summary / extract key points / with citations (RAG)*。

---

## 5) チャットボット｜Chatbot

**EN**: Chatbot
**输入/输出**：用户自然语言 → 多轮对话应答/办理流程。
**价值**：7×24 小时服务、缓解人工压力。
**AWS 常见实现**：

* **Lex**（意图/槽位/对话编排）+ **Bedrock LLM**（生成更自然的回答）
* 接 **Knowledge Bases / Kendra** 做企业文档问答
* 需要**Guardrails** 控制话题与敏感信息
  **考试关键词**：*intent / slot / multi-turn / FAQ / KB / guardrails*。

---

## 6) 翻訳【ほんやく】｜Translation

**EN**: Translation
**输入/输出**：源语言文本 → 目标语言文本（带术语表/风格要求）。
**价值**：跨语言沟通、国际化站点。
**AWS 常见实现**：

* **Amazon Translate**（机器翻译，支持自定义术语）
* 长文+上下文一致性 → 可用 **Bedrock LLM** 结合**术语表/RAG**
  **考试关键词**：*MT / glossary / domain adaptation*。

---

## 7) コード生成【せいせい】｜Code Generation

**EN**: Code Generation
**输入/输出**：需求/注释/上下文代码 → 代码片段/单元测试/重构建议。
**价值**：提高开发效率、减少模板性编码。
**风险**：许可证、依赖漏洞、准确性需要**人类审查**。
**AWS 常见实现**：

* **Amazon Q（Developer）/ CodeWhisperer**（AI 编码助手、建议与安全扫描）
* **Bedrock**（通用 LLM 生成脚本、SQL、Glue 作业）
  **考试关键词**：*code completion / unit tests / human-in-the-loop / security scan*。

---

## 8) カスタマーサービスエージェント｜Customer Service Agent

**EN**: Customer Service Agent（智能体/工作流）
**输入/输出**：用户请求 → 查知识 → 填单/调用 API → 回复并**执行动作**。
**价值**：把“答问题”升级为“**能办事**”（建工单、改预约、发送邮件）。
**AWS 常见实现**：

* **Bedrock Agents**（工具调用/函数调用/对话状态管理）
* 搭配 **Knowledge Bases**（检索公司政策/流程）
* 与 **Amazon Connect** 联动（联络中心）
  **考试关键词**：*agent / tool use / function calling / workflow / Connect*。

---

## 9) 検索【けんさく】｜Search（RAG 强化）

**EN**: Search (with RAG)
**输入/输出**：自然语言查询 → 最相关的文档片段 + 生成式摘要/答案。
**价值**：**语义级**查找；给可点击引用，可信度高。
**AWS 常见实现**：

* **Kendra**（企业搜索）或 **Knowledge Bases**（托管 RAG 管道：分块→嵌入→向量检索）
* **Bedrock LLM** 负责“读结果并生成回答”。
  **考试关键词**：*semantic search / embeddings / vector / citations*。

---

## 10) レコメンデーションエンジン｜Recommendation Engine

**EN**: Recommendation Engine
**输入/输出**：用户/物品画像 + 行为 → 推荐列表/解释。
**价值**：提升点击率、转化率与留存。
**生成式增强**：

* 传统个性化（**Amazon Personalize**）产出候选 → **LLM 生成理由/文案**，或让模型**基于向量相似度**生成“相似说明”。
  **考试关键词**：*personalization / ranking / explainable recommendation*。

---

## 11) 选择题“秒选”口诀（看到词就能归类）

* **看图出图** → 画像生成（Image Gen）
* **短片动效** → 動画生成（Video Gen）
* **读文成语音** → 音声生成（TTS/Polly）
* **长文要点** → 要約（Summarization + RAG）
* **多轮问答** → チャットボット（Lex + KB + Guardrails）
* **跨语言** → 翻訳（Translate / LLM+术语）
* **写代码** → コード生成（CodeWhisperer/Bedrock）
* **能做事** → カスタマーサービスエージェント（Agents）
* **找文档** → 検索（Kendra/KB + LLM）
* **猜你喜欢** → レコメンデーション（Personalize + LLM 解释）

---

## 12) 推理形态与成本（实战要点）

* **リアルタイム**：聊天/翻译/客服；**低延迟**优先。
* **バッチ**：大批量摘要/生成图；**夜间离线**省成本。
* **非同期**：长任务（视频生成/大图像处理）。
* **サーバレス**：**低频/突发**调用最省钱（有冷启动，模型越大等待越久）。

---

## 13) 治理与合规（每个用例都要想）

* **Guardrails**：敏感话题、PII 过滤、越权/越界防护。
* **引用与可追溯**：优先 **RAG + citation**，降低“幻觉”。
* **人类在回路（HITL）**：代码生成、客服关键动作务必二次确认。
* **数据与版权**：图片/视频/语音生成需审查素材与用途；商业落地看授权。

---

### 术语小词典（中・日・英）

* 要约：要約【ようやく】/ Summarization
* 翻译：翻訳【ほんやく】/ Translation
* 对话机器人：チャットボット / Chatbot
* 智能体：エージェント / Agent
* 搜索：検索【けんさく】/ Search
* 推荐引擎：レコメンデーションエンジン / Recommendation Engine
* 画像/動画/音声 生成：Image/Video/Audio Generation（Polly=语音合成）
* 检索增强生成：検索拡張生成 / Retrieval-Augmented Generation (RAG)

---

**一口气总括**：

> **文本类（要约/对话/翻译/代码）**，**媒体类（图/视频/语音）**，和**企业类（Agent/搜索/推荐）**构成生成式 AI 的主战场。
> 在 AWS：优先考虑 **Bedrock + Knowledge Bases/Kendra + Guardrails**，必要时配 **Lex/Polly/Transcribe/Personalize**，并按**实时/批/异步/无服**选择最省钱的推理形态。
