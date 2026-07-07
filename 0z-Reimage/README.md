Reimage

https://www.dell.com/support/product-details/en-us/product/dell-pro-max-fcm1253-micro/drivers

https://www.dell.com/support/kbdoc/en-us/000382042/how-to-reinstall-the-nvidia-dgx-operating-system-on-dell-pro-max-with-grace-blackwell-systems

https://www.dell.com/support/kbdoc/en-us/000379162/how-to-upgrade-the-bios-and-the-firmware-on-a-dell-pro-max-with-the-grace-blackwell-system

# Dell Pro Max with GB10 重灌記錄

| 階段 | 主要內容 | 操作時間 |
|------|----------|----------|
| 0. 下載 Image | 下載9GB Image iso | （ 視網路速度 約 10 分鐘） |
| 1. 燒錄 Recovery Image | 使用 `dd` 將 DGX OS7 OTA3 復原映像檔寫入 USB，並執行 `sync` 確保資料落盤 | （約 10 分鐘） |
| 2. 使用USB開機 | 開機按F7 選擇USB開機Recovery Image 程序 | （約 30~ 分鐘） |
| 3. 初始GB10 | 使用機身旁的貼紙 WIFI熱點連接 初始設定 | （約 30~ 分鐘） |
| 4. 完成初始化，進入系統 | OOBE 更新流程跑完，機器正常進入桌面，確認可連外 | （約 5~ 分鐘） |
| 5. 更新系統 | 檢查與更新系統含fwupdmgr   | （約 5 分鐘） |

詳細步驟說明
0. 下載 Image

sudo dd if=Dell_Pro_Max_with_GB10_FCM1253_WW_DGX_OS7_OTA3_Recovery_Image.iso bs=2048 of=/dev/sda
