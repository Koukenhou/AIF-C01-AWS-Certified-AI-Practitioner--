# AIF-C01｜5.1.4 **AIシステムのセキュリティとプライバシーに関する考慮事項**

*AIF-C01 Domain 5, Task 5.1.4 — Security & privacy considerations for AI systems (AppSec, threat detection, vulnerability management, infra protection, prompt injection, encryption at rest/in transit, etc.)*

> 目标：能**系统阐述**在 AWS 上构建/运行 AI 应用的安全与隐私要点，并能把这些要点**映射到具体服务**与**实操清单**。下文采用**中・日（读法）・英**三语标注关键术语，讲解以中文为主，辅以**政务/道路影像**案例。

---

## 0) 总览与应试抓手（按层分解）

* **责任边界**：**共有責任モデル【きょうゆう せきにん】/Shared Responsibility Model**（IaaS/托管服务中 AWS 与客户各自负责什么）。([Amazon Web Services, Inc.][1])
* **应用安全（AppSec）**：鉴权、输入验证、WAF/Shield、Secrets 与密钥管理。
* **威胁检测（Threat Detection）**：**GuardDuty** 等持续监控与告警。([AWS ドキュメント][2])
* **脆弱性管理（Vuln Mgmt）**：**Inspector** 对 EC2/ECR/Lambda 扫描、补丁自动化。([AWS ドキュメント][3])
* **基础设施防护（Infra Protection）**：VPC 隔离、Security Group/NACL、**Network Firewall**、**PrivateLink**、DDoS 防护（**WAF/Shield**）。([AWS ドキュメント][4])
* **LLM 特有风险**：**プロンプトインジェクション【prompt injection】** 等，对策：**Bedrock Guardrails**（包含“提示攻击检测/敏感信息过滤/否定主题/上下文扎实性检查”等）。([AWS ドキュメント][5])
* **加密**：**保存時の暗号化【ほぞんじ】/Encryption at rest**（KMS, S3 默认加密），**転送中の暗号化【てんそうちゅう】/In-transit**（TLS 1.2+，ACM 证书）。([AWS ドキュメント][6])

---

## 1) 责任与边界｜Shared Responsibility Model

* **要点**：AWS 负责**物理/基础设施/虚拟化层**，客户负责**操作系统/网络配置/身份授权/数据与应用安全**；随着你使用更多“托管服务”，你负责的**运维面**会更靠上层（配置与数据）。考题常要求你分辨**谁该做什么**。([Amazon Web Services, Inc.][1])

---

## 2) 应用安全（AppSec）

**日**：アプリケーションセキュリティ【せきゅりてぃ】｜**EN**：Application Security

**常用服务与做法**

1. **鉴权与会话**：

   * **Amazon Cognito【こぐにーと】/Cognito**：User Pool（用户管理）＋Identity Pool（签发临时 **IAM** 凭证），结合 **API Gateway**/ALB 授权下游资源访问。考题聚焦“最小权限”“多 IdP 联邦”。([AWS ドキュメント][7])
2. **输入验证与边界加固**：

   * **AWS WAF**：基于 **WebACL** 的托管/自定义规则（SQLi/XSS/机器人流量），能保护 **CloudFront / API Gateway / ALB / AppSync / Cognito** 等前置资源。([AWS ドキュメント][4])
   * **AWS Shield**：DDoS 防护（标准版默认，**Shield Advanced** 提供 L3–L7 增强防护与事件可视化/响应）。([AWS ドキュメント][8])
3. **机密与凭证**：

   * **AWS Secrets Manager**：安全存储/轮换 DB 密码、API Key、OAuth Token；与 **KMS** 集成加密；通过 IAM 控制访问。([AWS ドキュメント][9])
4. **证书与 TLS**：

   * **AWS Certificate Manager (ACM)**：签发/托管/自动续期公有或私有证书；官方**要求 TLS 1.2、推荐 1.3**。([AWS ドキュメント][10])

**道路影像例**：前端巡检小程序登录 → Cognito 颁发令牌 → API Gateway（WAF 保护）→ 后端推理 API（私网访问 Bedrock/SageMaker）。Secrets Manager 存摄像头接入密钥与数据库凭证；ACM 负责证书与 TLS。

---

## 3) 威胁检测（Threat Detection）

**日**：脅威検知【きょうい けんち】｜**EN**：Threat detection

* **Amazon GuardDuty**：托管威胁检测，持续分析 **CloudTrail 管理事件、VPC Flow Logs、Route 53 DNS** 等数据源；可扩展到 **EKS/Kubernetes、RDS 登录、S3 数据事件、EBS、Lambda 运行时网络** 等，输出可操作的**发现（Findings）**。([AWS ドキュメント][2])
* **Amazon Detective**：把多源日志构建为**行为图**，辅助**根因调查**与跨事件关联。([AWS ドキュメント][11])
* **AWS Security Hub**：集中汇总 GuardDuty/Inspector/Macie 等发现，按**最佳实践与基线**（CIS 等）出**态势评分与优先级**。用于**多账号**统一看板与自动化响应。([AWS ドキュメント][12])

**实操要点**：

* 打开 GuardDuty（组织级汇聚），将发现转发到 Security Hub；严重事件（如数据外流/异常探测）触发 EventBridge 自动化（隔离 SG/吊销密钥）。

---

## 4) 脆弱性管理（Vulnerability Management）

**日**：脆弱性管理【ぜいじゃくせい かんり】｜**EN**：Vulnerability management

* **Amazon Inspector**：自动发现**EC2、ECR 镜像、Lambda** 的**软件漏洞**与**网络暴露**，输出 CVE 发现并可联动修复。ECR 集成**镜像推送/定期**扫描。([AWS ドキュメント][3])
* **AWS Systems Manager Patch Manager**：定义基线与维护窗，**自动打补丁**并出合规报告（适配多账户/多区）。适合长期运维闭环。([AWS ドキュメント][13])

**道路影像例**：推理容器镜像入 ECR → Inspector 扫描阻断含高危 CVE 的部署；EC2 训练节点用 Patch Manager 每月例行更新。

---

## 5) 基础设施防护（Infrastructure Protection）

**日**：インフラ保護【ほご】｜**EN**：Infrastructure protection

* **网络分层**：VPC 私有子网、**Security Group**（有状态）与 **NACL**（无状态）双层控制。
* **AWS Network Firewall**：**状态检测＋IDS/IPS** 的托管防火墙，可在 VPC 出入口/跨网段实施策略（域名/IP/协议/应用层规则）。适合“总出口”控制与东西向流量审计。([AWS ドキュメント][14])
* **AWS PrivateLink**：通过 **VPC 接口端点**以**私网 IP**访问托管服务/合作方服务，避免流量经公网，缩小攻击面；常与 **`aws:SourceVpce`** 条件配合做**来源限制**。([AWS ドキュメント][15])
* **WAF/Shield**：见第 2 节（AppSec），配合 **CloudFront**/ALB 形成边缘—应用双层防护。([AWS ドキュメント][4])

**道路影像例**：业务 VPC 通过 PrivateLink 访问 Bedrock 与 S3；南北向出入口接入 Network Firewall；前端 API 走 CloudFront+WAF，并订阅 Shield Advanced 提升抗 DDoS 能力。

---

## 6) LLM 特有风险：プロンプトインジェクション（Prompt Injection）

**定义**：攻击者通过**指令注入**（直接或**间接**嵌入到网页/文档/RAG 数据源）诱导模型**越权**或**泄露**敏感信息，或执行不当的工具调用。
**核心防护（Bedrock 原生）**：

* **Amazon Bedrock Guardrails**：

  * **Prompt 攻击检测**（阻断“忽略先前指令”等）、**敏感信息过滤（PII/自定义 Regex）**、**否定主题**、**词汇过滤**、**上下文扎实性检查**（降低幻觉），可同时作用于**输入**与**输出**；支持在 **Agents/Flows/KB** 上关联。([AWS ドキュメント][5])
* 设计性加固：

  * **最小化工具能力**（只暴露必要 API），参数白名单；
  * **RAG 前置清洗**（去除可能的“诱导”片段），为检索结果加**来源引用**；
  * **会话分割**与**机密分层**（机密不进模型上下文，必要时仅返回引用链接）。

> **考点语言**：
>
> * プロンプトインジェクション【ぷろんぷと いんじぇくしょん】/Prompt injection（提示词注入）
> * ガードレール【がーどれーる】/Guardrails（安全护栏）
> * 脱獄/ジェイルブレイク【jailbreak】/Jailbreak（越狱式绕过）

---

## 7) 加密：保存時（静态）与転送中（传输）

**日**：保存時の暗号化【ほぞんじ】／転送中の暗号化【てんそうちゅう】｜**EN**：Encryption at rest / in transit

* **KMS（Key Management Service）**：集中创建/管理加密密钥，HSM（FIPS 140-3 L3）保护，广泛集成 S3/EBS/RDS/SageMaker 等；**密钥策略**与 **CloudTrail** 审计是考点。([AWS ドキュメント][6])
* **S3 默认加密**：自 2023-01-05 起**所有新上传对象默认加密**（SSE-S3）；可按需切换 **SSE-KMS** 以使用自管 CMK。([AWS ドキュメント][16])
* **TLS in transit**：**ACM** 证书用于 **ALB/CloudFront/API Gateway** 等，官方**要求 TLS 1.2、推荐 1.3**。([AWS ドキュメント][10])

**道路影像例**：原始影像与标注集落 S3（SSE-KMS，统一 CMK）；推理端点与前端交互全程 TLS（ACM 证书），训练/推理日志写入加密的 CloudWatch Log Group。

---

## 8) 场景化串联（道路影像 × 生成式报告）

1. **AppSec**：Cognito 鉴权 → API Gateway（WAF）→ 私网调用推理；Secrets Manager 管摄像头密钥。
2. **Threat Detection**：GuardDuty 开启并把发现聚合到 Security Hub，异常自动触发隔离剧本。
3. **Vuln Mgmt**：Inspector 对 ECR 镜像与 Lambda 扫描，Patch Manager 定期打补丁。
4. **Infra Protection**：PrivateLink 访问 Bedrock/S3，VPC 出口 Network Firewall，Shield 抗 DDoS。
5. **LLM 保护**：Bedrock Guardrails 开启“Prompt 攻击检测/PII 过滤/否定主题/上下文检查”；RAG 侧过滤外部文档中的“诱导语”。
6. **加密**：S3 SSE-KMS、EBS/RDS 加密，前后端 TLS 1.2+。

---

## 9) 实操清单（Checklist，直接照抄）

* [ ] **共享责任**：把“谁负责什么”写入架构说明与审计清单。([Amazon Web Services, Inc.][1])
* [ ] **GuardDuty** 组织级开启；关键发现进 **Security Hub**，EventBridge 触发隔离/告警。([AWS ドキュメント][2])
* [ ] **Inspector**：启用 EC2/ECR/Lambda 扫描；镜像推送即扫，阻断高危 CVE 部署。([AWS ドキュメント][3])
* [ ] **WAF/Shield**：WebACL 绑定 CloudFront/API；订阅 Shield Advanced 保护要害资源。([AWS ドキュメント][4])
* [ ] **Network Firewall**：在总出口/跨网段布防；日志送集中分析。([AWS ドキュメント][14])
* [ ] **PrivateLink**：Bedrock/S3 等托管服务走私网端点，并在 IAM 中限制 `aws:SourceVpce`。([AWS ドキュメント][15])
* [ ] **Guardrails**：为 Agents/Flows/KB 关联 Guardrail（开启 Prompt 攻击检测、PII 过滤、否定主题、上下文检查）。([AWS ドキュメント][5])
* [ ] **KMS/S3**：统一使用 CMK，S3 默认加密核验（必要时要求 SSE-KMS）。([AWS ドキュメント][6])
* [ ] **TLS**：前端与 API 使用 ACM 证书，强制 TLS 1.2+。([AWS ドキュメント][10])

---

## 10) 高频术语小词典（中・日・英，带读法）

* 共有责任模型：**共有責任モデル【きょうゆう せきにん】** / Shared Responsibility Model. ([Amazon Web Services, Inc.][1])
* 威胁检测：**脅威検知【きょうい けんち】** / Threat detection（**GuardDuty**）。([AWS ドキュメント][2])
* 脆弱性管理：**脆弱性管理【ぜいじゃくせい かんり】** / Vulnerability management（**Inspector**）。([AWS ドキュメント][3])
* 应用防护：**アプリケーション防御【ぼうぎょ】** / App protection（**WAF/Shield**）。([AWS ドキュメント][4])
* 私网连接：**プライベートリンク** / **AWS PrivateLink**（私有端点）。([AWS ドキュメント][15])
* 提示注入：**プロンプトインジェクション** / Prompt injection（**Bedrock Guardrails**）。([AWS ドキュメント][5])
* 保存时/传输中加密：**保存時の暗号化／転送中の暗号化** / Encryption at rest/in transit（**KMS/ACM**）。([AWS ドキュメント][6])

---

### ✅ 一句话总括

> **AI 安全 = 责任清晰 + 分层防护 + 持续检测 + 漏洞治理 + LLM 护栏 + 全程加密**。
> 在 AWS 上，对应**SRM**（谁负责）、**WAF/Shield/PrivateLink/Network Firewall**（边界/网络）、**GuardDuty/Security Hub/Detective**（检测与响应）、**Inspector/Patch Manager**（漏洞与补丁）、**Bedrock Guardrails**（LLM 风险）、**KMS/ACM/S3 默认加密**（加密与密钥）。这些实践组合成**可审计、可运营、可扩展**的 AI 安全底座。 ([Amazon Web Services, Inc.][1])

> 需要的话，我可以把\*\*“道路影像 AI 系统安全基线（按服务逐项勾选）”\*\*整理成一页表格，便于你们团队做上线前自查。

[1]: https://aws.amazon.com/compliance/shared-responsibility-model/?utm_source=chatgpt.com "Shared Responsibility Model - Amazon Web Services (AWS)"
[2]: https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html?utm_source=chatgpt.com "What is Amazon GuardDuty? - Amazon ..."
[3]: https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html?utm_source=chatgpt.com "What is Amazon Inspector? - Amazon Inspector"
[4]: https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html?utm_source=chatgpt.com "AWS WAF"
[5]: https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-injection.html?utm_source=chatgpt.com "Prompt injection security - Amazon Bedrock"
[6]: https://docs.aws.amazon.com/kms/latest/developerguide/overview.html?utm_source=chatgpt.com "AWS Key Management Service"
[7]: https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-identity.html?utm_source=chatgpt.com "Amazon Cognito identity pools"
[8]: https://docs.aws.amazon.com/waf/latest/developerguide/ddos-overview.html?utm_source=chatgpt.com "How AWS Shield and Shield Advanced work"
[9]: https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html?utm_source=chatgpt.com "What is AWS Secrets Manager? - ..."
[10]: https://docs.aws.amazon.com/acm/latest/userguide/infrastructure-security.html?utm_source=chatgpt.com "Infrastructure security in AWS Certificate Manager"
[11]: https://docs.aws.amazon.com/detective/?utm_source=chatgpt.com "Amazon Detective Documentation"
[12]: https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub-v2.html?utm_source=chatgpt.com "Introduction to AWS Security Hub"
[13]: https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-manager.html?utm_source=chatgpt.com "AWS Systems Manager Patch Manager"
[14]: https://docs.aws.amazon.com/network-firewall/latest/developerguide/what-is-aws-network-firewall.html?utm_source=chatgpt.com "AWS Network Firewall - AWS Documentation"
[15]: https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html?utm_source=chatgpt.com "What is AWS PrivateLink? - Amazon Virtual Private Cloud"
[16]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html?utm_source=chatgpt.com "Using server-side encryption with AWS KMS keys (SSE-KMS)"
