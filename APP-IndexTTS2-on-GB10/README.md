# Index TTS2 在 GB10 上佈署問題解決

大叔裝機安的 YouTube 影片語音都是靠 Text to Voice 作業（辦公室座位環境下不太可能一直唸稿）。

這些年試過一些 Text to Voice 工具，目前比較合我用的是 [IndexTTS2](https://github.com/index-tts/index-tts)。

之前都是在 Intel x64 server + T4 16G VRAM GPU Linux 環境下作業，雖然不快但是可以正常作業（但這台可能之後有其它任務）。

有想轉移到工作環境 Dell Pro Max 16 Windows NB，WSL 環境下，但是 GPU PRO 1000 8G VRAM 實在不給力，慢到無法接受。

所以囉，Dell Pro Max GB10 就是你了！幾個月前有試過，但是過程有問題沒有解決，今天重新檢視並解決問題。

## 問題如下

這台機器跟一般 x86_64 的 WSL/Linux 環境**架構完全不同**，需要 ARM64 專用的套件與 PyTorch build，且 Blackwell 是全新架構，許多套件的預編譯二進位檔還沒跟上，導致安裝過程需要多處手動處理。

在此記錄並創建檔案，作為之後新安裝時的參考。

請參考路徑下的文件 

GB10_IndexTTS2_安裝與除錯紀錄.md
GB10_IndexTTS2_標準安裝流程.md
