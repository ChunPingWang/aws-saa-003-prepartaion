---
tags: [AWS, EC2, IAM, security]
---
# EC2與IAM安全

## 一個 EC2 讀私有 S3 物件

`EC2 instance → instance profile → IAM role 暫時憑證 → S3 API`。角色的 **trust policy** 決定誰可 assume，**permissions policy** 決定能做什麼；S3 bucket policy、SCP、VPC endpoint policy、KMS key policy 等也可能限制結果。網路可達與授權是兩道不同檢查。避免將長期 access key 放在 AMI、user data 或程式設定。

| 需求 | 常見工具 | 邊界 |
|---|---|---|
| EC2 呼叫 AWS API | IAM role + instance profile | 最小權限，必要時檢查 metadata 服務設定 |
| 跨帳號讀取資源 | STS AssumeRole + 對方 role trust/permissions | 不能只在來源帳號給 Allow 就期待成功 |
| 管理人員登入多帳號 | IAM Identity Center | 與應用在 EC2 上的 instance role 用途不同 |
| 保存 DB 密碼並輪替 | Secrets Manager | 讀取 secret 需要 IAM，若自訂 KMS key 還要評估解密權限 |
| 加密 EBS/S3/RDS | KMS 整合與適當 key policy | 傳輸中 TLS 與靜態加密是不同需求 |
| TLS 憑證 | ACM 與支援的整合服務如 ALB/CloudFront | 不等於把憑證直接安裝到 EC2 OS |
| L7 Web 攻擊防護 | AWS WAF 附於受支援資源 | SG 管端口/來源；WAF 管 HTTP 規則 |
| 審計 API 操作 | CloudTrail | CloudWatch metrics/logs 用來觀測運作，職責不同 |

SCP 限制 Organizations 帳號的**權限上限**，一般不直接授予權限；root user 等例外與服務連結角色細節以官方文件為準。Resource policy 與 identity policy 的跨帳號組合要逐題看明確 deny、owner 與信任關係。

## 比較題

「EC2 連不到 S3」分成三個診斷問題：DNS/route/endpoint 是否通？身份與 bucket policy 是否允許 `s3:GetObject`？若物件使用 SSE-KMS，KMS 權限是否滿足？也參考 [[EC2與網路]] 與 [[S3與資料生命週期]]。

參考 [[官方資料]]；返回 [[服務關聯總圖]]。
