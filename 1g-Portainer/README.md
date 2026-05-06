部署 Portainer ，讓你的 GB10 現在具備完整圖形化 Docker 管理能力。

Portainer CE 就像是 Docker 的「網頁控制台」，把原本需要輸入指令的管理工作改成圖形化操作。

Portainer 可以做什麼？
1. 管理容器（Containers）

你可以直接：

啟動 / 停止
重啟
刪除
查看日誌
進入 Console（類似 SSH 到容器內）
2. 管理映像檔（Images）
拉取新 Image
刪除舊版
更新版本
查看容量使用
3. 管理 Volume（資料卷）
查看資料儲存位置
建立永久儲存
刪除未使用 Volume
4. 管理 Network（網路）
查看容器互聯方式
建立自訂網段
排查 Port 問題

| 功能         | Docker CLI  | Portainer |
| ---------- | ----------- | --------- |
| 查看容器       | docker ps   | Web UI    |
| 停止容器       | docker stop | 按鈕        |
| 查看日誌       | docker logs | 圖形化       |
| Compose 部署 | CLI         | Web 表單    |
| 資源監控       | 指令          | Dashboard |


最大優勢
初學者：

不用背指令。

進階使用者：

集中管理多服務、多主機。

Portainer 讓你用瀏覽器就能完整控制 Docker 生態，特別適合 GB10、本地 AI Server、NAS 與 HomeLab。

>原廠站站 https://www.portainer.io/
---
