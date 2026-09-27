---
tags: [AWS, Route53, CloudFront]
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

參考 [[官方資料]]。
