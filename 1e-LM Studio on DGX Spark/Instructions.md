# 安裝並使用 LM Studio（llmster）操作步驟

---

## 整體流程概覽

| 步驟 | 說明 | 執行位置 |
|------|------|----------|
| Step 1 | 安裝 llmster | GB10 終端機 |
| Step 2 | 下載輔助腳本 | 筆電本機終端機 |
| Step 3 | 啟動 LM Studio API 伺服器 | GB10 終端機 |
| Step 4 | （選用）設定 LM Link 加密連線 | GB10 + 筆電 |
| Step 5 | 下載 AI 模型 | GB10 終端機 |
| Step 6 | 載入模型 | GB10 終端機 |
| Step 7 | 從筆電發送測試提示 | 筆電本機終端機 |
| Step 8 | 後續擴充 | — |
| Step 9 | 清理與移除 | GB10 終端機 |

---

## Step 1：在 GB10 上安裝 llmster

**llmster** 是 LM Studio 的**終端機版本（無圖形介面）**，適合安裝在伺服器或沒有螢幕的機器上。安裝完成後，你就可以透過 API 從筆電連線到 GB10 上的 LLM。

**在 GB10 終端機執行：**

```bash
curl -fsSL https://lmstudio.ai/install.sh | bash
```

安裝完成後，請依照終端機輸出的說明，將 `lms` 加入 **PATH 環境變數**，這樣你才能在任何目錄下直接執行 `lms` 指令。

> 💡 **`lms` 是什麼？**  
> 這是 LM Studio 的命令列工具（CLI），後續所有操作（下載模型、啟動伺服器、載入模型）都會用到它。

---

## Step 2：下載輔助腳本

在你的**終端機**執行以下指令，下載後續測試用的腳本。依你偏好的語言選擇其中一個（或全部下載）：

```bash
# Bash
curl -L -O https://raw.githubusercontent.com/lmstudio-ai/docs/main/_assets/nvidia-spark-playbook/bash/run.sh
```

> 這些腳本將在 **Step 7** 用來向 GB10 發送測試提示，驗證整個連線流程是否正常。

---

## Step 3：啟動 LM Studio API 伺服器

在 **GB10 終端機**上，使用 `lms` 指令啟動 API 伺服器，並開放區域網路存取：

```bash
lms server start --bind 0.0.0.0 --port 1234
```

> `--bind 0.0.0.0` 表示允許同一區域網路內的所有裝置連線。請確認連線的裝置都是你信任的裝置。

**測試筆電是否能連上 GB10：**

在筆電的終端機執行（將 `<SPARK_IP>` 替換為 GB10 的實際 IP）：

```bash
curl http://<GB10_IP>:1234/api/v1/models
```

## Step 4（選用）：設定 LM Link 加密連線

> 如果你的筆電與 GB10 **不在同一個區域網路**，或不想手動記 IP 位址，可以使用 **LM Link** 透過端對端加密連線遠端存取模型。

**設定步驟：**

1. **建立連結：** 前往 https://lmstudio.ai/link，依照「建立你的連結」的步驟建立私人 LM Link 網路
2. **連接兩台裝置：** 分別在 GB10（llmster）和筆電上登入並加入同一個 Link
3. **使用遠端模型：** 在筆電上開啟 LM Studio，GB10 上的模型會自動出現在模型載入器中

連線成功後，所有工具只需指向 `localhost:1234` 即可使用 GB10 上的模型，**不需要修改任何端點設定**，相容工具包括 LM Studio SDK、Codex、Claude Code、OpenCode 以及 Step 7 的腳本。

> 📋 **目前限制（預覽版）：** 最多 2 位使用者免費使用，每位最多 5 台裝置（共 10 台）。

---

## Step 5：下載 AI 模型到 GB10

以 **NVIDIA Nemotron 3 Nano Omni** 為範例，在 GB10 終端機執行：

```bash
lms get nvidia/nemotron-3-nano-omni
```

> 📦 模型檔案較大26GB，下載需要一段時間，請耐心等待。

**下載完成後，確認模型已成功存在：**

```bash
lms ls
```

看到模型名稱出現在清單中即代表下載成功 ✅

---

## Step 6：載入模型

將模型載入記憶體，使其準備好回應來自筆電的請求：

```bash
lms load nvidia/nemotron-3-nano-omni
```

> 載入完成後，模型即處於待機狀態，可以開始接收推理請求。

---

## Step 7：從筆電發送測試提示

在筆電上安裝對應語言的 SDK，並執行 Step 2 下載的腳本。

> ⚠️ **執行前請先編輯腳本**，將其中的 `<SPARK_IP>` 替換為 GB10 的實際 IP 位址。

---

### JavaScript

**前置需求：** 已安裝 `npm` 與 `node`

```bash
npm install @lmstudio/sdk
node run.js
```

---

### Python

**前置需求：** 已安裝 `uv`

```bash
uv run --script run.py
```

---

### Bash

**前置需求：** 已安裝 `jq` 與 `curl`

```bash
bash run.sh
```

---

## Step 8：後續擴充

完成基本設定後，你可以繼續：

- 前往 [LM Studio 模型目錄](https://lmstudio.ai/models) 下載並試用不同的模型
- 使用 **LM Link** 連接更多裝置，從任何地方透過加密連線存取 GB10 上的模型

---

## Step 9：清理與移除

> **⚠️ 注意：** LM Studio 將模型與應用程式**分開儲存**。卸載 llmster 本身**不會**自動刪除已下載的模型，需手動清除。

| 要移除的項目 | 操作方式 |
|-------------|----------|
| llmster 應用程式 | 刪除資料夾 `~/.lmstudio/llmster` |
| 已下載的模型 | 刪除資料夾內容 `~/.lmstudio/models/` |
| 桌面版 LM Studio | 先從系統托盤退出，再將應用程式移至垃圾桶 |

---

## 相關資源

| 資源 | 說明 |
|------|------|
| [LM Studio 文件](https://lmstudio.ai/docs) | 官方安裝與使用說明 |
| [LM Link](https://lmstudio.ai/link) | 遠端使用本機模型 |
| [DGX Spark 文件](https://docs.nvidia.com/dgx-spark) | 硬體與系統參考 |
| [DGX Spark 論壇](https://forums.developer.nvidia.com) | 社群問答 |
| [LM Studio Discord](https://discord.gg/lmstudio) | 即時社群支援 |
