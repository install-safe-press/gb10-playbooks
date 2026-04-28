
## 要如何在自已的PC/NB連上並示GB10的 DGX Dashboard / JupyterLab web


你想從自己的 PC 瀏覽器直接打開 GB10 的 DGX Dashboard（現在是 localhost:11000），對嗎？
因為 localhost:11000 只有在 GB10 本機才能存取，你的 PC 需要透過 SSH Tunnel 把那個 port 轉到本地。

方法：SSH Port Forwarding


在你的 PC 上執行：

bash
ssh -L 11000:localhost:11000 user@<GB10的IP>


然後在你 PC 的瀏覽器打開：
http://localhost:11000
就會看到 GB10 的 DGX Dashboard ,如同你在GB10上操作 .



讓連線持續在背景跑（不佔終端機）
bashssh -fNL 11000:localhost:11000 user@<GB10的IP>

-f 背景執行
-N 不開 shell，只做 tunnel

相同方式
你想從自己的 PC 瀏覽器直接打開 GB10 的 DGX JupyterLab web（現在是 localhost:11002），對嗎？
怎麼看是11002？
1.在GB10的JupyterLab web瀏覽器的port
2.在GB10上執行

ps aux | grep jupyter



問題 症狀 原因 解決方法

使用者無法執行更新 使用者不在 sudo 群組中 將使用者新增至 sudo 群組：sudo usermod -aG sudo <使用者名稱>; 然後執行 newgrp docker

JupyterLab 無法啟動 目前虛擬環境存在問題 在 JupyterLab 面板中變更工作目錄並啟動新實例

SSH 隧道連線被拒絕 IP 或連接埠錯誤 驗證 Spark 設備 IP 並確保 SSH 服務正在執行

GPU 在監控中不可見 驅動程式問題 使用 nvidia-smi 檢查 GPU 狀態

有關最新的已知問題，請參閱 DGX Spark 使用者指南。https://docs.nvidia.com/dgx/dgx-spark/known-issues.html

GB10網頁連不上，Port Forwarding 之後你的PC 瀏覽器直接打開的地址是localhost 而不是GB10的IP