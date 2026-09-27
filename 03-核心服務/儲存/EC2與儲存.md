---
title: EC2與儲存
tags:
  - AWS/SAA-C03
  - EC2
  - storage
  - service/ebs
  - service/efs
  - service/fsx
  - exam/d3
  - numbers
status: 未讀
confidence: 1
importance: 5
updated: 2026-09-27
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

---

# 🧠 技術層
*它實際上怎麼運作——心智模型、機制、限制。**第一輪只讀這一層**，先把架構直覺建立起來。*

## 關聯 1：EC2 → EBS → Snapshot → 恢復

EBS volume 是 AZ 範圍。Snapshot 是備份機制，建立 snapshot 後可於支援的 AZ 建新 volume；跨 Region 需複製 snapshot。Snapshot 儲存於 AWS 管理的 S3，但不是你可用一般 S3 bucket API 瀏覽的物件。快照不是「資料庫交易一致性」的同義詞；需要時配合應用凍結/資料庫備份。AMI 可包含用於建立 root volume 的 EBS snapshots；AMI 與資料庫備份目的不同。

## 關聯 2：EC2 → 網路共享 → EFS

兩台跨 AZ EC2 要同時讀寫共享 Linux 目錄，可考慮 EFS；設 mount target，讓 EC2 到 EFS 的網路及 SG 允許 NFS 2049。EFS 的 API VPC endpoint 與 NFS data mount 是不同路徑，不能把 API endpoint 誤當 NFS mount target。見 [[EC2與網路]]。

## 關聯 3：EC2 → S3

用 SDK/API 對 S3 存取，不要把它想成 EBS。需同時考慮 IAM role、bucket policy、S3 endpoint/出網路徑、KMS 權限；詳見 [[EC2與IAM安全]]、[[S3與資料生命週期]]。

## EBS volume 類型

| 類型 | 介質 | 特性 | 何時選 |
|---|---|---|---|
| **gp3** | SSD | **IOPS 與吞吐量可獨立於容量調整**，比 gp2 便宜約 20% | **通用預設首選**。題目說「想在不增加容量的前提下提高 IOPS」→ gp3 |
| **gp2** | SSD | IOPS 綁定容量（3 IOPS/GB） | 舊世代，考題常作為「應該升級成 gp3」的現況 |
| **io2 / io2 Block Express** | SSD | **最高 IOPS 與耐久性**，支援 **Multi-Attach** | `critical database`、`sustained IOPS > 16000`、`highest durability` |
| **st1** | HDD | 吞吐量優化，**不能當開機碟** | `big data`、`log processing`、**大型循序讀寫** |
| **sc1** | HDD | 最低成本、冷資料，**不能當開機碟** | `infrequently accessed`、`lowest cost` |

> [!danger] 三個常見判斷
> **「需要極高 IOPS 的資料庫」** → io2（不是 gp3，若題目給的 IOPS 數字超過 gp3 上限）。
> **「大量循序掃描的巨量資料、成本敏感」** → **st1**（HDD 在循序吞吐上比 SSD 划算）。注意 **HDD 類型不能作為開機磁碟**。
> **「想提升效能但不想加大容量」** → **gp3**（gp2 做不到）。

## EBS 的其他機制

- **Multi-Attach**：僅 **io1/io2**，同一 AZ、最多 16 台實例，且**需要 cluster-aware 檔案系統**（如 GFS2）。**一般 ext4/XFS 掛兩台會壞資料**——這不是共享檔案系統的替代方案。
- **加密**：建立時啟用。**既有未加密 volume → snapshot → 複製 snapshot 時加密 → 由該 snapshot 建新 volume**。無法就地加密。加密的 snapshot 複製到其他 Region 時要注意目的地 Region 的 KMS key（見 [[威脅偵測與邊界防護]]）。
- **Snapshot 是增量的**，但刪除任一 snapshot 不會讓還原失效（AWS 會保留必要區塊）。
- **快速 snapshot 還原（FSR）**：新建 volume 免除首次讀取的延遲初始化。
- **Data Lifecycle Manager (DLM)**：自動化 snapshot 排程與保留（或用 **AWS Backup** 統一管理，見 [[治理與合規]]）。

## EFS 深入

| 面向 | 選項 |
|---|---|
| **儲存類別** | Standard、**One Zone**（單 AZ，便宜約 47%）、Infrequent Access（IA）、Archive |
| **生命週期管理** | 未存取 N 天後自動轉 IA/Archive |
| **吞吐量模式** | **Elastic**（預設，自動調整）、Provisioned、Bursting |
| **效能模式** | General Purpose（預設，低延遲）、Max I/O（高併發但延遲較高） |

> [!tip] EFS vs FSx 的分界
> **EFS = Linux / NFS**。跨 AZ、自動擴展、無容量規劃。
> 題幹出現 **Windows**、**SMB**、**Active Directory 整合** → **FSx for Windows File Server**，不是 EFS。

## FSx 四種類型

| 類型 | 協定 | 題幹關鍵字 |
|---|---|---|
| **FSx for Windows File Server** | **SMB** | `Windows`、`Active Directory`、`SMB share`、`NTFS` |
| **FSx for Lustre** | Lustre（POSIX） | **`HPC`**、`machine learning`、`high performance computing`、**可與 S3 連動**（延遲載入 S3 資料） |
| **FSx for NetApp ONTAP** | **NFS + SMB + iSCSI 都支援** | `multi-protocol`、`existing NetApp`、`snapshots/cloning` |
| **FSx for OpenZFS** | NFS | `ZFS`、從地端 ZFS 遷移 |

> [!important] 最好記的兩句
> `HPC` 或 `machine learning training with data in S3` → **FSx for Lustre**。
> `Windows 應用要共享檔案` → **FSx for Windows File Server**。

## 儲存選型速決表

| 題幹說 | 答案 |
|---|---|
| 單一實例的低延遲區塊裝置 | **EBS** |
| 多台 **Linux** 同時讀寫同一目錄 | **EFS** |
| 多台 **Windows** 同時讀寫（SMB/AD） | **FSx for Windows** |
| HPC / ML 高吞吐平行檔案系統 | **FSx for Lustre** |
| 物件、靜態資產、備份、資料湖 | **S3** |
| 暫存、可重建、要最高 IOPS | **Instance store** |
| 本地應用要繼續用 NFS/SMB/iSCSI，資料放雲端 | **Storage Gateway**（見 [[遷移與混合雲]]） |

### 四個典型場景的完整推理

上表是查表用的；實務與考題還要多想一層：

1. 單一 EC2 的資料庫需要持久 block I/O → EBS；再評估是否直接用 RDS。
2. 兩個 AZ 的 Linux web nodes 共享上傳檔 → EFS；若只是圖片物件，S3 也可能更合適。
3. 可重新產生、追求暫存速度 → instance store；設計節點失效時重建。
4. 備份 EC2 OS volume → EBS snapshot/AMI；若要應用資料跨 Region 恢復，檢查快照複製與 RPO/RTO。

---

# 🎯 考試層
*考試會怎麼問——關鍵字反射、誘答陷阱、閉卷檢核。**第二輪與考前讀這一層**。*

## 🎯 考點速記

看到 `POSIX` / `shared directory` / `NFS` → **EFS**（**不是 S3**）
看到 `Windows` / `SMB` / `Active Directory` → **FSx for Windows**
看到 `HPC` / `ML training` / `parallel file system` → **FSx for Lustre**
看到 `multi-protocol`（NFS+SMB+iSCSI）→ **FSx for NetApp ONTAP**
看到 `large sequential` + 成本敏感 + 非開機碟 → **st1（HDD）**
看到 `increase IOPS without increasing size` → **gp3**
看到 `highest IOPS / critical database` → **io2**
看到 `temporary` / `can be regenerated` / 最高 IOPS → **instance store**
看到 `on-prem app must keep using NFS/SMB/iSCSI` → **Storage Gateway**

## 💣 真實場景陷阱

- **EBS Multi-Attach 不是共享檔案系統**：掛一般 ext4/XFS 到兩台實例會**毀損資料**，必須用 cluster-aware 檔案系統（如 GFS2），且僅限 io1/io2、同一 AZ。
- **既有 volume 無法就地加密**：必須 snapshot → 複製 snapshot 時加密 → 由該 snapshot 建新 volume。
- **root volume 的 Delete on termination 預設為 true**，額外掛載的 volume 預設為 false。實務上刪了實例才發現資料沒了。
- **EFS 的 API endpoint 不是 NFS mount target**：兩者是不同路徑，把 interface endpoint 當 mount target 用會連不上。
- **跨 AZ 掛載 EFS 會產生跨 AZ 流量費**，高吞吐場景要納入成本評估。

## ✍️ 自我檢核

1. 五種儲存（EBS / instance store / EFS / FSx / S3）各用一句話說出存取介面與典型情境。
2. 為什麼 EBS Multi-Attach 不能當成 EFS 的替代方案？列出三個限制。
3. 「提高 IOPS 但不想加大容量」——gp2 做得到嗎？為什麼？
4. 既有的未加密 EBS volume 要加密，完整步驟是什麼？
5. st1 與 sc1 有什麼共同限制？什麼題幹會暗示可以用它們？

<details>
<summary>參考答案</summary>

1. **EBS**=區塊裝置、單一實例持久磁碟；**instance store**=主機本地區塊、暫存可重建；**EFS**=NFS、多台 Linux 跨 AZ 共享 POSIX；**FSx**=託管檔案系統（Windows SMB / Lustre HPC / ONTAP 多協定）；**S3**=物件 API、靜態資產與資料湖。
2. **① 僅 io1/io2 ② 僅同一 AZ（不能跨 AZ）③ 需要 cluster-aware 檔案系統**，最多 16 台。一般檔案系統會毀損資料。
3. **做不到**。gp2 的 IOPS 綁定容量（3 IOPS/GB），只能靠加大容量提升。**gp3 的 IOPS 與吞吐量可獨立調整**。
4. **建立 snapshot → 複製該 snapshot 並在複製時指定加密 → 由加密的 snapshot 建立新 volume → 換掛到實例。** 無法就地啟用。
5. 兩者都是 HDD，**不能作為開機磁碟**。題幹說「這些 volume 不作為開機碟」或強調「大型循序讀寫 + 成本最低」就是在暗示 HDD。

</details>

---

# 🔗 相關

參考 [[03 官方資源清單]]；返回 [[服務關聯總圖]]、[[00 考試總覽]]。
