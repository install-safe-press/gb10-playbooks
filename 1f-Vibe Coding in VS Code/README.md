Vibe Coding in VS Code AI 程式開發助手
> 原始出處 https://build.nvidia.com/spark/vibe-coding
---
使用 DGX Spark（GB10） 作為本地或遠端 Vibe Coding 助手，配合 Ollama 和 Continue.dev Extension AI 程式碼助手

https://www.continue.dev/

本指南將引導您完成 DGX Spark 的設置，使其成為Vibe 編碼助理——既可以本地使用，也可以透過 Continue.dev 將其作為 VSCode 的遠端編碼助理。
本指南使用Ollama和GPT-OSS 120B，方便您將編碼助理部署到 VSCode。指南中包含進階說明，指導您如何讓 DGX Spark 和 Ollama 透過本地網路提供編碼助理。
本指南基於全新安裝的作業系統編寫。如果您的作業系統並非全新安裝，並且遇到問題，請參閱故障排除標籤。

你將會取得的成就
您將擁有一個配置完整的 DGX Spark 系統，該系統具備以下功能：

透過 Ollama 運行本地程式碼協助。
為 Continue 和 VSCode 整合提供遠端服務模式。
使用統一記憶體託管大型 LLM，例如 GPT-OSS 120B。

先決條件
DGX Spark（建議使用 128GB 統一記憶體）
Ollama和您選擇的 LLM（例如gpt-oss:120b）
VSCode

繼續VSCode 擴展
用於模型下載的互聯網接入
具備開啟 Linux 終端機、複製貼上指令的基本操作能力。
擁有 sudo 權限。
可選：遠端存取配置的防火牆控制

時間與風險
時長：約30分鐘
風險：由於網路問題導致資料下載緩慢或失敗
回滾：在正常使用過程中未對系統進行任何永久性變更。
最後更新日期： 2025年10月21日
首次發表
