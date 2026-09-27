---
title: 02 Obsidian 設定與外掛建議
tags:
  - AWS/SAA-C03
  - meta
  - obsidian
status: 讀過
confidence: 3
importance: 4
updated: 2026-09-27
---

# 02 Obsidian 設定與外掛建議

## ⚙️ 必要設定（沒設會有東西不顯示）

| 設定 | 位置 | 值 | 為什麼 |
|---|---|---|---|
| **Use Wikilinks** | Settings → Files & Links | 開啟 | 本 Vault 全部用 `[[ ]]` 雙括號連結 |
| Default location for new notes | Files & Links | `Same folder as current file` | 新增筆記不會亂跑 |
| **Strict line breaks** | Editor | **關閉** | 表格與列表排版才正常 |
| Readable line length | Appearance | 開啟 | 長篇筆記好讀 |
| Show frontmatter | Editor | 開啟 | 才看得到 `confidence` 欄位 |

> [!note] 為什麼連結不用相對路徑
> 本 Vault 的 `[[連結]]` 一律只寫**檔名**，不寫路徑。這樣**搬動資料夾不會斷鏈**——實際上這份 vault 就重組過一次結構，26 篇筆記的連結全部不受影響。

## 🔌 外掛（皆非必要，但建議依序安裝）

| 外掛 | 作用 | 在本 Vault 的用途 |
|---|---|---|
| **Spaced Repetition** | 間隔重複 | `08-Flashcards` 的 `::` 閃卡 |
| **Dataview** | 查詢筆記 | [[README]] 的進度儀表板、[[每日追蹤模板]] 的弱點彙整 |
| **Advanced Tables** | 表格編輯 | 這份 vault 有大量比較表，手動對齊很痛苦 |
| **Templater** | 模板 | 每日回顧、錯題卡 |
| **Excalidraw** 或內建 **Canvas** | 畫圖 | Day 22 的「憑記憶畫架構圖」任務 |
| **Mermaid**（內建） | 流程圖 | 決策樹與架構圖已大量使用，不需外掛 |

> [!note] Mermaid 是內建的
> 本 Vault 的決策樹用 ` ```mermaid ` 語法，Obsidian 原生支援。若圖沒顯示，檢查程式碼區塊的語言標記有沒有打錯。

---

## 🃏 Spaced Repetition 設定

`Settings → Spaced Repetition`：

```
Flashcard tags:        #flashcards
Single-line separator: ::
Multiline separator:   ?
Convert highlights to clozes: on
```

設定完成後，`08-Flashcards` 的四個檔案會被自動辨識。建議節奏：**每天 10 分鐘**，通勤時用手機刷。

---

## 📝 錯題卡 Templater 模板

放在 `_templates/錯題卡.md`：

```markdown
---
tags: [weak, 錯題, AWS/SAA-C03]
date: <% tp.date.now("YYYY-MM-DD") %>
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
[[]]

## 一句話教訓
```

> [!tip] 「變形題」那一欄是關鍵
> 只記正解等於背答案。**寫下「改掉哪個條件答案會變」，才代表你抓到的是判準而不是關鍵字。** 詳見 [[模考與錯題模板]] 的錯因分類。

---

## 🎨 中文顯示

這份 vault 全部是繁體中文加英文術語混排。若在終端機或編輯器出現方框、字寬錯亂：

- **Obsidian**：`Settings → Appearance → Font`，選一套含中文的等寬字型（如 `Maple Mono NF CN`、`Sarasa Mono TC`、`PingFang TC`）。
- **終端機（iTerm2）**：純西文的 Nerd Font（`MesloLGS NF`、`Hack NF`）**不含中文字符**。改用 `Maple Mono NF CN` 這類中英合一字型，並勾選 **Treat ambiguous-width characters as double width**，中文標點才不會半格偏移。
- **git**：`git config --global core.quotepath false`，否則中文檔名會被印成 `\346\210\221` 這種八進位轉義。

---

## 📱 跨裝置

- **Obsidian Sync / iCloud / Git** 任一種同步都可以；本 Vault 是純 Markdown，沒有鎖定。
- **通勤時只讀** `08-Flashcards` 與 `06-速查表`（純文字、手機友善）。
- `05-實作實驗室` 需要終端機與 AWS Console，安排在有電腦的時段。

> [!warning] 若用 Git 同步
> `.gitignore` 已排除 `.obsidian/workspace.json`（各裝置的版面狀態），避免每次開啟都產生無意義的變更。

## 🔗 相關

- [[01 如何使用本 Vault]]
- [[README]]
- [[每日追蹤模板]]
