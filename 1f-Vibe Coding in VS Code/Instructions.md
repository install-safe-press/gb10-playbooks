步驟1
安裝 Ollama
請使用下列指令安裝最新版本的 Ollama：


巴什

複製
curl -fsSL https://ollama.com/install.sh | sh
服務運行後，拉取所需的模型：


巴什

複製
ollama pull gpt-oss:120b
步驟2
（可選）啟用遠端存取
若要允許遠端連線（例如，從使用 VSCode 和 Continue 的工作站），請修改 Ollama systemd 服務：


巴什

複製
sudo systemctl edit ollama
在註解部分下方新增以下幾行：


伊尼

複製
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="OLLAMA_ORIGINS=*"
重新載入並重啟服務：


巴什

複製
sudo systemctl daemon-reload
sudo systemctl restart ollama
如果使用防火牆，請開啟連接埠 11434：


巴什

複製
sudo ufw allow 11434/tcp
請確認工作站可以連接到您的 DGX Spark 的 Ollama 伺服器：


巴什

複製
curl -v http://YOUR_SPARK_IP:11434/api/version
請將YOUR_SPARK_IP替換為您的 DGX Spark 的 IP 位址。如果連線失敗，請參閱「故障排除」標籤。

步驟3
安裝 VSCode
對於 DGX Spark（基於 ARM 架構），請下載並安裝 VSCode：造訪https://code.visualstudio.com/download並下載 Linux ARM64 版本的 VSCode。下載完成後，記下下載的軟體包名稱。在下一個指令中，用該名稱取代 DOWNLOADED_PACKAGE_NAME。


巴什

複製
sudo dpkg -i DOWNLOADED_PACKAGE_NAME
如果使用遠端工作站，請安裝適合您系統架構的 VSCode。

第四步
安裝 Continue.dev 擴充
開啟 VSCode，然後從 Marketplace安裝Continue.dev ：

在 VSCode 中前往「擴充功能」檢視。
搜尋Continue.dev發布的Continue擴充功能並安裝。安裝完成後，點選右側欄的 Continue 圖示。
第五步
本地推理設定
點選Or, configure your own models
點選Click here to view more providers
選擇Ollama提供者
對於型號，請選擇Autodetect
透過發送測試提示來測試推理功能。
您下載的模型現在將成為推理的預設模型（例如，gpt-oss:120b）。

步驟6
設定工作站以連接到 DGX Spark' Ollama 伺服器
若要將執行 VSCode 的工作站連接到遠端 DGX Spark 實例，必須在該工作站上完成以下操作：

請依照步驟 4 中的說明繼續安裝。
點選Continue左側窗格中的圖示
點選Or, configure your own models
點選Click here to view more providers
選擇Ollama提供者
選擇Autodetect作為模型。
由於模型正在嘗試連接到本地託管的 Ollama 伺服器，因此「繼續」操作將無法偵測到該模型。

找到gear“繼續”視窗右上角的圖標，然後點擊它。
在左側窗格中，按一下「模型」。
在「聊天」下方的第一個下拉式選單旁邊，點擊齒輪圖示。
“繼續”按鈕config.yaml將會開啟。請記下您的DGX Spark的IP位址。
請將配置替換為以下內容。 YOUR_SPARK_IP應替換為您的 DGX Spark 的 IP 位址。

YAML
```
name: Config
version: 1.0.0
schema: v1

assistants:
  - name: default
    model: OllamaSpark

models:
  - name: OllamaSpark
    provider: ollama
    model: gpt-oss:120b
    apiBase: http://YOUR_SPARK_IP:11434
    title: gpt-oss:120b
    roles:
      - chat
      - edit
      - autocomplete
```

請將此處替換YOUR_SPARK_IP為您的 DGX Spark 的 IP 位址。
如果您希望遠端託管其他 Ollama 型號，請新增額外的型號條目。



故障排除
概述
指示
故障排除
症狀	原因	使固定
奧拉瑪沒有開始	GPU驅動程式可能未正確安裝。	在終端機中運行命令nvidia-smi。如果命令失敗，請檢查 DGX Dashboard 以取得 DGX Spark 的更新資訊。
繼續操作，但無法透過網路連線。	連接埠 11434 可能未開放或無法存取。	運行命令ss -tuln | grep 11434。如果輸出結果與預期不符tcp LISTEN 0 4096 *:11434 *:*，請返回步驟 2 並執行 ufw 命令。
繼續操作無法偵測到本地運行的 Ollama 模型	配置未正確設定或偵測到	檢查檔案中的 ` OLLAMA_HOSTand` 和 ` OLLAMA_ORIGINSin` /etc/systemd/system/ollama.service.d/override.conf。如果OLLAMA_HOST`and`OLLAMA_ORIGINS設定正確，請將以下行新增到您的~/.bashrc檔案中。
記憶體使用率高	模型尺寸過大	確認沒有其他大型模型或容器正在運行。對於輕量



複製
sudo sh -c 'sync; echo 3 > /proc/sys/vm/drop_caches'
資源
DGX Spark 文件
Ollama 文檔
VSCode
繼續開發
DGX Spark 論壇



