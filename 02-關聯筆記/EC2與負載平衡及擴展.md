---
tags: [AWS, EC2, ALB, AutoScaling]
---
# EC2與負載平衡及擴展

`Route 53 → (可選 CloudFront/WAF) → ALB → Target group → 多 AZ Auto Scaling EC2 → RDS`。Launch template 定義 AMI、instance type、角色、SG 等；ASG 負責期望數量及擴縮，target group 連接負載平衡器並將健康的 instance 作目標。健康檢查路徑必須和應用一致。

| 判斷 | ALB | NLB |
|---|---|---|
| 處理層 | HTTP/HTTPS、路徑/host 路由等 L7 | TCP/UDP/TLS 等 L4 |
| 典型需求 | Web API、微服務路由 | 非 HTTP 協定、特定網路能力或極低延遲需求 |

ASG 的目標追蹤可依 CPU、ALB request count per target 或自訂指標擴展；要看暖機、健康檢查、scale-in 與狀態外部化。增加 instance 不會自動擴大單一 RDS writer 容量；讀負載可用 read replica/cache。跨 AZ 部署改善 AZ 故障容忍，不自動提供跨 Region DR。見 [[資料庫與快取]]、[[高可用備援與災難復原]]。

Spring Boot 練習：`/actuator/health/readiness` 作 target health check；單一節點故障時，ALB 停止導流、ASG 補節點。不要將 session 或上傳檔只放在本機磁碟；改用外部 session store/物件儲存。詳見 [[EC2與儲存]]。

參考 [[官方資料]]。
