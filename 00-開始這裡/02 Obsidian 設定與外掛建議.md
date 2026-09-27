---
title: 02 Obsidian 設定與外掛建議
tags:
  - AWS/SAA-C03
  - meta
  - obsidian
status: 讀過
confidence: 3
importance: 3
updated: 2026-09-27
---

# 02 Obsidian 設定

> [!important] 這份 Vault 不需要任何外掛
> 全部內容都用**標準 Markdown + Obsidian 內建功能**寫成。
> 開啟就能讀，**一個社群外掛都不用裝**。下面只有三個設定要確認，其餘都是可選的。

---

## ⚙️ 三個要確認的設定

| 設定 | 位置 | 值 | 為什麼 |
|---|---|---|---|
| **Use Wikilinks** | Settings → Files & Links | 開啟 | 本 Vault 的連結都是 `[[檔名]]` |
| **Strict line breaks** | Settings → Editor | **關閉** | 表格與列表排版才正常 |
| Readable line length | Settings → Appearance | 開啟 | 長篇筆記好讀 |

其他維持預設即可。

---

## 📦 內建功能就夠用

| 需求 | 用什麼（全部內建） |
|---|---|
| 看筆記之間的關聯 | **Graph view**。某個主題孤零零沒連線，代表你還沒把它放進整體架構 |
| 在長筆記中跳段落 | **Outline 面板**。會看到 `🧠 技術層` / `🎯 考試層` / `🔗 相關` 三個區塊 |
| 找出誰引用了這篇 | **Backlinks 面板** |
| 找弱點 | **搜尋 `⌘⇧F`**：`confidence: 1`、`confidence: 2`、`tag:#weak` |
| 依主題篩選 | **Tags 面板**：`#exam/d1`、`#numbers`、`#service/s3` |
| 決策樹與架構圖 | **Mermaid**，Obsidian 原生支援 |
| 重點框（提示、警告） | **Callout**，Obsidian 原生支援 |
| 摺疊的解答 | 用標準 HTML `<details>`，**在 Obsidian、GitHub、VS Code 都能摺疊** |

---

## 🃏 閃卡怎麼用（不裝外掛也行）

`08-Flashcards` 的格式是 `問題::答案`：

```
S3 Standard-IA 的最低儲存期間是多久？::30 天
```

**不裝外掛的用法**：把游標或手指遮住 `::` 右邊，答完再看。手機上讀也適用。

**想要自動排程複習**（可選）：裝 **Spaced Repetition** 外掛，設定
`Flashcard tags: #flashcards`、`Single-line separator: ::`、`Multiline separator: ?`。
這是整份 Vault 唯一會用到外掛的地方，而且**不裝也完全能用**。

---

## 📝 錯題卡（複製貼上即可，不需 Templater）

每次做錯就複製這段到新筆記：

```markdown
---
tags: [weak, 錯題, AWS/SAA-C03]
date: 2026-09-28
domain:
錯因:
---
# 題目摘要


## 題幹的關鍵限制詞（圈出來）


## 我選了什麼／為什麼錯


## 正解成立的判準


## 每個錯誤選項踩到哪個條件
- A：
- B：
- C：

## 變形題：把哪個條件改掉，答案就會變？


## 對應筆記


## 一句話教訓
```

> [!tip] 「變形題」那一欄是關鍵
> 只記正解等於背答案。**寫下「改掉哪個條件答案會變」，才代表你抓到的是判準而不是關鍵字。**
> 錯因分類見 [[模考與錯題模板]]。

---

## 🌐 在 Obsidian 以外閱讀

這份 Vault 也能直接在 **GitHub 網頁**、**VS Code**、**任何 Markdown 編輯器**上讀：

| 元素 | Obsidian | GitHub / 其他 |
|---|:-:|---|
| 表格、清單、程式碼 | ✅ | ✅ |
| Mermaid 圖 | ✅ | ✅ GitHub 原生支援 |
| `<details>` 摺疊解答 | ✅ | ✅ |
| Callout 重點框 | ✅ 彩色框 | 顯示為引言，內容完整可讀 |
| `[[雙括號連結]]` | ✅ 可點擊 | 顯示為純文字，**看得到檔名但不能點** |

> [!note] 連結為什麼保留雙括號
> 改成標準 Markdown 連結雖然在 GitHub 上可點，但路徑含中文與空白會變得又長又難維護，而且**搬動檔案就得全部重寫**。
> 目前的寫法讓 Obsidian 在改檔名時自動更新所有連結——**維護成本低很多**。在 GitHub 上仍讀得到內容，只是要自己找檔案。

---

## 📱 跨裝置

- **Obsidian Sync / iCloud / Git** 任一種都可以；純 Markdown，沒有鎖定。
- **通勤時只讀** `08-Flashcards` 與 `06-速查表`（純文字、手機友善）。
- `05-實作實驗室` 需要終端機與 AWS Console，安排在有電腦的時段。
- `.gitignore` 已排除 `.obsidian/workspace.json`（各裝置的版面狀態），避免同步時一直產生無意義變更。

---

## 🎨 中文顯示

若出現方框、字寬錯亂：

- **Obsidian**：Settings → Appearance → Font，選含中文的等寬字型（`Maple Mono NF CN`、`Sarasa Mono TC`、`PingFang TC`）。
- **終端機（iTerm2）**：純西文的 Nerd Font（`MesloLGS NF`、`Hack NF`）**不含中文字符**。改用中英合一字型，並勾選 **Treat ambiguous-width characters as double width**。
- **git**：`git config --global core.quotepath false`，否則中文檔名會印成 `\346\210\221` 這類八進位轉義。

## 🔗 相關

- [[01 如何使用本 Vault]]
- [[README]]
- [[每日追蹤模板]]
