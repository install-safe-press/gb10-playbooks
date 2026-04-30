# NVIDIA DGX Spark（Dell Pro Max GB10）：個人 AI 超級電腦簡介

從桌上型 AI 開發一路延伸至資料中心級 AI 應用，**NVIDIA DGX Spark（Dell Pro Max GB10）** 定位為「個人 AI 超級電腦」等級的桌上型工作站。<br>
專為開發人員、研究人員、數據科學家，以及希望建立本地 AI 能力的企業用戶設計。<br>

採用最新 **NVIDIA Blackwell 架構**，體積極為精巧（約 15 公分見方），可直接放置於桌面環境，讓使用者在本地端即可完成：<br>

- 大型模型原型設計  
- 模型微調（Fine-tuning）  
- 模型推論（Inference）  
- AI Agent 開發  
- 多機叢集部署<br>

---

## 1. 核心技術規格

| 項目 | 詳細規格 |
| :--- | :--- |
| **處理器晶片** | **NVIDIA GB10**（Grace Blackwell 超級晶片） |
| **GPU 架構** | Blackwell（第 5 代 Tensor Core） |
| **CPU 架構** | 20 核心 Arm（10x 高性能核心 + 10x 節能核心） |
| **AI 算力** | 高達 **1 PetaFLOP**（FP4 精度） |
| **統一記憶體** | **128 GB LPDDR5x**（CPU 與 GPU 共享的一致性架構） |
| **記憶體頻寬** | 273 GB/s |
| **儲存空間** | 1TB ~ 4TB NVMe SSD |
| **網路連接** | 10 GbE RJ-45 + **ConnectX-7 SmartNIC**（最高 200 Gbps） |
| **無線技術** | Wi-Fi 7 / Bluetooth 5.4 |
| **尺寸與重量** | 150 x 150 x 50.5 mm / 1.2 kg |
| **最大功耗** | 約 240W |

---

## 2. 主要特點分析

### 🚀 革命性的統一記憶體架構
DGX Spark 最大的核心優勢，在於其 **128GB 統一記憶體（Unified Memory）**。<br>

由於 CPU 與 GPU 共用相同記憶體空間，可大幅降低資料搬移延遲，使其在極小體積下依然能直接運行高達 **200B（2,000 億參數）** 的大型語言模型。<br>

### 對初學者的意義：
- 不必過度擔心顯存不足  
- 可直接接觸更大型模型  
- 更適合本地 AI 開發與實驗<br>

---

### 🔌 高速擴展與群集能力
雖然體積小巧，但具備接近資料中心級的擴展能力：<br>

* **雙機串聯**：透過 ConnectX-7 高速網路，可將兩台 DGX Spark 串聯，共同處理高達 **405B（4,050 億參數）** 的模型（如 Llama 3.1 405B）。  
* **資料中心級網路**：支援最高 200Gbps 頻寬，讓桌面環境也能模擬企業級叢集架構。  

### 優勢：
- 本地叢集實驗  
- AI 基礎架構驗證  
- 未來可平滑升級至大型資料中心部署<br>

---

### 🛠️ 完整的 AI 開發堆疊
隨附 **NVIDIA DGX OS**，這是一套基於 Linux 並經過深度優化的 AI 作業系統。<br>

整合內容包含：
* **CUDA Toolkit & cuDNN**：GPU 加速核心函式庫  
* **NVIDIA AI Enterprise**：完整企業級 AI 管理  
* **開發框架**：支援 NVIDIA Isaac、Metropolis、Holoscan 等生態系  

### 初學者可理解為：
你買到的不只是硬體，而是一整套已經幫你準備好的 AI 開發平台。<br>

---

## 3. 應用場景

1. **本地模型開發與原型設計**  
   在將模型部署至昂貴雲端前，先於本地端快速測試。  

2. **大模型微調（Fine-tuning）**  
   單機可微調如 Llama 3.1 70B 等中大型模型。  

3. **邊緣計算（Edge AI）**  
   適合部署於辦公室、實驗室、醫療設施或企業內網。  

4. **隱私與安全**  
   可於完全離線環境中處理敏感資料，滿足企業或政府合規需求。  

---

## 4. 總結
**NVIDIA DGX Spark** 成功填補了：  
**高效能筆電 → AI 工作站 → 資料中心伺服器** 之間的重要空白。<br>

它將過去需要多台 GPU Server 才能完成的大模型推論與微調能力，濃縮進一台僅 1.2 公斤的桌上型設備中。<br>

### 一句話總結：
**GB10 是目前性能密度極高、最適合作為本地 AI 開發起點的平台之一。**

---

## NVIDIA GTC Taipei 2025 現場實機展示
以下為拍攝於 NVIDIA GTC Taipei 2025 的同晶片架構品牌機（GB10 平台）：  

![gtc2025-1](images/gtc-2025-tp-01.jpg)
![gtc2025-1](images/gtc-2025-tp-01.jpg)
---

## NVIDIA GB10 主機板
![gtc2025-2](images/gtc-2025-tp-02.jpg)

![gtc2025-3](images/gtc-2025-tp-03.jpg)

---

由於 NVIDIA 已將 GB10 平台開放給合作品牌，因此市場上出現多家規格近似產品。<br>

### 常見品牌型號：
- Acer Veriton GN100 AI Workstation  
- ASUS Ascent GX10  
- Dell Pro Max With GB10  
- Gigabyte AI TOP ATOM  
- HP ZGX Nano AI Station  
- Lenovo ThinkStation PGX  
- MSI EdgeXpert MS-C931  

### 差異主要在：
- 外殼設計  
- 散熱方案  
- 管理軟體  
- SSD/Wi-Fi 配置  
- 售價與售後服務  

---

# GB10（Grace Blackwell）平台產品橫向分析表
*以下內容為市場定位整理，實際仍應依各品牌官方資料為準。*

| 產品 | 平台定位 | 管理 / 軟體堆疊 | 核心差異點 | 適合族群 | 部署場景 |
|------|----------|------------------|------------|----------|----------|
| NVIDIA DGX Spark | 官方參考標準 AI 節點（Reference DGX Node） | 完整 DGX Stack（NGC / CUDA / AI Enterprise） | 最完整原生 NVIDIA 生態 | AI 研究員 / ML 工程師 / 高相容需求者 | AI Lab / 研發中心 / 企業 AI 中樞 |
| Dell Pro Max with GB10 | 企業級整合工作站 | NVIDIA Stack + Dell ProSupport | 強企業維運與售後整合 | IT 部門 / 企業 AI 導入 | 企業內部 AI 部署 |
| Lenovo ThinkStation PGX | 長時間穩定運算工作站 | NVIDIA Stack + Lenovo 管理工具 | 散熱 / 穩定性優化 | 研究機構 | 實驗室 |
| HP ZGX Nano AI Station | 高資安工作站 | NVIDIA Stack + HP Wolf Security | 安全性強 | 金融 / 政府 | 高安全企業環境 |
| ASUS Ascent GX10 | 創作者導向工作站 | NVIDIA Stack + ASUS ProArt | UI / 創作導向 | AI 創作者 | 工作室 |
| Gigabyte AI TOP ATOM | 低門檻 AI 平台 | AI TOP GUI | No-code / 易上手 | Startup / 教學 | 快速 PoC |
| MSI EdgeXpert MS-C931 | 工業級邊緣 AI 節點 | 工業管理介面 | 工規耐用 | 工廠 / 自動化 | Edge AI |
| Acer Veriton GN100 | 成本導向工作站 | 基礎 NVIDIA Stack | 高性價比 | 教育 / 公部門 | 大規模部署 |

---

# 核心差異總結
### 各品牌主要差異：
- Wi-Fi 模組  
- SSD 容量  
- 外觀設計  
- 售價  
- 管理工具  

### 核心運算能力：
大致相近。<br>

---

# build.nvidia.com 是什麼？
**build.nvidia.com** 是 NVIDIA 官方 AI 模型與服務平台。<br>

主要提供：

### 1️⃣ Blueprint（藍圖教學）
官方 AI 工作流 + 程式碼範例  
適合快速建立：
- 聊天機器人  
- AI Agent  
- RAG 系統  

---

### 2️⃣ Step-by-step Playbooks（逐步教學）
例如：
- AI Agent 建置  
- GPU 啟動  
- API 串接  
- 本地模型部署  

---

### 3️⃣ API Sandbox + 模型測試
可直接測試：
- Chat  
- Coding  
- Agent  
- 多模態模型  

---

## GB10 專屬 Playbooks
https://build.nvidia.com/spark

### Playbook 白話文：
**NVIDIA 幫你寫好的 AI 專案實戰教學腳本。**

內容包含：
- 架構流程  
- API 呼叫  
- 範例程式  
- 部署步驟  

### 適合初學者：
照著做即可快速理解 AI 專案完整流程。  

---

![playbooks](images/00-allplaybooks-20260420.jpg)

---

## 本專案核心目標
## 👉「先在桌上把 AI 玩熟，再把同一套能力放大到機房等級。」

以 Dell Pro Max GB10 作為本地 AI 開發起點：  
- 模型部署  
- 模型推論  
- 系統整合  
- 多機叢集  

進一步延伸至：
- H200  
- GPU Server  
- Data Center 級應用  

---

> 決策重點：  
> - 要快速導入 AI → 選 GB10  
> - 要極致算力 / 大規模訓練 → 選 GPU Server  
> - 要彈性與全球擴展 → 選雲端 AI <br>

---

| 決策面向 | GB10 | Mac（Apple） | 雲端 AI | GPU 工作站 | GPU Server |
|----------------|--------------------------|--------------|----------------|------------------|-----------------------|
| **資料安全** | ✅ 完全內部 | ✅ 本地 | ❌ 外部風險 | ✅ 本地 | ✅ 完全內部 |
| **成本模式** | 一次性（可控） | 一次性 | ❌ 持續支出 | 一次性 | ❌ 高資本投入 |
| **AI 能力** | ✅ 完整平台 | ⚠️ 個人開發 | ✅ 最強 | ✅ 強 | ✅ 最強（企業級） |
| **模型大小** | ✅ 大模型（128GB） | ❌ 受限 | ✅ 無限制 | ⚠️ 視 GPU | ✅ 超大模型 |
| **部署難度** | ✅ 低（已整合） | ✅ 低 | ⚠️ 中 | ❌ 高 | ❌ 很高 |
| **擴展性** | ✅ 可集群 | ❌ | ✅ 無限 | ⚠️ 有限 | ✅ 高（資料中心） |
| **延遲** | ✅ 低 | ✅ 低 | ❌ 高 | ✅ 低 | ✅ 低 |
| **維運成本** | ✅ 低 | ✅ 低 | ❌ 高 | ⚠️ 中 | ❌ 很高 |
| **導入速度** | ✅ 快（開箱即用） | ✅ 快 | ⚠️ 中 | ❌ 慢 | ❌ 很慢 |
| **適合角色** | 企業 IT / AI 導入 | 個人開發 | SaaS / 平台 | AI 工程師 | 大型企業 / 資料中心 |

---

## 本專案採用 Dell Pro Max GB10 作為主要操作示範平台。  
## 非常歡迎其他品牌使用者共同交流、補充與技術支援。

