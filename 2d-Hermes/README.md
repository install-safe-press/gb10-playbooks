> 原始內容 https://build.nvidia.com/spark/hermes-agent
---
本文件與原始內容環境略有不同, 請自行比對

使用本地模型運行 Hermes Agent。
在 GB10 上安裝並執行 Hermes 自改良 AI 代理程式。

Hermes Agent
GitHub   https://github.com/nousresearch/hermes-agent

Nous Research開發的這款自學習型 AI 智能體，是唯一內建學習循環的智能體——它能從經驗中積累技能，在使用過程中不斷改進，持續學習並鞏固知識，還能搜尋過往對話記錄，並在不同會話中逐步構建更深入的自我認知模型。它可以運行在 5 美元的 VPS、GPU 叢集或幾乎零成本的無伺服器基礎架構上。它不依賴你的筆記型電腦——即使它在雲端虛擬機上運行，你也可以透過 Telegram 與它互動。

Dell Pro MAX GB10環境
依gb10-playbooks專案文件配置
1a-Open WebUI with Ollama , docker 
2a-OpenClaw , 系統最外層

## 2d 安裝 Hermes  , 系統最外層 
```
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```


