# AIF-C01｜考点 1.3：ML 開発ライフサイクルについて説明する（中・日・英对照，日语含读法）

> 目标：熟悉 **机器学习开发生命周期** 的各阶段、每阶段要做什么、常见方法、以及在 **AWS** 上该用哪些托管功能（SageMaker 系列）。
> 记忆策略：**“收—看—洗—造—训—调—评—上—监”** 九步法 + “每步对应一个或几个 AWS 功能”。

---

## 0) 一页总览（背会就能拿分）

| 阶段     | 日文（读音）                 | English               | 你要做什么                            | 常用 AWS                                                                                |
| ------ | ---------------------- | --------------------- | -------------------------------- | ------------------------------------------------------------------------------------- |
| 数据收集   | データ収集【しゅうしゅう】          | Data Collection       | 汇聚原始数据                           | S3 / Glue / Kinesis                                                                   |
| 探索性分析  | 探索的データ分析【たんさくてき…】      | EDA                   | 看分布/缺失/异常                        | SageMaker Studio, **Data Wrangler** ([AWSドキュメント][1])                                  |
| 前处理    | データ前処理【まえしょり】          | Preprocessing         | 清洗、标准化、编码                        | **Data Wrangler**（GUI 流程） ([AWSドキュメント][1])                                            |
| 特征工程   | 特徴量エンジニアリング（とくちょうりょう…） | Feature Engineering   | 生成可训练的特征                         | **Feature Store**（特征仓库） ([AWSドキュメント][2])                                              |
| 训练     | モデルトレーニング【とれーにんぐ】      | Model Training        | 选择算法并拟合参数                        | SageMaker Training / JumpStart / Autopilot ([AWSドキュメント][3])                           |
| 超参调优   | ハイパーパラメータ…チューニング       | Hyperparameter Tuning | 搜最佳学习率/树深等                       | **Automatic Model Tuning (AMT)**（Bayesian/Random/Grid） ([AWSドキュメント][4])               |
| 评价     | 評価【ひょうか】               | Evaluation            | 指标：Accuracy/F1/AUC 等             | Studio/Processing；Autopilot 报告 ([AWSドキュメント][5])                                       |
| 部署     | デプロイ【でぷろい】             | Deployment            | Batch/Real-time/Async/Serverless | Batch Transform / Realtime EP / **Async** / **Serverless** ([AWSドキュメント][6])           |
| 监控     | モニタリング（もにたりんぐ）         | Monitoring            | 监控数据/模型/偏差/漂移                    | **Model Monitor** / CloudWatch / Model Dashboard ([AWSドキュメント][7])                     |
| 编排（贯穿） | パイプライン                 | Pipelines             | 把各步连成可复现流程                       | **SageMaker Pipelines**（DAG/可视化/SDK） ([AWSドキュメント][8], [Amazon Web Services, Inc.][9]) |

---

## 1) データ収集｜Data Collection（中文讲解）

**日/读**：データ収集【しゅうしゅう】
**EN**：Data Collection
**做什么**：从数据库/日志/物联网/第三方接口/文件系统抓取原始数据，落到对象存储或数据湖。
**要点**：明确**数据域**与**主键/时间戳**；保留**原始不可变**版本。
**AWS**：Amazon S3（数据湖）、AWS Glue（ETL 调度）、Amazon Kinesis（流数据）。
**考试提示**：没有高质量数据，后续都白搭（Garbage in, garbage out）。

---

## 2) 探索的データ分析 (EDA)｜Exploratory Data Analysis

**读**：たんさくてき でーた ぶんせき
**EN**：EDA
**做什么**：概览统计（均值/方差/分位数）、可视化（直方图、箱线图）、相关性与异常值识别。
**AWS**：

* **SageMaker Studio Notebooks**：用 Pandas/Matplotlib 做 EDA；
* **SageMaker Data Wrangler**：一站式**导入-分析-质量报告-可视化**（少写代码，也能自定义脚本）。([AWSドキュメント][1], [Amazon Web Services, Inc.][10])
  **考试关键词**：*EDA / visualization / data quality report / anomaly / leakage*。

---

## 3) データの前処理｜Data Preprocessing

**读**：まえしょり
**EN**：Preprocessing
**做什么**：缺失值处理、数值标准化/归一化、类别编码、文本清洗、采样/分割训练-验证-测试集。
**AWS**：**Data Wrangler** 支持多源导入（Athena/Redshift/S3 等）、自动类型推断与转换步骤**可视化流水**；可将流程直接导出到 SageMaker Pipeline/Processing 作业复用。([AWSドキュメント][11])
**考试关键词**：*normalize / encode / train-validation-test split / leakage*。

---

## 4) 特徴量エンジニアリング｜Feature Engineering

**读**：とくちょうりょう えんじにありんぐ
**EN**：Feature Engineering
**做什么**：从原始数据构造更有表达力的输入（日期衍生“星期/月份”、滚动统计、交叉特征、文本/图像嵌入等）。
**AWS**：**SageMaker Feature Store** 作为**特征仓库**，集中**创建-存储-共享-管理**特征，供训练与线上推理一致使用（在线/离线存储，降低重复处理）。([AWSドキュメント][2])
**考试关键词**：*feature store / online & offline store / time-travel / reuse*。

---

## 5) モデルトレーニング｜Model Training

**读**：とれーにんぐ
**EN**：Model Training
**做什么**：选任务（回归/分类/聚类…）、选算法/框架（XGBoost、TensorFlow、PyTorch…），在训练集上拟合参数。
**AWS**：SageMaker Training Jobs；也可用 **JumpStart/Autopilot** 快速起步（AutoML）。([AWSドキュメント][3])
**考试关键词**：*training job / estimator / algorithm / framework*。

---

## 6) ハイパーパラメータのチューニング｜Hyperparameter Tuning

**读**：はいぱーぱらめーたー ちゅーにんぐ
**EN**：Hyperparameter Tuning
**做什么**：搜索外部控制参数（学习率、批大小、树深度等），用验证集指标（F1/AUC/RMSE）选最佳组合。
**AWS**：**SageMaker Automatic Model Tuning (AMT)** 支持 **Random / Bayesian /（条件下）Grid** 等策略，自动并发多次训练，按目标指标选最优。([AWSドキュメント][4])
**考试关键词**：*HPO / AMT / objective metric / search strategy*。

---

## 7) 評価｜Evaluation

**读**：ひょうか
**EN**：Evaluation
**做什么**：用**独立验证/测试集**评估性能；分类看 Accuracy/Precision/Recall/F1/ROC-AUC，回归看 MSE/RMSE/MAE；绘制混淆矩阵、PR/ROC 曲线。
**AWS**：Studio/Processing 作业；**Autopilot** 生成模型性能报告与可视化指标。([AWSドキュメント][5])
**考试关键词**：*hold-out / confusion matrix / AUC / F1*。

---

## 8) デプロイ｜Deployment

**读**：でぷろい
**EN**：Deployment（选择合适的“推理形态”）

* **Batch Transform（バッチ推論）**：离线批量处理，结果写回 S3，适合“量大不急”。([AWSドキュメント][6])
* **Real-time Endpoint（リアルタイム推論）**：低延迟在线接口，支持自动扩缩与监控。([AWSドキュメント][12])
* **Asynchronous Inference（非同期推論）**：请求排队、长耗时/大载荷（最高 1GB）更合适，可空闲归零省成本。([AWSドキュメント][13])
* **Serverless Inference（サーバレス推論）**：按请求计费、免管实例，低频/突发负载理想；有容器/并发限制。([AWSドキュメント][14])
  **考试关键词**：*batch vs real-time vs async vs serverless / latency vs throughput / cost*。

---

## 9) モニタリング｜Monitoring

**读**：もにたりんぐ
**EN**：Monitoring
**做什么**：上线后持续监控**数据质量、模型质量、偏差漂移（bias drift）、特征归因漂移**，超阈值报警并触发再训练。
**AWS**：**SageMaker Model Monitor**（对实时端点、批处理定期检测；与 Clarify 集成做偏差监控）、Model Dashboard 汇总告警，CloudWatch 收集指标/日志。([AWSドキュメント][7])
**考试关键词**：*data drift / concept drift / baseline / alert / retraining trigger*。

---

## 10) パイプライン & 実験管理（贯穿全流程）

* **Pipelines（パイプライン） / Orchestration**：把“前处理→特征→训练→评估→部署→监控”编排成 **DAG**，可视化或用 SDK 定义，**可复现、可审计、可回滚**。([AWSドキュメント][8], [Amazon Web Services, Inc.][9])
* **Experiments（実験管理）**：记录每次实验的参数/数据版本/指标，方便比较最佳方案（与 Pipelines 常配合）。
  **记忆**：**把一次性脚本变成流水线**，MLOps 的核心。

---

## 11) 与业务指标的联动（考试常考）

* 技术好 ≠ 业务好。上线前后同时看 **模型指标**（Accuracy/F1/AUC 等）与 **业务指标**（单次推理成本、用户满意度、ROI）。
* 例：选择 **Async/Serverless** 可在**低频调用**下降本；**Batch** 适合“夜间全量”，**Realtime** 适合交互。（结合 8) 的形态与 9) 的监控一起作答更稳。）

---

## 12) 术语小词典（中・日・英）

* **Pipeline**：パイプライン / Pipeline（把步骤连成可复现流程）
* **Feature Store**：フィーチャーストア / Feature Store（特征仓库，在线/离线）
* **HPO**：ハイパーパラメータ最適化 / Hyperparameter Optimization
* **Drift**：ドリフト / Drift（数据/概念分布变化）
* **Endpoint**：エンドポイント / Endpoint（模型在线服务）

---

### 复习口令

> **“收—看—洗—造—训—调—评—上—监；管线连起来，指标一起看。”**
> 看到题干先判断**所处阶段**，再映射**对应 AWS 功能**，最后考虑**部署形态 + 监控策略**，十拿九稳。

——
**参考（官方文档）**：Pipelines（流程编排/DAG） ([AWSドキュメント][8], [Amazon Web Services, Inc.][9])；Data Wrangler（前处理/EDA/可视化） ([AWSドキュメント][1])；Feature Store（特征仓库） ([AWSドキュメント][2])；Model Monitor（生产监控/漂移） ([AWSドキュメント][7])；Autopilot（AutoML）与 HPO（AMT） ([AWSドキュメント][3])；部署形态：Batch/Realtime/Async/Serverless ([AWSドキュメント][6])。

[1]: https://docs.aws.amazon.com/sagemaker/latest/dg/data-wrangler.html?utm_source=chatgpt.com "Prepare ML Data with Amazon SageMaker Data Wrangler"
[2]: https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store.html?utm_source=chatgpt.com "Create, store, and share features with Feature Store"
[3]: https://docs.aws.amazon.com/sagemaker/latest/dg/autopilot-automate-model-development.html?utm_source=chatgpt.com "SageMaker Autopilot"
[4]: https://docs.aws.amazon.com/sagemaker/latest/dg/automatic-model-tuning.html?utm_source=chatgpt.com "Automatic model tuning with SageMaker AI"
[5]: https://docs.aws.amazon.com/sagemaker/latest/dg/autopilot-model-insights.html?utm_source=chatgpt.com "View an Autopilot model performance report"
[6]: https://docs.aws.amazon.com/sagemaker/latest/dg/batch-transform.html?utm_source=chatgpt.com "Batch transform for inference with Amazon SageMaker AI"
[7]: https://docs.aws.amazon.com/sagemaker/latest/dg/model-monitor.html?utm_source=chatgpt.com "Data and model quality monitoring with ..."
[8]: https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines.html?utm_source=chatgpt.com "Pipelines - Amazon SageMaker AI"
[9]: https://aws.amazon.com/sagemaker/ai/pipelines/?utm_source=chatgpt.com "Amazon SageMaker Pipelines"
[10]: https://aws.amazon.com/sagemaker/ai/data-wrangler/?utm_source=chatgpt.com "Amazon SageMaker Data Wrangler"
[11]: https://docs.aws.amazon.com/sagemaker/latest/dg/data-wrangler-import.html?utm_source=chatgpt.com "Import - Amazon SageMaker AI - AWS Documentation"
[12]: https://docs.aws.amazon.com/sagemaker/latest/dg/realtime-endpoints.html?utm_source=chatgpt.com "Real-time inference - Amazon SageMaker AI"
[13]: https://docs.aws.amazon.com/sagemaker/latest/dg/async-inference.html?utm_source=chatgpt.com "Asynchronous inference - Amazon SageMaker AI"
[14]: https://docs.aws.amazon.com/sagemaker/latest/dg/serverless-endpoints.html?utm_source=chatgpt.com "Deploy models with Amazon SageMaker Serverless ..."
