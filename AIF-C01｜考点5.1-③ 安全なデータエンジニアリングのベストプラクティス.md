# AIF-C01｜5.1.3 **安全なデータエンジニアリングのベストプラクティス**

*AIF-C01 Domain 5, Task 5.1.3 — Secure data engineering best practices (data quality, privacy-enhancing tech, access control, integrity, etc.)*

> 目标：能**定义**与**应用**在 AWS 上构建 AI/ML 数据管线的安全实践：
> **数据质量评估／データ品質評価【ひんしつ ひょうか】／Data quality assessment**、
> **隐私增强技术／プライバシー強化技術【きょうか ぎじゅつ】／Privacy-enhancing technologies (PETs)**、
> **数据访问控制／データアクセス制御【せいぎょ】／Data access control**、
> **数据完整性／データ完全性【かんぜんせい】／Data integrity**。
> 三语标注：**中・日（读法）・英**；讲解以中文为主，并配**实际例子**与**官方文档依据**。

---

## 1) 数据质量评估（Data Quality Assessment）

**日**：データ品質評価【ひんしつ ひょうか】｜**EN**：Data quality assessment

### 1.1 AWS Glue Data Quality（托管规则+评分）

* **是什么**：在 **AWS Glue** 中对数据表**生成/编写规则集**（完整率、唯一性、范围、时效性等），并计算**数据质量得分**（通过规则比例）。可直接在 Glue Console 的 **Data quality** 标签页开通监控。([AWS ドキュメント][1])
* **自动推荐规则**：当你“不知道写什么规则”时，可触发 **Rule Recommendation Run**，Glue 会**扫描数据并推荐规则**，随后你再**审阅/调整**。([AWS ドキュメント][2])
* **典型场景**：对 **S3→Glue Catalog** 的道路巡检表（`roads_inspection`）设规则：

  * `defect_severity` ∈ {A,B,C,D}；
  * `capture_ts` 最近 30 天的比例 ≥ 95%；
  * `image_uri` 非空 & 可达。
    当得分低于阈值，触发报警，**阻断**进入训练/KB 的管线。

### 1.2 Deequ / PyDeequ（开源库，规则即代码）

* **是什么**：Amazon 开源的**基于 Spark 的数据质量库**，支持**约束检测**（Completeness、Uniqueness、Anomaly）与**度量指标**；可在 **EMR/Lambda/Step Functions** 等环境跑批。([Amazon Web Services, Inc.][3])
* **用法提示**：把“质量规则”版本化，放入 CI/CD；每次数据到来自动**质量门禁**（Quality Gate），不达标则中止下游任务。

> **考试要点**：
>
> * 托管：**Glue Data Quality**（可视化、评分、规则推荐）。([AWS ドキュメント][1])
> * 代码化：**Deequ/PyDeequ**（可嵌入无服务器/批处理管线）。([Amazon Web Services, Inc.][4])

---

## 2) 隐私增强技术（PETs）

**日**：プライバシー強化技術【きょうか ぎじゅつ】｜**EN**：Privacy-enhancing technologies

### 2.1 Amazon Macie（S3 中敏感数据检测）

* **作用**：对 S3 对象**自动/按需**检测 **PII/敏感数据**（支持**托管识别器**与**自定义识别器**），并出具报告；常用于**训练前清洗**与**RAG 入库前把关**。([AWS ドキュメント][5])
* **实践**：

  * 对 `s3://road-raw/` 开启 **自动发现**；
  * 对 `attachments/`、`ocr/` 等高风险路径周期性跑 **Discovery Job**；
  * 将发现事件接入告警/工单，走脱敏或隔离流程。

### 2.2 AWS Clean Rooms Differential Privacy（差分隐私）

* **作用**：在多方数据协作/聚合分析里，提供**差分隐私**保护（**隐私预算**、**添加噪声**、**聚合次数约束**），**无需**合作成员改写查询方式。([Amazon Web Services, Inc.][6])
* **适配**：当需要与外部伙伴共享**统计洞察**但不泄露个体时（比如按行政区统计路面病害率），用 **Clean Rooms + DP** 暴露**聚合**而非**明细**。

### 2.3 其他常用 PETs（与安全域联动）

* **假名化/トークナイズ**（Pseudonymization/Tokenization）：在入湖前替换可识别标识（车牌、工单号）。
* **最小化公开**：RAG 上线前，对 KB **过滤**含 PII 的页；对模型响应做 **Guardrails** 检测与遮罩（参见 3.2.4 节所学）。

> **小结**：**先检测（Macie）→ 再最小化（过滤/脱敏）→ 必要时用差分隐私开放聚合**，贯穿数据生命周期。([AWS ドキュメント][5])

---

## 3) 数据访问控制（Data Access Control）

**日**：データアクセス制御【せいぎょ】｜**EN**：Data access control

### 3.1 IAM + Lake Formation（细粒度授权）

* **Lake Formation 行级/列级权限**：在数据湖中对表**行级**（Row-level）与**列级**（Column-level）实施访问限制，是**推荐**的细粒度授权方法；支持集中审计。([AWS ドキュメント][7])
* **典型规则**：

  * 只允许“北区维护组”访问 `region = 'Kita'` 行；
  * 对“客户服务台”隐藏 `owner_name`、`phone` 等列。
* **与 IAM 的关系**：IAM 负责**身份/动作**层的允许/拒绝，Lake Formation 负责**数据层资源**（库/表/位置/行列）授权与治理。([AWS ドキュメント][7])

### 3.2 私网访问（PrivateLink）

* **PrivateLink**：在你的 **VPC** 里创建**接口端点**，通过**私有 IP**访问 AWS 托管服务或对等账户服务；无需 IGW/NAT/公网 IP，且可用 **Security Group** 控制源。适用于**Bedrock、S3 等**服务的私网调用。([AWS ドキュメント][8])
* **常见做法**：给调用 Bedrock 的应用角色附上 **`aws:SourceVpce` 条件**，限定**只能经 VPC 端点**调用模型（与 5.1.1 呼应）。

### 3.3 加密与密钥（KMS）

* **KMS**：集中**创建/管理**数据加密密钥（FIPS 140-3 L3 HSM 保护；密钥从不以明文离开 KMS）。广泛用于 S3、EBS、RDS、Glue、SageMaker 等服务。([AWS ドキュメント][9])
* **实践**：S3 桶→默认 **SSE-KMS（CMK）**；SageMaker 训练/端点**同一把 CMK**；跨账户共享时用 **CMK + 资源策略**与 **Lake Formation** 权限联动。

---

## 4) 数据完整性（Data Integrity）

**日**：データ完全性【かんぜんせい】｜**EN**：Data integrity

### 4.1 S3 校验与校验和（Checksums）

* **作用**：在上传/下载/分段上传时，用 **MD5/SHA-1/SHA-256/CRC32** 等**校验和**核对传输完整性；S3 支持**计算与验证**，并可对分段上传**计算全对象校验**。([AWS ドキュメント][10])
* **建议**：对关键影像/标注文件启用**强校验**，并在 ETL 作业结束后**比对记录**（Glue/Spark 任务写入度量表）。

### 4.2 S3 Object Lock（WORM 防篡改）

* **作用**：以 **WORM（一次写入，多次读取）**方式**冻结**对象，**防删除与覆盖**，支持**合规/治理**保留期；适合应对**误删/勒索**与**审计保全**。([AWS ドキュメント][11])
* **实践**：对**原始采集影像**与**模型评估基准数据集**启用 **Object Lock**（合规桶 + 版本控制），确保事实来源可追溯。

---

## 5) 端到端“安全数据工程”蓝图（把要点串起来）

**例子**：*道路影像识别 + 企业 RAG 报告*

1. **编目**：S3 → **Glue Data Catalog** 建库/表；Crawler 每日更新。([AWS ドキュメント][12])
2. **质量门禁**：**Glue Data Quality** 规则 + 分数阈值（<90% 直接阻断）；对复杂规则用 **PyDeequ** 扩展。([AWS ドキュメント][1])
3. **隐私治理**：**Macie** 自动发现 S3 中 PII；高风险路径跑作业；RAG 入库前清理；与**Clean Rooms DP** 共享**聚合**指标（非明细）。([AWS ドキュメント][5])
4. **访问控制**：**Lake Formation** 行列级授权（按区域/角色）＋ **IAM** 最小权限；对 **Bedrock** 走 **PrivateLink** 并以 `aws:SourceVpce` 收紧来源。([AWS ドキュメント][13])
5. **加密与密钥**：S3/SageMaker/向量库统一 **SSE-KMS（CMK）**；禁止明文导出。([AWS ドキュメント][9])
6. **完整性**：S3 **Checksums** 验收 + **Object Lock** 保全关键版本。([AWS ドキュメント][10])

---

## 6) 备考一页通（中・日・英）

| 主题   | 日文（读法）                | English        | 关键 AWS 服务/要点                                                                  |
| ---- | --------------------- | -------------- | ----------------------------------------------------------------------------- |
| 数据质量 | データ品質評価【ひんしつ ひょうか】    | Data quality   | **Glue Data Quality**（规则/评分/推荐）; **Deequ**（约束即代码）。([AWS ドキュメント][1])           |
| 隐私增强 | プライバシー強化技術【きょうか ぎじゅつ】 | PETs           | **Macie**（PII 扫描）；**Clean Rooms DP**（隐私预算/噪声）。([AWS ドキュメント][5])               |
| 访问控制 | データアクセス制御【せいぎょ】       | Access control | **Lake Formation**（行/列级权限）；**PrivateLink**（私网）；**KMS**（CMK）。([AWS ドキュメント][7]) |
| 完整性  | データ完全性【かんぜんせい】        | Data integrity | **S3 Checksums**；**S3 Object Lock（WORM）**。([AWS ドキュメント][10])                  |

---

## 7) 实操清单（Checklist，直接照抄即可）

* [ ] **Glue Data Quality**：为关键表开启规则与监控；设置**得分阈值**与报警。([AWS ドキュメント][12])
* [ ] **PyDeequ**：将规则代码化并纳入 CI/CD；构建**质量门禁**。([Amazon Web Services, Inc.][4])
* [ ] **Macie**：自动发现 + 高风险路径扫描；把发现事件接入告警/隔离流程。([AWS ドキュメント][5])
* [ ] **Clean Rooms DP**：对外共享仅限**聚合**；配置**隐私预算**与**噪声级别**。([AWS ドキュメント][14])
* [ ] **Lake Formation**：按部门/区域设**行级/列级**查看权限；集中审计。([AWS ドキュメント][7])
* [ ] **PrivateLink**：Bedrock/S3 访问走私网；IAM 条件键限制 `aws:SourceVpce`。([AWS ドキュメント][8])
* [ ] **KMS**：S3/训练/端点统一 **CMK**；禁止明文导出；按需做密钥轮换。([AWS ドキュメント][9])
* [ ] **S3 完整性**：启用 **Checksums**；关键数据用 **Object Lock** 防篡改与误删。([AWS ドキュメント][10])

---

### ✅ 一句话总括

> **安全数据工程 = 质（规则与评分）+ 私（检测与最小化）+ 控（细粒度授权与私网）+ 真（校验与防篡改）**。
> 在 AWS 上，使用 **Glue Data Quality / Deequ** 把关质量，**Macie / Clean Rooms DP** 保障隐私，**Lake Formation / PrivateLink / KMS** 控制访问与加密，**S3 Checksums / Object Lock** 维护完整性；把这些**工程化为模板与门禁**，即可稳定支撑 AI/ML 的**可靠、合规、可审计**的数据底座。

[1]: https://docs.aws.amazon.com/glue/latest/dg/glue-data-quality.html?utm_source=chatgpt.com "AWS Glue Data Quality"
[2]: https://docs.aws.amazon.com/glue/latest/dg/aws-glue-api-data-quality-api.html?utm_source=chatgpt.com "Data Quality API - AWS Glue"
[3]: https://aws.amazon.com/blogs/big-data/test-data-quality-at-scale-with-deequ/?utm_source=chatgpt.com "Test data quality at scale with Deequ"
[4]: https://aws.amazon.com/blogs/big-data/build-a-serverless-data-quality-pipeline-using-deequ-on-aws-lambda/?utm_source=chatgpt.com "Build a serverless data quality pipeline using Deequ on ..."
[5]: https://docs.aws.amazon.com/macie/latest/user/data-classification.html?utm_source=chatgpt.com "Discovering sensitive data with Macie"
[6]: https://aws.amazon.com/clean-rooms/differential-privacy/?utm_source=chatgpt.com "AWS Clean Rooms Differential Privacy"
[7]: https://docs.aws.amazon.com/lake-formation/latest/dg/access-control-fine-grained.html?utm_source=chatgpt.com "Methods for fine-grained access control"
[8]: https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html?utm_source=chatgpt.com "What is AWS PrivateLink? - Amazon Virtual Private Cloud"
[9]: https://docs.aws.amazon.com/kms/latest/developerguide/overview.html?utm_source=chatgpt.com "AWS Key Management Service"
[10]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/checking-object-integrity.html?utm_source=chatgpt.com "Checking object integrity in Amazon S3"
[11]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html?utm_source=chatgpt.com "Locking objects with Object Lock"
[12]: https://docs.aws.amazon.com/glue/latest/dg/data-quality-getting-started.html?utm_source=chatgpt.com "Getting started with AWS Glue Data Quality for the Data Catalog"
[13]: https://docs.aws.amazon.com/lake-formation/latest/dg/cbac-tutorial.html?utm_source=chatgpt.com "Securing data lakes with row-level access control"
[14]: https://docs.aws.amazon.com/clean-rooms/latest/userguide/dp-settings.html?utm_source=chatgpt.com "Differential privacy policy - AWS Clean Rooms"
