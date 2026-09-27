---
title: Domain 3 設計高效能架構
tags:
  - AWS/SAA-C03
  - exam/d3
  - checklist
status: 未讀
confidence: 1
importance: 5
updated: 2026-09-27
---

# Domain 3：Design High-Performing Architectures（24%）

> [!abstract] 這篇怎麼用
> 官方 task statement 的逐條拆解 + 對應筆記。**約 12 道計分題。**
> 這個 domain **有五個 task statement，是四個 domain 中最寬的**——涵蓋儲存、運算、資料庫、網路、資料擷取五條線。好處是每條線的題目都很「選型」，靠決策樹就能拿分。

---

## Task 3.1：Determine high-performing and/or scalable storage solutions

- [ ] 混合儲存方案 → [[遷移與混合雲]]（Storage Gateway 四型）
- [ ] 儲存服務與適用情境（**S3、EFS、EBS**） → [[EC2與儲存]]、[[S3與資料生命週期]]
- [ ] 儲存類型特性（**object / file / block**） → [[決策樹 儲存選型]]
- [ ] 決定符合效能需求的儲存服務與組態
- [ ] 決定可擴展以因應未來需求的儲存服務

> [!important] 高頻考點
> **S3 不是 POSIX**（看到 `mount`/`shared directory` 別選）｜**EFS = Linux/NFS，FSx for Windows = SMB/AD，FSx for Lustre = HPC/ML**｜**EBS Multi-Attach 僅 io1/io2 同 AZ 且需 cluster-aware FS**｜**gp3 可獨立調 IOPS，gp2 不行**｜**st1/sc1 不能開機**

---

## Task 3.2：Design high-performing and elastic compute solutions

- [ ] 運算服務與適用情境（**Batch、EMR、Fargate**） → [[運算與容器選型]]
- [ ] 全球基礎架構與邊緣服務支援的分散式運算 → [[DNS與全球流量]]
- [ ] 佇列與訊息概念 → [[事件驅動與無伺服器]]
- [ ] 擴展能力（**EC2 Auto Scaling、AWS Auto Scaling**） → [[EC2與負載平衡及擴展]]
- [ ] 無伺服器技術與模式 → [[運算與容器選型]]
- [ ] 容器編排（ECS、EKS） → [[運算與容器選型]]
- [ ] **解耦工作負載使元件能獨立擴展** → [[事件驅動與無伺服器]]
- [ ] 識別執行擴縮動作的指標與條件
- [ ] 選擇適當的運算選項與功能（**EC2 instance type**）
- [ ] 選擇適當的資源類型與大小（**Lambda 記憶體配置**）

> [!important] 高頻考點
> **Lambda 15 分鐘上限**｜**provisioned concurrency 消除冷啟動 vs reserved concurrency 隔離額度**｜**Cluster placement group 給 HPC**｜**ASG 目標追蹤為預設首選、warm pool/predictive 解決啟動慢**｜**調大 Lambda 記憶體同時提升 CPU，可能反而更便宜**

---

## Task 3.3：Determine high-performing database solutions

- [ ] 全球基礎架構（AZ、Region） → [[高可用備援與災難復原]]
- [ ] **快取策略與服務（ElastiCache）** → [[資料庫與快取]]
- [ ] 資料存取模式（讀密集 vs 寫密集） → [[決策樹 資料庫選型]]
- [ ] 資料庫容量規劃（**capacity unit、instance type、Provisioned IOPS**） → [[數字與門檻速查]]
- [ ] **資料庫連線與 proxy** → [[資料庫與快取]]（RDS Proxy）
- [ ] 資料庫引擎與情境（**異質 vs 同質遷移**） → [[遷移與混合雲]]（DMS + SCT）
- [ ] 資料庫複寫（**read replica**） → [[資料庫與快取]]
- [ ] 資料庫類型（**serverless、關聯式 vs 非關聯式、記憶體內**） → [[決策樹 資料庫選型]]
- [ ] 設定 read replica、設計資料庫架構、選擇引擎與類型、整合快取

> [!important] 高頻考點
> **RDS Proxy 解決 Lambda 的 `too many connections`**｜**DAX 只服務 DynamoDB、幾乎不改程式**｜**Redis 有 HA/持久化，Memcached 沒有**｜**DynamoDB 熱分割區：容量有剩仍限流**｜**GSI 只支援最終一致性**｜**Aurora Serverless v2 給間歇性負載**

---

## Task 3.4：Determine high-performing and/or scalable network architectures

- [ ] 邊緣網路服務（**CloudFront、Global Accelerator**） → [[DNS與全球流量]]
- [ ] 網路架構設計（**subnet 分層、路由、IP 定址**） → [[EC2與網路]]
- [ ] 負載平衡概念（ALB） → [[EC2與負載平衡及擴展]]
- [ ] 網路連線選項（**VPN、Direct Connect、PrivateLink**） → [[遷移與混合雲]]、[[決策樹 網路與連線選型]]
- [ ] 為各種架構建立網路拓樸（**global / hybrid / multi-tier**）
- [ ] 決定可擴展的網路組態、資源放置位置、負載平衡策略

> [!important] 高頻考點
> **CloudFront 快取 vs Global Accelerator 非快取/固定 IP/秒級切換**｜**Route 53 七種 policy，latency vs geolocation**｜**Alias 才能用在 zone apex**｜**gateway endpoint 只有 S3/DynamoDB**｜**Peering 無遞移路由**｜**subnet 保留 5 個 IP**

---

## Task 3.5：Determine high-performing data ingestion and transformation solutions

- [ ] 分析與視覺化服務（**Athena、Lake Formation、Amazon Quick**） → [[資料分析與串流]]
- [ ] 資料擷取模式（**頻率**） → [[資料分析與串流]]
- [ ] 資料傳輸服務（**DataSync、Storage Gateway**） → [[遷移與混合雲]]
- [ ] 資料轉換服務（**Glue**） → [[資料分析與串流]]
- [ ] 擷取端點的安全存取 → [[威脅偵測與邊界防護]]
- [ ] 符合需求的規模與速度 → [[數字與門檻速查]]（Kinesis shard 吞吐）
- [ ] **串流資料服務（Kinesis）** → [[資料分析與串流]]
- [ ] 建置與保護 data lake、設計串流架構與資料傳輸方案
- [ ] 選擇資料處理的運算選項（**EMR**）
- [ ] **在格式之間轉換資料（.csv → .parquet）**

> [!important] 高頻考點
> **Data Streams（即時/多消費者/可重播）vs Firehose（近即時/零維運/固定目的地）**｜**Athena 降本三招：Parquet + 壓縮 + 分區**｜**DataSync 搬家 vs Storage Gateway 延伸**｜**頻寬夠就不要選 Snowball**｜**OpenSearch 給即時全文檢索與儀表板**

---

## 📚 本 Domain 的主力筆記

| 筆記 | 對應 Task |
|---|---|
| [[決策樹 儲存選型]]、[[EC2與儲存]]、[[S3與資料生命週期]] | 3.1 |
| [[決策樹 運算選型]]、[[運算與容器選型]]、[[EC2與負載平衡及擴展]] | 3.2 |
| [[決策樹 資料庫選型]]、[[資料庫與快取]] | 3.3 |
| [[決策樹 網路與連線選型]]、[[EC2與網路]]、[[DNS與全球流量]] | 3.4 |
| [[資料分析與串流]]、[[遷移與混合雲]] | 3.5 |

## ✍️ Domain 3 自我檢核

1. 五種儲存（EBS / instance store / EFS / FSx / S3）各用一句話說出**存取介面**與**典型情境**。
2. 「已經開了 Auto Scaling 但效能還是不好」——列出四種可能的瓶頸層與對應解法。
3. Kinesis Data Streams 與 Data Firehose，各自在什麼題幹關鍵字下是正解？
4. CloudFront 與 Global Accelerator 的三個分界點？
5. Athena 查詢又慢又貴，標準的三招優化是什麼？為什麼「換更大的 instance」是錯的？

> [!success]- 參考答案
> 1. **EBS**=區塊裝置、單一實例持久磁碟；**instance store**=主機本地區塊、暫存可重建；**EFS**=NFS、多台 Linux 跨 AZ 共享；**FSx**=託管檔案系統（Windows SMB / Lustre HPC / ONTAP 多協定）；**S3**=物件 API、靜態資產與資料湖。
> 2. **運算** → 加機器有效；**資料庫寫入** → 垂直擴展或改 DynamoDB/分片；**資料庫讀取** → Read Replica 或 ElastiCache/DAX；**靜態內容頻寬** → CloudFront；**後端處理速度** → SQS 解耦 + worker ASG。
> 3. **Data Streams**：`real-time`、`sub-second`、`multiple consumers`、`replay`。**Firehose**：`near real-time`、`deliver to S3/Redshift/OpenSearch`、`no administration`、`automatically scales`。
> 4. **可快取的 HTTP → CloudFront**；**非 HTTP 協定 / 需要固定 anycast IP / 需要不受 DNS 快取影響的秒級切換 → Global Accelerator**。
> 5. **列式格式（Parquet/ORC）+ 壓縮 + 分區**，因為 Athena **按掃描資料量計費**。「換更大 instance」錯在 **Athena 是無伺服器的，根本沒有 instance**（誘答套路：服務根本不支援）。

## 🔗 相關

- [[Domain 2 設計高韌性架構]]
- [[Domain 4 設計成本最佳化架構]]
- [[常見陷阱與誘答選項識別]]
- [[情境題庫]]（Q17–Q23、Q48–Q54）
