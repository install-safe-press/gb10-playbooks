佈建你的Tailscale 虛擬網路

> 原始說明 https://build.nvidia.com/spark/tailscale/instructions
---
1.https://tailscale.com/ >>註冊申請帳號 右上方Get Started - it's free
![ts-2](images/tailscale-2.jpg)<br>

2.下載你PC/NB 對映的作業系統安裝檔執行安裝 Download Tailscale for Windows
![ts-3](images/tailscale-3.jpg)<br>


3.GB10 

步驟1
驗證系統需求
請檢查您的 NVIDIA DGX Spark 裝置是否正在執行支援的 Ubuntu 版本，並已連接到網路。此步驟將在 DGX Spark 設備上運行，以確認所有先決條件。

# Check Ubuntu version (should be 20.04 or newer)
```text
lsb_release -a
```

# Test internet connectivity
```text
ping -c 3 google.com
```

# Verify you have sudo access
```text
sudo whoami
```
步驟2
安裝 SSH 伺服器（如有需要）
請確保您的 DGX Spark 設備上正在運行 SSH 伺服器，因為 Tailscale 提供網路連接，但需要 SSH 進行遠端存取。此步驟在 DGX Spark 設備上執行。

# Check if SSH is running
```text
systemctl status ssh --no-pager
```
如果 SSH 未安裝或未運作：---如果依照章節順序不太可能沒有ssh
# Install OpenSSH server
```text
sudo apt update
sudo apt install -y openssh-server

# Enable and start SSH service
sudo systemctl enable ssh --now --no-pager

# Verify SSH is running
systemctl status ssh --no-pager
```

步驟3
在 NVIDIA DGX Spark 上安裝 Tailscale
使用官方 Ubuntu 軟體倉庫在 DGX Spark 上安裝 Tailscale。此步驟會新增 Tailscale 軟體包倉庫並安裝用戶端。

# Update package list
```text
sudo apt update
```

# Install required tools for adding external repositories
```text
sudo apt install -y curl gnupg
```

# Add Tailscale signing key
```text
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.noarmor.gpg | \
  sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg > /dev/null
```

# Add Tailscale repository
```text
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.tailscale-keyring.list | \
  sudo tee /etc/apt/sources.list.d/tailscale.list
```
# Update package list with new repository
```text
sudo apt update
```

# Install Tailscale
```text
sudo apt install -y tailscale
```

第四步
驗證 Tailscale 安裝
在進行身份驗證之前，請確認 Tailscale 已正確安裝在您的 DGX Spark 裝置上。

![ts-4](images/tailscale-4.jpg)<br>
![ts-5](images/tailscale-5.jpg)<br>





# Check Tailscale version
```text
tailscale version
```

# Check Tailscale service status
```text
sudo systemctl status tailscaled --no-pager
```

第五步
將您的 DGX Spark 連接到 Tailscale 網絡
使用您選擇的身份提供者，透過 Tailscale 對您的 DGX Spark 設備進行身份驗證。這將建立您的私人尾網並指派一個穩定的 IP 位址。

# Start Tailscale and begin authentication
```text
sudo tailscale up
```
![ts-6](images/tailscale-6.jpg)<br>
![ts-7](images/tailscale-7.jpg)<br>

會拿到一段 URL ,貼到瀏覽器 認證進入 Tailscal
就會看到兩邊設備都在同一群驗證是否可以互連 ping 
![ts-8](images/tailscale-8.jpg)<br>


Windows CMD: 
```text
ipconfig
```
![ts-10](images/tailscale-10.jpg)<br>

GB10 : 
```text
ip a 
```
會看到二者相對映IP地址


設定已完成就可以使用  SSH, RDP, HTTP 連線GB10  Tailscal 發送給GB10的IP 
還有更多管理功能請造訪tailscale admin page

```text
https://login.tailscale.com/admin/machines
```




