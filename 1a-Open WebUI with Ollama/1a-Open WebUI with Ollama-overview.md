Open WebUI with Ollama 

安裝 Open WebUI 與 Ollama 應用大語言模型開始交談

🚀 快速上手：在 DGX Spark（Dell Pro MAX GB10） 部署 Open WebUI (整合 Ollama)
想在你的 DGX Spark 上擁有像 ChatGPT 一樣漂亮的操作介面，同時確保所有資料都保存在本地端嗎？
透過 Open WebUI 結合 Ollama，你可以直接在瀏覽器上調用 GPU 的強大算力來跑 AI 模型。

💡 核心概念
Open WebUI：你的 AI 控制面板（WEB介面）。
Ollama：後台的模型驅動引擎（大腦）。Ollama 是一個開源的本地大語言模型（LLM）執行框架，旨在讓使用者能夠在自己的電腦上（而非依賴雲端伺服器）輕鬆地執行、管理和部署各種 AI 模型。
Ollama 是一個開源的本地大語言模型（LLM）執行框架，旨在讓使用者能夠在自己的電腦上（而非依賴雲端伺服器）輕鬆地執行、管理和部署各種 AI 模型。
DGX Spark(Dell Pro MAX GB10)：在你本地的桌上電腦，運行Ubuntu Linux 作業系統，提供強大 GPU 加速的硬體基礎設施。

🛠️ 準備工作
在開始之前，請確認：

設備已就緒：你的 DGX Spark (Dell Pro MAX GB10) 已經開機並可以連線。
網路通暢：你可以透過本地網路（或 SSH 隧道）存取設備。
空間足夠：準備好幾 GB 的空間來存放 Docker 鏡像與 AI 模型。

⌛ 預估時間：15 - 20 分鐘（不含下載大型模型的時間）。
⚠️ 小提醒：如果遇到 Docker 權限報錯，可能需要將您的帳號加入 docker 使用者群組並重新登入。

📝連線工具
SSH
WEB Browser瀏覽器

 安裝步驟
第一步：啟動 Open WebUI 容器
我們使用 Docker 一鍵啟動。這個指令會自動下載 Open WebUI 並與內建的 Ollama 綁定，同時開啟 GPU 加速。


第二步：存取網頁介面
安裝完成後，開啟瀏覽器並輸入以下網址：

如果你遠端連線：請輸入設備的 IP 地址，例如 http://192.168.x.x:12000

提示：第一次登入需要註冊一個管理員帳號（這只是存在你本地設備上的，請放心）。

第三步：下載並開始對話
進入介面後：

點擊設定或模型管理。

輸入你想下載的模型名稱（例如 llama3 或 gemma）。

等待下載完成後，就可以開始在OpenWeb UI介面上與 AI 對話了！

🆗結果畫面呈現



