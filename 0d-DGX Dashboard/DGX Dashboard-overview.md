基本概念

DGX Dashboard 是一個運行在 DGX Spark 設備上的 Web 應用程序，它提供圖形介面，用於系統更新、資源監控以及整合 JupyterLab 環境。使用者可以透過應用程式啟動器在本地存取 DGX Dashboard，也可以透過 NVIDIA Sync 或 SSH 隧道遠端存取。遠端工作時，DGX Dashboard 是更新系統軟體包和韌體的最便捷方式。

您將掌握的內容

您將學習如何在 DGX Spark 設備上存取和使用 DGX Dashboard。完成本教學後，您將能夠啟動預先配置 Python 環境的 JupyterLab 執行個體、監控 GPU 效能、管理系統更新，並使用 Stable Diffusion 執行範例 AI 工作負載。您將了解多種存取方法，包括桌面捷徑、NVIDIA Sync 和手動 SSH 隧道。

開始之前須知

SSH 連線和連接埠轉送的基本終端用法

Python 環境和 Jupyter Notebook 的基礎知識

先決條件

硬體需求：

NVIDIA Grace Blackwell GB10 Superchip 系統

軟體需求：

NVIDIA DGX 作業系統

已安裝 NVIDIA Sync（用於遠端存取）或已設定 SSH 用戶端

輔助文件

SDXL 的 Python 程式碼片段可在 GitHub 上找到

