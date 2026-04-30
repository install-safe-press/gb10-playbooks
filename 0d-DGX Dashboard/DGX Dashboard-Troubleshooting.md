# 如何在自己的 PC/NB 連上並顯示 GB10 的 DGX Dashboard / JupyterLab Web
> 原文參考：　　https://build.nvidia.com/spark/dgx-dashboard/troubleshooting
---
## 背景說明

GB10 上的服務只綁定在本機：
- DGX Dashboard：`localhost:11000`
- JupyterLab：`localhost:11002`

你的 PC 無法直接訪問，需要透過 **SSH Tunnel（Port Forwarding）** 轉到本地。

> ⚠️ Port Forwarding 之後，你 PC 瀏覽器要開的是 `http://localhost:11000`，**不是 GB10 的 IP**

---

## 方法：SSH Port Forwarding

### 單次連線（終端機會保持開著）

```bash
ssh -L 11000:localhost:11000 -L 11002:localhost:11002 user@<GB10的IP>
```

### 背景執行（不佔終端機）

```bash
ssh -fNL 11000:localhost:11000 -L 11002:localhost:11002 user@<GB10的IP>
```

| 參數 | 說明 |
|------|------|
| `-f` | 背景執行 |
| `-N` | 不開 shell，只做 tunnel |
| `-L` | 本地 port 轉發 |

### 連線後，在你 PC 的瀏覽器開啟

| 服務 | 網址 |
|------|------|
| DGX Dashboard | http://localhost:11000 |
| JupyterLab | http://localhost:11002 |

---

## 如何確認 JupyterLab 的 Port

**方法一：** 看 GB10 瀏覽器的網址列 port 號

**方法二：** 在 GB10 上執行

```bash
ps aux | grep jupyter
```

---

## 常見問題排除

| 問題 | 症狀 | 原因 | 解決方法 |
|------|------|------|----------|
| 無法執行更新 | 權限不足 | 使用者不在 sudo 群組 | `sudo usermod -aG sudo <username>` 然後執行 `newgrp docker` |
| JupyterLab 無法啟動 | 啟動失敗 | 虛擬環境問題 | 在 JupyterLab 面板中變更工作目錄並啟動新實例 |
| SSH Tunnel 連線被拒絕 | 無法建立 tunnel | IP 或 Port 錯誤 | 確認 GB10 的 IP，並確保 SSH 服務正在執行 |
| GPU 在監控中不可見 | Dashboard 顯示異常 | 驅動程式問題 | 在 GB10 上執行 `nvidia-smi` 檢查 GPU 狀態 |

更多已知問題請參考：[DGX Spark 使用者指南](https://docs.nvidia.com/dgx/dgx-spark/known-issues.html)
