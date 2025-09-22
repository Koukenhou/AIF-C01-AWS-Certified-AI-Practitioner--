# AIF-C01｜5.2.1 **AIシステムの規制コンプライアンス基準を特定する**

*AIF-C01 Domain 5, Task 5.2.1 — Identify regulatory & assurance standards for AI systems (ISO, SOC, Algorithmic Accountability Act, etc.)*

> 目标：能够**点名 → 正确定义 → 说明在 AWS 上如何落地与取证**的合规/保障标准：
> **ISO 系列**（27001/27017/27018/27701、42001、23894）、**SOC（1/2/3）**、**美国 Algorithmic Accountability Act（算法说明责任法，法案动向）**，并知道借助 **AWS Artifact / AWS Audit Manager** 取证与对齐的方法。([Amazon Web Services, Inc.][1])

---

## 0) 三语速查（中・日（读法）・英）

| 主题                             | 日文（片假名/读法）                                                   | English                               | 一句话定位                                                     |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------- | --------------------------------------------------------- |
| 国际标准化组织/电工委员会                  | **国際標準化機構【こくさい ひょうじゅんか きこう】/ 国際電気標準会議【こくさい でんき ひょうじゅん かいぎ】** | **ISO/IEC**                           | 信息安全/云/隐私/AI 管理与风险标准族。                                    |
| ISO/IEC 27001                  | **情報セキュリティマネジメントシステム**【じょうほう せきゅりてぃ まねじめんと】                  | ISMS                                  | 建立/运行信息安全管理体系。([Amazon Web Services, Inc.][2])            |
| ISO/IEC 27017                  | **クラウドセキュリティ管理策**                                            | Cloud security controls               | 云环境专属控制指引。([Amazon Web Services, Inc.][2])                |
| ISO/IEC 27018                  | **クラウド上の個人データ保護**                                            | Protection of PII in public cloud     | 云上 PII 保护实践。([Amazon Web Services, Inc.][2])              |
| ISO/IEC 27701                  | **プライバシーマネジメント**                                             | Privacy Information Management        | 扩展 ISMS 的隐私管理体系。([Amazon Web Services, Inc.][1])          |
| ISO/IEC 42001:2023             | **AI マネジメントシステム**                                            | AI management systems                 | **首个 AI 管理体系标准**（治理/伦理/透明等）。([ISO][3])                    |
| ISO/IEC 23894:2023             | **AI リスクマネジメント指針**                                           | AI risk management guidance           | AI 风险治理方法与流程。([ISO][4])                                   |
| SOC（1/2/3）                     | **システム・アンド・オーガニゼーション・コントロールズ**                               | System and Organization Controls      | 独立审计报告，SOC2 针对**信任服务准则**。([Amazon Web Services, Inc.][5]) |
| Algorithmic Accountability Act | **アルゴリズム説明責任法【せつめい せきにん ほう】**                                | Algorithmic Accountability Act (U.S.) | 拟要求**自动化决策系统影响评估**（美国，**法案**进程）。([Congress.gov][6])       |
| NIST AI RMF 1.0                | **NIST AI リスク管理フレームワーク**                                     | NIST AI Risk Management Framework     | **自愿**框架，提“Govern-Map-Measure-Manage”。([NIST][7])         |

---

## 1) ISO / IEC 标准族（与 AWS 的关系）

### 1.1 经典“云+隐私”四件套

* **ISO/IEC 27001**（ISMS）：建立组织层面的**信息安全管理体系**；AWS 平台通过第三方审计取得 27001/27017/27018/27701 等认证，客户可据此在**共享责任模型**下继承基础控制并补齐自有控制。([Amazon Web Services, Inc.][2])
* **ISO/IEC 27017**：云专属控制（如租户隔离、虚拟化安全、供应商管理）。([Amazon Web Services, Inc.][2])
* **ISO/IEC 27018**：**公有云 PII** 保护的实践守则。([Amazon Web Services, Inc.][2])
* **ISO/IEC 27701**：在 27001/27002 之上，扩展**隐私信息管理**（角色/责任/处理记录/主体权利）。([Amazon Web Services, Inc.][1])

> 在 AWS 上取证：登录 **AWS Artifact** 下载 AWS 的 ISO 证书与声明（含适用服务清单），用于**第三方评估与尽职调查**。([AWS ドキュメント][8])

### 1.2 新的 AI 向标准

* **ISO/IEC 42001:2023（AI 管理体系）**：面向组织的**AI 治理管理体系**（政策、角色、风险、透明度、持续改进），是**首个**可认证的 AI 管理体系标准。适用于**自建/使用** AI 的企业。([ISO][3])
* **ISO/IEC 23894:2023（AI 风险指南）**：提供如何将**风险管理**嵌入 AI 全生命周期（数据 → 模型 → 部署 → 运行）的指南；常与 **NIST AI RMF** 联用。([ISO][4])
* AWS 官方亦给出 42001/NIST AI RMF 的治理实践解读与对齐思路，可作为实施参考。([Amazon Web Services, Inc.][9])

---

## 2) SOC 报告（独立审计取证）

* **SOC 1**：财务报告相关控制（适合会计/计费影响的系统）。
* **SOC 2**：**信任服务准则**（安全/可用性/处理完整性/保密/隐私）。
* **SOC 3**：SOC2 的**公开版本**（摘要，便于外部披露）。
* **获取方式**：**AWS Artifact** 可**按需下载** AWS 的 SOC1/SOC2 报告；最新 SOC3 可公开获取。AWS 安全博客会定期公告覆盖范围与时段。([Amazon Web Services, Inc.][5])

> 备考要点：**SOC 是“审计报告/鉴证”，ISO 是“管理体系/规范”**；两者都可作为**供应商尽调与客户保证**的证据。([Amazon Web Services, Inc.][5])

---

## 3) 美国：Algorithmic Accountability Act（算法说明责任法，动向）

* **定位**：**联邦法案**（119 届国会 **2025-06-25 提出** S.2164），拟授权 **FTC** 要求对**自动化决策系统**与**关键决策流程**做**影响评估（impact assessments）**；目前处于**Introduced**（已提交）阶段，**尚未生效**。([Congress.gov][6])
* **对企业含义**（若通过）：需要**登记系统范围与用途**、记录**训练数据与风险**、**偏差与隐私影响评估**、**治理/申诉**流程，并接受**抽查或执法**。
* **与 AWS 的结合**：

  * 用 **SageMaker Model Cards** 记录用途/限制/数据出典与评估；
  * 用 **Audit Manager** 汇集证据（访问控制、日志、加密）并对照框架；
  * 用 **Bedrock Guardrails**、**Macie**、**KMS** 作为技术控制的证据来源。
    （这些不等于“合规即达成”，但有助于**证明控制有效性**。）

> 现实里，跨境还会遇到 **EU AI Act** 等法制；AWS 已发布其**对 EU AI Act 的方法与承诺**（可信 AI、端到端责任、工具/文档支持），便于你把**技术管控**对齐**法律义务**。([Amazon Web Services, Inc.][10])

---

## 4) NIST AI RMF（虽非法律，但常被考）

* **定位**：**自愿性**的 AI 风险管理框架，核心四职能：**Govern（治理）/ Map（建模风险图谱）/ Measure（量化评估）/ Manage（管控与改进）**；强调**可信**（可靠、安全、可解释、公平、隐私增强）。([NIST][7])
* **与 ISO 42001/23894 的关系**：RMF 可作为**过程方法**，帮助组织落地**AI 管理体系**与**风险指南**。AWS 的安全/治理博文也提供了**对齐与落地**的实践。([Amazon Web Services, Inc.][9])

---

## 5) 在 AWS 上“取证/对齐”的两把钥匙

### 5.1 AWS Artifact（合规报告与证书的“自助金库”）

* **作用**：按需下载 **AWS 的 ISO 证书、SOC1/2 报告、其他合规文档**；也可获取与 ISV（Marketplace）相关的合规材料，便于**尽调**。([AWS ドキュメント][8])
* **考点句式**：“要获取 AWS 的 SOC2 报告或 ISO 证书 → **AWS Artifact**”。([Amazon Web Services, Inc.][5])

### 5.2 AWS Audit Manager（自动收集证据，映射到框架）

* **作用**：提供**预置框架库**（如 **SOC2、ISO 27001、NIST 800-53、GDPR、HIPAA** 等），**自动收集证据**并生成**审核就绪**的报告；可跨 **Organizations** 多账户集中管理。([AWS ドキュメント][11])
* **考试要点**：不是“自动合规”，而是**加速对齐与取证**；仍需你配置/运行**技术与管理控制**。([AWS ドキュメント][12])

---

## 6) 场景化示例（以“道路影像识别 + 生成式说明书”为例）

> 目标：让审计或监管问“你凭什么说系统合规？”时，有**标准—控制—证据**的闭环。

1. **确定适用标准**：

   * 信息安全/云/隐私：**ISO 27001/27017/27018/27701**；
   * AI 治理与风险：**ISO 42001 / 23894 / NIST AI RMF**；
   * 审计鉴证：**SOC2**（向客户/合作方证明平台控制）。([Amazon Web Services, Inc.][2])
2. **平台级证据**（AWS 责任边界内）：

   * 通过 **AWS Artifact** 下载 AWS 的 **ISO/SOC** 报告；列出你用到的**已覆盖服务**（如 S3、SageMaker、Bedrock）。([Amazon Web Services, Inc.][13])
3. **你方控制与证据**（客户责任边界）：

   * **技术控制**：KMS 加密、IAM 最小权限、PrivateLink、WAF/Shield、Macie、Guardrails；
   * **过程控制**：**SageMaker Model Cards** 记录用途/限制/数据出典与评估；**Audit Manager** 选用 **SOC2/ISO27001 框架**自动汇集证据并导出报告。([AWS ドキュメント][14])
4. **（若涉美国监管）影响评估草案**：参考 **Algorithmic Accountability Act** 的思路组织**AI 影响评估**（系统范围、数据/偏差、隐私、纠纷申诉）。([Congress.gov][6])
5. **（若涉欧盟）风险分类与技术/组织措施**：参考 **EU AI Act** 指南类材料与 AWS 的实施建议。([Amazon Web Services, Inc.][10])

---

## 7) 备考要点与易混辨析

* **“ISO vs SOC”**：ISO 是**管理体系或实践准则**，可证书化；SOC 是**独立审计报告**（尤其 SOC2），常用于**对外保证**。二者相辅相成。([Amazon Web Services, Inc.][2])
* **“法律 vs 框架”**：**Algorithmic Accountability Act** 目前是**法案提案**（Introduced），**NIST AI RMF** 是**自愿框架**；**EU AI Act** 属于**法规**（区域法），注意适用性差异。([Congress.gov][6])
* **“如何拿到 AWS 的合规材料？”**：**AWS Artifact**。**如何把证据结构化？**：**AWS Audit Manager**。([AWS ドキュメント][8])

---

## 8) 一页清单（回答面试官/审计官用）

1. **说出标准名**：ISO 27001/27017/27018/27701、ISO 42001、ISO 23894、SOC 1/2/3、NIST AI RMF、（美国）Algorithmic Accountability Act（法案动向）、（欧盟）AI Act。([Amazon Web Services, Inc.][1])
2. **拿到证据**：AWS 的 ISO/SOC 证书与报告 → **AWS Artifact**；你方控制的运行证据 → **AWS Audit Manager** 自动采集。([AWS ドキュメント][8])
3. **映射实践**：ISO 42001 / NIST AI RMF 的治理与风险流程 → 贯穿数据/模型/部署/监控（Model Cards + 审计日志 + Guardrails/Macie/KMS 等技术控制）。([Amazon Web Services, Inc.][9])
4. **法制敏感**：若在美国面向高风险领域，预备**影响评估**文档与流程（对齐 **Algorithmic Accountability Act** 的评估要素）。([Congress.gov][6])

---

### ✅ 一句话总括

> **识别并对齐标准 =（ISO/SOC 等“外部基准”）＋（NIST/ISO AI 风险与治理“方法论”）＋（AWS Artifact/Audit Manager“取证工具”）**；面对美国/欧盟法制再**补齐监管特有义务**（如影响评估/风险分类），即可把**AI 系统合规**从“口号”落到“证据链”。 ([Amazon Web Services, Inc.][1])

> 如果你愿意，我可以把**道路管理场景**下每个标准对应的**控制项—AWS 服务—证据产出物**做成一张“对照表”便于项目审计直接复用。

[1]: https://aws.amazon.com/compliance/iso-certified/?utm_source=chatgpt.com "ISO Certified"
[2]: https://aws.amazon.com/compliance/iso-27001-faqs/?utm_source=chatgpt.com "ISO/IEC 27001:2022 Compliance"
[3]: https://www.iso.org/standard/42001?utm_source=chatgpt.com "ISO/IEC 42001:2023 - AI management systems"
[4]: https://www.iso.org/standard/77304.html?utm_source=chatgpt.com "ISO/IEC 23894:2023 - AI — Guidance on risk management"
[5]: https://aws.amazon.com/compliance/soc-faqs/?utm_source=chatgpt.com "SOC Compliance - Amazon Web Services (AWS)"
[6]: https://www.congress.gov/bill/119th-congress/senate-bill/2164 "S.2164 - 119th Congress (2025-2026): Algorithmic Accountability Act of 2025 | Congress.gov | Library of Congress"
[7]: https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-ai-rmf-10?utm_source=chatgpt.com "Artificial Intelligence Risk Management Framework (AI ..."
[8]: https://docs.aws.amazon.com/artifact/latest/ug/what-is-aws-artifact.html?utm_source=chatgpt.com "What is AWS Artifact? - AWS Artifact"
[9]: https://aws.amazon.com/blogs/security/ai-lifecycle-risk-management-iso-iec-420012023-for-ai-governance/?utm_source=chatgpt.com "AI lifecycle risk management: ISO/IEC 42001:2023 for AI ..."
[10]: https://aws.amazon.com/blogs/machine-learning/building-trust-in-ai-the-aws-approach-to-the-eu-ai-act/?utm_source=chatgpt.com "Building trust in AI: The AWS approach to the EU AI Act"
[11]: https://docs.aws.amazon.com/audit-manager/latest/userguide/what-is.html?utm_source=chatgpt.com "AWS Audit Manager User Guide"
[12]: https://docs.aws.amazon.com/pdfs/audit-manager/latest/userguide/audit-manager-ug.pdf?utm_source=chatgpt.com "AWS Audit Manager - User Guide"
[13]: https://aws.amazon.com/artifact/?utm_source=chatgpt.com "Access AWS Compliance Reports On-Demand - AWS Artifact"
[14]: https://docs.aws.amazon.com/audit-manager/latest/userguide/SOC2.html?utm_source=chatgpt.com "SSAE-18 SOC 2 - AWS Audit Manager"
