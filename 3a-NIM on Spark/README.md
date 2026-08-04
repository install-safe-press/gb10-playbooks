https://build.nvidia.com/spark/nim-llm


在 DGX Spark（NVIDIA 最新的桌上型 AI 開發主機）上部署 NIM (NVIDIA Inference Microservice)。NIM 是容器化的軟體，讓你能在 NVIDIA GPU 上快速、穩定地做 AI 模型服務和推論。
這篇 playbook 示範如何在 DGX Spark 裝置上跑 LLM 的 NIM 微服務，透過簡單的 Docker 流程實現本地 GPU 推論——你會先跟 NVIDIA 的 registry 做身份驗證，啟動 NIM 推論微服務,再做基本的推論測試來驗證功能是否正常。
簡單說：用 Docker 一鍵把大型語言模型跑成一個本地的 OpenAI 相容 API server，示範用的模型是 Llama 3.1 8B，但也支援其他模型（如 Qwen3-32B）。

NIM 的架構本質上是把「模型 + 推論引擎 + API」封裝進一個 Docker 容器裡，讓你不用自己組裝底層技術棧。我畫一張結構圖讓你看清楚容器裡面到底裝了什麼、怎麼跟外部互動：

