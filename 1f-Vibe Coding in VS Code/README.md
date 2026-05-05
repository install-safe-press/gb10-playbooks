# Vibe Coding in VS Code：AI 程式開發助手

> 原始出處：https://build.nvidia.com/spark/vibe-coding  
---

## 什麼是 Vibe Coding？

**Vibe Coding** 是一種以 AI 輔助為核心的程式開發方式——你描述想要的功能，AI 幫你寫出程式碼，讓開發流程更快、更直覺。

本章節將帶你把 **DGX Spark（GB10）** 設定為本地（或遠端/雲端）的 AI 程式碼助手，搭配以下工具組合：

| 工具 | 說明 |
|------|------|
| **Ollama** | 在 GB10 上本地執行 LLM |
| **GPT-OSS 120B** | 作為程式碼助手的 AI 模型 |
| **Continue.dev** | VS Code 的 AI 程式碼助手擴充套件 |

🔗 Continue.dev 官網：https://www.continue.dev/

---

## 本章節目標

完成設定後，你的 GB10 將具備以下能力：

- 透過 **Ollama** 在本機執行 AI 程式碼輔助
- 作為**遠端服務**，供 VS Code 的 Continue 擴充套件連線使用
- 利用 GB10 的**統一記憶體（UMA）** 執行 GPT-OSS 120B 等大型模型

---

![vscode-vide](images/vscode-vibe-30fps-1920.gif)
## 先決條件

### 硬體需求

- **DGX Spark（GB10）**，使用 128GB 統一記憶體版本

### 軟體需求

- **Ollama**（已安裝於 GB10）及你選擇的 LLM（例如 `gpt-oss:120b`）
- **VS Code**（安裝於 Windows 筆電）
- **Continue** VS Code 擴充套件
- 可連線至網際網路（用於下載模型）

### 操作能力需求
先行理解之前章節

1a-Open WebUI with Ollama <br>
1b｜VS Code 遠端開發環境 <br>

---

輸出結果


