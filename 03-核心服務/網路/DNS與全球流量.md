---
title: DNS與全球流量
tags:
  - AWS/SAA-C03
  - Route53
  - CloudFront
  - service/route53
  - service/cloudfront
  - exam/d3
status: 未讀
confidence: 1
importance: 5
updated: 2026-09-27
---
# DNS與全球流量

| 服務 | 角色 | 選型線索 |
|---|---|---|
| Route 53 | DNS 與 routing policy / health checks | weighted、latency、failover 等 DNS 決策；TTL/快取影響切換 |
| CloudFront | CDN edge 快取及 HTTP(S) 交付 | 靜態內容、可快取的動態內容，S3/ALB 可作 origin |
| Global Accelerator | 固定 anycast IP，將用戶流量經 AWS 全球網路到受支援 endpoint | 非快取流量、固定 IP、全球網路路徑/快速健康切換等需求 |
| ALB | Region 內 HTTP(S) L7 分流 | path/host routing 至 target group |

典型靜態路徑：`Route 53 → CloudFront → 私有 S3 origin (OAC)`。動態路徑：`Route 53 → CloudFront (可選) → ALB → private EC2`。DNS failover 不等於跨 Region 資料已同步；見 [[高可用備援與災難復原]]。

常見陷阱：CloudFront 能快取並降低 origin load；Global Accelerator 不是 CDN 物件快取。Route 53 只處理名稱解析與策略，不取代資料平面中的 ALB。ACM 憑證與 WAF 的關聯依受支援服務及部署位置安排；見 [[EC2與IAM安全]]。

---

# 深化

## Route 53 的七種 routing policy

原本只列了三種，但考題會逐一點名，必須全部認得。**判斷方法：看題目是依「什麼條件」決定要回哪個 IP。**

| Policy | 依據 | 題幹關鍵字 |
|---|---|---|
| **Simple** | 無條件，單一記錄（可多值但不健康檢查） | 最基本 |
| **Weighted** | 按**權重百分比**分配 | `A/B testing`、`canary`、`blue-green`、`send 10% of traffic` |
| **Latency-based** | 使用者到**各 Region 的網路延遲** | `lowest latency`、`best performance for global users` |
| **Failover** | primary 健康就回 primary，否則回 secondary | `active-passive`、`disaster recovery`、`standby site` |
| **Geolocation** | 使用者**所在地理位置**（國家/洲） | `users in Europe must be served from EU`、**合規/在地化內容/語言** |
| **Geoproximity** | 資源的地理位置 + 可調整的 **bias** 偏移量 | `shift traffic toward a region`、需要**手動調整流量重心** |
| **Multivalue answer** | 回傳多筆（最多 8）**通過健康檢查**的記錄 | `simple load balancing with health checks`、不想用 ELB 時的輕量方案 |
| **IP-based** | 依使用者**來源 IP CIDR** | `route based on ISP / client subnet` |

> [!danger] Latency vs Geolocation（最常混淆）
> **Latency-based = 誰快去誰那**（效能導向）。
> **Geolocation = 誰在哪去哪**（合規/法規/語言導向）。
> 題幹出現 `data residency`、`regulatory requirement`、`must be served content in their language` → **Geolocation**，即使那不是延遲最低的 Region。

## Alias record vs CNAME（必考）

| | Alias（Route 53 專有） | CNAME |
|---|---|---|
| **可用於 zone apex**（`example.com`） | ✅ **可以** | ❌ **不行**（DNS 標準禁止） |
| 指向 AWS 資源 | ALB、CloudFront、S3 website、API Gateway、Global Accelerator、另一筆 Route 53 記錄 | 任意網域名稱 |
| 查詢費用 | **免費** | 收費 |
| 健康檢查整合 | 可評估目標健康狀態 | 否 |

> [!important] 一句話
> 「要把 `example.com`（根網域）指向 ALB / CloudFront」→ **只能用 Alias record**。這題出現頻率很高，選 CNAME 一定錯。

## TTL 與切換速度

DNS 的切換速度受 **TTL** 與**各層 resolver/瀏覽器快取**限制。題目說「failover 要快」時要注意：
- **降低 TTL**（例如 60 秒）可加快切換，但增加查詢次數與費用。
- 若要求**秒級、不受 DNS 快取影響**的切換 → **Global Accelerator**（固定 anycast IP，切換發生在 AWS 網路層，客戶端不需重新解析 DNS）。這是 GA 相對 Route 53 failover 的核心價值。

## CloudFront 深入

| 功能 | 用途 | 題幹關鍵字 |
|---|---|---|
| **OAC**（Origin Access Control） | 讓 S3 origin 保持私有，只接受 CloudFront 存取 | `keep S3 bucket private`、`restrict direct access`。**OAI 是舊機制，新設計一律用 OAC** |
| **Signed URL / Signed Cookie** | 限制**個別使用者**存取私有內容 | Signed **URL** = 單一檔案；Signed **Cookie** = 多個檔案（整個影片串流目錄） |
| **Geo restriction** | 依國家允許/封鎖 | `block users in certain countries` |
| **Cache behavior** | 依路徑樣式（`/api/*`、`/static/*`）套用不同 origin 與快取策略 | 同一網域同時服務靜態與動態 |
| **Origin group** | 主 origin 失敗時自動改用備援 origin | `origin failover`、`high availability for origin` |
| **Price Class** | 限制只使用部分 edge location | `reduce CloudFront cost`、不需要全球覆蓋時 |
| **Field-level encryption** | 特定欄位在 edge 就加密 | `encrypt sensitive fields at the edge` |
| **CloudFront Functions / Lambda@Edge** | edge 運算，見 [[運算與容器選型]] | 輕量改寫 vs 重量處理 |

> [!warning] 兩個容易踩的細節
> **(1) CloudFront 的 ACM 憑證必須簽發在 `us-east-1`（維吉尼亞北部）。** 其他 Region 的憑證掛不上去。ALB 則用 ALB 所在 Region 的憑證。
> **(2) CloudFront 也能加速動態內容。** 常見錯誤觀念是「動態就不該用 CloudFront」——實際上即使不快取，走 AWS 骨幹網路到 origin 仍能降低延遲並提供 TLS 終止、WAF 掛載點。

## CloudFront vs Global Accelerator（決策表）

| 條件 | 選擇 |
|---|---|
| 可快取的 HTTP(S) 內容、要降低 origin 負載 | **CloudFront** |
| 非 HTTP 協定（TCP/UDP、遊戲、IoT、VoIP） | **Global Accelerator** |
| 需要**固定 anycast IP**（客戶端防火牆白名單） | **Global Accelerator** |
| 需要**秒級**跨 Region failover、不受 DNS 快取影響 | **Global Accelerator** |
| 靜態網站 + S3 | **CloudFront** |

## 與其他筆記的接點

WAF 的掛載位置與 DDoS 分層：[[威脅偵測與邊界防護]]；跨 Region 切換與資料同步的落差：[[高可用備援與災難復原]]；edge 運算選型：[[運算與容器選型]]。

參考 [[03 官方資源清單]]；返回 [[00 考試總覽]]。

---

## 🎯 考點速記

看到 `lowest latency` → **Latency-based routing**
看到 `data residency` / `regulatory` / 語言 → **Geolocation routing**
看到 `A/B testing` / `canary` / `10% of traffic` → **Weighted routing**
看到根網域指向 ALB/CloudFront → **Alias record**（CNAME 不行）
看到 `static anycast IP` 或 `秒級切換不受 DNS 影響` → **Global Accelerator**
看到 `cacheable content` / `reduce origin load` → **CloudFront**
看到 `keep S3 origin private` → **OAC**
看到 CloudFront 的憑證 → **必須在 us-east-1**

## 💣 真實場景陷阱

- **DNS failover 不等於資料已同步**：Route 53 把流量切到第二 Region，但那邊的資料庫可能落後數分鐘。切換機制與資料複寫是兩件事。
- **TTL 沒調低就談秒級切換**：客戶端與各層 resolver 會快取到 TTL 到期。真的要秒級請用 Global Accelerator。
- **CloudFront 快取了不該快取的東西**：未正確設定 cache key（忽略了 Authorization header 或 query string）會把 A 使用者的內容送給 B。
- **誤以為動態內容不該用 CloudFront**：即使不快取，走 AWS 骨幹到 origin 仍降低延遲，並提供 TLS 終止與 WAF 掛載點。

## ✍️ 自我檢核

1. Route 53 的七種 routing policy，各在什麼題幹關鍵字下是正解？
2. Latency-based 與 Geolocation 的根本差別是什麼？哪一個會在「合規」題出現？
3. `example.com` 要指向 ALB，為什麼不能用 CNAME？
4. CloudFront 與 Global Accelerator 的三個分界點？
5. 為什麼「DNS failover 完成」不代表「災難復原完成」？

> [!success]- 參考答案
> 1. Simple（無條件）、**Weighted**（canary/A-B）、**Latency**（效能）、**Failover**（active-passive）、**Geolocation**（合規/語言）、Geoproximity（bias 調流量重心）、Multivalue（最多 8 筆健康記錄）、IP-based（依來源 CIDR）。
> 2. **Latency = 誰快去誰那（效能導向）；Geolocation = 誰在哪去哪（合規導向）。** 出現 `data residency`、`must be served from EU` 就是 **Geolocation**，即使那不是延遲最低的 Region。
> 3. **DNS 標準禁止 CNAME 用於 zone apex（根網域）**。Route 53 的 **Alias record** 是專有擴充，可以用在 apex，而且查詢免費、能評估目標健康狀態。
> 4. **可快取 HTTP → CloudFront**；**非 HTTP 協定 / 需要固定 anycast IP / 需要不受 DNS 快取影響的秒級切換 → Global Accelerator**。
> 5. DNS 只切**流量入口**，不處理**資料**。第二 Region 的資料可能因為非同步複寫而落後（RPO），應用也可能還沒預先部署（RTO）。切換機制、資料同步、環境就緒是三件獨立的事。
