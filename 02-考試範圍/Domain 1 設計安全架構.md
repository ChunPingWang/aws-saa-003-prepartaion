---
title: Domain 1 設計安全架構
tags:
  - AWS/SAA-C03
  - exam/d1
  - checklist
status: 未讀
confidence: 1
importance: 5
updated: 2026-09-27
---

# Domain 1：Design Secure Architectures（30%）

> [!abstract] 這篇怎麼用
> 這是**官方 task statement 的逐條拆解 + 對應筆記**，當成檢核清單反覆回看，不是拿來讀的內容。
> 每一條你要能回答：「這一點我能不能說出對應的 AWS 服務與判準？」不能就去讀右欄的筆記。

> [!danger] 這是最大的單一區塊
> **30%，約 15 道計分題。** 比成本（20%）多出一半。而且安全題很少是「背 IAM 語法」，多半是**串聯題**——「網路通了為什麼還是 403」需要同時理解網路與權限。

---

## Task 1.1：Design secure access to AWS resources

**Knowledge of**

- [ ] 跨多帳號的存取控制與管理 → [[治理與合規]]
- [ ] AWS 聯合存取與身分服務（IAM、IAM Identity Center） → [[身分聯合與應用存取]]
- [ ] AWS 全球基礎架構（AZ、Region） → [[服務關聯總圖]]
- [ ] AWS 安全最佳實務（最小權限原則） → [[EC2與IAM安全]]
- [ ] **責任共擔模型（shared responsibility model）** → [[00 考試總覽]]

**Skills in**

- [ ] 對 IAM user 與 root user 套用最佳實務（**MFA**） → [[EC2與IAM安全]]
- [ ] 設計彈性授權模型：user / group / role / policy → [[EC2與IAM安全]]
- [ ] 設計 role-based 存取策略：**STS、role switching、cross-account** → [[身分聯合與應用存取]]
- [ ] 多帳號安全策略：**Control Tower、SCP** → [[治理與合規]]
- [ ] 判斷何時該用**資源政策（resource policy）** → [[EC2與IAM安全]]
- [ ] 判斷何時該把 directory service 與 IAM role 聯合 → [[身分聯合與應用存取]]

> [!important] 這一節的高頻考點
> **SCP 只設上限不授權、對管理帳號無效**｜**permission boundary 限制單一身分**｜**跨帳號要兩邊都同意**｜**External ID 防 confused deputy**｜**員工用 IAM Identity Center，App 使用者用 Cognito，工作負載用 role**

---

## Task 1.2：Design secure workloads and applications

**Knowledge of**

- [ ] 應用組態與憑證安全 → [[治理與合規]]（Parameter Store / Secrets Manager）
- [ ] **AWS service endpoints** → [[EC2與網路]]
- [ ] 控制 AWS 上的 port、protocol、網路流量 → [[EC2與網路]]
- [ ] 安全的應用存取 → [[身分聯合與應用存取]]
- [ ] 安全服務與適用情境（**Cognito、GuardDuty、Macie**） → [[威脅偵測與邊界防護]]
- [ ] 來自 AWS 外部的威脅向量（**DDoS、SQL injection**） → [[威脅偵測與邊界防護]]

**Skills in**

- [ ] 設計含安全元件的 VPC 架構（SG、route table、NACL、NAT gateway） → [[EC2與網路]]
- [ ] 網路分段策略（public / private subnet） → [[EC2與網路]]
- [ ] 整合 AWS 服務保護應用（**Shield、WAF、IAM Identity Center、Secrets Manager**） → [[威脅偵測與邊界防護]]
- [ ] 保護進出 AWS 的外部網路連線（**VPN、Direct Connect**） → [[遷移與混合雲]]

> [!important] 這一節的高頻考點
> **WAF 掛不上 NLB**｜**Shield Standard 免費自動、Advanced 有 DRT 與費用保護**｜**GuardDuty 看行為、Inspector 看漏洞、Macie 看 S3 資料**｜**Session Manager 免入站端口**｜**DX 不加密，要加密得疊 VPN**

---

## Task 1.3：Determine appropriate data security controls

**Knowledge of**

- [ ] 資料存取與治理 → [[治理與合規]]
- [ ] 資料復原 → [[高可用備援與災難復原]]
- [ ] 資料保留與分類 → [[S3與資料生命週期]]
- [ ] **加密與適當的金鑰管理** → [[威脅偵測與邊界防護]]

**Skills in**

- [ ] 對齊法規遵循需求 → [[治理與合規]]（Artifact、Config、Control Tower）
- [ ] **靜態加密（KMS）** → [[威脅偵測與邊界防護]]
- [ ] **傳輸中加密（ACM + TLS）** → [[DNS與全球流量]]
- [ ] 為加密金鑰實作存取政策（**key policy**） → [[威脅偵測與邊界防護]]
- [ ] 實作資料備份與複寫 → [[高可用備援與災難復原]]、[[治理與合規]]（AWS Backup）
- [ ] 實作資料存取、生命週期與保護政策 → [[S3與資料生命週期]]
- [ ] **輪替金鑰與更新憑證** → [[威脅偵測與邊界防護]]

> [!important] 這一節的高頻考點
> **KMS CMK 是 Region 專屬的**（跨 Region 複寫加密資料必考）｜**SSE-KMS 需要兩層授權：`s3:GetObject` + `kms:Decrypt`**｜**Object Lock Compliance mode 連 root 都不能刪**｜**CloudFront 的 ACM 憑證必須在 us-east-1**｜**既有資源無法就地加密**（要 snapshot → 加密複製 → 還原）

---

## 📚 本 Domain 的主力筆記

| 筆記 | 涵蓋 |
|---|---|
| [[EC2與IAM安全]] | 工作負載如何取得權限、instance profile、三層診斷 |
| [[身分聯合與應用存取]] | Cognito、Identity Center、Directory Service、STS、**IAM 權限評估流程圖** |
| [[威脅偵測與邊界防護]] | GuardDuty 系列、WAF/Shield、**KMS 深入**、S3 五種加密 |
| [[治理與合規]] | Organizations/SCP、Config、Systems Manager、AWS Backup |
| [[EC2與網路]] | SG/NACL、endpoint、排錯決策樹 |

## ✍️ Domain 1 自我檢核

1. SCP、permission boundary、IAM policy、resource policy——**有效權限**怎麼算出來？哪一層的 Deny 最強？
2. 「網路通了但還是 403」——你會依序檢查哪五層？
3. 員工、App 終端使用者、EC2 工作負載，分別用什麼機制取得身分？各自的錯誤選項長什麼樣？
4. GuardDuty / Inspector / Macie / Security Hub / Detective 各一句話。
5. 跨 Region 複寫 SSE-KMS 加密的 S3 物件，為什麼常常失敗？
6. 「連 root 都不能刪除的備份」有哪兩種實作？

> [!success]- 參考答案
> 1. **有效權限 = IAM policy ∩ SCP ∩ permission boundary ∩ resource policy**；**任何一層的 explicit Deny 直接否決**，沒有例外。預設是隱含拒絕。
> 2. IAM identity policy → 資源 policy（bucket/key policy）→ VPC endpoint policy → SCP → **KMS 解密權限**。（**不查 route table**——那是 timeout 的症狀。）
> 3. 員工 → **IAM Identity Center**（錯誤選項：每帳號建 IAM user）；App 使用者 → **Cognito**（錯誤選項：為每位使用者建 IAM user）；工作負載 → **IAM role + instance profile**（錯誤選項：把 access key 放進 user data/AMI）。
> 4. GuardDuty=威脅行為偵測；Inspector=漏洞/CVE 掃描；Macie=S3 敏感資料發現；Security Hub=findings 彙整與合規計分；Detective=根因調查。
> 5. **CMK 是 Region 專屬的**。複寫角色必須能在來源 Region `kms:Decrypt`、在目的地 Region `kms:Encrypt`；解法是授予兩邊金鑰權限或用 **multi-Region key**。
> 6. **S3 Object Lock（Compliance mode）** 與 **AWS Backup Vault Lock（合規模式）**。Governance mode 允許特權使用者繞過，不符合。

## 🔗 相關

- [[00 考試總覽]]
- [[Domain 2 設計高韌性架構]]
- [[常見陷阱與誘答選項識別]]
- [[情境題庫]]（Q1–Q8、Q31–Q39）
