---
tags: [AWS, EC2, storage]
---
# EC2與儲存

EC2 是運算主機；儲存選型同時決定存取介面、生命週期、AZ 邊界與備份方式。

| 方案 | EC2 看到的介面 | 共享/範圍 | 生命週期及典型用途 |
|---|---|---|---|
| EBS | Block device，格式化後掛載 | volume 位於一個 AZ；一般附於同 AZ instance；Multi-Attach 僅特定類型/條件，不等於一般共享檔案系統 | volume 可獨立於 instance 存在，root volume 的 Delete on termination 預設設定需檢查；OS、交易資料、需要低延遲 block |
| Instance store | 主機本地 block | 綁定宿主；不能當跨主機共享資料 | 臨時 scratch/cache，停止、休眠或終止等情況資料可能失去；不得作唯一持久副本 |
| EFS | NFS 共享檔案 | 多 EC2 跨 AZ 同時掛載（依 EFS storage class/掛載點設計） | 多主機共享 Linux 檔案；注意 NFS 網路、延遲、吞吐和費用 |
| S3 | Object API/HTTP，非原生 POSIX 磁碟 | 多應用經 API 存取，bucket/資料跨 AZ（依 storage class） | 靜態物件、備份、媒體與資料湖；應用改用 object semantics |
| FSx | 受管檔案系統，依類型選 SMB/NFS 等 | 按 FSx 類型與部署模式 | 需要 Windows SMB 或特定檔案系統能力時評估 |

## 關聯 1：EC2 → EBS → Snapshot → 恢復

EBS volume 是 AZ 範圍。Snapshot 是備份機制，建立 snapshot 後可於支援的 AZ 建新 volume；跨 Region 需複製 snapshot。Snapshot 儲存於 AWS 管理的 S3，但不是你可用一般 S3 bucket API 瀏覽的物件。快照不是「資料庫交易一致性」的同義詞；需要時配合應用凍結/資料庫備份。AMI 可包含用於建立 root volume 的 EBS snapshots；AMI 與資料庫備份目的不同。

## 關聯 2：EC2 → 網路共享 → EFS

兩台跨 AZ EC2 要同時讀寫共享 Linux 目錄，可考慮 EFS；設 mount target，讓 EC2 到 EFS 的網路及 SG 允許 NFS 2049。EFS 的 API VPC endpoint 與 NFS data mount 是不同路徑，不能把 API endpoint 誤當 NFS mount target。見 [[EC2與網路]]。

## 關聯 3：EC2 → S3

用 SDK/API 對 S3 存取，不要把它想成 EBS。需同時考慮 IAM role、bucket policy、S3 endpoint/出網路徑、KMS 權限；詳見 [[EC2與IAM安全]]、[[S3與資料生命週期]]。

## 情境題

1. 單一 EC2 的資料庫需要持久 block I/O → EBS；再評估是否直接用 RDS。
2. 兩個 AZ 的 Linux web nodes 共享上傳檔 → EFS；若只是圖片物件，S3 也可能更合適。
3. 可重新產生、追求暫存速度 → instance store；設計節點失效時重建。
4. 備份 EC2 OS volume → EBS snapshot/AMI；若要應用資料跨 Region 恢復，檢查快照複製與 RPO/RTO。

參考 [[官方資料]]；返回 [[服務關聯總圖]]。
