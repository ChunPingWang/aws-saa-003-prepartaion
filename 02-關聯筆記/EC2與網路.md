---
tags: [AWS, EC2, VPC, networking]
---
# EC2與網路

**VPC 是網路範圍，subnet 屬於一個 AZ；EC2 的 ENI、私有 IP 與 SG 實際承接流量。** 「public subnet」取決於關聯 route table 有通向 IGW 的路由；instance 要直接 IPv4 網際網路通訊還須適當 public IPv4/EIP 及 SG/NACL 規則。

## 三條不同的路

| 目標 | 路徑與必要條件 | 易錯點 |
|---|---|---|
| Internet → 應用 | Internet-facing ALB 放 public subnets → target group → private EC2；ALB/EC2 SG 各自放行相應端口 | private EC2 不需要 public IP；ALB 仍需健康檢查通過 |
| Private EC2 → Internet IPv4 | private route table 的 0.0.0.0/0 → public NAT gateway（位於 public subnet 並可至 IGW） | NAT 供出站與回應，不開放主動入站；多 AZ 高可用要檢視各 AZ 路徑 |
| Private EC2 → S3/DynamoDB | gateway endpoint + 關聯路由表；IAM、endpoint policy、資源 policy 仍需允許 | gateway endpoint 不是所有服務通用；不是「有網路就有權限」 |
| Private EC2 → 其他支援的 AWS API | interface endpoint / PrivateLink + DNS、endpoint SG、policy | interface endpoint 通常按 AZ/時數與流量計費 |

SG 關聯至網路介面，**stateful、只列允許規則**；NACL 在 subnet 邊界，**stateless、可 allow/deny**，回程與 ephemeral ports 也要考慮。SG 來源可引用另一 SG，適合 ALB SG → App SG → DB SG。SG/NACL 不替代 IAM 權限；見 [[EC2與IAM安全]]。

## VPC 之間與混合網路

- 少量 VPC 點對點：VPC Peering；注意 CIDR 不重疊，peering 不提供任意轉送。
- 多 VPC / on-prem hub：Transit Gateway；設路由與費用。
- 消費對方提供的單一服務而非整網互通：PrivateLink。
- on-prem 加密連線：Site-to-Site VPN；專用連線需求：Direct Connect，並另思考備援/加密。

## 自問

EC2 在 private subnet 為何 `curl` 外網失敗？依序檢查 DNS、route、NAT/endpoint、SG egress、NACL 雙向、目標是否允許。若是 AWS API 403，網路可能已通，繼續查 IAM/resource/endpoint/KMS policy。

連到 [[EC2與負載平衡及擴展]]、[[DNS與全球流量]]、[[成本與可觀測性]]；參考 [[官方資料]]。
