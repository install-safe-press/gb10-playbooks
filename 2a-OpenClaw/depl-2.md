# 在 GB10 上部署 OpenClaw

> **前提：** 已依照 `1a-Open WebUI with Ollama` 章節，使用 Docker 完成 Ollama 佈建。  
> **本章目標：** 在 GB10 的主機作業系統（非容器內）直接安裝 OpenClaw，並讓它存取 Ollama 的模型。

---

## 準備作業：確認 Ollama 正確使用 GPU

在安裝 OpenClaw 之前，先確認 Ollama 容器有正確使用 GPU，否則 AI 推理速度會很慢。

**執行以下指令查看 Ollama 目前使用的處理器：**

```bash
docker exec ollama ollama ps
```

**結果判斷：**

```
# ❌ 沒有用到 GPU（效能差）
NAME           ID              SIZE     PROCESSOR    CONTEXT
qwen3.5:35b    3460ffeede54    32 GB    100% CPU     262144

# ✅ 有用到 GPU（效能正常）
NAME           ID              SIZE     PROCESSOR    CONTEXT
qwen3.5:35b    3460ffeede54    34 GB    100% GPU     262144
```

**如果顯示 `100% CPU`，表示 GPU 設定未生效，請執行以下步驟修復：**

確認你的 `docker-compose.yml` 內有包含 GPU 設定：

```yaml
deploy:
  resources:
    reservations:
      devices:
        - driver: nvidia
          count: all
          capabilities: [gpu]
```

然後重新建立容器：

```bash
cd ~/openwebui
docker compose down
docker compose up -d
```

等待約 30 秒後，再次執行 `docker exec ollama ollama ps` 確認已切換為 `100% GPU` ✅

---

## 安裝 OpenClaw

確認 Ollama GPU 正常後，回到 Home 目錄，執行以下一鍵安裝指令：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

> 以下以 **v2026.5.7** 版本為示範。

安裝過程會自動偵測環境、安裝套件並啟動 Gateway 服務，完整輸出如下：

```
🦞 OpenClaw Installer
✓ Detected: linux

[1/3] Preparing environment
✓ Node.js v22.22.2 found

[2/3] Installing OpenClaw
✓ OpenClaw npm package installed
✓ OpenClaw installed

[3/3] Finalizing setup
✓ Gateway service metadata refreshed
🦞 OpenClaw installed successfully (2026.5.7)!
```

> ⚠️ 安裝過程中若出現 `Gateway restart failed`，這是正常的初次啟動警告，安裝程式會繼續完成設定，不影響後續操作。

---

## 初次設定精靈

安裝完成後，會自動進入互動式設定精靈。以下依序說明各設定步驟：

---

### 1. 安全聲明

> OpenClaw 目前仍是 beta 版本，預設為個人使用模式。  
> 它可以讀取檔案並執行操作，**請勿在不了解安全風險的情況下開放給多人或公開網路使用。**

選擇 **Yes** 繼續。

---

### 2. 設定模式

選擇 **QuickStart**（快速設定，細節之後可以用 `openclaw configure` 調整）。

---

### 3. 選擇 AI 模型提供者

在提供者清單中選擇 **Ollama（Cloud and local open models）**。

---

### 4. Ollama 模式

選擇 **Local only（僅使用本地模型）**。

---

### 5. Ollama 連線網址

保持預設值：

```
http://127.0.0.1:11434
```

---

### 6. 選擇預設模型

點選 **Browse all models**，從清單中選擇你想使用的模型。  
本示範選擇：

```
ollama/qwen3.5:35b
```

---

### 7. 選擇通訊頻道

此步驟讓你設定 OpenClaw 透過哪個通訊軟體接收指令（Telegram、Slack、Discord 等）。  
初次設定可選 **Skip for now**，之後再用 `openclaw channels add` 新增。

---

### 8. 網路搜尋提供者

選擇 **Ollama Web Search（本地，免 API Key）**，或選 **Skip for now** 跳過。

---

### 9. 設定 Skills（技能外掛）

選擇 **Yes** 進入技能設定，然後選 **Skip for now** 略過個別依賴安裝（可之後再裝）。

---

### 10. 啟用 Hooks（自動化觸發）

建議啟用以下三個預設 Hooks：

- 📝 `command-logger`：記錄所有指令事件到稽核日誌
- 🧹 `compaction-notifier`：Session 壓縮時發送通知
- 💾 `session-memory`：下 `/new` 或 `/reset` 時自動儲存 Session 記憶

---

### 11. 啟動方式

選擇 **Hatch in Terminal（推薦）**，在終端機中直接開啟 OpenClaw TUI 介面。

---

## 啟動成功確認

設定完成後，終端機會顯示類似以下畫面：

```
🦞 OpenClaw 2026.5.7 — Greetings, Professor Falken

Control UI:
Web UI: http://127.0.0.1:18789/
Web UI (with token): http://127.0.0.1:18789/#token=xxxxxxxx...
Gateway: reachable
```

---

## 開啟 OpenClaw 網頁控制台

**在 GB10 上直接開啟瀏覽器：**

```
http://127.0.0.1:18789/#token=<你的token>
```

**透過 SSH 隧道，在 Windows 瀏覽器開啟：**

```
http://192.168.101.159:18789/#token=<你的token>
```

> 將 IP 替換為你的 GB10 實際 IP 位址。Token 在安裝完成時會顯示在終端機中，請複製保存。

---

## 與 AI 對話測試

在 TUI 終端機介面中，你可以直接與 OpenClaw 對話：

```
hi who are you
```

OpenClaw 會詢問你想給它取什麼名字、什麼個性，完成後就能正常使用。

**建議同時在另一個終端機視窗監控 GPU 使用狀況：**

```bash
watch -n 1 nvidia-smi
```

---

## 加入 Telegram Bot（選用）

讓你可以直接透過手機 Telegram 對 OpenClaw 下指令。

### Step 1：建立 Telegram Bot

1. 在 Telegram 中搜尋並開啟 **@BotFather**
2. 輸入 `/newbot`
3. 依提示為 Bot 取名（名稱結尾必須是 `bot`，例如 `MyAssistantBot`）
4. 完成後 BotFather 會給你一串 **HTTP API Token**，請妥善保存

### Step 2：在 OpenClaw 中建立連結

在 TUI 介面中對話，請求建立 Telegram 連結並輸入你的 HTTP API Token。

### Step 3：與 Bot 開始對話

打開你建立的 Telegram Bot，開始傳送訊息。

> 第一次傳訊息時 Bot 可能不會立即回應——這是因為還需要完成配對。Bot 會回覆你一組**配對碼**。

### Step 4：在 GB10 完成配對

回到 GB10 終端機，輸入以下指令（將 `00000000` 替換為 Bot 回傳的配對碼）：

```bash
openclaw pairing approve telegram 00000000
```

配對成功後，Telegram Bot 就可以正常對話了 ✅

---

## 常用指令速查

| 指令 | 說明 |
|------|------|
| `openclaw tui` | 開啟終端機互動介面 |
| `openclaw logs --follow` | 即時查看 OpenClaw 日誌 |
| `openclaw config` | 查看目前設定 |
| `openclaw doctor --fix` | 自動診斷並修復問題 |
| `openclaw configure --section web` | 設定網路搜尋 API Key |
| `openclaw channels add` | 新增通訊頻道 |
| `openclaw pairing approve <channel> <code>` | 核准配對請求 |

> 💡 **小提示：** 不需要自己手動改設定檔，直接在 TUI 中用自然語言告訴 OpenClaw 你想改什麼就好。  
> 設定檔位置：`~/.openclaw/openclaw.json`

---

## 停止與完整移除 OpenClaw

### 停止 OpenClaw

```bash
pkill -f openclaw || true
pkill -f "npm run dev" || true
pkill -f node || true
```

### 完整刪除所有檔案

```bash
# 刪除主程式目錄
rm -rf ~/openclaw

# 刪除使用者設定與資料
rm -rf ~/.openclaw

# 刪除 Python 虛擬環境（如有）
rm -rf ~/openclaw-env

# 清除 npm 快取
npm cache clean --force

# 搜尋並清除殘留資料
find ~ -type d -name "*openclaw*" -exec rm -rf {} + 2>/dev/null
```

> ⚠️ 執行刪除後，所有對話記憶、設定與已安裝的技能都會被清除，且無法還原。

---

## 相關資源

| 資源 | 說明 |
|------|------|
| [OpenClaw 官網](https://openclaw.ai/) | 官方首頁 |
| [OpenClaw 文件](https://docs.openclaw.ai/) | 完整使用說明 |
| [ClawhHub](https://clawhub.ai/) | 社群技能外掛庫 |
| [安全設定指南](https://docs.openclaw.ai/security) | 強化安全的建議 |
| [Hooks 說明](https://docs.openclaw.ai/automation/hooks) | 自動化觸發設定 |
