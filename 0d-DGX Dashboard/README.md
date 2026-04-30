
## DGX Dashboard：Monitor your DGX system and launch JupyterLab

DGX Dashboard 提供兩大核心功能：

---

### 1️⃣ DGX Dashboard Monitor（系統監控）

- 以 Web 方式呈現 GPU / 記憶體使用狀態
- 類似「即時儀錶板（Dashboard）」
- 用來觀察 AI 運算負載、資源消耗

---

### 2️⃣ JupyterLab（互動式開發環境）

- 一鍵啟動 JupyterLab
- 適合：
  - AI 開發
  - Python 程式設計
  - 資料科學分析
  - 實驗與研究

---

## GB10 入口捷徑

在 GB10 桌面上會看到：

👉 **DGX Spark Resources**

點擊後會開啟：

🔗 https://build.nvidia.com/spark

---

![dashboard-0](images/dashboard-0.jpg)

進入後可看到 DGX Dashboard 入口頁面：

![dashboard-1](images/build.nvidia.com-spark.jpg)

---

## DGX Dashboard Monitor 操作說明

DGX Dashboard Monitor 的本質是：

👉 在本機開啟 **http://localhost:11000**  
👉 透過 Web UI 顯示 GPU / 記憶體圖形化資訊

---

### Dashboard 主畫面

![dashboard-2](images/dashboard-1.jpg)

---

### 系統更新（Settings）

- 可進行系統 Update
- 更新韌體或套件

![dashboard-3](images/dashboard-2.jpg)

---

### 修改主機名稱

路徑：

👉 Settings → System → Edit Hostname

![dashboard-4](images/dashboard-3.jpg)

---

## 文件與社群資源

### 📘 官方文件

- DGX Spark 使用手冊  
  https://docs.nvidia.com/dgx/dgx-spark/

### 💬 開發者論壇

- NVIDIA DGX Spark 討論區  
  https://forums.developer.nvidia.com/c/accelerated-computing/dgx-spark-gb10/

---

📌 注意：

以上連結與 Dashboard 操作皆為 **本機 localhost 資源**

如果是從其他設備存取，請參考：

👉 `DGX Dashboard-Instructions.md`

---

## 2️⃣ 啟用 JupyterLab（AI 開發環境）

JupyterLab 是 GB10 上的核心開發環境，用於：

- Python 程式開發
- AI 模型測試
- Notebook 實驗
- GPU 加速運算

---

### ▶ 啟動方式

點擊：

👉 **JupyterLab → Start**

首次啟動可能需要：

⏳ 等待初始化數分鐘

完成後點擊：

👉 **Open in Browser**

---

### JupyterLab 啟動畫面

![jupyter](images/jupyter-1.jpg)

---

### 進入工作區

![jupyter](images/jupyter-2.jpg)

---

### 建立 Python Notebook

![jupyter](images/jupyter-create.png)

---

## 🧪 AI 測試範例（SDXL）

在 Notebook 中貼上 Prompt：

```

一個舒適的現代閱讀角落，配有大窗戶，柔和的自然光，照片級真實感

```

---

### 執行 AI 生成流程

![jupyter](images/jupyter-post.png)

---

### SDXL 圖像生成結果

![jupyter](images/jupyter-run-sd.png)

---

## 🧠 SDXL 簡單說明

Stable Diffusion XL（SDXL）是：

👉 由 Stability AI 推出的高階文字生成圖片模型（Text-to-Image）

---

### 在 GB10 上的運作方式

輸入文字 Prompt → GPU 運算 → 自動生成圖片

---

## 🎯 常見應用場景

### 👍 創作者
- 插畫
- 封面設計
- YouTube 縮圖
- Logo 草圖

### 👍 商業應用
- 商品圖生成
- 廣告素材
- 行銷視覺設計

### 👍 技術玩家
- 本地 GPU AI 部署
- AI 工作站開發
- 自建圖像生成服務

---

## 📊 實務操作建議

在執行 SDXL 或 AI 任務時：

👉 可以同時開啟 DGX Dashboard  
👉 觀察 GPU / RAM 使用率變化  
👉 用來理解 AI 運算負載狀態
```
