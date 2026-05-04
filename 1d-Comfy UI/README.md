Comfy UI ： 安裝與使用 AI 繪圖工具 Comfy UI 生成圖像.
> 原始出處 https://build.nvidia.com/spark/comfy-ui
---

基本思路
ComfyUI 是一款開源的 Web 伺服器應用程序，用於使用 SDXL、Flux 等基於擴散的模型生成 AI 影像。它擁有基於瀏覽器的使用者介面，允許使用者建立、編輯和運行包含多個步驟的圖像生成和編輯工作流程。這些生成和編輯步驟（例如，載入模型、新增文字或取樣）在使用者介面中以節點的形式進行配置，使用者可以透過連接線將節點連接起來，從而形成工作流程。

ComfyUI 使用主機的 GPU 進行推理，因此您可以將其安裝在GB10 DGX Spark 或者是帶有Nvidia GPU 的桌機筆電上，並直接在您的裝置上完成所有影像產生和編輯。

工作流程以 JSON 檔案格式儲存，因此您可以對其進行版本控制，以便將來進行工作、協作和可重現性管理。

你將會取得的成就
您將在 NVIDIA GB10 DGX Spark 裝置上安裝和設定 ComfyUI，以便您可以使用統一記憶體來處理大型模型。

開始前需要了解什麼
具備使用 Python 虛擬環境和套件管理的經驗
熟悉命令列操作和終端使用
對深度學習模型部署和檢查點有基本的了解
了解容器工作流程與 GPU 加速概念
了解存取 Web 服務所需的網路配置
先決條件

硬體需求：
NVIDIA Grace Blackwell GB10 超級晶片系統
穩定擴散模型至少需要 8GB GPU 顯存 
至少 20GB 可用儲存空間
軟體需求：

已安裝 Python 3.8 或更高版本：python3 --version
pip 套件管理器可用：pip3 --version
與 Blackwell 相容的 CUDA 工具包：nvcc --version
Git 版本控制：git --version
透過網路存取 Hugging Face 下載模型
Web 瀏覽器存取連接<SPARK_IP>:8188埠
輔助文件
所有必需的資源都可以在 GitHub 上的 ComfyUI 程式碼庫中找到。

requirements.txt- ComfyUI 安裝所需的 Python 依賴項
main.py- ComfyUI 伺服器應用程式的主要入口點
v1-5-pruned-emaonly-fp16.safetensors- 穩定擴散 1.5 檢查點模型
時間與風險
預計時間： 30-45分鐘（含模型下載）
風險等級：中等
模型下載檔案較大（約2GB），可能因網路問題而失敗。
連接埠 8188 必須可存取才能使用 Web 介面功能。
回滾：可以刪除虛擬環境以移除所有已安裝的軟體套件。可以手動從檢查點目錄中刪除已下載的模型。
最後更新時間： 2025年11月10日
將 ComfyUI PyTorch 更新至 CUDA 13.0
