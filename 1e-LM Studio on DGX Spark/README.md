# LM Studio on DGX Spark(GB10)

> 原始資訊：https://build.nvidia.com/spark/lm-studio  
> 最後更新：2026 年 4 月 28 日

---

## 什麼是 LM Studio？

**LM Studio** 是一款讓你在自己電腦上執行大型語言模型（LLM）的應用程式，完全**私密、免費、不需要網路**。

支援的模型包括：`gpt-oss`、`Qwen3`、`Gemma3`、`DeepSeek` 等主流開源模型。

將 LM Studio 部署在 **DGX Spark（GB10）** 上，等於擁有一台屬於自己的高效能私有 LLM 伺服器，所有推理運算都在本機 GPU 上完成。

---

## 本章節目標

完成本章節後，你將能夠：

- 在 GB10 上安裝 **llmster**（LM Studio 的無介面終端機版本）
- 透過 API 在 GB10 本機執行 LLM 推理
- 使用 **LM Studio SDK** 從筆記型電腦與模型互動
- （選用）使用 **LM Link** 透過加密連線，讓遠端 GB10 上的模型看起來像在本機一樣

> 本指南以 **Nemotron 3 Nano Omni**（`nvidia/nemotron-3-nano-omni`）作為範例模型。

---

## 開始前需要具備的基礎知識

- 能夠設定 DGX Spark 的本地網路存取
- 熟悉終端機 / 命令列操作
- 了解 REST API 的基本概念

---

## 先決條件

### 硬體需求

| 項目 | 需求 |
|------|------|
| 處理器架構 | ARM64（GB10 Grace Blackwell） |
| GPU 顯示記憶體 | 最低 65GB，建議 70GB 以上 |
| 可用儲存空間 | 最低 65GB，建議 70GB 以上 |

### 軟體需求

- NVIDIA DGX 作業系統
- 用戶端裝置（Mac、Windows 或 Linux 筆電）
- **筆電與 GB10 必須連接到同一個區域網路**（若使用 LM Link 則不受此限制）
- 可連線至網路以下載套件與模型

---

## 支援的模型清單

完整模型清單請參考 [LM Studio 模型目錄](https://lmstudio.ai/models)。

| 模型 | 支援狀態 | 模型路徑 |
|------|----------|----------|
| Nemotron 3 Nano Omni | ✅ | `nvidia/nemotron-3-nano-omni` |
| Qwen3.6-35B-A3B | ✅ | `qwen/qwen3.6-35b-a3b` |
| GPT-OSS-120B | ✅ | `openai/gpt-oss-120b` |

---

## LM Link（選用功能）

**LM Link** 讓你可以從任何地方遠端使用 GB10 上的模型，就像模型跑在自己的電腦上一樣。

**核心特色：**

- **端對端加密：** 基於 Tailscale Mesh VPN，裝置不會暴露在公開網際網路上
- **相容本機工具：** 任何連接到 `localhost:1234` 的工具（如 Codex、Claude Code、LM Studio SDK）都能直接使用 LM Link 上的模型
- **免費預覽版：** 最多 2 位使用者、每人最多 5 台裝置（共 10 台）

> 🔗 建立你的連結：https://lmstudio.ai/link

**使用 LM Link 的好處：**  
不需要將伺服器綁定到 `0.0.0.0` 或記憶 GB10 的 IP 位址。裝置連接後，將筆電指向 `localhost:1234`，遠端模型就會自動出現在模型載入器中。

---

## 輔助腳本

以下範例腳本用於測試步驟 7（向 GB10 發送測試提示），可依需求選用：

| 腳本 | 說明 |
|------|------|
| `run.js` | JavaScript 版本測試腳本 |
| `run.py` | Python 版本測試腳本 |
| `run.sh` | Bash 版本測試腳本 |

---

## 預估時間與風險

| 項目 | 說明 |
|------|------|
| ⏱️ 預計時間 | 15～30 分鐘（含模型下載，視網路速度而定） |
| ⚠️ 風險等級 | 低 |

**復原方式（如需移除）：**
- 手動從模型目錄刪除已下載的模型
- 卸載 LM Studio 或 llmster

---

## 相關資源

| 資源 | 說明 |
|------|------|
| [LM Studio 文件](https://lmstudio.ai/docs) | 官方安裝與使用說明 |
| [LM Link](https://lmstudio.ai/link) | 遠端使用本機模型 |
| [DGX Spark 文件](https://docs.nvidia.com/dgx-spark) | 硬體與系統參考 |
| [DGX Spark 論壇](https://forums.developer.nvidia.com) | 社群問答 |
| [LM Studio Discord](https://discord.gg/lmstudio) | 即時社群支援 |

---

## 附錄：LM Studio vs Ollama 比較

兩者都是本機執行 LLM 的熱門工具，但定位不同，選擇前可先參考以下比較：

| 項目 | LM Studio | Ollama |
|------|-----------|--------|
| 使用方式 | GUI 圖形介面為主 | CLI / API 為主 |
| 適合族群 | 初學者、一般使用者 | 開發者、企業部署 |
| 安裝難度 | 非常簡單 | 中等 |
| 模型下載 | 內建模型商店 | 指令下載 |
| 本地聊天 | 內建完整 UI | 需額外搭配前端 |
| API 支援 | OpenAI 相容 | 強大 REST API |
| Docker 部署 | 較弱 | 強 |
| 自動化整合 | 中等 | 非常強 |
| 模型管理 | 視覺化介面 | Modelfile 設定 |
| 資源監控 | 直觀 | 較少 |

### LM Studio 的優缺點

**優點：**
- 安裝最快、上手門檻最低
- 介面直觀，類似 ChatGPT 的使用體驗
- 內建模型搜尋，方便測試不同模型
- 可快速建立本地 API
- Windows / Mac 新手友善

**缺點：**
- 自動化整合能力較弱
- Docker 支援不足，不適合大規模部署
- 部分情況下 GGUF 模型效能略低於 Ollama

---

> 💡 **選擇建議：**  
> 如果你只是想快速試用 LLM、不想寫指令，選 **LM Studio**。  
> 如果你需要自動化整合、API 開發或大規模部署，選 **Ollama**。
