# NemoClaw Applications 系列 — 官方 Troubleshooting 對照總結

> 對照來源：[NemoClaw Applications Troubleshooting](https://build.nvidia.com/spark/nemoclaw-applications/troubleshooting)
> 涵蓋文章：News Digest、Software Development Agent、Deck Reviewer、Calendar Negotiator

---

## 為什麼要做這份對照

這四篇實測過程中，我們獨立踩到、排查、記錄了好幾個「模型行為不穩定」的案例。回頭看官方 Troubleshooting 頁面才發現：**這些現象官方早就知道，並且已經明文記錄在文件裡**，只是官方文件只給「症狀 → 原因 → 修法」的三行摘要，沒有像我們這樣做完整的重現過程與量化分析。這份對照表把兩邊的內容並排放在一起，證明我們的踩坑不是操作失誤，而是這套系統目前已知、尚待根本解決的限制。

---

## 一、`Version:` 大小寫 bug（我們在 News Digest 那篇踩到）

| | 內容 |
|---|---|
| **官方記錄** | `openshell policy set` fails with `unknown field 'Version'`。原因：`openshell 0.0.44` 的 round-trip bug——`policy get --full` 印出大寫 `Version:`，但 `policy set` 只接受小寫 `version:` |
| **官方建議解法** | 應急：`sed -i 's/^Version:/version:/' policy.yaml`。**更推薦**：完全跳過完整匯出/匯入流程，改用增量式的 `nemoclaw policy-add --from-file` |
| **我們的發現** | 完全一致。我們當時手動用 `openshell policy get --full` → 編輯 → `policy set` 這條路，真的撞上同樣的錯誤訊息，用一模一樣的 `sed` 指令修正 |
| **結論** | 這不是我們的操作問題，是官方承認的已知 bug，且官方自己也建議不要走這條路 |

---

## 二、`preset.name` 命名規則（解答了我們當時的疑惑）

| | 內容 |
|---|---|
| **官方記錄** | `Preset must declare preset.name (lowercase, hyphenated RFC 1123 label)`。**限制只在最外層 `preset.name`**——`network_policies.<group>` 這個 key 跟它內部的 `name` 欄位「可以」用底線 |
| **我們的發現** | 我們手動改 YAML 時用了底線（如 `news_sources`）沒被擋下來，但一開始不確定為什麼；這條說明解答了範圍限制只在最外層這件事 |

---

## 三、Approval Gate 被繞過（Software Development Agent）

| | 內容 |
|---|---|
| **官方記錄** | `Agent modifies files outside the plan` → 原因：`Plan-approval checkpoint was disabled`。修法：在設定裡把 Q5 回答 `yes`，agent 必須印出 `PLAN READY` 並停下來等 `approve` |
| **我們的發現** | 我們實測到 agent 在 prompt 跟 feature request 一次貼上時,會自己判斷 approval gate = no,跳過核准直接動手。指出問題並用更明確措辭（"mandatory, no exceptions, ever"）後才穩定遵守 |
| **官方沒寫但我們補上的** | 具體的「怎麼會發生」的機制——一次貼上 prompt + feature request 會讓 agent 誤判成使用者已經跳過設定流程 |

---

## 四、CRITICAL 忽略被繞過（Deck Reviewer）—— 官方也點名這是 regression

| | 內容 |
|---|---|
| **官方記錄** | `CRITICAL findings disappear from the report` → 原因：`Auto-dismiss was attempted (should be impossible)`。官方原文：**"This is a regression"**。修法：`re-paste the full prompt to restore the rule` |
| **我們的發現** | 完全一致的現象——第一次 `dismiss` 一個 CRITICAL 發現時，agent 完全沒有觸發 prompt 要求的二次確認，直接寫入 `dismissals.jsonl` |
| **我們額外驗證的部分** | 官方只給了「重貼 prompt」這個解法，**我們實際做了複測**：指出錯誤 → agent 承認並撤銷 → 用完全相同請求複測 → 這次正確要求 `yes, dismiss critical` 二次確認 → 報告裡還主動加註稽核標記。證實了「規則違反後可以被修正，且修正是穩定持久的」 |

---

## 五、Booking 檔案被覆寫 / 角色邊界規則被繞過（Calendar Negotiator）

| | 內容 |
|---|---|
| **官方記錄** | `Booking file overwrites a confirmed prior booking` → 原因：`Agent did not honor the "never overwrite" rule`。修法：檢查有沒有 `-v2.md`，如果被覆寫,`re-paste the full prompt to restore the rule` |
| **我們的發現** | 我們踩到的是同一類別但不同案例：在 **Propose-only** 模式下，agent 宣稱「Proposal sent」、「sending... now」——但這個模式下 agent 根本沒有發送能力,這是角色邊界規則被違反,不是檔案覆寫,但修正模式完全一致（指出→承認→修正） |

---

## 六、模型冷啟動很慢（我們在 NemoClaw 安裝篇跟 Policy Setup 都遇過）

| | 內容 |
|---|---|
| **官方記錄**（Policy Setup 分頁） | `Telegram bot receives messages but returns nothing for 60+ seconds` → 原因：`First response on a 120B model is slow (cold start)`。這是**預期行為**，如果之後的回應也慢，建議在 onboard 精靈裡換小一點的模型 |
| **我們的發現** | 這跟我們在安裝篇、以及切換到 Nemotron 測試時反覆遇到的「載入要等 5-10 分鐘」完全對應，官方直接承認這是預期的正常現象，不是故障 |

---

## 七、UMA 記憶體問題（我們沒踩到，但很可能是稍早那次詭異卡住的解答）

| | 內容 |
|---|---|
| **官方記錄** | DGX Spark 用 Unified Memory Architecture,很多應用還沒完全適配，即使容量足夠也可能遇到記憶體問題。解法：`sudo sh -c 'sync; echo 3 > /proc/sys/vm/drop_caches'` |
| **我們的發現** | 我們今天遇到過幾次模型載入詭異卡住、GPU 記憶體沒有正確釋放的情況，最後是用「重開機」解決的——**這個指令可能可以在下次遇到同樣狀況時，作為比重開機更輕量的第一步嘗試** |

---

## 八、官方給的除錯優先順序建議

> 如果問題發生在 `nemoclaw`/`openshell` 指令層級,先查 **General sandbox & policy issues**；**多數回報的問題其實來自安裝層,而非應用層**——建議先查 [NemoClaw 安裝文章的 Troubleshooting 分頁](https://build.nvidia.com/spark/nemoclaw)。

這跟我們四篇文章的架構安排不謀而合——我們也是先把安裝篇的環境問題排查清楚，才進到應用層的測試。

---

## 總結：這四篇實測記錄補上了官方文件沒寫的東西

官方 Troubleshooting 頁面提供的是「症狀 → 原因 → 修法」的三行摘要，這對於快速排查已知問題很有幫助。但我們四篇文章額外補上了三件事，是官方文件沒有的：

1. **完整的重現過程**——不只是「這會發生」，而是「在什麼具體條件下會發生、log 裡長什麼樣子」
2. **量化的驗收數據**——例如 Deck Reviewer 的 Precision/Recall 計算，官方文件只說「建議自己校準」，沒有給出實際跑出來的數字長什麼樣子
3. **複測驗證**——官方文件的解法大多停在「重貼 prompt」，我們實際做了「指出錯誤 → 修正 → 用完全相同請求複測」的完整循環，證實了修正是否真的持久有效

這四篇文章加上這份對照總結，構成了一套完整、有紮實證據支撐的 NemoClaw Applications 系列實測記錄。
