---
title: Domain 2 設計高韌性架構
tags:
  - AWS/SAA-C03
  - exam/d2
  - checklist
status: 未讀
confidence: 1
importance: 5
updated: 2026-09-27
---

# Domain 2：Design Resilient Architectures（26%）

> [!abstract] 這篇怎麼用
> 官方 task statement 的逐條拆解 + 對應筆記。當檢核清單反覆回看。
> **約 13 道計分題。** 這個 domain 的核心是「**壞掉什麼還能活**」與「**元件之間夠不夠鬆耦合**」。

---

## Task 2.1：Design scalable and loosely coupled architectures

**Knowledge of**

- [ ] API 建立與管理（**API Gateway、REST API**） → [[事件驅動與無伺服器]]
- [ ] AWS 託管服務與適用情境（Transfer Family、SQS、Secrets Manager） → [[遷移與混合雲]]、[[事件驅動與無伺服器]]
- [ ] **快取策略** → [[資料庫與快取]]
- [ ] 微服務設計原則（**stateless vs stateful**） → [[EC2與負載平衡及擴展]]
- [ ] **事件驅動架構** → [[事件驅動與無伺服器]]
- [ ] **水平擴展 vs 垂直擴展** → [[EC2與負載平衡及擴展]]
- [ ] 邊緣加速器（CDN）的適當使用 → [[DNS與全球流量]]
- [ ] 如何把應用遷移到容器 → [[運算與容器選型]]
- [ ] 負載平衡概念（ALB） → [[EC2與負載平衡及擴展]]
- [ ] 多層式架構 → [[服務關聯總圖]]
- [ ] 佇列與訊息概念（**pub/sub**） → [[決策樹 訊息與事件選型]]
- [ ] 無伺服器技術與模式（**Fargate、Lambda**） → [[運算與容器選型]]
- [ ] 儲存類型特性（**object / file / block**） → [[決策樹 儲存選型]]
- [ ] 容器編排（**ECS、EKS**） → [[運算與容器選型]]
- [ ] **何時使用 read replica** → [[資料庫與快取]]
- [ ] 工作流程編排（**Step Functions**） → [[事件驅動與無伺服器]]

**Skills in**

- [ ] 依需求設計事件驅動 / 微服務 / 多層式架構
- [ ] 決定各元件的擴展策略
- [ ] 判斷達成鬆耦合所需的 AWS 服務
- [ ] **判斷何時使用容器**（以及何時不要選 EKS）
- [ ] 判斷何時使用無伺服器
- [ ] 依需求推薦運算 / 儲存 / 網路 / 資料庫技術
- [ ] 使用專用（purpose-built）服務

> [!important] 這一節的高頻考點
> **單一 SQS 做不到多方各收一份**（要 SNS fan-out + 每消費者一個 SQS）｜**Kinesis 才能重播**｜**Amazon MQ 是「既有 JMS/AMQP 不改程式」的唯一正解**｜**沒說 Kubernetes 就不選 EKS**｜**Lambda 15 分鐘硬上限**｜**session 要外部化**

---

## Task 2.2：Design highly available and/or fault-tolerant architectures

**Knowledge of**

- [ ] AWS 全球基礎架構（AZ、Region、**Route 53**） → [[DNS與全球流量]]
- [ ] AWS 託管 AI 服務與情境（Comprehend、Polly 等） → [[00 考試總覽]]（一句話對照表）
- [ ] 基本網路概念（route table） → [[EC2與網路]]
- [ ] **DR 策略**：backup and restore、pilot light、warm standby、active-active、**RPO / RTO** → [[高可用備援與災難復原]]
- [ ] 分散式設計模式 → [[服務關聯總圖]]
- [ ] **故障切換策略** → [[高可用備援與災難復原]]
- [ ] 不可變基礎設施（immutable infrastructure） → [[運算與容器選型]]（instance refresh）
- [ ] 負載平衡概念 → [[EC2與負載平衡及擴展]]
- [ ] **Proxy 概念（RDS Proxy）** → [[資料庫與快取]]
- [ ] **服務配額與節流** → [[數字與門檻速查]]
- [ ] 儲存選項特性（**durability、replication**） → [[S3與資料生命週期]]
- [ ] 工作負載可見性（**X-Ray**） → [[成本與可觀測性]]

**Skills in**

- [ ] 用自動化確保基礎設施完整性 → [[治理與合規]]（CloudFormation、Config remediation）
- [ ] 判斷跨 AZ 或跨 Region 達成 HA / 容錯所需的服務
- [ ] 依業務需求識別衡量高可用的指標
- [ ] **實作設計以消除單點故障**
- [ ] 實作資料耐久性與可用性策略（備份）
- [ ] **選擇符合業務需求的 DR 策略**
- [ ] 用 AWS 服務提升 legacy 應用的可靠性（**當應用無法修改時**）
- [ ] 使用專用服務

> [!important] 這一節的高頻考點
> **Multi-AZ ≠ DR、Read Replica ≠ HA、備份 ≠ 高可用**｜**DR 題先用 RTO 篩選再比價**（看到 DR 就選 active/active 是最常見錯誤）｜**ASG health check type 預設是 EC2，要改成 ELB**｜**NAT gateway 要每 AZ 一個**｜**Aurora Global Database RPO≈1s RTO≈1min**｜**DynamoDB Global Tables 多 Region 寫入**

---

## 📚 本 Domain 的主力筆記

| 筆記 | 涵蓋 |
|---|---|
| [[高可用備援與災難復原]] | RPO/RTO、四類 DR 策略量級、跨 Region 複寫對照、可用性數學 |
| [[事件驅動與無伺服器]] | SQS 四機制、SNS fan-out、EventBridge、Step Functions |
| [[EC2與負載平衡及擴展]] | ALB/NLB/GWLB、ASG 深入、瓶頸判斷圖 |
| [[決策樹 訊息與事件選型]] | 佇列 vs 廣播 vs 串流的分水嶺 |
| [[資料庫與快取]] | Multi-AZ / Read Replica / Global Database |

## ✍️ Domain 2 自我檢核

1. Multi-AZ、Read Replica、跨 Region replica、Global Database——各解決什麼？哪兩組最常被題目對調？
2. 題目給「RTO 4 小時、成本最低」，四類 DR 策略你怎麼篩？
3. 三個下游都要處理同一筆事件且彼此不能互相影響——畫出架構。
4. ASG 沒有替換一台「應用當掉但 OS 還活著」的實例，為什麼？
5. 單一 AZ 的 NAT gateway 故障，為什麼會影響**其他** AZ 的私有子網？
6. Lambda 非同步呼叫失敗的事件為什麼會消失？怎麼保留？

> [!success]- 參考答案
> 1. **Multi-AZ = HA（同 Region 自動切換）**；**Read Replica = 讀取擴展（非同步、不自動切換）**；**跨 Region replica / Global Database = DR**。最常被對調的是 Multi-AZ 與 Read Replica，其次是 Multi-AZ 被當成 DR。
> 2. **先用 RTO 篩掉達不到的，再在合格者中選最便宜的。** RTO 4 小時 → pilot light 就夠；active/active 與 warm standby 雖然也達標但更貴，屬過度設計。
> 3. `事件 → SNS topic → 三個 SQS（各一訂閱）→ 各自 worker + DLQ`。單一 SQS 做不到，因為訊息被一個消費者取走就消失。
> 4. ASG 的 health check type **預設是 `EC2`**，只看 instance status check。要改成 **`ELB`** 才會依 target group 健康狀態替換。
> 5. 若所有私有子網的 `0.0.0.0/0` 都指向同一個 AZ 的 NAT gateway，那個 AZ 故障就全斷。正解是**每 AZ 一個 NAT，各自路由指向自己 AZ 的 NAT**（同時省跨 AZ 傳輸費）。
> 6. 非同步呼叫**預設只重試 2 次**，之後丟棄。要保留必須設 **DLQ** 或 **Lambda Destinations（on-failure）**。

## 🔗 相關

- [[Domain 1 設計安全架構]]
- [[Domain 3 設計高效能架構]]
- [[常見陷阱與誘答選項識別]]
- [[情境題庫]]（Q9–Q16、Q40–Q47）
