# NemoClaw CLI 常用指令參考

> 版本：`NemoClaw v0.0.81`（`nemoclaw` 指令本身可執行 `nemoclaw` 查看完整說明）
> 官方文件：https://docs.nvidia.com/nemoclaw/user-guide/openclaw/home

## 指令格式說明

NemoClaw 指令分兩種：

- **全域指令**：不需要 sandbox 名稱前綴，例如 `nemoclaw status`、`nemoclaw list`
- **Sandbox 指令**：需要以 sandbox 名稱開頭，例如 `nemoclaw gb10-assistant status`

# NemoClaw / OpenClaw 指令分層參考

## 核心觀念：兩層、兩套指令，不互通






**判斷原則只有一句話：**
> 要管「這個 sandbox 容器本身」→ 用 `nemoclaw`（在 host 打）
> 要跟「容器裡跑的那個 agent」互動 → 用 `openclaw`（連進 sandbox 後打）

`nemoclaw` 這個名字容易讓人誤會 sandbox 裡也有對應指令 — **實際上沒有**，一旦 `connect` 進去，你面對的是完全獨立的 `openclaw` 生態系，兩邊指令不重疊、不互通。

---

## 第一層：`nemoclaw`（在 Host 上打，不需要 connect 進 sandbox）

### 入門 / 建立

```bash
nemoclaw onboard                          # 互動精靈，建立新 sandbox
nemoclaw list                             # 列出所有 sandbox
nemoclaw use gb10-assistant                # 設定預設 sandbox
```

### Sandbox 生命週期管理

```bash
nemoclaw gb10-assistant connect            # 連進 sandbox（唯一會跨到第二層的入口）
nemoclaw gb10-assistant status             # 看這個 sandbox 健康狀態
nemoclaw gb10-assistant logs --follow      # 串流 sandbox log
nemoclaw gb10-assistant rebuild            # 重建 sandbox（換模型後常用）
nemoclaw gb10-assistant destroy --yes      # 刪除 sandbox
nemoclaw gb10-assistant doctor --fix       # 診斷並修復 sandbox/gateway 問題
```

### 檔案傳輸（host ↔ sandbox）

```bash
nemoclaw gb10-assistant download <sandbox路徑> <host路徑>
nemoclaw gb10-assistant upload <host路徑> <sandbox路徑>
```

### Policy（網路/檔案系統白名單）

```bash
nemoclaw gb10-assistant policy-list                     # 看目前套用的 preset
nemoclaw gb10-assistant policy-add                      # 加內建 preset（僅限固定清單，如 npm/telegram/discord）
nemoclaw gb10-assistant policy-explain --json            # 解釋目前完整生效的 policy
```

> ⚠️ `policy-add` 只支援約 11 個內建服務，**不接受自訂網域**。要放行一般網站需改走底層 `openshell policy get --full` / `openshell policy set --policy <file> --wait`（這已經不算 `nemoclaw` 系列，是更底層的 `openshell`）。

### Messaging Channels

```bash
nemoclaw gb10-assistant channels list
nemoclaw gb10-assistant channels status --channel telegram
```

### 推論模型

```bash
nemoclaw inference get                                          # 查目前推論路由
nemoclaw inference set --model qwen3.6:35b --provider ollama-local --sandbox gb10-assistant
```

### 全域狀態 / 資源 / 服務

```bash
nemoclaw status --json          # 全域 sandbox/host 服務狀態
nemoclaw resources --json       # CPU/RAM/GPU VRAM
nemoclaw tunnel start           # 啟動對外 tunnel（cloudflared）
```

---

## 第二層：`openclaw`（先 `nemoclaw <name> connect` 進 sandbox 後才打）

進去之後，提示字元會變成：
```
sandbox@a7ec382e478f:~$
```
**這裡開始，指令一律是 `openclaw`，不是 `nemoclaw`。**

### 對話 / TUI

```bash
openclaw tui                            # 開互動終端機介面對話
openclaw agent -m "hello"               # 非互動跑一次 agent turn
```

### 狀態 / 診斷（今天沒用過，但很實用）

```bash
openclaw status                         # 一次看 Gateway/channel/model/最近 session 狀態
openclaw doctor --fix                   # 診斷並自動修復 config/gateway/plugin/channel 問題
openclaw --version                      # 看目前 OpenClaw 版本
```

### Cron（排程任務）

```bash
openclaw cron create "0 8 * * *" "訊息內容" --name daily-tech-news --announce --channel discord --to <channel_id>
openclaw cron list                      # 列出所有 cron job
openclaw cron run <job_id>              # 手動觸發一次
```

### Session（對話紀錄）

```bash
openclaw sessions list                  # 列出所有儲存的對話 session
openclaw sessions reset <key>           # 重設某個 session
openclaw sessions export <key>          # 匯出 session JSONL
```

### Memory（記憶檔案）

```bash
openclaw memory --help                  # 查看正確用法（搜尋/檢視/重建索引 memory 檔案）
```

### Agents（sandbox 內的 agent 設定）

```bash
openclaw agents list                    # 列出這個 sandbox 裡設定的 agent 與 routing 規則
```

### Channels（通訊管道，sandbox 內視角）

```bash
openclaw channels status --probe        # 即時連線測試 channel 健康狀態（比看 log 更直接）
```

### 離開 sandbox

```bash
/exit    # 先離開 openclaw 對話（如果在 tui 裡）
exit     # 再離開 sandbox shell，回到 host
```

---

## 對照速查表

| 我想做的事 | 該用哪一層 | 指令 |
|---|---|---|
| 建立新 sandbox | nemoclaw（host） | `nemoclaw onboard` |
| 看 sandbox 是否健康 | nemoclaw（host） | `nemoclaw <name> status` |
| 換推論模型 | nemoclaw（host） | `nemoclaw inference set ...` |
| 放行網路白名單（內建服務） | nemoclaw（host） | `nemoclaw <name> policy-add` |
| 放行網路白名單（自訂網域） | openshell（host，更底層） | `openshell policy set ...` |
| 從 sandbox 抓檔案回 host | nemoclaw（host） | `nemoclaw <name> download ...` |
| 跟 agent 對話 | openclaw（sandbox 內） | `openclaw tui` |
| 建立/查看排程任務 | openclaw（sandbox 內） | `openclaw cron ...` |
| 查對話 session 紀錄 | openclaw（sandbox 內） | `openclaw sessions list` |
| 查/整理 memory 檔案 | openclaw（sandbox 內） | `openclaw memory ...` |
| 診斷 agent 本身的問題 | openclaw（sandbox 內） | `openclaw doctor --fix` |
| 診斷 sandbox 容器本身的問題 | nemoclaw（host） | `nemoclaw <name> doctor --fix` |

> 注意最後兩行：`doctor` 這個指令**兩層都有**，但診斷的對象不同——host 端的 `nemoclaw <name> doctor` 看的是容器/gateway 層級；sandbox 內的 `openclaw doctor` 看的是 agent 本身的設定/plugin/channel。
