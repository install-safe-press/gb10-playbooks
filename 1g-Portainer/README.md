# 部署 Portainer： WEB 圖形化 Docker 管理介面

> 官方網站：https://www.portainer.io/  提供企業解決方案詳見網站資訊, 本章節是採用（Community Edition，社群版）

---
（本章節不屬於 Nvidia DGX Spark playbooks , 是針對容器管理所創建的工具 ）

## 什麼是 Portainer？

**Portainer CE** 就像是 Docker 的「**網頁控制台**」，把原本需要記憶並輸入指令的管理工作，全部改成在瀏覽器中點擊按鈕操作。
另有商業版請看Portainer官方網站。

---

## Portainer 可以做什麼？

### 管理容器（Containers）

直接在網頁上對容器進行：啟動 / 停止、重啟、刪除、查看日誌、進入 Console（類似 SSH 進入容器內部）

### 管理映像檔（Images）

拉取新 Image、刪除舊版、更新版本、查看容量使用狀況

### 管理 Volume（資料卷）

查看資料儲存位置、建立永久儲存、刪除未使用的 Volume

### 管理 Network（網路）

查看容器互聯方式、建立自訂網段、排查 Port 連線問題

---

## CLI 指令 vs Portainer 比較

| 功能 | Docker CLI 指令 | Portainer 操作方式 |
|------|----------------|-------------------|
| 查看容器 | `docker ps` | Web UI 一覽表 |
| 停止容器 | `docker stop` | 按鈕點擊 |
| 查看日誌 | `docker logs` | 圖形化即時顯示 |
| Compose 部署 | CLI 指令 | Web 表單 |
| 資源監控 | 指令查詢 | Dashboard 圖表 |

> 💡 **最大優勢：**  
> 初學者不用背指令，進階使用者可集中管理多個服務與主機。  
> Portainer 特別適合 GB10、本地 AI Server、NAS 與 HomeLab 環境。

---

## 開始部署 Portainer

以下步驟均在 **GB10 終端機**執行。

### Step 1：建立 Portainer 目錄

```bash
mkdir portainer
```

### Step 2：進入目錄

```bash
cd portainer/
```

### Step 3：建立 docker-compose.yml 設定檔

使用任意編輯器（`vi` 或 `nano`）建立檔案，內容如下：

```yaml
version: "3.8"
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    ports:
      - "8000:8000"
      - "9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
volumes:
  portainer_data:
```

> **Port 說明：**
> - `8000`：Agent 通訊用（多主機管理時使用）
> - `9443`：Portainer 網頁介面（HTTPS）

### Step 4：啟動 Portainer 容器

```bash
docker compose up -d
```

### Step 5：確認容器是否正常執行

```bash
docker ps
```

看到 `portainer` 容器狀態為 `Up` 即代表啟動成功 ✅

### Step 6：查看 Portainer 啟動日誌

```bash
docker logs portainer
```

![portainer-1](images/portainer-1.jpg)<br>

---

## 首次設定 Portainer

### Step 7：開啟 Portainer 網頁介面

在瀏覽器輸入以下網址（將 `GB10_IP` 替換為 GB10 的實際 IP）：

```
https://GB10_IP:9443
```

> 第一次開啟時，系統會要求你設定 **admin 帳號密碼**，請設定一組你記得住的密碼。

![portainer-2](images/portainer-2.jpg)<br>

---

### Step 8：完成初始設定精靈

登入後依序點擊：**Quick Start** → **Environment Wizard** → **Get Started**

![portainer-3](images/portainer-3.jpg)<br>

畫面會顯示 GB10 本地 Docker 的運行狀態。點擊 **Live connect** 進入本地環境的詳細 Dashboard。

![portainer-4](images/portainer-4.jpg)<br>

---

## 使用 Portainer 管理容器

### Dashboard 總覽

進入後可以看到 **Dashboard** 與 **Environment info**，顯示目前有幾個容器在運行。點擊數字可查看容器清單細節。

![portainer-5](images/portainer-5.jpg)<br>

### 容器清單（等同 docker ps）

容器清單頁面列出所有容器的狀態——是不是很像 `docker ps` 指令的輸出？

![portainer-6](images/portainer-6.jpg)<br>

### 進入容器操作

點擊 **ollama** 容器，可以查看詳細狀態或對它進行操作。點擊 **`>_ Console`** 可以進入該容器的命令列介面。

![portainer-8](images/portainer-8.jpg)<br>

點擊 **Connect** 連入容器的 CLI Console：

![portainer-9](images/portainer-9.jpg)<br>

進入 Console 後，就像在終端機中操作一樣，可以直接在這個畫面對容器下指令：

![portainer-10](images/portainer-10.jpg)<br>

### 在容器內執行指令範例

**列出 Ollama 容器內的所有模型：**

```bash
ollama list
```

![portainer-11](images/portainer-11.jpg)<br>

**查看容器內的磁碟空間使用狀況：**

```bash
df -h
```

![portainer-12](images/portainer-12.jpg)<br>

> ⚠️ **注意：** 以上指令都是在 **ollama 容器的內部環境**執行，不是在 GB10 主機系統上。離開 Console 後，這些操作不會影響到 GB10 本身。
