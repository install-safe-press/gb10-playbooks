## docker 說明
在 Linux 上，Docker 透過容器技術把應用程式與其相依環境一起打包，利用作業系統核心（kernel）共享資源，實現快速部署、環境一致性與輕量化運行，比傳統虛擬機更省資源且啟動更快。<br>
在 Linux 上，Docker 透過容器技術將應用程式、函式庫與設定一起打包成映像檔（image），執行時以隔離的容器（container）運行，並共享同一個系統核心（kernel），
因此具備**快速啟動、資源占用低、跨環境一致（開發/測試/正式）與易於擴展（scaling）**等優點；同時可搭配映像版本控管與自動化部署，大幅提升系統維運與開發效率。<br>
舉例：在一個作業系統之下 , ollama 是一個 docker , Openweb UI 是一個 docker .

```text
docker ps
```
![docker-ps](images/docker-ps.jpg)

💻🗄️🧩 簡單理解 docker-compose.yml 設定檔,  Docker Compose 使用的設定檔，用來定義並一次啟動多個 Docker 容器服務。
👉 它就像「服務的藍圖」<br>
把一整個系統（例如：Web + DB + Cache）寫在一個檔案裡，一鍵啟動。<br>
📦 主要功能<br>
定義多個服務（services）<br>
設定映像（image）或建置方式（build）<br>
設定網路（network）<br>
掛載資料（volume）<br>
控制啟動順序與依賴<br>




