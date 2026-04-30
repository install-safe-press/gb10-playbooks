```md id="p9q7k1"
# DGX Dashboard 基本概念（入門版）

---

## 🧠 基本概念

DGX Dashboard 是一個運行在 **DGX Spark 設備上的 Web 應用程式**。

它提供一個圖形化介面（GUI），主要用途包括：

- 🔧 系統更新（System Update）
- 📊 GPU / 記憶體資源監控
- 🚀 整合 JupyterLab 開發環境

---

### 🌐 存取方式

使用者可以透過以下方式進入 DGX Dashboard：

#### 📍 本地存取
- 從桌面應用程式啟動器直接開啟

#### 🌍 遠端存取
- NVIDIA Sync
- SSH Tunnel（SSH 隧道）

---

### 🔧 遠端管理重點

在遠端工作情境下：

👉 DGX Dashboard 是最方便的方式  
用來進行：

- 系統更新
- 軟體套件管理
- 韌體更新

---

## 🎯 你將學到什麼？

完成本教學後，你將能夠：

### 💻 JupyterLab 操作
- 啟動預先配置好的 Python 環境
- 使用 Jupyter Notebook 開發 AI 程式

### 📊 系統監控
- 監控 GPU 使用率
- 查看記憶體與資源狀態

### 🔧 系統管理
- 執行系統更新
- 管理 DGX Spark 軟體環境

### 🤖 AI 工作負載
- 執行 Stable Diffusion（SDXL）範例
- 跑本地 AI 圖像生成任務

---

## 🌐 多種存取方式

DGX Dashboard 支援多種進入方式：

- 🖥️ 桌面捷徑（最簡單）
- ☁️ NVIDIA Sync（遠端同步）
- 🔐 SSH 隧道（進階使用者）

---

## 📚 開始之前須知

在開始操作前，建議具備以下基礎能力：

### 💡 SSH 與網路概念
- SSH 連線基本操作
- Port Forwarding（連接埠轉發）

### 🐍 Python 基礎
- Python 執行環境概念
- Jupyter Notebook 基本操作（Windows 使用者尤重要）

---

## 🧰 先決條件

### 🖥️ 硬體需求

- NVIDIA Grace Blackwell GB10 Superchip 系統

---

### 💿 軟體需求

- NVIDIA DGX 作業系統
- NVIDIA Sync（用於遠端存取）
  或
- SSH Client（Windows / Linux / macOS）

---

## 📎 輔助文件

### 🧪 SDXL 程式範例

Stable Diffusion XL（SDXL）相關 Python 程式碼：

👉 可在 GitHub 上取得

（用於執行文字生成圖片 AI 範例）
```
