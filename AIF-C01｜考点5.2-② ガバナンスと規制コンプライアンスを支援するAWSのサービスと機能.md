# AIF-C01｜5.2.2 **ガバナンスと規制コンプライアンスを支援するAWSのサービスと機能**

*AIF-C01 Domain 5, Task 5.2.2 — Identify AWS services & features that support governance and regulatory compliance (AWS Config, Amazon Inspector, AWS Audit Manager, AWS Artifact, AWS CloudTrail, AWS Trusted Advisor, etc.)*

> 目标：能**点名 → 定义 → 会用**这些“治理与合规”服务，并把它们套进你的 **AI/ML（含 Bedrock / SageMaker / RAG）** 项目流程里。下文按服务逐项说明：**中文讲解为主**，关键术语给出 **日语读法**与 **English**。

---

## 0) 三语速查（中・日（读法）・英）

| 服务                      | 日文（读法）         | English             | 一句话定位                                                                                  |
| ----------------------- | -------------- | ------------------- | -------------------------------------------------------------------------------------- |
| **AWS Config**          | コンフィグ          | AWS Config          | 记录与评估**资源配置合规**，用**规则**与**一致性包**（Conformance Pack）+ 自动整改。([AWS ドキュメント][1])             |
| **Amazon Inspector**    | インスペクター        | Amazon Inspector    | 托管**漏洞与暴露**评估（EC2/ECR/Lambda；含**网络可达性**与**无代理/混合扫描**）。([Amazon Web Services, Inc.][2]) |
| **AWS Audit Manager**   | オーディット マネージャー  | AWS Audit Manager   | **自动采集证据**、按**合规模板/框架**生成审计就绪报告。([AWS ドキュメント][3])                                      |
| **AWS Artifact**        | アーティファクト       | AWS Artifact        | **自助下载** AWS 的 **ISO/SOC** 等合规报告与协议；做供应商尽调凭证库。([Amazon Web Services, Inc.][4])         |
| **AWS CloudTrail**      | クラウドトレイル       | AWS CloudTrail      | 记录**账户 API/用户活动**；事件类型含**管理/数据/洞察/网络活动**。([AWS ドキュメント][5])                             |
| **AWS Trusted Advisor** | トラスティッド アドバイザー | AWS Trusted Advisor | 持续**最佳实践体检**（成本/性能/韧性/安全/运维/配额）。([Amazon Web Services, Inc.][6])                       |

---

## 1) AWS Config（コンフィグ / AWS Config）

**是什么**：集中记录各资源**配置变更历史**，用**Config 规则**评估是否满足政策与基线；把若干规则与整改动作打包为**一致性包**（Conformance Pack），可一键在**单账号或整个 Organizations** 推广。([Amazon Web Services, Inc.][7])

**考点要语**

* **Managed Rules**（托管规则，直接用）：如 S3 公共读写、加密是否启用等。([AWS ドキュメント][8])
* **Conformance Pack**（一致性包）：**一揽子**规则＋整改动作；可跨组织集中下发。([AWS ドキュメント][9])
* **Auto Remediation**：把不合规资源自动交给 **Systems Manager Automation** 修复（如自动关掉 `0.0.0.0/0:22`）。([AWS ドキュメント][10])

**怎么用（AI 场景）**

* 给 **SageMaker Endpoint/S3 数据桶**套一包规则：**强制 SSE-KMS、阻断公共访问、VPC-Only**；一旦检测到不合规，**自动整改**。([AWS ドキュメント][11])
* 在 **Organizations** 级别下发“AI 安全基线一致性包”，统一治理多账号环境。([AWS ドキュメント][12])

---

## 2) Amazon Inspector（インスペクター / Amazon Inspector）

**是什么**：托管的**漏洞与暴露评估**。扫描 **EC2、容器镜像（ECR）、Lambda**，并对 EC2 做**网络可达性**分析（每 12 小时）；**混合/无代理**扫描支持用 **EBS 快照**收集软件清单。([Amazon Web Services, Inc.][2])

**怎么用（AI 场景）**

* 你的**推理容器镜像**入 ECR → Inspector 自动扫描 **CVE**；发现高危则阻断部署。
* 训练/推理 EC2 的**网络可达性**发现提示“对公网开放端口”→ 联动整改关闭暴露。([AWS ドキュメント][13])

---

## 3) AWS Audit Manager（オーディット マネージャー / AWS Audit Manager）

**是什么**：选择一个**合规框架**（如 **SOC 2 / ISO 27001 / PCI DSS** 等），Audit Manager 按框架**自动化收集证据**（来自 CloudTrail、配置/加密状态等），并**生成审计就绪报告**；2025 年仍在持续**更新框架与证据模型**。([AWS ドキュメント][3])

**怎么用（AI 场景）**

* 为“**AI/数据治理**”创建评估：证据自动抓取**加密是否启用**、**IAM 最小权限**、**CloudTrail 是否全量打开**等，**减少手工截图粘贴**。([Amazon Web Services, Inc.][14])

---

## 4) AWS Artifact（アーティファクト / AWS Artifact）

**是什么**：**合规资料金库**。自助下载 AWS 的 **ISO/SOC** 报告、证书与**协议**；把这些文档递交审计或客户以证明你所依托的 AWS 基础设施的合规性。也可集中**管理与跟踪**与 AWS 的各类合规协议。([Amazon Web Services, Inc.][4])

**关联记忆**：要 **SOC1/SOC2 报告** → 去 **Artifact**；**SOC3** 可公开获取。([Amazon Web Services, Inc.][15])

---

## 5) AWS CloudTrail（クラウドトレイル / AWS CloudTrail）

**是什么**：记录与审计**用户与 API 活动**。

* **默认**提供**90 天**的**管理事件**“事件历史”（控制台/CLI/API 调用），长期留存请建 **trail 或 event data store**。([AWS ドキュメント][5])
* 事件类型：**管理事件、数据事件、网络活动事件、Insights 事件**（异常行为检测）。([AWS ドキュメント][16])
* 还能为 **VPC 端点**记录**网络活动事件**，帮助构建**数据边界**监控。([Amazon Web Services, Inc.][17])

**怎么用（AI 场景）**

* 追溯“谁在何时调用了 **bedrock\:InvokeModel** / 修改了 S3 策略”，并把证据喂给 **Audit Manager** 做合规报告。([AWS ドキュメント][16])

---

## 6) AWS Trusted Advisor（トラスティッド アドバイザー / AWS Trusted Advisor）

**是什么**：持续性**最佳实践体检**与建议，覆盖 **成本、性能、韧性、安全、运维卓越、服务配额** 等分类；不同支持计划的**可用检查项**不同（基础/开发者计划仅开放部分安全/容错与**配额**检查）。([Amazon Web Services, Inc.][6])

**怎么用（AI 场景）**

* 关注**服务配额**（如 **Bedrock TPS / SageMaker 端点并发 / VPC 资源**）与**安全类检查**（S3 公共桶、Root MFA）。([AWS ドキュメント][18])

---

## 7) 把它们串成“治理合规闭环”（AI/ML 落地蓝图）

> 以“**道路影像识别 + 生成式解释**”为例，构建**证据链**：

1. **配置与策略落地**：

   * **AWS Config** 下发“AI 安全一致性包”：S3 **SSE-KMS** 强制、禁止公共访问、端点必须在 **VPC**；非合规→ **SSM Automation 自动整改**。([AWS ドキュメント][1])
2. **漏洞与暴露治理**：

   * **Inspector** 扫 ECR 镜像与 EC2 网络可达性；高危阻断上线。([AWS ドキュメント][13])
3. **活动审计与追溯**：

   * **CloudTrail** 开通跨账户/跨区 trail（管理+数据+必要的网络活动事件）；查“谁改了桶策略/谁调了模型”。([AWS ドキュメント][16])
4. **合规体检与配额**：

   * **Trusted Advisor** 监控配额与高危配置，持续出建议。([Amazon Web Services, Inc.][6])
5. **证据自动化与对外取证**：

   * **Audit Manager** 选择 SOC2/ISO27001 框架，自动收集加密/IAM/日志等证据并生成报告；
   * **Artifact** 下载 AWS 的 ISO/SOC 文档，“平台合规”对外可审。([AWS ドキュメント][19])

---

## 8) 服务 × 能力 × 典型控制（备考表）

| 能力        | 用哪个服务                   | 典型考点/动作                                                                                    |
| --------- | ----------------------- | ------------------------------------------------------------------------------------------ |
| 配置基线与自动整改 | **AWS Config**          | 托管规则、**Conformance Pack** 跨组织推广、**SSM Automation** 自动修复（如禁 0.0.0.0/0:22）。([AWS ドキュメント][9]) |
| 漏洞与暴露     | **Amazon Inspector**    | EC2/ECR/Lambda 扫描；**网络可达性**每 12 小时；**无代理/混合**扫描（EBS 快照）。([AWS ドキュメント][13])                 |
| 证据自动采集    | **AWS Audit Manager**   | 选 SOC2/ISO27001 框架→ 自动拉证据→ 审计就绪报告；框架持续更新。([AWS ドキュメント][3])                                 |
| 合规文档获取    | **AWS Artifact**        | 自助下载 AWS **ISO/SOC**；管理协议（多账号可跟踪）。([AWS ドキュメント][20])                                       |
| 活动审计      | **AWS CloudTrail**      | **90 天管理事件历史**；类型：管理/数据/洞察/网络活动；长期请建 trail/data store。([AWS ドキュメント][5])                    |
| 最佳实践体检    | **AWS Trusted Advisor** | 类别：**成本/性能/韧性/安全/运维/配额**；不同支持计划可用检查不同。([Amazon Web Services, Inc.][6])                     |

---

## 9) 术语小词典（中・日・英）

* 一致性包：**コンフォーマンスパック**／**Conformance Pack**（Config 规则+整改打包，支持跨组织部署）。([AWS ドキュメント][1])
* 自动整改：**自動リメディエーション**／**Auto Remediation**（Config 调用 **SSM Automation**）。([AWS ドキュメント][10])
* 漏洞管理：**脆弱性管理【ぜいじゃくせい かんり】**／**Vulnerability Management**（Inspector）。([Amazon Web Services, Inc.][2])
* 证据自动化：**証拠の自動収集【しょうこ じどう しゅうしゅう】**／**Automated Evidence Collection**（Audit Manager）。([AWS ドキュメント][3])
* 合规资料库：**コンプライアンス文書リポジトリ**／**Compliance Reports Repository**（Artifact）。([Amazon Web Services, Inc.][4])
* 事件类型：**管理/データ/インサイト/ネットワーク活動**／**Management/Data/Insights/Network Activity**（CloudTrail）。([AWS ドキュメント][16])
* 体检检查：**ベストプラクティスチェック**／**Best Practice Checks**（Trusted Advisor）。([Amazon Web Services, Inc.][6])

---

### ✅ 一句话总括

> **治理合规闭环** = **Config**（基线与整改）＋ **Inspector**（漏洞暴露）＋ **CloudTrail**（可审计活动）＋ **Trusted Advisor**（持续体检）＋ **Audit Manager**（证据自动化）＋ **Artifact**（对外取证）。把这套组合嵌进你的 **Bedrock/SageMaker/RAG** 流水线，就能做到**标准可对齐、控制可落地、证据可复核**。

[1]: https://docs.aws.amazon.com/config/latest/developerguide/conformance-packs.html?utm_source=chatgpt.com "Conformance Packs for AWS Config"
[2]: https://aws.amazon.com/inspector/faqs/?utm_source=chatgpt.com "Automated Vulnerability Management - Amazon Inspector FAQs"
[3]: https://docs.aws.amazon.com/audit-manager/latest/userguide/how-evidence-is-collected.html?utm_source=chatgpt.com "Understanding how AWS Audit Manager collects evidence"
[4]: https://aws.amazon.com/artifact/?utm_source=chatgpt.com "Access AWS Compliance Reports On-Demand - AWS Artifact"
[5]: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events.html?utm_source=chatgpt.com "Working with CloudTrail event history"
[6]: https://aws.amazon.com/premiumsupport/technology/trusted-advisor/?utm_source=chatgpt.com "AWS Trusted Advisor"
[7]: https://aws.amazon.com/documentation-overview/config/?utm_source=chatgpt.com "AWS Config Documentation - Amazon.com"
[8]: https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html?utm_source=chatgpt.com "List of AWS Config Managed Rules"
[9]: https://docs.aws.amazon.com/config/latest/developerguide/conformance-pack-organization-apis.html?utm_source=chatgpt.com "Managing Conformance Packs for AWS Config Across all ..."
[10]: https://docs.aws.amazon.com/config/latest/developerguide/remediation.html?utm_source=chatgpt.com "Remediating Noncompliant Resources with AWS Config"
[11]: https://docs.aws.amazon.com/config/latest/developerguide/setup-autoremediation.html?utm_source=chatgpt.com "Setting Up Auto Remediation for AWS Config"
[12]: https://docs.aws.amazon.com/cli/latest/reference/configservice/put-organization-conformance-pack.html?utm_source=chatgpt.com "put-organization-conformance-pack - AWS Documentation"
[13]: https://docs.aws.amazon.com/inspector/latest/user/findings-types.html?utm_source=chatgpt.com "Amazon Inspector finding types"
[14]: https://aws.amazon.com/blogs/security/streamlining-evidence-collection-with-aws-audit-manager/?utm_source=chatgpt.com "Streamlining evidence collection with AWS Audit Manager"
[15]: https://aws.amazon.com/compliance/soc-faqs/?utm_source=chatgpt.com "SOC Compliance - Amazon Web Services (AWS)"
[16]: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-events.html?utm_source=chatgpt.com "Understanding CloudTrail events"
[17]: https://aws.amazon.com/cloudtrail/?utm_source=chatgpt.com "Track User and API Activity - AWS CloudTrail"
[18]: https://docs.aws.amazon.com/awssupport/latest/user/trusted-advisor-check-reference.html?utm_source=chatgpt.com "AWS Trusted Advisor check reference - AWS Support"
[19]: https://docs.aws.amazon.com/audit-manager/latest/userguide/review-frameworks.html?utm_source=chatgpt.com "Reviewing a framework in AWS Audit Manager"
[20]: https://docs.aws.amazon.com/artifact/latest/ug/what-is-aws-artifact.html?utm_source=chatgpt.com "What is AWS Artifact? - AWS Artifact"
