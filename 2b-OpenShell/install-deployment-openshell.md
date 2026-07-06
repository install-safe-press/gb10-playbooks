本操作說明與nvidia playbooks Instructions 有所差異, 請進行比對
https://build.nvidia.com/spark/openshell/instructions


依據NVIDIA OpenShell Developer Guide 
# 安裝 OpenShell
使用一條指令即可安裝 OpenShell：

```
curl -LsSf https://raw.githubusercontent.com/NVIDIA/OpenShell/main/install.sh | sh
```
該腳本會偵測您的作業系統，並使用您作業系統自帶的套件管理器安裝 OpenShell 命令列介面和網關。然後，它會啟動本機網關伺服器，以便您可以開始建立沙箱。


以GB10安裝為例，Ubuntu 系統上，安裝腳本使用 Debian 軟體套件。此 Debian 軟體包會安裝openshell命令列介面 (CLI)、openshell-gateway守護程式、虛擬機器沙箱支援以及 systemd 使用者服務。

Linux 用戶服務監聽端口https://127.0.0.1:17670，從內建預設值啟動，並在網關啟動前產生本地 mTLS 封包。~/.config/openshell/gateway.toml僅當需要覆寫這些預設值時才建立該服務。

CLI 從以下位置讀取客戶端包~/.config/openshell/gateways/openshell/mtls/：

安裝程式會自動啟動該服務。當您需要檢查、重新啟動或停止網關服務時，請使用 systemd 使用者命令：

```
systemctl --user status openshell-gateway
systemctl --user restart openshell-gateway
journalctl --user -u openshell-gateway -f
```

為使用戶服務在登出後繼續運行，請啟用 linger：

```
sudo loginctl enable-linger $USER
```

OpenShell cli 的參數有可能因為版本異動而有所差異 , 強烈建議要參考help

# 使用 OpenShell 建 sanbox 沙箱
注意：先確保之前已經建立的ollama正常運作且有QWEN模型 , 因為之前已經使用 process方式在GB10 OS 安裝openclaw port 18789 故下面的步驟改為18790
以下程序會建立一個sanbox 名為gb10-openshell-demo  port 18790 會forared接著會進行openclaw的設定程序 
```
openshell sandbox create \
  --keep \
  --forward 18790 \
  --name gb10-openshell-demo \
  --from openclaw \
  -- openclaw-start
```

