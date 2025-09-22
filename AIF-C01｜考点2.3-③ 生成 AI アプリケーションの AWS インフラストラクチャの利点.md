# AIF-C01｜2.3.3 生成 AI アプリの**AWS インフラ利点**

*Benefits of AWS infrastructure for generative-AI apps（安全性・コンプライアンス・責任・セーフティ）*

> 目标：能明确说明在 **AWS** 上构建生成式 AI 应用时，基础设施在**安全（Security）**、**合规（Compliance）**、**责任分界（Shared Responsibility）**、\*\*模型安全/内容安全（Safety）\*\*方面的优势，并能把这些优势与具体服务/文档对上号。术语均含 **中・日（含读法）・英**。

---

## 1) セキュリティ【Security｜安全性】

**要点**：**私网访问 + 加密 + 细粒度权限 + 审计与监控**，并覆盖到 Bedrock 的 RAG/Agent 等关键链路。

* **私网访问（VPC/PrivateLink）**｜**VPC エンドポイント／プライベートリンク**｜**VPC endpoints / AWS PrivateLink**
  通过 **Interface VPC Endpoint** 私网直连 **Amazon Bedrock**，无需公网 IP、NAT、VPN 或 DX，即可在 VPC 内安全调用模型与 Agent 组件。([AWS ドキュメント][1])

* **传输与静态加密**｜**通信・保存時の暗号化**｜**Encryption in transit & at rest**
  Bedrock 要求 **TLS 1.2+** 的安全连接；静态数据与中转数据默认加密，并可自选 **KMS** 自管密钥（CMK），**Knowledge Bases** 相关资源与评估作业亦支持 KMS。([AWS ドキュメント][2])

* **数据湖/对象存储默认加密**｜**S3 既定の暗号化**｜**Default S3 encryption**
  所有新上传到 **Amazon S3** 的对象默认静态加密（SSE-S3），可改为 **SSE-KMS** 并通过 **S3 Bucket Keys** 优化成本。([AWS ドキュメント][3])

* **身份与权限**｜**IAM ポリシー**｜**IAM identity-based policies**
  以 **最小权限** 控制谁能调用哪些模型/资源、在何条件下访问；Bedrock 明确支持基于身份的访问控制。([Amazon Web Services, Inc.][4])

* **日志审计与可观测性**｜**監査ログ／モニタリング**｜**CloudTrail & CloudWatch**
  Bedrock 与 **CloudTrail** 集成，所有控制台与 API 调用会被记录；可用 **CloudWatch** 追踪使用量与自定义仪表盘满足审计。([AWS ドキュメント][5], [Amazon Web Services, Inc.][4])
  *（扩展：可细化记录数据事件，但需额外开启与计费。）* ([AWS ドキュメント][6])

---

## 2) コンプライアンス【Compliance｜合规】

**要点**：AWS 提供广泛的合规对齐与佐证文档；你在其之上配置控制来满足行业/地域要求。

* **合规计划与在范围服务**｜**コンプライアンス・プログラム／サービス適用範囲**｜**Compliance programs & services-in-scope**
  AWS 支持众多国际/行业合规（如 **GDPR、HIPAA、PCI-DSS、FedRAMP** 等）；你可查询各服务的“**在范围（in scope）**”列表与对齐文档。([Amazon Web Services, Inc.][7])

* **合规的实现方式**：
  AWS 提供**治理友好**的功能（KMS、IAM、CloudTrail、私网连接、区域选择等），帮助你在云上落实自身的合规控制。([Amazon Web Services, Inc.][8])

---

## 3) 責任【Shared Responsibility｜责任分界】

**要点**：**云上安全是“共同责任”**。

* **提供方（AWS）**：物理设施、网络、虚拟化层、托管服务平台的安全与合规；
* **客户**：数据、身份/访问、网络分段、加密密钥、应用/提示治理、合规配置等。
  参考 **共享责任模型**白皮书与 Well-Architected 安全支柱文档。([Amazon Web Services, Inc.][9], [AWS ドキュメント][10])

> 记忆点：**“AWS 管底座，你管数据与配置。”**

---

## 4) セーフティ（AI Safety）【模型/内容安全】

**要点**：**Guardrails** 把安全策略“平台化”，跨模型一致生效；与审计、加密、私网、权限形成合力。

* **Guardrails for Amazon Bedrock**｜**ガードレール**｜**Configurable safeguards**
  可针对**提示与回复**配置不当内容过滤、PII 识别/遮罩、话题限制、提示注入防护等，并可跨**多家 FM** 复用统一策略（也可通过 API 独立调用）。([AWS ドキュメント][11], [Amazon Web Services, Inc.][12])

* **数据使用与隐私承诺**
  官方 FAQ/安全页面明确：**Bedrock 不会将你的输入/输出用于训练 Amazon 或第三方模型**；模型定制数据经**加密**与**私网**传输，仅用于你的用例，不共享给第三方。([Amazon Web Services, Inc.][13], [AWS ドキュメント][14])

---

## 5) 生成式 AI 专属链路的“端到端”安全闭环

> 从“调用模型”到“RAG/Agent 执行”的每一段，都有官方文档对应的安全/合规支撑。

1. **调用路径**：VPC Endpoint/PrivateLink → TLS → IAM 细粒度授权 → CloudTrail 记录。([AWS ドキュメント][1])
2. **知识检索（KB/RAG）**：文档入湖与分块/嵌入/向量存储过程加密（含 Bedrock 管理的向量库与你自建的 OpenSearch/Aurora pgvector 等）。([AWS ドキュメント][15])
3. **Agent 工具调用**：可用 PrivateLink 连接 **AgentCore** 与后端服务，并由 IAM/Secrets/KMS 控制密钥与凭据的解密与最小授权。([AWS ドキュメント][16])
4. **安全策略**：Guardrails 统一过滤/限制；CloudWatch 监控毒性/拒答/延迟指标。([Amazon Web Services, Inc.][12])

---

## 6) 术语速查（中・日・英）

* 私有连接：**プライベートリンク**／VPC **エンドポイント**｜*AWS PrivateLink / VPC endpoint* ([AWS ドキュメント][1])
* 加密密钥：**KMS キー**｜*AWS Key Management Service (KMS)* ([AWS ドキュメント][2])
* 访问控制：**IAM アイデンティティベースポリシー**｜*IAM identity-based policies* ([Amazon Web Services, Inc.][4])
* 日志审计：**CloudTrail（クラウドトレイル）**｜*AWS CloudTrail* ([AWS ドキュメント][5])
* 监控：**CloudWatch（クラウドウォッチ）**｜*Amazon CloudWatch* ([Amazon Web Services, Inc.][4])
* 共享责任模型：**共有責任モデル**｜*Shared Responsibility Model* ([Amazon Web Services, Inc.][9])
* 合规计划：**コンプライアンス・プログラム**｜*Compliance Programs* ([Amazon Web Services, Inc.][7])
* 模型守护栏：**ガードレール**｜*Guardrails for Amazon Bedrock* ([AWS ドキュメント][11])

---

## 7) 考试“秒选”提示（题干关键词 → 答案方向）

* **“私网/不走公网”** → **VPC Endpoint / PrivateLink**（Bedrock/Kendra/AgentCore）。([AWS ドキュメント][1])
* **“加密/密钥自管”** → **KMS + at-rest/in-transit encryption**。([AWS ドキュメント][2])
* **“谁在用数据/是否用于训练”** → **Bedrock 不用你的输入/输出训练模型**。([Amazon Web Services, Inc.][13])
* **“统一内容安全/PII/话题限制”** → **Guardrails**。([Amazon Web Services, Inc.][12])
* **“合规/审计”** → **Compliance programs + CloudTrail + IAM**。([Amazon Web Services, Inc.][7], [AWS ドキュメント][5])
* **“职责边界”** → **Shared Responsibility Model**。([Amazon Web Services, Inc.][9])

---

### 一句话总括

> 在 AWS 上做生成式 AI，可把“**网络隔离（PrivateLink）→ 加密（KMS/TLS）→ 访问控制（IAM）→ 审计监控（CloudTrail/CloudWatch）→ 模型安全（Guardrails）**”串成闭环；同时以 **合规计划** 与 **共享责任模型** 为底座，满足企业级的**安全、合规、责任、Safety** 要求。

[1]: https://docs.aws.amazon.com/bedrock/latest/userguide/vpc-interface-endpoints.html?utm_source=chatgpt.com "Use interface VPC endpoints (AWS PrivateLink) to create a ..."
[2]: https://docs.aws.amazon.com/bedrock/latest/userguide/data-encryption.html?utm_source=chatgpt.com "Data encryption - Amazon Bedrock"
[3]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingEncryption.html?utm_source=chatgpt.com "Protecting data with encryption"
[4]: https://aws.amazon.com/bedrock/security-compliance/?utm_source=chatgpt.com "Amazon Bedrock Security and Privacy - AWS"
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/logging-using-cloudtrail.html?utm_source=chatgpt.com "Monitor Amazon Bedrock API calls using CloudTrail"
[6]: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html?utm_source=chatgpt.com "Logging data events - AWS CloudTrail"
[7]: https://aws.amazon.com/compliance/?utm_source=chatgpt.com "Cloud Compliance - Amazon Web Services (AWS)"
[8]: https://aws.amazon.com/compliance/programs/?utm_source=chatgpt.com "Compliance Programs - Amazon Web Services (AWS)"
[9]: https://aws.amazon.com/compliance/shared-responsibility-model/?utm_source=chatgpt.com "Shared Responsibility Model - Amazon Web Services (AWS)"
[10]: https://docs.aws.amazon.com/whitepapers/latest/aws-risk-and-compliance/shared-responsibility-model.html?utm_source=chatgpt.com "Shared responsibility model - Amazon Web Services"
[11]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect and filter harmful content by using Amazon Bedrock ..."
[12]: https://aws.amazon.com/bedrock/guardrails/?utm_source=chatgpt.com "Amazon Bedrock Guardrails"
[13]: https://aws.amazon.com/bedrock/faqs/?utm_source=chatgpt.com "Amazon Bedrock FAQs - Generative AI - AWS"
[14]: https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html?utm_source=chatgpt.com "Data protection - Amazon Bedrock"
[15]: https://docs.aws.amazon.com/bedrock/latest/userguide/encryption-kb.html?utm_source=chatgpt.com "Encryption of knowledge base resources - Amazon Bedrock"
[16]: https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/vpc-interface-endpoints.html?utm_source=chatgpt.com "Use interface VPC endpoints (AWS PrivateLink) to create a ..."
