# AIF-C01｜5.1.1 **AIシステムを保護するためのAWSのサービスと機能**

*AIF-C01 Domain 5, Task 5.1.1 — Identify AWS services & features to protect AI systems (IAM roles, policies, permissions, encryption, Amazon Macie, AWS PrivateLink, AWS Shared Responsibility Model, etc.)*

> 目标：能**点名 + 正确定义 + 会用**以下能力来保护 **生成式/传统 ML 系统**：
> **IAM ロール（ろーる，Roles）／ポリシー（Policies）／アクセス許可（Permissions）／暗号化（Encryption/KMS）／Amazon Macie／AWS PrivateLink／AWS 責任共有モデル（Shared Responsibility Model）**。并能把它们组合到 **Bedrock / SageMaker / RAG** 的实战里（含示例与考试关键表述）。

---

## 0) 总览（看到题就能答）

* **身份与访问控制**：用 **IAM 角色/策略**授予**最小权限**；策略是 JSON 文档，决定“谁（Principal）可对哪些资源（Resource）执行哪些操作（Action）在何种条件（Condition）下”。([AWS ドキュメント][1])
* **数据保护**：**传输中**用 **TLS**；**静态**用 **KMS** 加密（S3、EBS、SageMaker 工件等）。([AWS ドキュメント][2])
* **敏感数据发现**：**Amazon Macie** 自动发现/报告 S3 中的 **PII/敏感数据**，支持**自动发现**与**按需任务**。([AWS ドキュメント][3])
* **私网访问**：**AWS PrivateLink**（VPC 接口端点）把对 **Bedrock 等服务**的访问**留在私网**（无需公网 IP/IGW/NAT）。([AWS ドキュメント][4])
* **责任边界**：**Shared Responsibility Model**——**AWS 负责“云的安全”**，**你负责“云上的安全”**（数据、身份、网络、合规配置等）。([AWS ドキュメント][5])
* **Bedrock 安全补充**：**不会使用你的输入/输出训练任何模型，也不会分发给第三方**；支持 **KMS 加密、CloudWatch/CloudTrail 监控**。([AWS ドキュメント][2])

---

## 1) 身份与访问控制（IAM）

### 1.1 IAM ロール【Roles】× ポリシー【Policies】× アクセス許可【Permissions】

* **Roles（角色）**：给人/应用/服务的**临时身份**；权限来自附加的**策略**。可用 **STS** 发放短期凭证，避免长期密钥。([AWS ドキュメント][6])
* **Policies（策略）**：**JSON 文档**，定义 *Effect/Action/Resource/Condition*；是授权的**根本单位**（可挂载到**身份**或**资源**）。([AWS ドキュメント][7])
* **Permissions（访问许可）**：由“**谁**（User/Role）+ **什么策略**（Policy）”共同决定是否**Allow/Deny**。([AWS ドキュメント][1])

**Bedrock/SageMaker 考点要点**

* **Bedrock**：用 **IAM 身份策略**控制 `bedrock:InvokeModel`、`bedrock:InvokeModelWithResponseStream`、`bedrock:ListFoundationModels` 等操作；结合 **VPC 端点策略**进一步限制来源。官方还建议启用 **MFA/TLS**。([AWS ドキュメント][2])
* **SageMaker**：**Notebook/Job/Endpoint** 都需要**执行角色**访问 S3（数据/工件）、ECR（镜像）、CloudWatch（日志），按**最小权限**拆分。([AWS ドキュメント][8])

**小例子｜最小权限调用 Bedrock：**

* 建一条策略：仅允许角色在 `ap-northeast-1` 调 `bedrock:InvokeModel`，并限制到你的 VPC 端点 ARN（条件 `aws:SourceVpce`）。策略挂到一枚**应用角色**，再把该角色供 API 层（例如 Lambda）扮演。

---

## 2) データ保護（数据保护）

### 2.1 暗号化【Encryption】at rest / in transit

* **KMS（キー管理サービス）**：集中管理密钥；KMS 密钥受 **FIPS 140-3 L3 HSM** 保护，密钥**永不以明文离开 KMS**。([AWS ドキュメント][9])
* **SageMaker**：**默认加密**模型工件、训练/处理作业输出、Studio 笔记本等（SSE-S3）；跨账号共享时建议改用**自管 KMS CMK**。([AWS ドキュメント][10])
* **Bedrock**：数据**传输加密**（TLS1.2+，推奨 1.3），**在写入服务侧资源时可用 KMS**；支持 **CloudWatch/CloudTrail** 审计。([AWS ドキュメント][2])

**小例子｜道路影像训练数据（S3）**

* 桶级开启**默认加密（SSE-KMS，CMK）**；
* SageMaker 训练作业指定同一把 **CMK**；
* 下游端点（Invoke）全程 **HTTPS**；
* 若要跨账号分享数据集，使用 **CMK + 资源策略**明确定义可访问主体。([AWS ドキュメント][11])

### 2.2 Bedrock 的数据使用与隐私

* **不会存储/记录**你的提示与完成；**不会使用你的输入/输出训练 AWS 或第三方模型**；模型提供方**无法访问**部署账号与日志。（考试常问）([AWS ドキュメント][2])
* Bedrock 官方页也强调：**数据在传输与静态均可加密，KMS 可自管；支持 CloudWatch/CloudTrail** 做监控与审计。([Amazon Web Services, Inc.][12])

---

## 3) Amazon Macie（マシー）：敏感数据发现

* **定义**：**托管的数据安全/隐私服务**；利用 **ML + 模式匹配**，**自动发现/记录/报告** S3 数据中的 **PII/敏感信息**（支持“自动发现”或“发现任务”两种工作方式）。([AWS ドキュメント][13])
* **为何重要**：避免把 **PII/机密**误喂入训练/微调或发给第三方；为 **GDPR/CCPA** 等合规做证据。
* **怎么用**：

  1. 对存放原始/整理/特征数据的 S3 桶启用 **自动发现**；
  2. 对高风险分区（例如“用户工单附件/原始上传”）定期跑**发现任务**；
  3. 把 Macie 发现事件接入 **Security Hub/CloudWatch** 报警 → **阻断数据入湖/入模的管线**。([AWS ドキュメント][14])

**小例子｜RAG 知识库接入前**

* 在把企业 PDF/CSV 导入 **Bedrock Knowledge Bases** 之前，先用 **Macie** 扫 S3 源，若识别到 PII → 走脱敏/红线清理，再允许 KB 抓取。([AWS ドキュメント][14])

---

## 4) AWS PrivateLink（プライベートリンク）：私网访问 Bedrock/托管服务

* **概念**：在**你的 VPC**里创建 **接口端点（Interface VPC Endpoint）**，通过 **私有 IP** 访问目标服务（AWS/第三方/自定义端点），**无需 IGW/NAT/公网 IP/VPN/DC**。支持用 **安全组** 控制入站访问。([AWS ドキュメント][4])
* **Bedrock 专用**：可为 **Amazon Bedrock** 建立 **PrivateLink** 连接，使调用 **InvokeModel** 等 API **完全走内网**；也可在 **VPC** 中启用 **Flow Logs** 监控进出容器/作业流量。([AWS ドキュメント][15])

**小例子｜政务内网的生成式应用**

* 在 `ap-northeast-1` 为 **Bedrock** 创建 **VPC 接口端点**；
* API 层（Lambda/ECS）仅允许访问该 **VPCE**；
* **IAM 条件键**限制仅当 `aws:SourceVpce` 为端点 ARN 时才可 `InvokeModel`；
* 结合 **安全组** 控制来源子网，审计用 **VPC Flow Logs + CloudTrail**。([AWS ドキュメント][15])

---

## 5) AWS 責任共有モデル（Shared Responsibility Model）

* **定义**：**Security & Compliance = AWS + 你**。AWS 管理从**物理设施/虚拟化层**往下的一切；**你**负责**数据、身份、网络、应用配置与合规**。([AWS ドキュメント][5])
* **对 AI 的含义**：

  * AWS **保证** Bedrock/SageMaker 等平台级安全；
  * 你需**配置 IAM/加密/PrivateLink/Macie/监控**、清理数据（不要把 PII/受限数据误喂入），并**记录用途与来源**（参见 **Model Cards/Lineage**）。([AWS ドキュメント][16])

---

## 6) 监控与审计（建议一起背）

* **CloudWatch**：监控推理/吞吐/Token 指标，建看板 & 告警；**CloudTrail**：记录 API 调用（谁在何时调用了哪种模型/端点）。Bedrock 安全页推荐把它们用于**治理与审计**。([Amazon Web Services, Inc.][12])

---

## 7) 把它们拼起来：两套“考试即用”的安全基线

### 7.1 生成式应用（Bedrock + KB）

1. **身份**：仅赋予应用角色 `bedrock:InvokeModel`；加上 `aws:SourceVpce` 条件；控制台操作要 **MFA**。([AWS ドキュメント][2])
2. **网络**：对 Bedrock 建 **PrivateLink**；API/Agent 只走私网端点；VPC 打开 **Flow Logs**。([AWS ドキュメント][15])
3. **数据**：S3/向量库**启用 KMS**；KB 抓取前先跑 **Macie**；提示/输出均走 **TLS**。([AWS ドキュメント][9])
4. **合规**：记得**Bedrock 不会使用你的数据训练，也不会分发给第三方**；开启 **CloudTrail + CloudWatch** 审计与监控。([AWS ドキュメント][2])

### 7.2 传统 ML（SageMaker 训练 + 托管端点）

1. **身份**：给 Notebook/Training/Endpoint **不同执行角色**（最小权限访问 S3/ECR/Logs）。([AWS ドキュメント][8])
2. **数据加密**：S3 桶 + 训练工件 + Batch Transform/Processing/Studio **默认加密**；跨账号需要**自管 CMK**。([AWS ドキュメント][10])
3. **网络**：把 Endpoint 放在 **VPC**；仅允许来自应用子网；必要时用 **PrivateLink** 访问其他托管服务。([AWS ドキュメント][4])
4. **隐私**：训练集入湖前跑 **Macie**；输出落桶同样**KMS**。([AWS ドキュメント][14])

---

## 8) 术语小词典（中・日・英，按本题关键字）

* 身份与访问管理：**IAM（アイエーエム）**／Identity and Access Management（访问控制总框架）。([AWS ドキュメント][17])
* 角色：**IAM ロール【ろーる】**／IAM Role（临时身份，由策略赋权）。([AWS ドキュメント][6])
* 策略：**ポリシー【ぽりしー】**／Policy（JSON 文档，定义允许/拒绝）。([AWS ドキュメント][7])
* 访问许可：**アクセス許可【きょか】**／Permissions（通过策略实现的授权）。([AWS ドキュメント][1])
* 加密：**暗号化【あんごうか】**／Encryption（At rest/in transit；KMS 管钥）。([AWS ドキュメント][9])
* 敏感数据发现：**Amazon Macie（マシー）**／Amazon Macie（自动发现 S3 中 PII/敏感数据）。([AWS ドキュメント][3])
* 私网互联：**AWS PrivateLink（プライベートリンク）**／AWS PrivateLink（VPC 接口端点）。([AWS ドキュメント][4])
* 共享责任：**AWS 責任共有モデル【せきにん きょうゆう もでる】**／Shared Responsibility Model。([AWS ドキュメント][5])

---

## 9) 速记口诀（中文）

> **“身数网责”：身（身份 IAM）｜数（数据加密+Macie）｜网（PrivateLink 私网）｜责（Shared Responsibility）**。
> 先**锁身份**，再**清/密数**，用**私网走**，明**责边界**；Bedrock 自带**不留存/不训练**的数据边界，配合 **CloudWatch/CloudTrail** 可审计可追溯。([AWS ドキュメント][2])

---

### ✅ 一句话总括

**保护 AWS 上的 AI 系统 = IAM 最小权限 + KMS/TLS 全链路加密 + Macie 敏感数据发现 + PrivateLink 私网访问 + 共享责任模型下的合规与审计（CloudWatch/CloudTrail）**。对 **Bedrock** 再强调一次：**数据不用于训练，也不分发给第三方**。([AWS ドキュメント][2])

如果你愿意，我可以把上面的两套“安全基线”进一步细化成**逐步执行清单**（Checklist），直接贴到你的项目 Wiki 里用。

[1]: https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_access-management.html?utm_source=chatgpt.com "How permissions and policies provide access management"
[2]: https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html?utm_source=chatgpt.com "Data protection - Amazon Bedrock"
[3]: https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html?utm_source=chatgpt.com "What is Amazon Macie? - Amazon Macie"
[4]: https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html?utm_source=chatgpt.com "What is AWS PrivateLink? - Amazon Virtual Private Cloud"
[5]: https://docs.aws.amazon.com/whitepapers/latest/aws-risk-and-compliance/shared-responsibility-model.html?utm_source=chatgpt.com "Shared responsibility model - Amazon Web Services"
[6]: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html?utm_source=chatgpt.com "IAM roles - AWS Identity and Access Management"
[7]: https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html?utm_source=chatgpt.com "Policies and permissions in AWS Identity and Access ..."
[8]: https://docs.aws.amazon.com/config/latest/developerguide/security-best-practices-for-SageMaker.html?utm_source=chatgpt.com "Security Best Practices for Amazon SageMaker AI"
[9]: https://docs.aws.amazon.com/kms/latest/developerguide/overview.html?utm_source=chatgpt.com "AWS Key Management Service"
[10]: https://docs.aws.amazon.com/sagemaker/latest/dg/encryption-at-rest.html?utm_source=chatgpt.com "Protect Data at Rest Using Encryption"
[11]: https://docs.aws.amazon.com/whitepapers/latest/sagemaker-studio-admin-best-practices/data-protection.html?utm_source=chatgpt.com "Data protection - SageMaker Studio Administration Best ..."
[12]: https://aws.amazon.com/bedrock/security-compliance/?utm_source=chatgpt.com "Amazon Bedrock Security and Privacy - AWS"
[13]: https://docs.aws.amazon.com/macie/?utm_source=chatgpt.com "Amazon Macie Documentation"
[14]: https://docs.aws.amazon.com/macie/latest/user/data-classification.html?utm_source=chatgpt.com "Discovering sensitive data with Macie"
[15]: https://docs.aws.amazon.com/bedrock/latest/userguide/vpc-interface-endpoints.html?utm_source=chatgpt.com "Use interface VPC endpoints (AWS PrivateLink) to create a ..."
[16]: https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/shared-responsibility.html?utm_source=chatgpt.com "Shared responsibility - Security Pillar"
[17]: https://docs.aws.amazon.com/iam/?utm_source=chatgpt.com "AWS Identity and Access Management Documentation"
