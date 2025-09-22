# AIF-C01｜3.4.2 **基盤模型性能的相关指标（ROUGE・BLEU・BERTScore 等）**／**基盤モデルのパフォーマンスを評価するための関連メトリクス**

*（中文为主；术语三语：中・日〔カタカナ读法〕・英）*

---

## 0) 本题要点（考试瞄准）

* 能**正确识别与解释**：**ROUGE**（ルージュ／*Recall-Oriented Understudy for Gisting Evaluation*）、**BLEU**（ブルー／*Bilingual Evaluation Understudy*）、**BERTScore**（バートスコア／*BERTScore*）等主流**文本生成**评估指标的**定义、适用场景、优缺点**与**常见误用**。
* 能**结合 AWS 官方**能力说明：在 **Amazon Bedrock Evaluations** 中如何使用**程序化（Programmatic）**与**LLM-as-a-Judge**评估；哪些指标内置、哪些需**自带/自定义**；以及在 **RAG**（ナレッジベース）场景的专用指标选择。([Amazon Web Services, Inc.][1])

---

## 1) 术语对照表（中・日〔读法〕・英）

| 指标                 | 日文（读法）         | English                                             | 核心思想                                             | 典型用途                                                 |
| ------------------ | -------------- | --------------------------------------------------- | ------------------------------------------------ | ---------------------------------------------------- |
| **ROUGE**          | ルージュ           | *Recall-Oriented Understudy for Gisting Evaluation* | **召回导向**的 n-gram/LCS **重叠度**；常见 **ROUGE-1/2/L**  | **摘要**（要涵盖尽量多的参考要点）([ACL Anthology][2])              |
| **BLEU**           | ブルー            | *Bilingual Evaluation Understudy*                   | **精确率导向**的 n-gram 匹配 + **简短惩罚（brevity penalty）** | **机器翻译/释义**（与参考译文接近）([ACL Anthology][3])             |
| **BERTScore**      | バートスコア         | *BERTScore*                                         | 基于 **上下文向量** 的词级相似度（P/R/F1）                      | **同义改写**丰富的生成任务；**摘要/翻译**增强版([arXiv][4])             |
| **METEOR**         | メテオール          | *METEOR*                                            | 词干/同义词匹配 + F-score                               | 翻译/摘要的补充（较贴近人工）([Amazon Web Services, Inc.][5])      |
| **chrF**           | シーエイチアールエフ     | *chrF*                                              | **字符 n-gram** 的 F-score                          | 跨语种/黏着语友好（如日语）                                       |
| **Exact Match/F1** | イグザクト・マッチ／エフワン | *Exact Match / F1*                                  | 与**真值文本**精确匹配或**词级**P/R/F1                       | **抽取式 QA、分类**（结构化任务）([Amazon Web Services, Inc.][1]) |

> **Bedrock Evaluations** 支持两路：
>
> 1. **Programmatic（程序化）**：提供传统算法与指标（如 **BERTScore、F1、EM** 等）；
> 2. **LLM-as-a-Judge**：由评审模型按“正确性、完整性、有害性”等**自定义标准**打分。([Amazon Web Services, Inc.][1])

---

## 2) 三大主角的“正确打开方式”

### 2.1 ROUGE（ルージュ／ROUGE）

* **定义**：比较生成文本与参考文本的 **n-gram 重叠** 或 **最长公共子序列（LCS）**；常用 **ROUGE-1**（Unigram）、**ROUGE-2**（Bigram）、**ROUGE-L**（LCS）。**偏召回**：覆盖到参考里的要点越多，分越高。([ACL Anthology][2])
* **适用**：**自动摘要**与**要点覆盖**场景（抽取式/生成式均可）。AWS 官方也在**摘要评估**实践中采用 ROUGE 系列。([Amazon Web Services, Inc.][5])
* **优点**：简单直观、社区基准广；对“**漏要点**”敏感（这是摘要的关键）。
* **局限**：对**同义/改写**不够鲁棒；高 ROUGE ≠ 一定事实正确；对**短答案**不友好（分母小）。
* **小例**：参考：“A,B,C 三点”。模型甲覆盖 A,B → **ROUGE-1/2 提高**；模型乙长篇发挥但漏 C → **ROUGE-L 下降**（未覆盖关键序列）。

### 2.2 BLEU（ブルー／BLEU）

* **定义**：**几何平均**的 n-gram **精确率**（Precision，常取 1–4-gram），再乘以**简短惩罚**防止输出过短。原生为**语料级**指标。([ACL Anthology][3])
* **适用**：**机器翻译/释义**需接近参考表达；句式、用词贴近参考时得分高。
* **优点**：历史最悠久的 MT 指标之一、可衡量“**过度简短**”问题（BP）。
* **局限**：**语料级**更稳定，**句级 BLEU**波动大；对**同义改写**不敏感；在**摘要**（偏召回）上不如 ROUGE 合拍。
* **小例**：参考：“I’m going home now.”

  * 输出 A：“I’m going home.” → n-gram 少一块且更短 → **BLEU 降 + BP 惩罚**；
  * 输出 B：“I am going home now.” → n-gram 多匹配 → **BLEU 升**。

### 2.3 BERTScore（バートスコア／BERTScore）

* **定义**：基于 **预训练语言模型（如 BERT/roberta）** 的**上下文向量**做词-词**余弦相似**，计算 **Precision / Recall / F1**；能识别**同义改写**、词序变化。([arXiv][4])
* **适用**：**自由生成**、**改写**、**抽象式摘要**、**跨语言/跨风格**；当你担心 ROUGE/BLEU **错杀同义表达**时，用它补充。
* **优点**：与人工相关性更高（多篇研究验证），能给**P/R/F1 三面**视角。([OpenReview][6])
* **局限**：对**所用嵌入模型**敏感；可能**偏爱冗长**（Recall 高）；**跨语种**需挑选恰当多语模型。
* **小例**：参考：“患者出现发热与咳嗽，应评估肺部感染可能。”

  * 输出甲：“患者有发热、咳嗽，考虑肺炎可能。”→ **BERTScore 高**（语义近），**ROUGE/BLEU 中等**（字面差异大）。

---

## 3) “选什么指标”：任务 × 指标 速配表（含 AWS 参考）

| 任务类型                  | 首选指标                          | 备选/补充                     | 说明                                                                                                 |
| --------------------- | ----------------------------- | ------------------------- | -------------------------------------------------------------------------------------------------- |
| **摘要（Summarization）** | **ROUGE-1/2/L**（召回导向）         | **BERTScore**、METEOR      | AWS 官方摘要评估文章示例使用 **ROUGE、METEOR、BERTScore**。([Amazon Web Services, Inc.][5])                       |
| **机器翻译/释义**           | **BLEU**（精确率导向）               | **BERTScore、chrF、METEOR** | BLEU 做主，BERTScore/chrF 捕捉同义/字符级变化。([ACL Anthology][3])                                             |
| **开放式问答（生成）**         | **BERTScore**、LLM-as-Judge 指标 | ROUGE/BLEU（参考）            | 正确性常需**人工/LLM 评审**（有用性/真实性/有害性）。([AWS Documentation][7])                                           |
| **抽取式 QA/信息抽取**       | **Exact Match、F1**            | —                         | **Programmatic**在 Bedrock 支持 **F1/EM** 等传统指标。([Amazon Web Services, Inc.][1])                      |
| **RAG（检索+生成）**        | **RAG 专用内置指标**                | 自定义指标 + 传统指标              | 在 Bedrock 里分 **Retrieve-only** 与 **Retrieve-and-Generate** 两类评估，指标内置且可自定义。([AWS Documentation][8]) |

---

## 4) 在 Amazon Bedrock 里，怎么“落地这些指标”

### 4.1 两种评估模式

* **Programmatic（程序化）**：可直接使用**传统算法**与指标（官方页面示例：**BERTScore、F1、EM** 等），也能用**你自带的基准数据集**。([Amazon Web Services, Inc.][1])
* **LLM-as-a-Judge（评审模型）**：挑选评审模型，按**正确性、完整性、有害性**或**自定义准则**打分，适合**主观维度**与**事实核查**。([AWS Documentation][7])

> **ROUGE/BLEU 如何用？**
> ① 直接在**程序化**流程里作为传统算法（若服务端未列内置，可在**自定义指标**里实现/接入）；② 也可在 **SageMaker / HF evaluate** 线下算分，再把结果并入 **Bedrock 评估对比**。AWS 官方博客在**摘要与 RAG**评估中示例性使用了 **ROUGE、BLEU、BERTScore**。([Amazon Web Services, Inc.][5])

### 4.2 RAG 专用（补充）

* **Retrieve-only / Retrieve-and-Generate** 两类作业、**内置指标集**与**自定义指标**入口，均由官方文档定义；数据集 **JSONL** 且有**条数上限**（≤1000）。([AWS Documentation][8])

---

## 5) 轻公式 & 常见坑（超高频考点）

### 5.1 BLEU（精确率几何均值 × 简短惩罚）

* **核心**：
  \- $\text{BLEU} = \text{BP} \cdot \exp\left(\sum_{n=1}^N w_n \log p_n\right)$
  \- $p_n$：n-gram **精确率**；$\text{BP}$：**brevity penalty**。
* **坑**：

  1. **句级不稳定**：尽量用**语料级**；
  2. **过短输出**会被 **BP** 惩罚；
  3. **不识别同义** → 搭配 **BERTScore** 看**语义**。([ACL Anthology][3])

### 5.2 ROUGE（召回导向）

* **常用**：ROUGE-1（unigram）、ROUGE-2（bigram）、ROUGE-L（LCS）。
* **坑**：

  1. **冗词**也能抬高召回；
  2. **事实错误**不一定被惩罚；
  3. **中文/日语**需注意**分词/字符级**配置（可考虑 **ROUGE-L** 或 **chrF** 辅助）。([ACL Anthology][2])

### 5.3 BERTScore（语义相似）

* **输出**：P/R/F1（建议报告 **F1** 并附 **R** 用于召回敏感任务如摘要）。
* **坑**：

  1. **模型选择**影响分数（英文 vs 多语模型）；
  2. **过长输出**可能抬高 Recall；
  3. 请**固定版本/权重**保证可复现。([arXiv][4])

---

## 6) 三个“带手感”的微型案例

1. **中文新闻摘要评测**

   * **做法**：基准集含“标题+人写摘要”；报告 **ROUGE-1/2/L**（覆盖主谓宾/关键短语）+ **BERTScore-F1**（同义补偿）。
   * **解读**：A 模型 ROUGE 高但 BERTScore 低 → 更像“摘抄”但缺语义顺滑；B 模型相反 → 更像“改写”，要复核事实性。
   * **AWS 参考**：官方博客示例使用 **ROUGE、METEOR、BERTScore** 评摘要。([Amazon Web Services, Inc.][5])

2. **英⇄日 MT 对比**

   * **做法**：**BLEU-4**（精确率+BP）为主，**BERTScore** 与 **chrF** 为辅（照顾同义与假名/汉字形态）。
   * **解读**：BLEU 接近但 chrF 差异大 → 字符层面错字较多；BERTScore 更能揭示语义错位。([ACL Anthology][3])

3. **企业 RAG 文档问答**

   * **做法**：跑 **RAG Retrieve-and-Generate** 评估（内置指标 + 自定义“是否引用到正确片段”）。
   * **补充**：离线再测 **ROUGE/BERTScore** 观察“语言表述质量”。([AWS Documentation][8])

---

## 7) 实操清单（在 AWS 上交付）

* **选模式**：

  * 结构化任务（抽取/分类）→ **Programmatic：EM/F1**；
  * 摘要/翻译/生成 → **ROUGE/BLEU/BERTScore + LLM-Judge 合规性**；
  * 知识库 → **RAG 专用评估** +（可选）传统文本指标补充。([Amazon Web Services, Inc.][1])
* **准备数据**：**Prompt Dataset**（JSONL）；RAG 评估 **≤1000** 条。([AWS Documentation][9])
* **创建作业**：控制台/CLI/API（`CreateEvaluationJob`），可带**自定义指标**提示词与评分标准。([AWS Documentation][7])
* **读报告**：查看**自动评估报告**与**人评报告卡**（S3 同步原始结果）。([AWS Documentation][10])

---

## 8) 备忘卡（一句话记忆）

* **摘要看 ROUGE（召回），翻译看 BLEU（精确），改写看 BERTScore（语义）**；
* **结构化任务**用 **EM/F1**，**主观/合规**交给 **LLM-Judge**；
* **RAG**用**官方内置 RAG 指标**，必要时加 ROUGE/BERTScore 做语言质量补充。([Amazon Web Services, Inc.][1])

---

## 参考（官方优先）

* **Bedrock Evaluations 产品页**：Programmatic（BERTScore、F1、EM 等）与 LLM-as-a-Judge 两路能力。([Amazon Web Services, Inc.][1])
* **Model Evaluation（LLM-as-Judge 内置指标表、自定义指标）**。([AWS Documentation][7])
* **RAG 评估指标与作业**（Retrieve-only / Retrieve-and-Generate、数据上限与格式）。([AWS Documentation][8])
* **AWS 博客：摘要评估用 ROUGE/METEOR/BERTScore**。([Amazon Web Services, Inc.][5])
* **AWS 博客：评估 RAG（包含 ROUGE/BLEU/BERTScore 的实践提及）**。([Amazon Web Services, Inc.][11])
* **原始论文**：BLEU（Papineni+，2002）、ROUGE（Lin，2004）、BERTScore（Zhang+，2019）。([ACL Anthology][3])

---

### 收尾一句

> **没有万能指标，只有合适组合。**在 AWS 上，把 **Programmatic**（ROUGE/BLEU/BERTScore/EM/F1）与 **LLM-Judge**（正确性/完整性/有害性）结合，再配 **RAG 专用评估**，你就能对**不同任务**和**真实业务目标**给出**可复现、可比较、可落地**的评测结论。([Amazon Web Services, Inc.][1])

[1]: https://aws.amazon.com/bedrock/evaluations/?utm_source=chatgpt.com "Evaluate Foundation Models - Amazon Bedrock Evaluations"
[2]: https://aclanthology.org/W04-1013.pdf?utm_source=chatgpt.com "ROUGE: A Package for Automatic Evaluation of Summaries"
[3]: https://aclanthology.org/P02-1040.pdf?utm_source=chatgpt.com "BLEU: a Method for Automatic Evaluation of Machine ..."
[4]: https://arxiv.org/abs/1904.09675?utm_source=chatgpt.com "BERTScore: Evaluating Text Generation with BERT"
[5]: https://aws.amazon.com/blogs/machine-learning/evaluate-the-text-summarization-capabilities-of-llms-for-enhanced-decision-making-on-aws/?utm_source=chatgpt.com "Evaluate the text summarization capabilities of LLMs for ... - AWS"
[6]: https://openreview.net/forum?id=SkeHuCVFDr&utm_source=chatgpt.com "BERTScore: Evaluating Text Generation with BERT"
[7]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-evaluation-metrics.html?utm_source=chatgpt.com "Use metrics to understand model performance"
[8]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-evaluation-metrics.html?utm_source=chatgpt.com "Use metrics to understand RAG system performance"
[9]: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-evaluation-create-randg.html?utm_source=chatgpt.com "Creating a retrieve-and-generate RAG evaluation job"
[10]: https://docs.aws.amazon.com/bedrock/latest/userguide/model-evaluation-report-programmatic.html?utm_source=chatgpt.com "Review metrics for an automated model evaluation job in ..."
[11]: https://aws.amazon.com/blogs/machine-learning/evaluate-rag-responses-with-amazon-bedrock-llamaindex-and-ragas/?utm_source=chatgpt.com "Evaluate RAG responses with Amazon Bedrock, LlamaIndex ..."
