---
title: Domain 4 設計成本最佳化架構
tags:
  - AWS/SAA-C03
  - exam/d4
  - checklist
status: 未讀
confidence: 1
importance: 5
updated: 2026-09-27
---

# Domain 4：Design Cost-Optimized Architectures（20%）

> [!abstract] 這篇怎麼用
> 官方 task statement 的逐條拆解 + 對應筆記。**約 10 道計分題。**
> 這個 domain 看起來最小，但**成本條件會混進其他 domain 的題目裡**（「高可用**且**成本最低」），實際影響遠超過 20%。

> [!danger] 這個 domain 唯一的解題順序
> **先用其他所有條件淘汰選項，剩下的才比價。**
> 反過來做（先找最便宜的再檢查條件）就會掉進陷阱——題目給的「最便宜」選項通常違反某個硬性條件。

---

## Task 4.1：Design cost-optimized storage solutions

- [ ] 存取選項（**Requester Pays**） → [[S3與資料生命週期]]
- [ ] 成本管理功能（**cost allocation tag、多帳號帳務**） → [[成本與可觀測性]]、[[治理與合規]]
- [ ] 成本管理工具（**Cost Explorer、Budgets、CUR**） → [[成本與可觀測性]]
- [ ] 儲存服務與情境（FSx、EFS、S3、EBS） → [[決策樹 儲存選型]]
- [ ] **備份策略** → [[治理與合規]]（AWS Backup）
- [ ] 區塊儲存選項（**HDD vs SSD volume 類型**） → [[EC2與儲存]]
- [ ] **資料生命週期** → [[S3與資料生命週期]]
- [ ] 混合儲存（DataSync、Transfer Family、Storage Gateway） → [[遷移與混合雲]]
- [ ] 儲存存取模式、**分層（cold tiering）**、儲存類型特性
- [ ] 決定最低成本的資料傳輸方式、正確的儲存大小、何時需要 storage auto scaling

> [!important] 高頻考點
> **各類別的最低儲存期間（30/90/180 天）——提前刪除仍要付滿**｜**Intelligent-Tiering 是唯一沒有取回費的分層方案**｜**未完成的 multipart upload 持續計費且看不見**｜**One Zone-IA 只在資料可重建時才對**｜**st1 給循序吞吐、成本遠低於 SSD**

---

## Task 4.2：Design cost-optimized compute solutions

- [ ] 成本管理功能與工具 → [[成本與可觀測性]]
- [ ] 全球基礎架構（AZ、Region） → [[服務關聯總圖]]
- [ ] **購買選項（Spot、Reserved Instances、Savings Plans）** → [[決策樹 運算選型]]
- [ ] 分散式運算策略（邊緣處理） → [[DNS與全球流量]]
- [ ] 混合運算選項（**Outposts**） → [[遷移與混合雲]]
- [ ] **Instance type / family / size**（記憶體最佳化、運算最佳化） → [[運算與容器選型]]
- [ ] 運算使用率最佳化（容器、無伺服器、微服務） → [[運算與容器選型]]
- [ ] 擴展策略（**auto scaling、hibernation**） → [[EC2與負載平衡及擴展]]
- [ ] **決定負載平衡策略（ALB L7 vs NLB L4 vs GWLB）** → [[EC2與負載平衡及擴展]]
- [ ] 決定彈性工作負載的擴展方式（水平 vs 垂直、**EC2 hibernation**）
- [ ] 判斷具成本效益的運算服務、不同等級工作負載的可用性需求、instance family 與 size

> [!important] 高頻考點
> **Compute SP 涵蓋 Fargate 與 Lambda**，EC2 Instance SP 不涵蓋｜**只有 Dedicated Host 看得到 socket/core，BYOL 必選**｜**Spot = fault-tolerant/可中斷**｜**mixed instances policy：基準 On-Demand + 其餘 Spot**｜**Capacity Reservation 沒有折扣**｜**Aurora Serverless v2 給間歇性負載**

---

## Task 4.3：Design cost-optimized database solutions

- [ ] 成本管理功能與工具 → [[成本與可觀測性]]
- [ ] 快取策略 → [[資料庫與快取]]
- [ ] **資料保留政策** → [[S3與資料生命週期]]、[[治理與合規]]
- [ ] 資料庫容量規劃（**capacity unit**） → [[資料庫與快取]]（on-demand vs provisioned）
- [ ] 資料庫連線與 proxy → [[資料庫與快取]]（RDS Proxy）
- [ ] 資料庫引擎與情境（異質 vs 同質遷移） → [[遷移與混合雲]]
- [ ] 資料庫複寫（read replica） → [[決策樹 資料庫選型]]
- [ ] 資料庫類型與服務（關聯式 vs 非關聯式、Aurora、DynamoDB）
- [ ] 設計備份與保留政策（**snapshot 頻率**）
- [ ] **判斷具成本效益的資料庫類型（時序格式、列式格式）**
- [ ] 遷移 schema 與資料到不同位置或引擎

> [!important] 高頻考點
> **DynamoDB on-demand（不可預測）vs provisioned + Auto Scaling（可預測且要省）**｜**Aurora Serverless v2 取代「手動停 RDS」**｜**RDS 自動備份保留 0–35 天**｜**手動 snapshot 不隨 DB 刪除消失**｜**報表拖垮生產庫：Read Replica（輕）或 Redshift（重）**

---

## Task 4.4：Design cost-optimized network architectures

- [ ] 成本管理功能與工具 → [[成本與可觀測性]]
- [ ] 負載平衡概念 → [[EC2與負載平衡及擴展]]
- [ ] **NAT gateway（NAT instance 成本 vs NAT gateway 成本）** → [[EC2與網路]]
- [ ] 網路連線（專線、VPN） → [[遷移與混合雲]]
- [ ] **網路路由、拓樸與 peering（Transit Gateway、VPC peering）** → [[決策樹 網路與連線選型]]
- [ ] 網路服務與情境（DNS） → [[DNS與全球流量]]
- [ ] **設定適當的 NAT gateway 型態（單一共用 vs 每 AZ 一個）**
- [ ] 設定適當的網路連線（DX vs VPN vs 網際網路）
- [ ] **設定路由以最小化傳輸成本**（Region 間、AZ 間、私有到公開、Global Accelerator、**VPC endpoint**）
- [ ] 判斷 CDN 與邊緣快取的策略需求
- [ ] 檢視既有工作負載的網路最佳化、節流策略、頻寬配置

> [!important] 高頻考點
> **gateway endpoint 免費 vs NAT gateway 按 GB 收處理費**（最經典的成本題）｜**每 AZ 一個 NAT 同時解決可用性與跨 AZ 傳輸費**｜**CloudFront egress 比 origin 直出便宜、回源免費**｜**進入 AWS 免費、跨 AZ 雙向計費、出網際網路最貴**

---

## 💰 資料傳輸計費速記

| 流向 | 費用 |
|---|---|
| **進入 AWS（inbound）** | **免費** |
| 同一 AZ、使用私有 IP | **免費** |
| **跨 AZ（同 Region）** | **雙向計費** |
| 跨 Region | 計費 |
| **出到網際網路** | **最貴** |
| 經 CloudFront 出去 | 比 origin 直出便宜，回源免費 |
| S3/DynamoDB 經 **gateway endpoint** | **免費** |

## 📚 本 Domain 的主力筆記

| 筆記 | 對應 Task |
|---|---|
| [[成本與可觀測性]] | 全部（成本工具、傳輸計費、最佳化五問） |
| [[S3與資料生命週期]]、[[決策樹 儲存選型]] | 4.1 |
| [[運算與容器選型]]、[[決策樹 運算選型]] | 4.2 |
| [[資料庫與快取]]、[[決策樹 資料庫選型]] | 4.3 |
| [[EC2與網路]]、[[決策樹 網路與連線選型]] | 4.4 |

## ✍️ Domain 4 自我檢核

1. 成本題的解題順序是什麼？為什麼不能先找最便宜的？
2. 資料只保留 45 天，為什麼 Glacier Deep Archive 反而更貴？
3. 私有子網大量存取 S3，帳單很高——正解是什麼？省在哪裡？
4. Compute Savings Plans 相對 EC2 Instance Savings Plans，多了什麼？
5. 「找出閒置資源」與「找出規格過大的實例」分別該用哪個工具？
6. Budgets、Cost Explorer、CUR、Trusted Advisor 各一句話。

<details>
<summary>參考答案</summary>

1. **先用其他所有條件（可用性、延遲、合規、不可中斷）淘汰，剩下的才比價。** 先找最便宜的會選到違反硬性條件的選項——那是題目故意放的。
2. **Deep Archive 最低儲存期間 180 天**，只放 45 天仍要付滿 180 天。成本題要先檢查**最低儲存期間**，不是看單價。
3. **建立 S3 gateway endpoint 並更新私有子網路由表。** gateway endpoint **免費**，而 NAT gateway 按**小時 + 處理資料量**計費；流量也不再離開 AWS 網路。
4. **跨 instance family、跨 Region、跨 OS，而且涵蓋 Fargate 與 Lambda。** EC2 Instance SP 折扣稍高但鎖定 family + Region 且不含 Fargate/Lambda。
5. 閒置資源 → **Trusted Advisor**（完整檢查需 Business/Enterprise Support）；規格建議 → **Compute Optimizer**。
6. **Budgets = 超標告警**；**Cost Explorer = 歷史分析與預測**；**CUR = 最細帳單明細供自訂分析**；**Trusted Advisor = 五大支柱檢查與閒置資源**。

</details>

## 🔗 相關

- [[Domain 3 設計高效能架構]]
- [[Domain 1 設計安全架構]]
- [[常見陷阱與誘答選項識別]]
- [[情境題庫]]（Q24–Q30、Q55–Q60）
