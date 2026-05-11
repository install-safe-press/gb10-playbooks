使用 Dell Pro MAX GB10 佈建 openclaw

前提  1a-Open WebUI with Ollama  已使用 Docker 佈建 
作業  於外層作業系統直接佈建openclaw 並存取 ollama 模型

準備作業-1：
在外層確認 ollama 有使用到 GPU,跑作ollama查詢接下來   docker exec ollama ollama ps
```
沒用到 GPU
user@promaxgb10-0a25:~/Desktop$ docker exec ollama ollama ps
NAME           ID              SIZE     PROCESSOR    CONTEXT    UNTIL              
qwen3.5:35b    3460ffeede54    32 GB    100% CPU     262144     3 minutes from now    
user@promaxgb10-0a25:~/Desktop$
```
```
有用到 GPU
user@promaxgb10-0a25:~/openwebui$ docker exec ollama ollama ps
NAME           ID              SIZE     PROCESSOR    CONTEXT    UNTIL              
qwen3.5:35b    3460ffeede54    34 GB    100% GPU     262144     4 minutes from now    
user@promaxgb10-0a25:~/openwebui$

問題在於 docker compose yml 
   deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

Config 有 GPU 設定，但 container 沒有重建過。執行：
bashcd ~/openwebui
docker compose down
docker compose up -d

準備作業-2：確認可以對外連網 , 回到 Home 目錄 .
