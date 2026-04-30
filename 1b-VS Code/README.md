# 1b｜VS Code 遠端開發環境

> 原始參考：https://build.nvidia.com/spark/vscode

---

## 📌 本章節說明

本章節內容與原廠文件不同，主要著重於從 **Windows PC／NB 上的 VS Code 遠端連線到 GB10** 進行開發，適合習慣在自己電腦上工作的使用者。

---

## 什麼是 VS Code？

**VS Code（Visual Studio Code）** 是由 Microsoft 開發的免費程式碼編輯器。

你可以把它理解成：「一個**專門寫程式用的超強記事本**」，支援語法高亮、自動補全、遠端連線等強大功能。

🔗 下載點：https://code.visualstudio.com/download

> 請先在 Windows PC／NB 上完成 VS Code 安裝，再繼續以下步驟。

---

## Windows VS Code 透過 Remote SSH 連接 GB10

### 前置準備

開始之前，請確認你已知道 GB10 的：

- **IP 位址**（例如 `192.168.101.159`）
- **登入帳號與密碼**

---

### Step 1：安裝 Remote SSH 擴充套件

1. 在 VS Code 中按 `Ctrl+Shift+X` 開啟 Extensions 面板
2. 搜尋並安裝：

```
Remote - SSH
```

> 請選擇作者為 **Microsoft** 的版本。安裝完成後，左側側邊欄會出現一個**電腦螢幕圖示**。

![vs-1](images/vs-1.jpg)<br>

---

### Step 2：新增 SSH 連線設定

1. 按 `Ctrl+Shift+P` 開啟命令列（Command Palette）
2. 輸入並選擇：

```
Remote-SSH: Add New SSH Host
```

3. 輸入以下連線指令（替換成你的帳號與 GB10 的 IP）：

```bash
ssh your-username@192.168.x.x
```

4. 選擇將設定儲存至：

```
C:\Users\你的帳號\.ssh\config
```

---

### Step 3：連線到 GB10

![vs-2](images/vs-2.jpg)<br>

1. 按 `Ctrl+Shift+P` → 選擇 `Remote-SSH: Connect to Host`
2. 從清單中選擇剛才新增的 GB10 主機
3. 彈出視窗詢問作業系統時，選擇 **Linux**
4. 輸入 GB10 的**登入密碼**

連線成功後，VS Code **左下角**會顯示：

```
SSH: 192.168.x.x
```

看到這個就代表你已成功連上 GB10 ✅

---

### Step 4：開啟 GB10 上的資料夾

1. 按 `Ctrl+Shift+E` 開啟 Explorer 面板
2. 點擊「**Open Folder**」
3. 選擇你想開啟的目錄（例如 `/home/your-username`）
4. 再次輸入密碼確認

![vs-3](images/vs-3.jpg)

---

### Step 5：安裝遠端 Python 套件

連線成功後，VS Code 的 Extensions 會區分「本機」與「遠端」兩種安裝位置。

搜尋 `Python`，點擊「**Install in SSH: GB10**」，讓 VS Code 能夠在 GB10 上執行與偵錯 Python 程式。

![vs-4](images/vs-4.jpg)<br>

---

### Step 6：開啟終端機測試連線

按 `` Ctrl+` `` 開啟終端機。

> 💡 這個終端機是**直接執行在 GB10 上的**，不是你的 Windows 本機。

輸入以下指令驗證 GPU 是否正常：

```bash
nvidia-smi
```

看到 GPU 資訊就代表完全連線成功 ✅

![vs-5](images/vs-5.jpg)<br>

---

## 建立測試檔案並執行

參照原廠範例 Step 6，寫一段簡單的 Python 程式並執行，驗證整體環境是否正常運作。

在終端機中貼上以下指令，建立測試目錄與檔案：

```bash
mkdir ~/vscode-test
cd ~/vscode-test
echo 'print("Hello from DGX Spark!")' > test.py
code test.py
```

檔案開啟後，點擊右上角的 **▶️ Run Python File** 按鈕執行程式。  
終端機出現 `Hello from DGX Spark!` 即代表環境設定完全正確 ✅

![vs-6](images/vs-6.jpg)<br>

---

## 💡 常用情境小提示

| 情境 | 做法 |
|------|------|
| 每次連線都要輸入密碼很麻煩 | 設定 SSH 金鑰，之後免密碼自動登入 |
| 想在遠端跑 Jupyter Notebook | VS Code 內建支援 `.ipynb`，直接在遠端開啟即可 |
| 想用 GB10 的 GPU 跑程式 | 選好遠端的 Python Interpreter 後直接按 `Run`，自動使用 GB10 的 GPU |
