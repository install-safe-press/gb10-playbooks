這種方式：Docker Compose 分離部署
這使用的是標準的 微服務架構，將 Ollama 和 Open WebUI 分成兩個獨立的容器執行。

架構：兩個獨立容器，透過網路（及 host.docker.internal）通訊。

優點：

模組化：你可以隨時更換 ollama/ollama:latest 為特定版本，而不影響前端。
專業管理：透過 Docker Compose，你可以更細緻地設定 GPU 分配（deploy.resources 區塊）。
擴充性強：你的 Ollama 容器現在是一個獨立的「模型伺服器」，除了 Open WebUI，你也可以讓其他的工具同時連進來使用。
網路配置靈活：例如你將埠位改成了 12000:8080，避免了常見的 8080 埠位衝突。

缺點：
配置較複雜：需要撰寫 YAML 檔案，並正確設定環境變數（如 OLLAMA_BASE_URL）來讓兩者對接。

本專案之操作應用盡可能會以此微服務架構 呈現

Docker Compose 部署方式
Docker Compose 是一種用來定義與執行「多個容器」應用程式的工具。

簡單來說，如果 docker run 是手動點菜（一次只能點一盤），那 Docker Compose 就是套餐選單（寫好一份清單，一次上齊整桌菜）。

## 核心概念：YAML 設定檔
Docker Compose 的核心是一個名為 docker-compose.yml 的文字檔。在這個檔案裡，你會定義：

Services (服務)：你要跑哪些容器（例如一個 Ollama，一個 WebUI）。

Networks (網路)：容器之間要怎麼互相溝通。

Volumes (資料卷)：資料要存在硬碟的哪個位置。

## 為什麼要用 Docker Compose？
一鍵啟動：原本要下好幾行長長的指令，現在只要輸入 docker-compose up -d，所有容器就會按照設定自動跑起來。

容易維護：所有的設定（如 Port、路徑掛載、GPU 分配）都寫在檔案裡，不用去記當初指令怎麼下的。

服務解耦：像你提到的第二種部署方式，將 Ollama 和 WebUI 分開，這讓你可以單獨重啟或升級其中一個，而不會影響另一個。

環境一致：這份 .yml 檔案拿到哪台電腦，只要執行同一個指令，跑出來的環境都會一模一樣。

## 操作流程簡述
撰寫檔案：使用任一你熟悉的純文字編輯器例如 vi nano  建立一個 docker-compose.yml 檔案並寫入服務配置。
請查看 本文件 yml  路徑下  docker-compose.yml 檔案內容
```text
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: always
    network_mode: host
    volumes:
      - open-webui-ollama:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: always
    ports:
      - "12000:8080"
    environment:
      - OLLAMA_BASE_URL=http://host.docker.internal:11434
    volumes:
      - open-webui:/app/backend/data
    extra_hosts:
      - "host.docker.internal:host-gateway"

volumes:
  open-webui:
  open-webui-ollama:

```


執行指令：在該檔案目錄下執行：
```text
docker-compose up -d
```
( -d 代表在背景執行 )

查看服務：

## 1. 系統容器狀態 (docker ps)
這個表格顯示了主機上目前正在運行的所有容器詳細資訊。

```text
+--------------+---------------------------+---------------------+---------------------+------------------------------------------+--------------+
| CONTAINER ID | IMAGE                     | COMMAND             | STATUS              | PORTS                                    | NAMES        |
+--------------+---------------------------+---------------------+---------------------+------------------------------------------+--------------+
| 3d6f8286db62 | open-webui:main           | "bash start.sh"     | Up 3 hours (healthy)| 0.0.0.0:12000->8080/tcp, [::]:12000->8080| open-webui   |
| dba4d665af98 | ollama/ollama:latest      | "/bin/ollama serve" | Up 3 hours          | (host mode / no specific port binding)   | ollama       |
+--------------+---------------------------+---------------------+---------------------+------------------------------------------+--------------+
```

## 2. Compose 專案狀態 (docker compose ps)
這個表格顯示了在你 ~/openwebui 目錄下，由 Compose 檔案定義的服務邏輯關係。

```text
+-------------+---------------------------+---------------------+------------+---------------------+------------------------------------------+
| NAME        | IMAGE                     | COMMAND             | SERVICE    | STATUS              | PORTS                                    |
+-------------+---------------------------+---------------------+------------+---------------------+------------------------------------------+
| ollama      | ollama/ollama:latest      | "/bin/ollama serve" | ollama     | Up 3 hours          | (Host Network)                           |
| open-webui  | open-webui:main           | "bash start.sh"     | open-webui | Up 3 hours (healthy)| 0.0.0.0:12000->8080/tcp, [::]:12000->8080|
+-------------+---------------------------+---------------------+------------+---------------------+------------------------------------------+
```

### 關鍵觀察點：
埠位映射 (Port Mapping)：你的 Open WebUI 正對外開放 12000 埠位，你可以透過 http://<伺服器IP>:12000 存取。
健康檢查 (Healthcheck)：open-webui 顯示為 healthy，代表容器內部的服務已成功啟動並通過自我檢測。
網路模式：ollama 容器看起來是以 network_mode: host 或類似方式執行，因此在 docker ps 的 PORTS 欄位中沒有顯示傳統的埠位映射。
執行時間：兩個容器都已經穩定運行了 3 小時。


停止服務：
```text
docker-compose down
```
( 這會同時停止並移除清單中所有的容器，非常乾淨 )


對於像 Open WebUI + Ollama 這種需要多個組件協作的應用，使用 Docker Compose 是最推薦的專業做法，因為它讓「管理」變得像「讀文件」一樣直觀。

## 細節說明

🧩 1️⃣ 定義多個服務（services）
services:
  ollama:
  open-webui:

👉 你這裡定義了 兩個服務：

ollama：跑 LLM（模型服務）
open-webui：提供 Web UI 介面

📌 重點：
這兩個容器會一起被 Docker Compose 管理與啟動

🐳 2️⃣ 設定映像（image）或建置方式（build）
ollama:
  image: ollama/ollama:latest
open-webui:
  image: ghcr.io/open-webui/open-webui:main

👉 這裡使用的是 image（現成映像），沒有用 build

📌 代表：

直接從 Docker Hub / GHCR 拉 image
不需要自己寫 Dockerfile
🌐 3️⃣ 設定網路（network）
ollama
network_mode: host

👉 使用 host network

容器直接用主機網路
port 直接開在主機上（11434）
open-webui
ports:
  - "12000:8080"

👉 使用 Docker 預設 bridge network

主機 12000 → 容器 8080
兩者如何互通？
environment:
  - OLLAMA_BASE_URL=http://host.docker.internal:11434
extra_hosts:
  - "host.docker.internal:host-gateway"

👉 這段是關鍵：

open-webui 透過 host.docker.internal
去連 host 上的 ollama

📌 因為：

ollama 用 host network
open-webui 在 bridge network
➡️ 不能直接用 container name 連
💾 4️⃣ 掛載資料（volumes）
volumes:
  - open-webui-ollama:/root/.ollama

👉 ollama 模型存放位置
（模型下載後不會消失）

volumes:
  - open-webui:/app/backend/data

👉 open-webui 的資料（帳號、設定）

volumes:
  open-webui:
  open-webui-ollama:

👉 宣告兩個 volume（由 Docker 管理）

📌 重點：

容器刪掉，資料還在
避免模型重新下載（很重要）
🔗 5️⃣ 控制啟動順序與依賴

👉 這份 沒有寫 depends_on

例如你可以加：

depends_on:
  - ollama

📌 目前狀況是：

open-webui 可能比 ollama 早啟動
但通常會自動 retry（問題不大）
🚀 補充：GPU 設定（你這份很關鍵）
deploy:
  resources:
    reservations:
      devices:
        - driver: nvidia
          count: all
          capabilities: [gpu]

👉 讓 ollama 使用 NVIDIA GPU

📌 前提：

有裝 NVIDIA driver
有裝 nvidia-container-toolkit
🧾 總結（你的架構）

👉 這份 compose 在做的事是：

ollama：跑模型（用 GPU + host network）
open-webui：提供 UI（透過 HTTP 呼叫 ollama）
用 volume 保存模型與資料
用 host + bridge 混合網路讓兩者溝通

## 結果輸出
