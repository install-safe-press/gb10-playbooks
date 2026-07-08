## Dell Pro Max GB10 系統重灌初始
警語:Dell Pro Max GB10 系統重灌初始資料會全部不見,如有重要資料請先進行備份

程序參考文件：<br>
系統重灌ISO檔案： <br>
https://www.dell.com/support/product-details/en-us/product/dell-pro-max-fcm1253-micro/drivers <br>

系統重灌原廠說明：<br>
https://www.dell.com/support/kbdoc/en-us/000382042/how-to-reinstall-the-nvidia-dgx-operating-system-on-dell-pro-max-with-grace-blackwell-systems <br>

系統更新原廠說明：<br>
https://www.dell.com/support/kbdoc/en-us/000379162/how-to-upgrade-the-bios-and-the-firmware-on-a-dell-pro-max-with-the-grace-blackwell-system <br>

Dell Pro Max with GB10 重灌步驟記錄

| 階段 | 主要內容 | 操作時間 |
|------|----------|----------|
| 0. 下載 Image | 下載9GB Image iso | （ 視網路速度 約 10 分鐘） |
| 1. 燒錄 Recovery Image | 使用 `dd` 將 DGX OS7 OTA3 復原映像檔寫入 USB，並執行 `sync` 確保資料確實寫入完成 | （約 10 分鐘） |
| 2. 使用USB開機 | 開機按F7 選擇USB開機Recovery Image 程序 | （約 30~ 分鐘） |
| 3. 初始GB10 | 使用機身旁的貼紙 WIFI熱點連接 初始設定 | （約 30~ 分鐘） |
| 4. 完成初始化，進入系統 | OOBE 更新流程跑完，機器正常進入桌面，確認可連外 | （約 5~ 分鐘） |
| 5. 更新系統 | 檢查與更新系統含fwupdmgr   | （約 5 分鐘） |


詳細步驟說明

0. 下載 Image 
到Dell 網站https://www.dell.com/support/product-details/en-us/product/dell-pro-max-fcm1253-micro/drivers <br>
下載 Recovery_Image.iso 檔名Dell_Pro_Max_with_GB10_FCM1253_WW_DGX_OS7_OTA3_Recovery_Image.iso <br>

![download-1](images/0-download-1.jpg)  <br>

下載完成後查驗檔案完整性

![checksum1](images/0-download-2.jpg)
使用md5sum 檔案的checksum必需一致
```
md5sum Dell_Pro_Max_with_GB10_FCM1253_WW_DGX_OS7_OTA3_Recovery_Image.iso
```
![checksum2](images/0-md5sum.jpg)


2. 燒錄 Recovery Image
GB10插上 USB 隨身碟,(要大於16G)
```
lsblk
```
![lsblk](images/lsblk.jpg)
指令lsblk 確認USB的裝置代號顯示 為 /dev/sda

```
sudo dd if=Dell_Pro_Max_with_GB10_FCM1253_WW_DGX_OS7_OTA3_Recovery_Image.iso bs=2048 of=/dev/sda
```
使用 dd 指令將ISO寫入USB 隨身碟 
![dd](images/1-dd-1.jpg)

當寫入完成時再用 sync 指令確認寫入完成
```
sudo sync
```
#以上程序必需耐心等待 一定要確認執行完成且磁碟無寫入動作了
可同時開啟另一CLI 使用 iostat 進行磁碟動作觀察裝置 /dev/sda
```
iostat 1
```
![iostat](images/iostat.jpg)


3. 先接上螢幕鍵盤,重啟GB10 REBOOT , 使用USB開機 | 開機按F7 選擇USB開機Recovery Image 程序
```
sudo reboot
```
重新開機在黑螢幕狀態時就要按下鍵盤上的 F7 按鍵 才會啟動開機選單
![usb-boot-1](images/usb-boot-1.jpg)
使用上下鍵移動到 UEFI： USB .......... 這個選單按下 Enter 
![usb-boot-2](images/usb-boot-2.jpg)
之後就不用再按任何動作讓它自動跑下去進行DGX Spark Installation Options 作業
![usb-boot-3](images/usb-boot-3.jpg)
解析度比較低的字體畫面要跑一陣子大約20分鐘，過程會有重新開機

![usb-boot-4](images/usb-boot-4.jpg)
如果到這種畫面 , 表示已經進入系統 , 建議也接上網路線備
如果螢幕畫面沒有進入到初始GUI 改使用WIFI進行初始 

4. 初始GB10 | 使用機身旁的貼紙 WIFI熱點連接 初始設定
使用另一台設備連接GB10的初始WIFI ,使用機身旁的貼紙 WIFI熱點連接初始設定

![boot-for-wifi-init](images/boot-for-wifi-init.jpg)

6. 完成初始化會重新開機，進入系統
![login](images/login.jpg)

   
7. 更新系統
