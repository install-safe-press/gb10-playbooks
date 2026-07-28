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
| `my-app.tar.gz` | 專案打包檔，把檔案放到 sandbox |
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



## Software Development Agent — 完整成功案例報告

> 測試日期：2026-07-28
> 環境：NVIDIA GB10 (DGX Spark) + NemoClaw sandbox `gb10-assistant`
> 模型：`nemotron-3-super:120b-64k`（Ollama 本地推論，context 65536）
> 參考 Playbook：[Software Development Agent](https://build.nvidia.com/spark/nemoclaw-applications/developer-agent)

---

## 一、結論先講：這次是完全成功的一次自主執行

經過前兩天的多次失敗排查（工具呼叫格式錯誤、approval gate 被跳過、文字轉義導致的編輯失敗、模型在系統錯誤後產生幻覺報告），這次套用累積下來的三項修正後，**agent 完整、自主、無需人工介入地走完了 SCAN → PLAN → approve → IMPLEMENT → SELF-REVIEW → REPORT → HANDOFF 整套流程**，且每一步都經過檔案系統實際查證，確認不是幻覺。

---

## 二、這次成功的三個關鍵修正

在 prompt 最前面加入以下三項，是這次成功的核心原因：

### 修正一：明確告知工具介面的正確使用方式

```
IMPORTANT: This sandbox's tool interface only exposes tool_call, tool_describe,
and tool_search as top-level tools. There is no direct "exec" or "read" tool.
To run a shell command or read/write a file, use tool_call with the correct
tool id (e.g. "openclaw:core:exec"). Never call "exec" or "read" directly.
```

### 修正二：修改既有檔案時，要求「整份覆寫」而非「精確編輯」

```
When you need to modify an existing file, prefer overwriting the entire file
content (create mode) rather than precise text-replacement edits, since the
text-replacement tool has been unreliable with multi-line content in this
environment.
```

### 修正三：approval gate 用堅定、明確、不留模糊空間的語氣要求

```
Yes. This is mandatory — after producing the PLAN, you must print "PLAN READY"
and stop, waiting for me to reply "approve". Do not skip this step under any
circumstance, even for small or seemingly unambiguous changes. This applies to
every feature request, not just the first one.
```

在核准 PLAN 時，額外重申了 fixture 名稱與整份覆寫策略：

```
approve

Reminders:
1. The existing pytest fixture in tests/test_app.py is named client
   (yields app.test_client()). Reuse it — do not define a new fixture.
2. When modifying tests/test_app.py, read the full existing content first,
   then compose the complete new file content (old content + new test
   function), and write it using full-file overwrite (create mode) —
   do not use step-by-step text-replacement/edit tools.
3. Use the short form of the commit SHA (git rev-parse --short HEAD),
   falling back to "unknown" on failure.
```

---

## 三、六題 Setup 問答記錄

| # | 問題 | 回答 |
|---|---|---|
| 1 | Project purpose | A minimal Flask demo app used to test the NemoClaw Software Development Agent workflow. |
| 2 | Source tree include/exclude | Include: `app.py`, `tests/`　Exclude: `.git/`, `__pycache__/`, `.pytest_cache/` |
| 3 | Code style reference | Match what's already there — infer from `app.py` and `tests/test_app.py`. |
| 4 | Test policy | Write tests for every change. |
| 5 | Approval gate | Mandatory — 每次 feature request 都要暫停等 approve，沒有例外。 |
| 6 | Report path | 預設值：`/sandbox/project/develop-and-review.md` |

Agent 正確將設定寫入 `/sandbox/project/.devprofile.yaml`，內容經查證完全正確：

```yaml
# NemoClaw Dev Agent — Project Profile
# Auto-saved Tue 2026-07-28 00:42 UTC
directory: /sandbox/project
project_purpose: >-
  A minimal Flask demo app used to test the NemoClaw Software Development Agent workflow.
include_dirs:
  - app.py
  - tests/
exclude_dirs:
  - .git/
  - __pycache__/
  - .pytest_cache/
style_guidance: Infer from existing code (app.py and tests/test_app.py)
test_policy: Write tests for every change
approval_after_plan: true
  # Mandatory — prints "PLAN READY" and waits for "approve" before any code changes.
report_path: /sandbox/project/develop-and-review.md
```

---

## 四、Feature Request

```
Add a /healthz endpoint that returns {status: 'ok', commit: <git sha>} with a test.
```

---

## 五、PLAN（核准前完整內容）

**Goal**：Serve `GET /healthz` → `{"status": "ok", "commit": "<8-char git SHA>"}`，並提供完整測試覆蓋。

**Affected Files**

| File | Action | Why |
|---|---|---|
| `app.py` | Modify | 新增 `/healthz` 路由 + 在啟動時讀取 git SHA 的 helper |
| `tests/test_app.py` | Modify | 新增測試，驗證狀態碼、JSON 結構與欄位值 |

**Step Order**

1. 在 `app.py` 建立 `_get_commit_sha()`，於 import 時執行 `git rev-parse --short HEAD`（失敗時 fallback 為 `"unknown"`），結果快取於模組層級變數 `COMMIT_SHA`
2. 新增 `@app.route("/healthz")`，回傳 `jsonify({"status": "ok", "commit": COMMIT_SHA})`
3. 在 `tests/test_app.py` 新增 `test_healthz_endpoint()`，驗證 HTTP 200、正確 content-type、回應欄位合理性

**Risks & Mitigations**

- Git 不存在或不在 repo 內 → 優雅 fallback 為 `"unknown"`，端點仍可運作
- 模組層級的 side effect（import 時觸發 subprocess）→ 可接受，SHA 不會隨請求改變，快取一次是合理設計

---

## 六、實際驗證結果（檔案系統查證，非文字自述）

### `git diff` 完整內容

```diff
diff --git a/app.py b/app.py
index 4e85161..44ac39b 100644
--- a/app.py
+++ b/app.py
@@ -9,16 +9,42 @@ The classic first feature request to try with the agent is:
      {status: 'ok', commit: <git sha>} with a test."
 """
+import subprocess
+
 from flask import Flask, jsonify
 app = Flask(__name__)
+def _get_commit_sha() -> str:
+    """Return the short git revision SHA at import time, or 'unknown'."""
+    try:
+        sha = subprocess.run(
+            ["git", "rev-parse", "--short", "HEAD"],
+            capture_output=True,
+            text=True,
+            check=True,
+            timeout=5,
+        )
+        return sha.stdout.strip()
+    except Exception:
+        return "unknown"
+
+
+COMMIT_SHA = _get_commit_sha()
+
+
 @app.route("/")
 def index():
     """Basic landing route so the app has something to serve."""
     return jsonify({"message": "Hello from my-app!"})
+@app.route("/healthz")
+def healthz():
+    """Infrastructure probe: returns status and the build commit SHA."""
+    return jsonify({"status": "ok", "commit": COMMIT_SHA})
+
+
 if __name__ == "__main__":
     app.run(host="0.0.0.0", port=5000)

diff --git a/tests/test_app.py b/tests/test_app.py
index 20890ab..3eb7a7e 100644
--- a/tests/test_app.py
+++ b/tests/test_app.py
@@ -5,6 +5,8 @@ Only the "/" route is covered right now. When the agent adds
 test module) as part of its SELF-REVIEW / test-policy step.
 """
+import re
+
 import pytest
 from app import app
@@ -21,3 +23,25 @@ def test_index_returns_hello_message(client):
     response = client.get("/")
     assert response.status_code == 200
     assert response.get_json() == {"message": "Hello from my-app!"}
+
+
+def test_healthz_endpoint(client):
+    response = client.get("/healthz")
+    data = response.get_json()
+
+    # HTTP status
+    assert response.status_code == 200
+
+    # Content type is JSON
+    assert "application/json" in response.content_type.lower()
+
+    # Exactly two keys, no extras
+    assert set(data.keys()) == {"status", "commit"}
+
+    # Status must be 'ok'
+    assert data["status"] == "ok"
+
+    # Commit is a non-empty hex string of length ≤ 40 (or 'unknown' fallback)
+    if data["commit"] != "unknown":
+        assert re.fullmatch(r"[0-9a-f]+", data["commit"])
+        assert len(data["commit"]) <= 40
```

### `pytest -v` 實際輸出

```
============================= test session starts ==============================
platform linux -- Python 3.13.5, pytest-8.2.0, pluggy-1.6.0 -- /usr/bin/python3
cachedir: .pytest_cache
rootdir: /sandbox/project
collected 2 items
tests/test_app.py::test_index_returns_hello_message PASSED               [ 50%]
tests/test_app.py::test_healthz_endpoint PASSED                          [100%]
============================== 2 passed in 0.06s ===============================
```

---

## 七、Agent 產出的 Self-Review（原文摘錄）

| Severity | 位置 | 問題 | 判斷 |
|---|---|---|---|
| Medium | `app.py`, `_get_commit_sha()` | `except Exception` 範圍過廣，會連 `KeyboardInterrupt`/`SystemExit` 都吃掉 | 標記為 informational，不需修正——這段只在 import 時執行一次，寬鬆的例外處理反而能避免 git 不存在時整個 app 掛掉 |
| Low | `app.py`, subprocess 呼叫 | 是否有注入風險 | 無風險，指令是靜態的 list，不是字串拼接 |
| Low | `test_app.py`, regex 檢查 | regex 只驗證是 hex 字串，沒有限制一定要 8 碼 | 可接受，端點本身對任何長度的 SHA 都能正常運作 |
| Low | Scope creep | 有沒有動到計畫外的檔案 | 沒有，只改了 `app.py` 和 `tests/test_app.py` |

---

## 八、HANDOFF

```
📝 Report at /sandbox/project/develop-and-review.md
Feature /healthz implemented across 2 files (app.py, tests/test_app.py);
0 findings in self-review (1 informational note, no action needed);
tests pass (2/2 ✅)
```

---

## 九、與前兩天失敗案例的對照

| 項目 | 之前失敗的嘗試 | 這次成功的關鍵差異 |
|---|---|---|
| 工具呼叫方式 | 反覆呼叫不存在的 `exec`/`read`，或參數位置放錯 | Prompt 開頭就明確說明 `tool_call` 三層架構 |
| 編輯既有測試檔案 | 連續近 20 次因文字轉義失敗 | 明確要求「讀取完整內容 → 組合 → 整份覆寫」，這次一次成功 |
| Approval gate | Agent 有時自行判定為不需暫停，或用預設值帶過 | 用堅定、重複強調「no exceptions, ever」的語氣，並在 approve 時再次提醒 |
| 面對系統錯誤（`LLM request failed`） | 傾向生成一整套看似完整、實則虛構的報告 | 這次全程沒有觸發任何 `LLM request failed`，因此沒有機會觸發幻覺行為 |

---

## 十、結論

這次完整成功證實：**Software Development Agent 這個 Playbook 官方描述的自動化流程，在本地模型（`nemotron-3-super:120b-64k`）上是可以完全自主、可靠達成的**——前提是使用者必須先了解並補足官方文件沒有寫出的三個實務細節（工具介面說明、整份覆寫策略、堅定的 approval gate 措辭）。這些細節屬於「環境特定的隱性知識」，是透過大量實測才能發掘出來的，值得整理進正式文件，作為其他人依循官方 Playbook 操作時的重要補充。


# Software Development Agent 專案中使用到的 NemoClaw 指令（Sandbox 外層操作）

本文整理 Software Development Agent 測試過程中，所有在 **Host（Sandbox
外層）** 使用過的 NemoClaw 指令，以及各指令的用途。

## 檔案傳輸

### 上傳檔案至 Sandbox

``` bash
nemoclaw gb10-assistant upload /home/user/my-app.tar.gz /sandbox/my-app.tar.gz
```

用途： - 將本機檔案上傳至 Sandbox。 - 本次用於傳送已打包的專案壓縮檔。 -
官方建議的 `tar` 串流方式測試失敗，因此改用 `upload`。

------------------------------------------------------------------------

## 非互動執行（Exec）

``` bash
nemoclaw gb10-assistant exec -- <command>
```

常見用途：

### 讀取檔案

``` bash
nemoclaw gb10-assistant exec -- cat app.py
```

### 查看 Git 狀態

``` bash
nemoclaw gb10-assistant exec -- git --no-pager status
nemoclaw gb10-assistant exec -- git --no-pager diff
nemoclaw gb10-assistant exec -- git --no-pager log
```

### 重置 Git

``` bash
nemoclaw gb10-assistant exec -- git reset --hard
```

### 執行測試

``` bash
nemoclaw gb10-assistant exec -- python3 -m pytest -v
```

### 建立目錄

``` bash
nemoclaw gb10-assistant exec -- mkdir -p /sandbox/project
```

### 解壓縮

``` bash
nemoclaw gb10-assistant exec -- tar xzf my-app.tar.gz --strip-components=1
```

### 清除暫存

``` bash
nemoclaw gb10-assistant exec -- rm -rf __pycache__
nemoclaw gb10-assistant exec -- rm -rf .pytest_cache
```

------------------------------------------------------------------------

## 互動連線

``` bash
nemoclaw gb10-assistant connect
```

------------------------------------------------------------------------

## 日誌查看

``` bash
nemoclaw gb10-assistant logs --follow
```

------------------------------------------------------------------------

## 狀態檢查

``` bash
nemoclaw gb10-assistant status
```

用途： - 檢查 Sandbox 與 Gateway 狀態 - 排查 Provisioning 卡住 - 排查
inference unhealthy

------------------------------------------------------------------------

## Gateway 管理

``` bash
nemoclaw gb10-assistant gateway restart
```

------------------------------------------------------------------------

## 推論設定

``` bash
nemoclaw inference get
```

``` bash
nemoclaw inference set --model nemotron-3-super:120b-64k --provider ollama-local --sandbox gb10-assistant
```

> 本次測試發現，直接修改設定檔未生效，最終透過 `openclaw onboard`
> 才完成模型設定。

------------------------------------------------------------------------

# 本次未使用但常見指令

  |指令|                                                     原因|
  |-|-|
  |`nemoclaw gb10-assistant policy-add`                     |未調整 Policy|
  |`nemoclaw gb10-assistant download`                       |尚未回傳成果至 Host|
  |`nemoclaw gb10-assistant rebuild`                        |未重建 Sandbox|
  |`nemoclaw gb10-assistant snapshot create/list/restore`   |未使用快照|

------------------------------------------------------------------------

nemoclaw 操作層級示意圖

```
Host
│
├── upload
├── download
├── status
├── logs
├── gateway restart
├── inference set
├── connect
└── exec
        │
        ▼
+------------------+
|     Sandbox      |
|                  |
| git              |
| pytest           |
| python           |
| tar              |
| mkdir            |
| rm               |
| cat              |
+------------------+
```
