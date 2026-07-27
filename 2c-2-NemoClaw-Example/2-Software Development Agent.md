# Software Development Agent Playbook 完整說明

參考官方 Playbook：[Software Development Agent](https://build.nvidia.com/spark/nemoclaw-applications/developer-agent)

---

## 一、這個 Playbook 是做什麼事

**一句話**：讓已經裝好的 NemoClaw sandbox，從「只能聊天」升級成「能真正讀取、修改、測試你的程式碼，並交出一份完整報告」的自主開發助理——但透過**副本隔離 + 網路封鎖 + approval gate** 三層機制，確保這個能力擴張不會變成風險。

**核心價值主張：**

- Agent 有真正的檔案讀寫、指令執行能力（不是純聊天建議）
- 但被限制在一個**複製出來的副本目錄**，不會碰到你真正的工作目錄
- 除了跟本地模型對話，**完全連不上外網**（`inference.local` 是唯一出口）
- 寫程式碼之前，會先停下來**印出計畫、等你人工核准**

---

## 二、要事先準備什麼

| 項目 | 說明 |
|---|---|
| 已完成的 NemoClaw sandbox | 前一篇安裝文章的成果，`nemoclaw list` 能看到 |
| 一個測試專案的副本 | **絕對不要**用你唯一的正式工作目錄，官方原話：「Point it at a project copy or a clean clone, not your only working tree.」 |
| 確認本地模型可用 | `ollama list` 確認模型存在，使用 context 夠大的（我們今天用 `nemotron-3-super:120b-64k`，262k context） |
| `my-app.tar.gz` | 專案打包檔，操作步驟見 [`2-upload-my-app-to-sandbox.md`](./2-upload-my-app-to-sandbox.md)，負責把檔案放到 sandbox |
| Prompt 檔案（選其一） |    英文：`Software Development Agent-dev-agent-prompt.txt`／     中文：`Software Development Agent-dev-agent-prompt-zh.txt` |
---
my-app.tar.gz 是「自製最小 Flask 範例，用於測試官方建議的第一個 feature request」官方文件本身沒有提供範例專案原始碼，my-app 這個測試專案是為了示範第一個建議 feature request（/healthz）而額外準備的最小 Flask 範例，並非 NVIDIA 官方素材。


## 三、要如何操作（照官方 Step 1 → Step 2 → Step 3）

### Step 1：把專案送進 sandbox

```bash
mkdir -p ~/nemoclaw-projects
cp -r ~/projects/my-app ~/nemoclaw-projects/my-app
```

```bash
tar czf - -C ~/nemoclaw-projects/my-app . \
  | nemoclaw gb10-assistant exec -- bash -lc 'mkdir -p /sandbox/project && tar xzf - -C /sandbox/project'
```

> ⚠️ **今天發現的修正**：官方這行 `tar` 管道指令，在我們的環境實測會出現 `gzip: stdin: unexpected end of file`。如果遇到同樣狀況，改用：

```bash
nemoclaw gb10-assistant upload my-app.tar.gz /sandbox/my-app.tar.gz
nemoclaw gb10-assistant exec -- bash -lc 'mkdir -p /sandbox/project && tar xzf /sandbox/my-app.tar.gz -C /sandbox/project --strip-components=1 && rm /sandbox/my-app.tar.gz'
```




#### 驗證三件事（官方原文）

```bash
nemoclaw gb10-assistant exec -- ls /sandbox/project
nemoclaw gb10-assistant exec -- bash -lc 'curl -sS --max-time 5 https://example.com'
nemoclaw gb10-assistant exec -- bash -lc 'curl -sf https://inference.local/v1/models'
```

预期：專案樹看得到、`example.com` 被 `403` 擋下、`inference.local` 回傳模型清單。
進入sanbox  : nemoclaw gb10-assistant connect
啟動openclaw tui : openclaw tui
驗證模型與回應正常



  進入 
---

### Step 2：貼上官方完整 prompt

> ⚠️ **今天發現的關鍵修正——這是官方文件沒寫、但實測必要的一段**：
>
> 這個 sandbox 的工具介面只有三個頂層工具：`tool_call`、`tool_describe`、`tool_search`，**沒有**可以直接呼叫的 `exec`/`read`。所有動作都要包在 `tool_call` 裡用完整 ID 呼叫（例如 `openclaw:core:exec`）。建議在官方 prompt 最前面加這段：

```
IMPORTANT: This sandbox's tool interface only exposes tool_call, tool_describe,
and tool_search as top-level tools. There is no direct "exec" or "read" tool.
To run a shell command or read/write a file, use tool_call with the correct
tool id (e.g. "openclaw:core:exec"). Never call "exec" or "read" directly.
```

把這段加在官方 prompt 最開頭，再貼完整內容，**一次貼完，不要分段**。

---

### Step 3：回答六個 setup 問題，逐題確認，尤其 Q5

- **Q5（approval gate）務必明確回答「需要」**，並加強語氣強調「絕不跳過」

#### 送出 feature request

```
Add a /healthz endpoint that returns {status: 'ok', commit: <git sha>} with a test.
```

#### 看到 PLAN READY 後，人工核准

```
approve
```

#### 等它走完 IMPLEMENT → SELF-REVIEW → REPORT → HANDOFF

#### 最後查證（不要只信任文字輸出）

```bash
nemoclaw gb10-assistant exec -- bash -lc 'cd /sandbox/project && git --no-pager diff'
nemoclaw gb10-assistant exec -- bash -lc 'cd /sandbox/project && python3 -m pytest -v'
nemoclaw gb10-assistant exec -- cat /sandbox/project/develop-and-review.md
```

#### 拉回 host（官方 Step 1 收尾動作）

```bash
nemoclaw gb10-assistant exec -- bash -lc 'cd /sandbox/project && tar czf - .' | tar xzf - -C ~/nemoclaw-projects/my-app
```

---

## 四、中間過程有什麼要調整（今天實測發現的三個關鍵落差）

| 官方步驟 | 實測發現的問題 | 調整方式 |
|---|---|---|
| Step 1 的 tar 管道指令 | `gzip: stdin: unexpected end of file` | 改用 `nemoclaw upload` + `--strip-components=1` |
| Step 2 的 prompt（直接貼上） | Agent 呼叫不存在的 `exec`/`read` 工具，導致 `Tool exec not found` 反覆失敗 | 在 prompt 前面加上工具介面說明（見上方） |
| PLAN 核准機制（Q5） | 如果 prompt 跟 feature request 一次貼上，agent 會自己跳過問答、自行判定 approval gate = no | 分兩則訊息送，先問完六題再送 feature request |

---

## 五、最後結果是什麼

- `app.py` 新增 `/healthz` 路由，回傳 `{"status": "ok", "commit": "<sha>"}`
- `tests/test_app.py` 新增對應測試，`pytest` 顯示 `2 passed`
- `/sandbox/project/develop-and-review.md` 產出完整報告，包含 SCAN 摘要、PLAN、實作明細、**自我發現並修正的一個 out-of-plan 錯誤**、測試結果
- 全程 agent 自主完成，人類只需在 approval gate 按一次 approve

---

## 附註：開始前先確認 `my-app` 是乾淨狀態

```bash
nemoclaw gb10-assistant exec -- bash -lc 'cd /sandbox/project && git reset --hard 2965131'
nemoclaw gb10-assistant exec -- bash -lc 'cd /sandbox/project && rm -f develop-and-review.md PROJECT_CONFIG.md'
nemoclaw gb10-assistant exec -- bash -lc 'cd /sandbox/project && git --no-pager status'
```

確認乾淨（只有 `/` 路由，沒有 `/healthz`，沒有多餘檔案）後，即可開始正式錄製「完全參照官方 + 套用今天修正」的完整流程。
