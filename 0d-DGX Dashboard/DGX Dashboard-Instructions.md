
步驟 1

存取 DGX 控制面板
選擇以下方法之一存取 DGX 控制面板 Web 介面：

選項 A：桌面捷徑（本機存取）
如果您可以本地存取 DGX Spark 設備：
登入 DGX Spark 裝置上的 Ubuntu 桌面環境
點擊螢幕左下角開啟 Ubuntu 應用程式啟動器
點擊應用啟動器中的 DGX 控制面板快捷方式
控制台將在您的預設 Web 瀏覽器中打開，位址為 http://localhost:11000

接螢幕鍵盤直接操作或遠端桌面


選項 B：NVIDIA Sync（建議用於遠端存取）
如果您已在本機上安裝 NVIDIA Sync：
點選系統托盤中的 NVIDIA Sync 圖標
從設備清單中選擇您的 DGX Spark 設備
點擊“連接”
點選「DGX 控制面板」啟動控制面板
控制面板將透過自動 SSH 隧道在您的預設 Web 瀏覽器中打開，位址為 http://localhost:11000
沒有安裝 NVIDIA Sync？在此安裝 https://build.nvidia.com/spark/connect-to-your-spark/sync
//就是用成圖形化APP記錄裝置進行啟動 選項 C： SSH 隧道

選項 C：手動下指令建立 SSH 隧道
如需在不使用 NVIDIA Sync 的情況下手動遠端訪問，您必須先手動設定 SSH 隧道。
您必須為 Dashboard 伺服器（連接埠 11000）和 JupyterLab 開啟一個隧道（如果您想遠端存取 JupyterLab）。每個使用者帳戶都會被指派一個不同的 JupyterLab 連接埠號碼。