1c-Set up Tailscale on Your Spark
>原文參考https://build.nvidia.com/spark/tailscale
---

Tailscale 是一款基於 WireGuard® 協議的虛擬私人網路（VPN）服務，它的核心目標是「讓聯網變得簡單且安全」。與傳統 VPN 不同，它採用 P2P（點對點）全網狀架構（Mesh Network），讓不同網路環境下的裝置就像連在同一個 Wi-Fi 下。

以下是 Tailscale 的核心功能與特色：

1. 零配置虛擬內網 (Mesh VPN)
這是 Tailscale 最基本的功能。只要在裝置（如電腦、手機、NAS、伺服器）安裝並登入同一個帳號，這些裝置就會獲得一個固定的內部 IP（Tailscale IP）。

突破 NAT 與防火牆： 即使裝置在不同的路由器後方，或是在公司、咖啡廳的防火牆內，通常不需要設定轉址（Port Forwarding）就能直接連通。

全網狀連接： 裝置之間是直接點對點傳輸，不會所有流量都經過中央伺服器，延遲更低。

2. Taildrop (跨平台檔案傳輸)
這功能類似於 Apple 的 AirDrop，但不受限於同一個區域網路。你可以在 Android、iOS、Windows、Linux、macOS 之間互相傳送圖片或檔案，只要裝置都有安裝 Tailscale 並登入即可。

總結來說：
如果你需要遠端存取家裡的 NAS、在公司連回家的開發環境、或是想要一個極簡化的跨裝置連網方案，Tailscale 目前是公認門檻最低且最穩定的選擇之一。

![ts-1](images/tailscale-1.jpg)<br>
![ts-2](images/tailscale-2.jpg)<br>
![ts-3](images/tailscale-3.jpg)<br>
![ts-4](images/tailscale-4.jpg)<br>
