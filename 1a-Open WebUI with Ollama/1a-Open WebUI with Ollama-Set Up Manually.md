# Open WebUI 安裝與設定指南

> 原文參考：https://build.nvidia.com/spark/open-webui/instructions

---

## Step 1：配置 Docker 權限

> **說明：** 此步驟讓你不需要每次都加 `sudo` 才能執行 Docker。透過將使用者加入 `docker` 群組，大幅提升日常操作便利性。

**1. 先測試目前是否有權限存取 Docker：**

```bash
docker ps
```

**2. 將目前使用者加入 Docker 群組（預設是沒有的）：**

```bash
sudo usermod -aG docker $USER
```

**3. 立即套用群組變更，不需要登出再重新登入：**

```bash
newgrp docker
```

---

## Step 2：驗證 Docker 設定並下載映像檔

> **說明：** 從 GitHub Container Registry 下載整合了 Ollama 引擎的 Open WebUI 映像檔。這是「All-in-One 全功能版」——一個容器內同時包含網頁介面與 AI 模型執行環境。

```bash
docker pull ghcr.io/open-webui/open-webui:ollama
```

---

## Step 3：啟動 Open WebUI 容器

執行以下指令來啟動容器：

```bash
docker run -d -p 8080:8080 --gpus=all \
  -v open-webui:/app/backend/data \
  -v open-webui-ollama:/root/.ollama \
  --name open-webui ghcr.io/open-webui/open-webui:ollama
```

**參數說明：**

| 參數 | 說明 |
|------|------|
| `--gpus=all` | 讓容器使用 GPU 硬體加速，執行 AI 模型時至關重要 |
| `-v` | 建立「持久化儲存」，確保下載的模型與對話紀錄在關閉容器後不會消失 |
| `-p 8080:8080` | 將容器的 8080 埠對映到本機的 8080 埠，開放網頁連線 |

啟動完成後，在瀏覽器輸入以下網址即可開啟 Open WebUI：

```
http://localhost:8080
```

> 📝 下方圖片範例使用的是 Port `12000`，這是透過 `docker-compose.yml` 佈建方式的設定，詳見後段說明。

![openwebui-1](images/owui-1.jpg)<br>

---

## Step 4：建立管理員帳號

首次進入網頁時，系統會要求你註冊帳號。

點擊「**開始使用**」並填寫表單，即可建立本地管理員帳戶並進入主介面。

> **說明：** 這是本地帳號，僅存在你的電腦內，用於保護你的 Web 介面，不需要連線到任何外部服務。

---

## Step 5：下載與設定 AI 模型

> **說明：** 在介面中輸入模型名稱，系統會自動從 Ollama 官方庫下載對應的模型權重。

**進入模型管理頁面的路徑：**

右上角圓圈圖示 → **Admin Panel** → **Settings** → **Models** → 右上角 **Manage** → **Manage Models**

![openwebui-2](images/owui-2.jpg)<br>

點擊「**click here**」可開啟 Ollama 官方模型清單：  
🔗 https://ollama.com/library

![openwebui-3](images/owui-3.jpg)<br>

**以 Llama 3.1 為例：**  
在模型庫中找到想要的模型後，將名稱（例如 `llama3.1:8b`）輸入至下圖標示 5 的欄位中。

![ollama-lib](images/ollama-lib.jpg)<br>

🔗 https://ollama.com/library/llama3.1

![openwebui-4](images/owui-4.jpg)<br>

---

### 📖 補充：模型名稱中的「B」代表什麼？

以 `llama3.1:8b` 為例：

- **B = Billion（十億）**
- **8B = 約 80 億個參數（Parameters）**

**參數可以想像成 AI 大腦的神經連結數量，數量越多，能力越強：**

| 規模 | 參數量 |
|------|--------|
| 3B | 30 億參數 |
| 7B / 8B | 70～80 億參數 |
| 13B | 130 億參數 |
| 70B | 700 億參數 |

**參數越大的優點：**
- 理解能力更強，回答更準確
- 推理與邏輯能力更好
- 處理長文的能力提升

**參數越大的缺點：**
- 需要更多 VRAM / RAM
- 回應速度較慢
- 硬體需求更高

![openwebui-5](images/owui-5.jpg)<br>

---

### 原廠範例模型

原廠官方範例使用的是 `gpt-oss:20b`，容量約 **14 GB**，下載需要一些時間，請耐心等待。

操作步驟：  
點擊「**Select a model**」→ 輸入 `gpt-oss:20b` → 點擊「**Pull from Ollama.com**」→ 等待進度條完成。

模型下載完成後，左上角的模型選單中就會出現該模型名稱：

![model-sel](images/model-sel.jpg)<br>

---

## Step 6：測試模型

> **說明：** 發送指令測試模型的反應速度與回答品質。若 GPU 設定正確，回覆速度會非常快。（若選用較大的模型，第一次回應前需要等待一小段時間載入）

在對話框中輸入以下測試句子，按下送出：

```
Write me a haiku about GPUs.
```

確認模型有正常回應即代表設定成功。

---

## Step 7：後續操作

你可以到 Ollama 官方庫嘗試下載更多不同的模型（如 Llama 3、Mistral 等）：  
🔗 https://ollama.com/library

**更新容器映像檔至最新版本：**

```bash
docker pull ghcr.io/open-webui/open-webui:ollama
```

---

## Step 8：清理與還原（移除所有內容）

> **⚠️ 警告：** 執行 `volume rm` 後，所有對話紀錄與已下載的模型（可能高達數 GB）都會被**徹底刪除**，且無法還原。

**停止並移除容器：**

```bash
docker stop open-webui
docker rm open-webui
```

**刪除映像檔與持久化資料 Volume：**

```bash
docker rmi ghcr.io/open-webui/open-webui:ollama
docker volume rm open-webui open-webui-ollama
```

---

## 附錄：兩種部署方式比較

### 方式一：All-in-One 容器（本文使用的方式）

使用 `open-webui:ollama` 這個特殊映像檔，將 **Ollama（後端引擎）** 和 **Open WebUI（前端介面）** 全部打包在同一個 Docker 容器內執行。

**架構：** 單一容器，內部執行兩個程序。

**優點：**
- 部署最簡單，一條指令搞定所有元件
- 前後端在容器內部直接溝通，不需要處理複雜的網路配置

**缺點：**
- 彈性低：無法單獨升級 Ollama 版本或 WebUI 版本
- 資源管理困難：若 Ollama 發生問題，會直接影響網頁介面
- 耦合度高：若要讓其他應用程式連線到這個 Ollama，設定較複雜

---

### 方式二：Docker Compose 分離部署（建議長期使用）

採用標準的**微服務架構**，將 Ollama 和 Open WebUI 分成兩個獨立容器執行，透過網路（`host.docker.internal`）相互溝通。

**架構：** 兩個獨立容器，透過網路通訊。

**優點：**
- **模組化：** 可隨時單獨升級 Ollama 或 WebUI，互不干擾
- **專業管理：** 透過 Docker Compose YAML，可細緻設定 GPU 資源分配
- **擴充性強：** Ollama 容器作為獨立的「模型伺服器」，可同時被多個工具連線使用
- **網路配置靈活：** 可自訂埠位（如改為 `12000:8080`）避免衝突

**缺點：**
- 配置較複雜，需撰寫 YAML 檔案並正確設定環境變數（如 `OLLAMA_BASE_URL`）

> 📄 詳細設定方式請參考目錄下：`1a-Open WebUI with Ollama-docker-compose.md`

---

## 總結建議

| 情境 | 建議方式 |
|------|----------|
| 只是想快速試玩，看看 AI 跑起來的樣子 | **方式一：All-in-One 容器** |
| 要在伺服器 / 工作站長期使用，未來可能嘗試不同模型或介面 | **方式二：Docker Compose 分離部署** |

> 💡 方式二更符合運維的最佳實踐，也更容易進行故障排除。建議長期使用者盡早熟悉 Docker Compose 的撰寫方式。
