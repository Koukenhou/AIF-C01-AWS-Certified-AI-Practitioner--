# AIF-C01｜5.2.3 **データガバナンス戦略（ライフサイクル／ログ／レジデンシー／モニタリング／オブザーバビリティ／保持）**

*AIF-C01 Domain 5, Task 5.2.3 — Explain data governance strategy (data lifecycle, logging, residency, monitoring, observability, retention)*

> 目标：掌握在 AWS 上如何把**数据从产生到归档的全生命周期**进行**可治理、可审计、可留存**的工程化设计。下面按六个要素展开，并给出**中・日（片假名/读法）・英**术语、**官方文档依据**与**道路影像×生成式报告**的实战示例。

---

## 0) 术语速查（中・日・英）

| 主题   | 日文（读法）          | English         | 一句话定位                                                        |
| ---- | --------------- | --------------- | ------------------------------------------------------------ |
| 数据治理 | データガバナンス【がばなんす】 | Data Governance | 以**政策/流程/技术**确保数据“可用、可控、可审”。([Amazon Web Services, Inc.][1]) |
| 生命周期 | データライフサイクル      | Data lifecycle  | 采集→处理→使用→归档→删除。                                              |
| 数据常驻 | データレジデンシー       | Data residency  | 数据**驻留/处理**所在的**区域与边界**。([AWS ドキュメント][2])                    |
| 监控   | モニタリング          | Monitoring      | 指标/日志/事件**采集 + 告警**。([AWS ドキュメント][3])                        |
| 可观测性 | オブザーバビリティ       | Observability   | **指标/日志/追踪**三件套与端到端调用链。([AWS ドキュメント][4])                     |
| 留存   | 保持【ほじ】          | Retention       | **保存期限/保全**（WORM、法务留存、审计）。([AWS ドキュメント][5])                  |

---

## 1) 数据生命周期（Data Lifecycle）

**日**：データライフサイクル｜**EN**：Data lifecycle

### 1.1 S3 生命周期策略（冷热分层与到期删除）

* 用 **S3 Lifecycle** 规则在对象达到指定天数后**自动转储/到期删除**（示例：日志 30 天转低频、90 天归档、365 天删除）。([AWS ドキュメント][6])
* 典型归档目标：**S3 Glacier** 系列存储类：**Instant Retrieval / Flexible Retrieval / Deep Archive**（取回时延与成本不同；注意最短计费周期：Flexible 90 天、Deep Archive 180 天）。([AWS ドキュメント][7])

### 1.2 数据分层与“冷冻”

* 热数据：S3 Standard；温：Standard-IA；冷：Glacier 族；极冷：Deep Archive。根据**访问频率/恢复时延/合规期**选择。([AWS ドキュメント][7])

**实战例（道路影像）**

* 原始影像（近 30 天）留在 **S3 Standard** 便于标注与复核；
* 30–180 天转 **Standard-IA**；
* 180 天后转 **Glacier Deep Archive**；
* 事故取证集单独桶，并启用“**不可变保全**”，见 §6。([AWS ドキュメント][6])

---

## 2) 日志与审计（Logging & Audit）

**日**：ログ記録【ろく きろく】｜**EN**：Logging

### 2.1 CloudTrail：统一审计“谁做了什么”

* 记录**账户级**API/用户活动；事件类型包含 **管理事件 / 数据事件 / ネットワーク活動（Network activity）/ Insights 异常**。缺省仅开管理事件；数据/网络活动/Insights 需按需开启。([AWS ドキュメント][8])
* **最佳实践**：建**组织级、跨区** trail 记录管理事件；为 S3/Lambda 等关键资源**单独加数据事件**；需要长周期留存与可查询，用 **CloudTrail Lake**（SQL 查询、定价与默认/最大留存依所选档位而异）。([AWS ドキュメント][9])

### 2.2 CloudWatch Logs & Alarms：把日志变成可告警信号

* 给日志组设置 **Metric Filter** 并创建 **Alarm**，把“**推理失败率**、**越权访问**、**RAG 检索失败**”等从日志转为度量并告警。([AWS ドキュメント][10])

**实战例（道路影像×生成式）**

* 对 **bedrock\:InvokeModel**、S3 桶策略变更、SageMaker 端点更新，开启 CloudTrail 数据/管理事件；把“调用失败 5xx 比例”做成 CloudWatch 告警；关键问责查询用 **CloudTrail Lake** 保存与检索。([AWS ドキュメント][8])

---

## 3) 数据常驻与边界（Data Residency & Boundaries）

**日**：データレジデンシー｜**EN**：Data residency

### 3.1 选区与隔离

* 按**合规与业务**要求选择 **分区（aws / aws-cn / aws-us-gov）与 Region**；使用 **多账户/多组织**隔离不同敏感度的数据与工作负载。([AWS ドキュメント][11])
* **Well-Architected Data Residency Lens** 给出在**混合云/多区**下实现常驻的系统性做法。([AWS ドキュメント][2])

### 3.2 常驻控制与防外泄

* **Control Tower 的 Data Residency 控制**：在 OU 层面实施**检测/阻断**跨指定 Region 的创建/复制/共享（多账号统一治理）。([AWS ドキュメント][12])
* 生成式应用补充：**Bedrock** 支持 CloudWatch/CloudTrail 监控且处于多项合规范围（ISO、SOC、HIPAA 等）；**不应在标签/自由文本**写入机密（会进入记账/诊断日志）。([Amazon Web Services, Inc.][13])

**实战例**

* 把“**大阪市道路数据**”限定在 `ap-northeast-1`，用 **OU 级 Data Residency 控制**防误扩散；Bedrock 访问全程私网（与 §5 网络治理联动），日志纳入 CloudTrail。([AWS ドキュメント][12])

---

## 4) 监控（Monitoring）与可观测性（Observability）

**日**：モニタリング／オブザーバビリティ｜**EN**：Monitoring / Observability

### 4.1 CloudWatch：指标、日志、告警的统一面板

* CloudWatch 原生收集多数资源指标；可创建**复合告警**与**跨账户/跨区域**观测视图。([AWS ドキュメント][14])

### 4.2 OpenTelemetry（ADOT）＋ X-Ray：端到端追踪

* 用 **AWS Distro for OpenTelemetry (ADOT)** 采集**指标/日志/追踪**并投递到 **X-Ray / CloudWatch / OpenSearch / Managed Service for Prometheus**，构建**调用链**与瓶颈定位；结合 **CloudWatch RUM** 做前端真实用户监测。([AWS ドキュメント][15])

**实战例**

* 为“**上传影像 → 预处理 → SageMaker 推理 → RAG 检索 → Bedrock 生成**”串起 **X-Ray Trace**，发现高延迟集中在“RAG 检索”；据此调整向量库/检索并发，同时在 CloudWatch 建 TPS/错误率告警。([AWS ドキュメント][15])

---

## 5) 保持与合规留存（Retention & Preservation）

**日**：保持【ほじ】｜**EN**：Retention

### 5.1 S3 Object Lock（WORM）与法务留存

* **Object Lock** 提供 **Retention Period（固定期限）** 与 **Legal Hold（法务留存，无固定期限）** 两种方式，防止对象版本被删除或覆盖；可批量操作与权限细分。适合审计/取证/监管要求的**不可变**存档。([Amazon Web Services, Inc.][16])

### 5.2 CloudTrail Lake：审计事件的长期留存与查询

* 把跨账号/跨区的活动事件沉淀到 **CloudTrail Lake** 的 **Event Data Store**，支持 **SQL 查询、ORC 列式存储**、按定价档位设置默认/最大留存期。适合“**保存 1–7 年**并随时检索审计证据”的要求。([AWS ドキュメント][17])

**实战例**

* “事故调查集” → **S3 Object Lock（合规模式）** 保全 7 年；
* 审计活动 → **CloudTrail Lake** 留存并提供按“谁在什么时候改了端点配置”进行 SQL 检索的能力。([AWS ドキュメント][17])

---

## 6) 组织级治理拼图（Catalog／Access／Policy）

> 此节与 5.1.2/5.2.2 呼应，放在**策略蓝图**里一起背。

* **Amazon DataZone（データゾーン）**：企业数据目录与共享治理，支持**订阅审批/跨账户与跨区访问**、对数据资产（S3/Glue/Redshift/模型等）的**发现/编目/访问治理**；越来越多官方方案展示将 DataZone 与数据湖/流数据打通。([Amazon Web Services, Inc.][18])
* **AWS Organizations + Config Conformance Pack**：多账号统一基线（加密、阻断公共访问、VPC-only 等）与自动整改（SSM Automation）。([AWS ドキュメント][19])

---

## 7) 一页蓝图：把六要素落到你的 AI/ML 平台

1. **生命周期**：S3 Lifecycle 规则把**原始→清洗→特征**分层存储与到期删除；冷数据进 Glacier。([AWS ドキュメント][6])
2. **日志**：CloudTrail（组织级、跨区、含数据/网络活动）＋ CloudWatch（日志→指标过滤→告警）。([AWS ドキュメント][8])
3. **常驻**：限定 Region，启用 **Control Tower 数据常驻控制**防“跨区外流”；生成式应用遵从 Bedrock 安全与日志实践。([AWS ドキュメント][12])
4. **监控**：CloudWatch 指标/告警 + 指标洞察；
5. **可观测**：ADOT + X-Ray 端到端追踪，RUM 采集前端性能。([AWS ドキュメント][4])
6. **保持**：S3 Object Lock（法务/合规留存）＋ CloudTrail Lake（长期审计仓）。([Amazon Web Services, Inc.][16])
7. **治理**：DataZone 数据目录与订阅审批；Organizations + Config 一致性包做“**基线即代码**”。([Amazon Web Services, Inc.][18])

---

## 8) 备考“关键词—服务—动作”表

| 关键词（中・日・英）                      | 选型                  | 关键动作                                                                         |
| ------------------------------- | ------------------- | ---------------------------------------------------------------------------- |
| 生命周期／ライフサイクル／Lifecycle          | **S3 Lifecycle**    | 规则：转 IA/Glacier、到期删除；注意 Glacier 最短计费周期。([AWS ドキュメント][6])                     |
| 日志／ログ／Logging                   | **CloudTrail**      | 事件类型（管理/数据/网络/Insights）与组织级 trail；CloudTrail Lake 长期留存+SQL。([AWS ドキュメント][8]) |
| 监控／モニタリング／Monitoring            | **CloudWatch**      | 日志→指标过滤→告警；复合/跨账户告警。([AWS ドキュメント][10])                                       |
| 可观测性／オブザーバビリティ／Observability    | **ADOT + X-Ray**    | 采集指标/日志/追踪，构建端到端调用链；RUM 采集前端。([AWS ドキュメント][15])                              |
| 常驻／レジデンシー／Residency             | **Control Tower**   | Data Residency 控制（OU 级检测与抑制跨区外流）。([AWS ドキュメント][12])                          |
| 保持／保持／Retention                 | **S3 Object Lock**  | Retention Period / Legal Hold（WORM），配合审计取证。([Amazon Web Services, Inc.][16]) |
| 目录与共享／カタログと共有／Catalog & Sharing | **Amazon DataZone** | 资产编目、订阅审批、跨账户/跨区访问控制。([Amazon Web Services, Inc.][20])                       |

---

## 9) 小结（记忆法）

> **“生（生命周期）—记（日志）—驻（常驻）—看（监控/可观测）—存（保持）—治（治理）”**
> 在 AWS：**S3 Lifecycle/Glacier**（生）→ **CloudTrail + CloudWatch**（记/看）→ **Control Tower 常驻控制**（驻）→ **ADOT+X-Ray**（看/可观测）→ **S3 Object Lock + CloudTrail Lake**（存）→ **DataZone + Organizations/Config**（治）。这套组合把数据治理做成“**政策即代码／证据即查询**”。

如需，我可以把以上蓝图改造成\*\*“道路管理部门落地清单（逐项复核）”\*\*版本，便于你直接贴到项目 Wiki 使用。

[1]: https://aws.amazon.com/what-is/data-governance/?utm_source=chatgpt.com "What is Data Governance?"
[2]: https://docs.aws.amazon.com/wellarchitected/latest/data-residency-hybrid-cloud-services-lens/data-residency-with-hybrid-cloud-services-lens.html?utm_source=chatgpt.com "Data Residency with Hybrid Cloud Services Lens"
[3]: https://docs.aws.amazon.com/prescriptive-guidance/latest/logging-monitoring-for-application-owners/cloudwatch.html?utm_source=chatgpt.com "Application logging and monitoring using Amazon CloudWatch"
[4]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-OpenTelemetry-Sections.html?utm_source=chatgpt.com "OpenTelemetry - Amazon CloudWatch"
[5]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html?utm_source=chatgpt.com "Locking objects with Object Lock"
[6]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html?utm_source=chatgpt.com "Managing the lifecycle of objects"
[7]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/glacier-storage-classes.html?utm_source=chatgpt.com "Understanding S3 Glacier storage classes for long-term data ..."
[8]: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-events.html?utm_source=chatgpt.com "Understanding CloudTrail events"
[9]: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/best-practices-security.html?utm_source=chatgpt.com "Security best practices in AWS CloudTrail"
[10]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Alarm-On-Logs.html?utm_source=chatgpt.com "Alarming on logs - Amazon CloudWatch - AWS Documentation"
[11]: https://docs.aws.amazon.com/prescriptive-guidance/latest/strategy-aws-semicon-workloads/meeting-data-residency-requirements.html?utm_source=chatgpt.com "Meeting data residency requirements on AWS"
[12]: https://docs.aws.amazon.com/controltower/latest/controlreference/data-residency-controls.html?utm_source=chatgpt.com "Controls that enhance data residency protection"
[13]: https://aws.amazon.com/bedrock/security-compliance/?utm_source=chatgpt.com "Amazon Bedrock Security and Privacy - AWS"
[14]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html?utm_source=chatgpt.com "Metrics in Amazon CloudWatch"
[15]: https://docs.aws.amazon.com/xray/latest/devguide/xray-services-adot.html?utm_source=chatgpt.com "AWS Distro for OpenTelemetry and AWS X-Ray"
[16]: https://aws.amazon.com/s3/features/object-lock/?utm_source=chatgpt.com "Amazon S3 Object Lock"
[17]: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-lake.html?utm_source=chatgpt.com "Working with AWS CloudTrail Lake"
[18]: https://aws.amazon.com/datazone/?utm_source=chatgpt.com "Govern Analytics – Amazon DataZone – AWS"
[19]: https://docs.aws.amazon.com/whitepapers/latest/aws-overview/management-governance.html?utm_source=chatgpt.com "AWS Management and Governance category icon"
[20]: https://aws.amazon.com/datazone/features/data-access/?utm_source=chatgpt.com "Amazon DataZone: Govern Data Access"
