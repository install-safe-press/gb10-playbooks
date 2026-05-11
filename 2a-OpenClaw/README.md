2a-OpenClaw
OpenClaw 是一款在 2026 年初爆紅的開源（MIT 授權）AI Agent 框架，被網友戲稱為「養龍蝦」 因為 OpenClaw 的 Logo 是一隻紅色小龍蝦（名為 Molty）。
它不只是像 ChatGPT 那樣的對話機器人，而是像「動手」操作電腦的「自主代理人」（AI Agent）。

--OpenClaw 的核心功能 (Skills)
OpenClaw 的特色在於其高自由度與多通路整合，可以作為個人或企業的「AI分身助手」：
本機任務自動化： 能在您的電腦上運行，直接存取並管理檔案、電子郵件、瀏覽器、終端機與各類軟體。
多通路整合： 可連接 Telegram、WhatsApp 或 Slack 等通訊軟體，讓您直接在手機上像聊天一樣對它下達指令。
自主計畫與執行： 能夠將複雜的目標拆解為小步驟，並自動執行「端到端」的完整工作流（例如：監控Email -> 抓取數據 -> 產出報告 -> 發送Slack）。
持久記憶： 具有記憶功能，可保留使用者互動歷史，提供個人化的服務。

--運作方式與架構
OpenClaw 採用「自託管」模式，也就是運行在您自己的硬體（如個人電腦、伺服器）上，不僅速度快，且遵循您設定的規則。
使用者介面（Input）： 用戶透過聊天軟體（Telegram/Discord）發送指令。
Gateway/Agent 核心（Brain）： 接收指令，並與 LLM（大語言模型）互動，解析用戶意圖。
Hooks 系統（Action）： 一種簡單的自動化機制，讓 AI 在不修改核心代碼的情況下，透過掛鉤（Hooks）在網關內部自動運行特定的腳本來執行任務。
Skills/Plugin（Tools）： 外掛元件，擴展 AI 的技能，如操作瀏覽器、執行代碼等。

--應用案例
社群公關分身： 自動回覆粉專留言，代入您的語氣和規則。
智慧採購： 監控電商平台，價格低於預算時自動下單。
自動化發佈： 自動將文章改寫成不同平台（FB/IG/LinkedIn）風格並發布。
日常助手： 定期整理信箱、排程日曆、監控競爭對手網站。

> Nvidia DGX spark Playbooks 網址 https://build.nvidia.com/spark/openclaw
> 本章節做法與描述與上述不同,請自行參考
---
> Openclaw 網站 https://openclaw.ai/
---


核心元件說明
1. 使用者介面層
多通路互動（Slack / Telegram / Discord / Web）
提供聊天與任務下達入口
2. Gateway 控制層
OpenClaw 核心中樞
處理所有訊息流
管理身份驗證、Session、路由
3. Agent Runtime
AI Agent 思考核心
任務拆解
工作流程管理
決策與執行
4. Memory System
長短期記憶
可持續保存上下文
支援知識庫型應用
5. LLM Router
支援多模型切換
本地/雲端模型混合
成本與效能優化
6. Skills / Tools
OpenClaw 最大價值所在
可接：
Browser
Email
API
File System
Automation Scripts
7. Device / Browser Nodes
實際執行操作
可控制網頁、手機、系統




